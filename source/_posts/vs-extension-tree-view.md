---
title: 'vs-extension: tree-view'
date: 2022-03-10 10:20:13
tags: vs-extension
---

## 新增custrom view containers

### 1- 在package.json内部添加 contributes.views

```json
"contributes": {
    "views": { // the view showed in sideBar
        "custom-view": [
            {
              "id": "customViewId",
              "name": "view title show ",
              "icon": "media/transfer.svg",
              "contextualTitle": "custom view"
            }
        ]
    },
    "viewsContainers": { // when the view show
        "activitybar": [
            {
                "id": "containerId",
                "title": "hover msg",
                "icon": "img url"
            }
        ]
    }

}
```

添加好上面的配置， 启动插件， 就会在active bar 看到一个图标，点击后会在sideBar看到配置的view
现在还是空白的。

### 2- 给view提供数据: create a TreeDataProvider

一定会使用到的apis

1- ```getChildren(element?: T): ProviderResult<T[]>```: 获取指定element/root的子元素

2- ```getTreeItem(element: T): TreeItem | Thenable<TreeItem>```: 获取在视图中显示的元素的 UI（TreeItem)

```react
export default CustomViewProvider implements vscode.ThreeDataProvider<TreeData> {
    constructor(private workspaceRoot: string) {}

    getTreeItem(element: TreeData): vscode.TreeItem {
        // 获取展示数据的容器
        return element;
    }

    getChildren(element: TreeData) Thenable<TreeData[]>  {
        // 获取展示数据
    }
}

class TreeData extends vscode.TreeItem {
    constructor(
        public readonly collapsibleState: vscode.TreeItemCollapsibleState
    ) {
        super(collapsibleState);
    }
}

````

### 3- Registering the TreeDataProvider to your view

有两种方法可以实现：

1- ```vscode.window.registerThreeDataProvider```: 通过viewId 和上面的TreeDataProvider来注册

Register the tree data provider by providing the registered view ID and above data provider.

```react
const rootPath = vscode.workspace.workspaceFolders && vscode.workspace.workspaceFolders.length > 0 &&
  vscode.workspace.workspaceFolders[0].uri.fsPath : undefined

vscode.window.registerThreeDataProvider(
    "customViewId",
    new CustomViewProvider(rootPath)
)

```

2- ```vscode.window.createTreeView```:根据 viewId and data provider创建treeView

Create the Tree View by providing the registered view ID and above data provider.
This will give access to the TreeView, which you can use for performing other view operations. 
Use createTreeView, if you need the TreeView API.

```react
vscode.window.createTreeView('customViewId', {
    treeDataProvider: new CustomViewProvider(rootPath)
})
```

### 更新Tree view content(refresh)

use ```onDidChangeTreeData``` event

```onDidChangeTreeData?: Event<T | undefined | null | void>```

Implement this if your tree data can change and you want to update the treeview.

Add the following to your CustomViewProvider.

```react
  private _onDidChangeTreeData: vscode.EventEmitter<Dependency | undefined | null | void> = new vscode.EventEmitter<Dependency | undefined | null | void>();
  readonly onDidChangeTreeData: vscode.Event<Dependency | undefined | null | void> = this._onDidChangeTreeData.event;

  refresh(): void {
    this._onDidChangeTreeData.fire();
  }
```

We can add a command to call refresh.

In the contributes section of your package.json, add:

```json
    "commands": [
            {
                "command": "customView.refreshEntry",
                "title": "Refresh",
                "icon": {
                    "light": "resources/light/refresh.svg",
                    "dark": "resources/dark/refresh.svg"
                }
            },
    ]
```

## 注册插件

```react
import * as vscode from 'vscode';
import { CustomViewProvider } from './customViewProvider';

export function activate(context: vscode.ExtensionContext) {
  const rootPath =
    vscode.workspace.workspaceFolders && vscode.workspace.workspaceFolders.length > 0
      ? vscode.workspace.workspaceFolders[0].uri.fsPath
      : undefined;
  const customViewProvider = new CustomViewProvider(rootPath);
  vscode.window.registerTreeDataProvider('customViewId', customViewProvider);
  vscode.commands.registerCommand('customView.refreshEntry', () =>
    customViewProvider.refresh()
  );
}
```

### refresh 通过按钮来触发: 在contributions里增加下面配置

```json
"menus": {
    "view/title": [
        {
            "command": "customView.refreshEntry",
            "when": "view == customViewId",
            "group": "navigation"
        },
    ]
}
```

## 激活插件: 在package.json注册激活事件

```json
"activationEvents": [
        "onView:customViewId",
],
```

## View Container

view Container包含两部分： 活动栏或面板中显示的视图列表以及内置的视图容器
内置视图容器的示例是源代码管理和资源管理器。

### register View Container in contributes.viewsContainers

```json
"contributes": {
  "viewsContainers": {
    "activitybar": [
      {
        "id": "package-explorer", // The name of the new view container you're creating.
        "title": "Package Explorer", // The name that will show up at the top of the view container.
        "icon": "media/dep.svg" //  An image that will be displayed for the view container when in the Activity Bar.
      }
    ]
  }
}
```

你也可以注册到panel下

```json
"contributes": {
  "viewsContainers": {
    "panel": [
      {
        "id": "package-explorer",
        "title": "Package Explorer",
        "icon": "media/dep.svg"
      }
    ]
  }
}
```

### Contributing views to View Containers

可以把view挂载到view Containers

```json
"contributes": {
  "views": {
    "package-explorer": [ // key是view container id
      {
        "id": "nodeDependencies",
        "name": "Node Dependencies",
        "icon": "media/dep.svg",
        "contextualTitle": "Package Explorer"
      }
    ]
  }
}
```

可以设置view的可见性：```visibility: visible/collapsed/ hidden;``` 仅在第一次使用此视图打开工作区时，VS Code 才会尊重此属性

## View Actions: 视图交互

通过在package.json中添加contributions
可以添加交互的位置，共三个：

1- ```view/title```: 主要操作添加配置```"group": "navigation"```, 次要操作在```...```内
2- ```view/item/context```:  
Location to show actions for the tree item. Inline actions use "group": "inline" and rest are secondary actions, which are in ... menu.

![vscode-action]("../../public/css/images/vscode-view-action.png")

```json
"contributes": {
  "commands": [
    {
      "command": "nodeDependencies.refreshEntry",
      "title": "Refresh",
      "icon": {
        "light": "resources/light/refresh.svg",
        "dark": "resources/dark/refresh.svg"
      }
    },
    {
      "command": "nodeDependencies.addEntry",
      "title": "Add"
    },
    {
      "command": "nodeDependencies.editEntry",
      "title": "Edit",
      "icon": {
        "light": "resources/light/edit.svg",
        "dark": "resources/dark/edit.svg"
      }
    },
    {
      "command": "nodeDependencies.deleteEntry",
      "title": "Delete"
    }
  ],
  "menus": {
    "view/title": [
      {
        "command": "nodeDependencies.refreshEntry",
        "when": "view == nodeDependencies",
        "group": "navigation"
      },
      {
        "command": "nodeDependencies.addEntry",
        "when": "view == nodeDependencies"
      }
    ],
    "view/item/context": [
      {
        "command": "nodeDependencies.editEntry",
        "when": "view == nodeDependencies && viewItem == dependency",
        "group": "inline"
      },
      {
        "command": "nodeDependencies.deleteEntry",
        "when": "view == nodeDependencies && viewItem == dependency"
      }
    ]
  }
}
```

