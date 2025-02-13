
# fork自[移植“沉浸式翻译”到Thunderbird实现邮件翻译](https://linux.do/t/topic/285525)

## 1. 注入 `background.js`

向 `background.js` 中的主体代码添加以下代码以注入 JS：

```javascript
async function registerMsgDisplayScript() {
    await messenger.messageDisplayScripts.register({
        js: [{file: "/content_script.js"}, {file: "/content_start.js"}]
    });
}
registerMsgDisplayScript();
```

## 2. 修改 `manifest.json`

### permissions
- 将 `"contextMenus"` 修改为 `"menus"`（右键菜单相关），并添加 `"messagesModify"` 权限（实现邮件翻译的核心权限）。

### browser_specific_settings
- 由于第一步中 `messageDisplayScripts` 的 API 从 Thunderbird 78 开始支持，所以修改 `"strict_min_version"` 最低版本为 `78.0`。
```
  "browser_specific_settings": {
    "gecko": {
      "id": "{5efceaa7-f3a2-4e59-a54b-85319448e305}",
      "strict_min_version": "78.0"
    }
  }
```

## 3. 修改右键菜单相关 API（可选）

（如果不需要右键菜单翻译的功能，可以跳过此步。）

- 重命名 `background.js`、`offscreen.js`、`content_script.js`、`content_start.js`、`popup.js`和`options.js`中所有调用名为 `"contextMenus"` 的 API 为 `"menus"`。（Thunderbird 中，`"contextMenus"` 并非是`"menus"`的别名）
- 删除 `background.js` 中列表 `["browser_action", "page_action"]` 中的 `"page_action"`，否则无法创建右键菜单。（该 `ContextType` 适用于 Firefox，而 Thunderbird 中无）


## 4. 打包与安装插件
- 在完成所有修改后，将 `thunderbird` 目录下的所有文件打包为`.zip`文件，然后修改后缀名为 `.xpi`。
- 进入Thunderbird扩展管理页面，点击右上角的齿轮图标，选择“从文件安装附加组件...”，选择刚刚生成的 `.xpi` 文件进行安装。
> **注意**：插件的快捷键设置唯一入口也在此页面。

![image](https://github.com/user-attachments/assets/4ad206ff-9299-438f-8237-bac491dec6a1)



> **注意**：初次安装后弹出引导要求翻译时，由于Thunderbird置顶的扩展图标在除邮件页外的第三方页面中不可见，所以可以点击页面右侧的悬浮窗或使用默认快捷键`Alt+A`翻译，进而完成引导。

![image](https://github.com/user-attachments/assets/122a39fc-f589-4267-9d26-d459f62fcd81)



