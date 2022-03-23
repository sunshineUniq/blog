---
title: vscode插件打包、发布、升级
date: 2022-01-21 12:05:49
tags: vscode, extensions, publish, package
---

# 1- 本地打包

```bash
npm i vsce -g
# 打包成vsix文件
vsce package  
```

打包的时候需要设置repository，如果没有设置`repository`会有提示，所以最好设置一下。

