---
title: 自定义vscode插件
date: 2022-01-14 16:24:00
Tags: vscode, typescript, extensions
---

# VScode组成结构

VSCode 是基于 Electron 构建的，主要由三部分构成：

- Electron: UI

- - Monaco Editor
  - Extension Host

- Language Server Protocol & Debug Adapter Protocol

# 插件生命周期

从生命周期上来看，插件编写有三大个部分：

- **Activation Event**：设置插件激活的时机。位于 package.json 中。
- **Contribution Point**：设置在 VSCode 中哪些地方添加新功能，也就是这个插件增强了哪些功能。位于 package.json 中。
- **Register**：在 `extension.ts` 中给要写的功能用 `vscode.commands.register...` 给 `Activation Event` 或 `Contribution Point` 中配置的事件绑定方法或者设置监听器。位于入口文件（默认是 `extension.ts`）的 `activate()` 函数中。

### packages.json

package 中和插件有关的主要内容是如下几个项目，其中 main 是插件代码的入口文件。

```json
"activationEvents": [
        "onCommand:extension.helloWorld"
    ],
    "main": "./out/extension.js",
    "contributes": {
        "commands": [
            {
                "command": "extension.helloWorld",
                "title": "Hello World"
            },
            {
                "command": "extension.helloVscode",
                "title": "Hello vscode"
            }
        ]
    },
```

- activationEvents: 配置插件什么时候被激活， 目前支持下面8种配置

```bash
onLanguage:${language}
onCommand:${command}
onDebug
workspaceContains:${toplevelfilename}
onFileSystem:${scheme}
onView:${viewId}
onUri
*
```

- contributes

```bash
configuration：设置
commands：命令
menus：菜单
keybindings：快捷键绑定
languages：新语言支持
debuggers：调试
breakpoints：断点
grammars
themes：主题
snippets：代码片段
jsonValidation：自定义JSON校验
views：左侧侧边栏视图
viewsContainers：自定义activitybar
problemMatchers
problemPatterns
taskDefinitions
colors
```

### viscode插件脚手架

```bash
npm install -g yo generator-code
yo code
# ? What type of extension do you want to create? New Extension (TypeScript)
# ? What's the name of your extension? HelloWorld
### Press <Enter> to choose default for all options below ###

# ? What's the identifier of your extension? helloworld
# ? What's the description of your extension? LEAVE BLANK
# ? Initialize a git repository? Yes
# ? Bundle the source code with webpack? No
# ? Which package manager to use? npm

# ? Do you want to open the new folder with Visual Studio Code? Open with `code`
```

生成插件目录如下

```bash
// tree -I "node_modules"
.
├── CHANGELOG.md
├── README.md
├── out
│   ├── extension.js
│   ├── extension.js.map
│   └── test
│       ├── runTest.js
│       ├── runTest.js.map
│       └── suite
│           ├── extension.test.js
│           ├── extension.test.js.map
│           ├── index.js
│           └── index.js.map
├── package-lock.json
├── package.json
├── src
│   ├── extension.ts
│   └── test
│       ├── runTest.ts
│       └── suite
│           ├── extension.test.ts
│           └── index.ts
├── tsconfig.json
└── vsc-extension-quickstart.md
```



