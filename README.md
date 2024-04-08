# smaple
```
// コンテキストメニューを作成する関数
function createContextMenusFromJSON(jsonData, parentId = null, parentMenuId = null) {
  for (const key in jsonData) {
    const item = jsonData[key];
    const id = parentId ? `${parentId}-${key}` : key;
    const menuId = parentMenuId ? `${parentMenuId}-${key}` : key;
    const menuItem = {
      id: menuId,
      title: item.title,
      contexts: item.contexts || ["editable"]
    };

    if (item.items) {
      // サブメニューの場合は再帰的に作成
      menuItem.parentId = parentId;
      browser.contextMenus.create(menuItem);
      createContextMenusFromJSON(item.items, id, menuId);
    } else {
      // アイテムの場合
      menuItem.parentId = parentId;
      browser.contextMenus.create(menuItem);
    }
  }
}

// メニューがクリックされたときの処理
browser.contextMenus.onClicked.addListener((info, tab) => {
  // 選択されたメニューに対応する定数値を取得して挿入
  const constantValue = findConstantValue(info.menuItemId);
  if (constantValue) {
    insertConstantValue(tab, constantValue);
  }
});

// 指定されたメニューIDに対応する定数値を再帰的に検索する関数
function findConstantValue(menuId, jsonData = contextMenuData) {
  for (const key in jsonData) {
    const item = jsonData[key];
    const fullMenuId = item.fullMenuId || item.id;
    console.log(fullMenuId)
    if (fullMenuId === menuId && item.constantValue) {
      return item.constantValue;
    }
    if (item.items) {
      const constantValue = findConstantValue(menuId, item.items);
      if (constantValue) {
        return constantValue;
      }
    }
  }
  return null;
}

// 定数値を挿入する関数
function insertConstantValue(tab, constantValue) {
  // 選択されている入力欄に定数値を入力するスクリプトを実行
  browser.tabs.executeScript({
    code: `document.activeElement.value += '${constantValue}';`
  });
}

let contextMenuData;

// Manifestが読み込まれた時にJSONファイルを読み込んでコンテキストメニューを作成
fetch(browser.runtime.getURL("context_menu.json"))
  .then(response => response.json())
  .then(data => {
    // JSONデータにフルのメニューIDを追加
    addFullMenuId(data);
    contextMenuData = data;
    createContextMenusFromJSON(contextMenuData);
  })
  .catch(error => console.error("Error fetching context menu JSON:", error));

// JSONデータにフルのメニューIDを追加する関数
function addFullMenuId(jsonData, parentMenuId = null) {
  for (const key in jsonData) {
    const item = jsonData[key];
    const menuId = parentMenuId ? `${parentMenuId}-${key}` : key;
    item.fullMenuId = menuId;
    if (item.items) {
      addFullMenuId(item.items, menuId);
    }
  }
}

```
```
{
  "parent-menu": {
    "title": "診断入力値",
    "contexts": ["editable"],
    "items": [
      {
        "id": "氏名",
        "title": "Examples",
        "contexts": ["editable"],
        "items": [
          {
            "id": "example1",
            "title": "Example 1",
            "contexts": ["editable"],
            "items": [
              {
                "id": "example1-1",
                "title": "Example 2",
                "constantValue": "Example 2 Constant Value",
                "contexts": ["editable"]
              }
            ]
          },
          {
            "id": "example2",
            "title": "Example 2",
            "constantValue": "Example 2 Constant Value",
            "contexts": ["editable"]
          }
        ]
      },
      {
        "id": "custom-submenu",
        "title": "Custom",
        "contexts": ["editable"],
        "items": [
          {
            "id": "custom1",
            "title": "Custom 1",
            "constantValue": "Custom 1 Constant Value",
            "contexts": ["editable"]
          },
          {
            "id": "custom2",
            "title": "Custom 2",
            "constantValue": "Custom 2 Constant Value",
            "contexts": ["editable"]
          }
        ]
      }
    ]
  }
}
```
```
{
  "manifest_version": 2,
  "name": "Input Constant Extension",
  "version": "1.0",
  "description": "Input a constant value into selected input field.",
  "permissions": [
    "contextMenus",
    "activeTab",
    "*://*/*"
  ],
  "background": {
    "scripts": ["background.js"]
  },
  "icons": {
    "48": "icon.png"
  },
  "browser_action": {
    "default_popup": "popup.html",
    "default_icon": {
      "48": "icon.png"
    }
  }
}
```
