---
title: vscode插件开发2 
date: 2022-01-17 19:27:00
Tags: vscode, extension
---

# 1 命令

注册命令

```js
// tension.ts
const disposable = vscode.commands.registerCommand('extension.sayHello', ()=> {
  // ...
})
context.subscriptions.push(disposable)
```

声明命令

```json
// package.json
"commands": [
  {
    "command": "extension.sayHello",
    "title": "Hello World"
  }
]
```

Note:  所有注册类API执行后都会返回一个disposable对象，必须放到```context.subscriptions```中去

## 1.1 回调函数参数

可选参数-uri

- 资源管理器： 选中的资源（文件/文件夹）路径
- 文件编辑器： 文件路径
- 快捷键： 空

示例：

```tsx
	let disposable1 = vscode.commands.registerCommand('extension.demo.getCurrentFilePath', (uri) => {
		vscode.window.showInformationMessage(`current file/folder's path is: ${uri? uri.path : 'empty'}`);
	});
	context.subscriptions.push(disposable1);
```

```json
	// 声明命令
	"activationEvents": [
				"onCommand:extension.demo.getCurrentFilePath"
	],	
	// 命令的具体信息
	"commands": [
			{
				"command": "extension.demo.getCurrentFilePath",
				"title": "get current file/folder path"
			}
	],
	// 命令触发时机-- 编辑器获焦后邮件下拉框 & 文件系统右键下拉框
	"menus": {
			"editor/context": [
				{
					"when": "eidtorFocus",
					"command": "extension.demo.getCurrentFilePath",
					"group": "navigation"
				}
			],
			"explorer/context": [
				{
					"command": "extension.demo.getCurrentFilePath",
					"group": "navigation"
				}
			]
		}
```

## 1.2 编辑器命令

```js
vscode.commands.registerTextEditorCommand('commandId', (textEditor, edit) => {
  
})
// package.json
"activationEvents": [
  "onCommand:extension.testEditorCommand"
],
"contributes": {
    "commands": [
      {
        "command": "extension.testEditorCommand",
        "title": "TEST EDITOR COMMAND"
      }
    ],
}
```

-- 按照上述没有看到该命令被激活？？？

激活： 只有在编辑器被激活时调用才生效， 参数textEditor可以拿到活动编辑器

## 1.3 执行命令

```js
vscode.commands.executeCommand("命令"， params1, ...).then((res) => {// ...})
```

## 1.4 获取所有命令

```js
vscode.commands.getCommands().then(allCommands => {//...})
```

# 2 菜单

完整的菜单配置

```json
"menus": {
  "editor/title": [
    {
      "when": "editorFocus",
      "command": "extension.sayHello",
      "alt": "extension.sayHello",
      "group": "navigation"
    },
  ],
}
// editor/title是key值，定义这个菜单出现在哪里；
// when控制菜单何时出现；
// command定义菜单被点击后要执行什么操作；
// alt定义备用命令，按住alt键打开菜单时将执行对应命令；
// group定义菜单分组；
```

## 2.1 出现的位置

目前插件可以给以下场景配置自定义菜单

```js
资源管理器上下文菜单 - explorer/context
编辑器上下文菜单 - editor/context
编辑标题菜单栏 - editor/title
编辑器标题上下文菜单 - editor/title/context
调试callstack视图上下文菜单 - debug/callstack/context
SCM标题菜单 -scm/title
SCM资源组菜单 -scm/resourceGroup/context
SCM资源菜单 -scm/resource/context
SCM更改标题菜单 -scm/change/title
左侧视图标题菜单 -view/title
视图项菜单 -view/item/context
控制命令是否显示在命令选项板中 - commandPalette
```

最常见的是`explorer/context`和`editor/context`

## 2.2 when

通过when语句，vs code可以很好的控制什么时候显示菜单项

when的常用语法

```json
resourceLangId == javascript：当编辑的文件是js文件时；
resourceFilename == test.js：当当前打开文件名是test.js时；
isLinux、isMac、isWindows：判断当前操作系统；
editorFocus：编辑器具有焦点时；
editorHasSelection：编辑器中有文本被选中时；
view == someViewId：当当前视图ID等于someViewId时；
...
```

多个条件可以通过与或非进行组合，例如：`editorFocus && isWindows && resourceLangId == javascript`。

[官方文档-when](https://code.visualstudio.com/docs/getstarted/keybindings#_when-clause-contexts)

## 2.3 alt

表示没有按下alt键时，点击右键菜单执行的是`command`对应的命令，而按下了alt键后执行的是alt对应的命令

## 2.4 group

### 2.4.1 组间排序

控制菜单的分组和排序。不同的菜单拥有不同的默认分组。

```json
navigation- 放在这个组的永远排在最前面；
1_modification - 更改组；
9_cutcopypaste - 编辑组
z_commands - 最后一个默认组，其中包含用于打开命令选项板的条目。
```

除了`navigation`是强制放在最前面之外，其它分组都是按照0-9、a-z的顺序排列的

`explorer/context`有这些默认组：

 ```json
 navigation - 放在这个组的永远排在最前面；
 2_workspace - 与工作空间操作相关的命令。
 3_compare - 与差异编辑器中的文件比较相关的命令。
 4_search - 与在搜索视图中搜索相关的命令。
 5_cutcopypaste - 与剪切，复制和粘贴文件相关的命令。
 7_modification - 与修改文件相关的命令。
 ```

在`编辑器选项卡上下文菜单`有这些默认组：

 ```json
 1_close - 与关闭编辑器相关的命令。
 3_preview - 与固定编辑器相关的命令。
 ```

在`editor/title`有这些默认组：

 ```json
 1_diff - 与使用差异编辑器相关的命令。
 3_open - 与打开编辑器相关的命令。
 5_close - 与关闭编辑器相关的命令
 ```

### 2.4.2 组内排序

默认同一个组的顺序取决于菜单名称，如果想自定义排序的话可以再组后面通过`@<number>`的方式来自定义顺序，例如：

```json
"editor/context": [
	{
		"when": "editorFocus",
		"command": "extension.sayHello",
		// 强制放在navigation组的第2个
		"group": "navigation@2"
	},
	{
		"when": "editorFocus",
		"command": "extension.demo.getCurrentFilePath",
		// 强制放在navigation组的第1个
		"group": "navigation@1"
	}
]
```

# 3 快捷键

```json
"contributes": {
	"keybindings": [{
		// 指定快捷键执行的操作
		"command": "extension.sayHello",
		// windows下快捷键
		"key": "ctrl+f10",
		// mac下快捷键
		"mac": "cmd+f10",
		// 快捷键何时生效
		"when": "editorTextFocus"
	}]
}
```

[官方文档--快捷键绑定](https://code.visualstudio.com/docs/getstarted/keybindings)

