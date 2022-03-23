---
title: vscode插件开发3-跳转、补全、提示
date: 2022-01-18 15:17:55
tags: viscose, extension, 跳转到定义, 自动补全, 悬停提示
---

# 1- 跳转到定义

```
vscode.language.registerDefinitionProvider
new vscode.location()
```

```react
"activationEvents": [
	"onLanguage: json"
]
```

`new vscode.Location`接收2个参数，第一个是要跳转到文件的路径，第二个是跳转之后光标所在位置，接收`Range`或者`Position`对象，`Position`对象的初始化接收2个参数，行`row`和列`col`。

# 2- 高亮显示范围

```tsx
vscode.languages.registerCompletionItemProvider

// 参数
- 第一个是要关联的文件类型
- 第二个是一个对象，里面必须包含provideCompletionItems 和 resolveCompletionItem 2个方法
- 第三个是一个可选的触发提示的字符列表
```

# 3- 悬停提示

```tsx
vscode.languages.registerHoverProvider
```

