---
title: vscode插件开发-webview
date: 2022-01-21 14:18:36
tags: vscode, extensions, webview
---

# 1- 创建webview

```tsx
import * as vscode from 'vscode';

function testWebview(context: vscode.ExtensionContext): void {
  context.subscriptions.push(
    vscode.commands.registerCommand('extension.demo.openWebview', (uri) => {
      console.log('uri', uri);
      const panel = vscode.window.createWebviewPanel(
        'testWebview',
        'Webview test',
        vscode.ViewColumn.One,
        {
          enableScripts: true,
          retainContextWhenHidden: true,
        },
      );
      panel.webview.html = `<html><body>你好，我是webview</body></html>`;
    }));
}

export default testWebview;

```

声明命令

```json
	"activationEvents": [
				"onCommand:extension.demo.openWebview"
	],
		"contributes": {
		"commands": [
			{
				"command": "extension.demo.openWebview",
				"title": "OPEN WEBVIEW"
			}
		],
		"menus": {
			"editor/context": [
				{
					"command": "extension.demo.openWebview",
					"group": "navigation"
			}
			],
		}
	},
```

# 2- 加载本地资源

出于安全考虑，Webview默认无法直接访问本地资源，它在一个孤立的上下文中运行，想要加载本地图片、js、css等必须通过特殊的`vscode-resource:`协议，网页里面**所有的静态资源都要转换成这种格式，否则无法被正常加载**。

`vscode-resource:`协议类似于`file:`协议，但它只允许访问特定的本地文件。和`file:`一样，`vscode-resource:`从磁盘加载绝对路径的资源。

转换本地磁盘文件路径成webview 能打开的文件路径

```js
getExtensionFileVscodeResource: function(context, relativePath) {
	const diskPath = vscode.Uri.file(path.join(context.extensionPath, relativePath));
	return diskPath.with({ scheme: 'vscode-resource' }).toString();
}
```

默认情况下，`vscode-resource:`只能访问以下位置中的资源：

 ```
 - 扩展程序安装目录中的文件。
 - 用户当前活动的工作区内。
 - 当然，你还可以使用dataURI直接在Webview中嵌入资源，这种方式没有限制；
 ```

# 3- 从文件加载HTML内容

默认不支持从文件加载HTML，需要自己封装代码

```js
  getWebviewContent(
    context: vscode.ExtensionContext,
    templatePath: string,
  ): string {
    const resourcePath = path.join(context.extensionPath, templatePath);
    const dirPath = path.dirname(resourcePath);
    let html = fs.readFileSync(resourcePath, 'utf-8');
    // 处理html内部的资源路径成可以被vscode webview 访问的路径
    html = html.replace(
      /(<link.+?href="|<script.+?src="|<img.+?src=")(.+?)"/g,
      (m, $1, $2) => {
        return $1 + vscode.Uri.file(path.resolve(dirPath, $2)).with(
          { scheme: 'vscode-resource' }).toString();
      });
    return html;
  },
```

# 4- 消息通信

重头戏来了，`Webview`和普通网页非常类似，不能直接调用任何`VSCode`API，但是，它唯一特别之处就在于多了一个名叫`acquireVsCodeApi`的方法，执行这个方法会返回一个超级阉割版的`vscode`对象，这个对象里面有且仅有如下3个可以和插件通信的API：

```
- getState
- postMessage
- setState
```

插件给`Webview`发送消息: 支持发送任意可以被`JSON`化的数据

```tsx
panel.webview.postMessage({text: "hello"})
```

webview 端接收

```tsx
window.addEventListener('message', event => {
	const message = event.data;
	console.log('Webview接收到的消息：', message);
}
```

`Webview`主动发送消息给插件：

```tex
vscode.postMessage({text: '你好，我是Webview啊！'});
```

插件接收：

```tsx
panel.webview.onDidReceiveMessage(message => {
	console.log('插件收到的消息：', message);
}, undefined, context.subscriptions);
```

# 5- 封装通信



# 6- 主题适配

`Webview`可以根据`VS Code`的当前主题更改其外观，原理是body上面添加当前主题名称，主要有以下三种：

 ```json
 vscode-light - 浅色主题；
 vscode-dark -深色主题；
 vscode-high-contrast - 高对比度主题;
 ```

可以通过自己写样式来适配不同主题：

```css
/* 浅色主题 */
.body.vscode-light {
	background: white;
	color: black;
}
/* 深色主题 */
body.vscode-dark {
	background: #252526;
	color: white;
}
/* 高对比度主题 */
body.vscode-high-contrast {
	background: white;
	color: red;
}
```

# 7- 生命周期

`webview`由创建它的扩展程序所有，返回的`panel`对象你必须自己保存，如果你的扩展程序丢失了这个引用，那么将无法再次重新访问该`webview`，即使Web视图继续显示在`vscode`中。

用户也可以随时关闭`webview`面板。当用户关闭webview面板时，webview本身将被销毁，此时不能再使用panel引用，否则将会出现异常，可以通过监听`onDidDispose`事件在这里面做一些销毁操作。

可以通过`panel.dispose()`方法主动关闭webview。

# 8- 状态保持

当webview移动到后台又再次显示时，webview中的任何状态都将丢失。

解决此问题的最佳方法是使你的webview无状态，通过消息传递来保存webview的状态。

## 8.1- state

在webview的js中我们可以使用`vscode.getState()`和`vscode.setState()`方法来保存和恢复JSON可序列化状态对象。当webview被隐藏时，即使webview内容本身被破坏，这些状态仍然会保存。当然了，当webview被销毁时，状态将被销毁。

## 8.2 序列化

通过注册`WebviewPanelSerializer`可以实现在`VScode`重启后自动恢复你的`webview`，当然，序列化其实也是建立在`getState`和`setState`之上的。

注册方法：`vscode.window.registerWebviewPanelSerializer`

## 8.3- retainContextWhenHidden

对于具有非常复杂的UI或状态且无法快速保存和恢复的`webview`，我们可以直接使用`retainContextWhenHidden`选项。设置`retainContextWhenHidden: true`后即使webview被隐藏到后台其状态也不会丢失。

尽管`retainContextWhenHidden`很有吸引力，但它需要很高的内存开销，一般建议在实在没办法的时候才启用。
`getState`和`setState`是持久化的首选方式，因为它们的性能开销要比`retainContextWhenHidden`低得多。

# 9- API

| Guide on VS Code Website                                     | API & Contribution                                           |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [Command](https://code.visualstudio.com/api/extension-guides/command) | [commands](https://code.visualstudio.com/api/references/vscode-api#commands) |
| [Color Theme](https://code.visualstudio.com/api/extension-guides/color-theme) | [contributes.themes](https://code.visualstudio.com/api/references/contribution-points#contributes.themes) |
| [File Icon Theme](https://code.visualstudio.com/api/extension-guides/file-icon-theme) | [contributes.iconThemes](https://code.visualstudio.com/api/references/contribution-points#contributes.iconThemes) |
| [Product Icon Theme](https://code.visualstudio.com/api/extension-guides/product-icon-theme) | [contributes.productIconThemes](https://code.visualstudio.com/api/references/contribution-points#contributes.productIconThemes) |
| [Tree View](https://code.visualstudio.com/api/extension-guides/tree-view) | [window.createTreeView](https://code.visualstudio.com/api/references/vscode-api#window.createTreeView) [window.registerTreeDataProvider](https://code.visualstudio.com/api/references/vscode-api#window.registerTreeDataProvider) [TreeView](https://code.visualstudio.com/api/references/vscode-api#TreeView) [TreeDataProvider](https://code.visualstudio.com/api/references/vscode-api#TreeDataProvider) [contributes.views](https://code.visualstudio.com/api/references/contribution-points#contributes.views) [contributes.viewsContainers](https://code.visualstudio.com/api/references/contribution-points#contributes.viewsContainers) |
| [Webview](https://code.visualstudio.com/api/extension-guides/webview) | [window.createWebviewPanel](https://code.visualstudio.com/api/references/vscode-api#window.createWebviewPanel) [window.registerWebviewPanelSerializer](https://code.visualstudio.com/api/references/vscode-api#window.registerWebviewPanelSerializer) |
| [Custom Editors](https://code.visualstudio.com/api/extension-guides/custom-editors) | [window.registerCustomEditorProvider](https://code.visualstudio.com/api/references/vscode-api#window.registerCustomEditorProvider) [CustomTextEditorProvider](https://code.visualstudio.com/api/references/vscode-api#CustomTextEditorProvider) [contributes.customEditors](https://code.visualstudio.com/api/references/contribution-points#contributes.customEditors) |
| [Virtual Documents](https://code.visualstudio.com/api/extension-guides/virtual-documents) | [workspace.registerTextDocumentContentProvider](https://code.visualstudio.com/api/references/vscode-api#workspace.registerTextDocumentContentProvider) [commands.registerCommand](https://code.visualstudio.com/api/references/vscode-api#commands.registerCommand) [window.showInputBox](https://code.visualstudio.com/api/references/vscode-api#window.showInputBox) |
| [Virtual Workspaces](https://code.visualstudio.com/api/extension-guides/virtual-workspaces) | [workspace.fs](https://code.visualstudio.com/api/references/vscode-api#workspace.fs) capabilities.virtualWorkspaces |
| [Workspace Trust](https://code.visualstudio.com/api/extension-guides/workspace-trust) | [workspace.isTrusted](https://code.visualstudio.com/api/references/vscode-api#workspace.isTrusted) [workspace.onDidGrantWorkspaceTrust](https://code.visualstudio.com/api/references/vscode-api#workspace.onDidGrantWorkspaceTrust) capabilities.untrustedWorkspaces |
| [Task Provider](https://code.visualstudio.com/api/extension-guides/task-provider) | [tasks.registerTaskProvider](https://code.visualstudio.com/api/references/vscode-api#tasks.registerTaskProvider) [Task](https://code.visualstudio.com/api/references/vscode-api#Task) [ShellExecution](https://code.visualstudio.com/api/references/vscode-api#ShellExecution) [contributes.taskDefinitions](https://code.visualstudio.com/api/references/contribution-points#contributes.taskDefinitions) |
| [Source Control](https://code.visualstudio.com/api/extension-guides/scm-provider) | [workspace.workspaceFolders](https://code.visualstudio.com/api/references/vscode-api#workspace.workspaceFolders) [SourceControl](https://code.visualstudio.com/api/references/vscode-api#SourceControl) [SourceControlResourceGroup](https://code.visualstudio.com/api/references/vscode-api#SourceControlResourceGroup) [scm.createSourceControl](https://code.visualstudio.com/api/references/vscode-api#scm.createSourceControl) [TextDocumentContentProvider](https://code.visualstudio.com/api/references/vscode-api#TextDocumentContentProvider) [contributes.menus](https://code.visualstudio.com/api/references/contribution-points#contributes.menus) |
| [Debugger Extension](https://code.visualstudio.com/api/extension-guides/debugger-extension) | [contributes.breakpoints](https://code.visualstudio.com/api/references/contribution-points#contributes.breakpoints) [contributes.debuggers](https://code.visualstudio.com/api/references/contribution-points#contributes.debuggers) [debug](https://code.visualstudio.com/api/references/vscode-api#debug) |
| [Markdown Extension](https://code.visualstudio.com/api/extension-guides/markdown-extension) | markdown.previewStyles markdown.markdownItPlugins markdown.previewScripts |
| [Test Extension](https://code.visualstudio.com/api/extension-guides/testing) | [TestController](https://code.visualstudio.com/api/references/vscode-api#TestController) [TestItem](https://code.visualstudio.com/api/references/vscode-api#TestItem) |
| [Custom Data Extension](https://code.visualstudio.com/api/extension-guides/custom-data-extension) | contributes.html.customData contributes.css.customData       |

