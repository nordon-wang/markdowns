- vscode 配置

```json

{
    "editor.tabSize": 2,
    "workbench.startupEditor": "newUntitledFile",
    "workbench.iconTheme": "vscode-icons",
    // 以下为stylus配置
    "stylusSupremacy.insertColons": false, // 是否插入冒号
    "stylusSupremacy.insertSemicolons": false, // 是否插入分好
    "stylusSupremacy.insertBraces": false, // 是否插入大括号
    "stylusSupremacy.insertNewLineAroundImports": false, // import之后是否换行
    "stylusSupremacy.insertNewLineAroundBlocks": false, // 两个选择器中是否换行
    "vetur.format.defaultFormatter.html": "js-beautify-html",
    "eslint.autoFixOnSave": true,
    "eslint.validate": [
        "javascript",
        {
            "language": "html",
            "autoFix": true
        },
        {
            "language": "vue",
            "autoFix": true
        },
        "javascriptreact",
        "html",
        "vue"
    ],
    "eslint.options": { "plugins": ["html"] },
    "prettier.singleQuote": true,
    "prettier.semi": false,
    "javascript.format.insertSpaceBeforeFunctionParenthesis": false,
    "vetur.format.js.InsertSpaceBeforeFunctionParenthesis": false,
    "vetur.format.defaultFormatter.js": "prettier",
    "window.zoomLevel": 0,
    "editor.fontSize":17,
    "typescript.check.tscVersion": false,
    "terminal.integrated.shell.windows": "C:\\Program Files\\Git\\bin\\bash.exe",
    "workbench.panel.location": "bottom",
    // "prettier.eslintIntegration": true,
    "editor.tabSize": 2,
    "editor.fontSize": 22
}

```

- vscode 插件

```json
Auto Close Tag
Vetur
HTML Snippets
HTML CSS Support 
JQuery Code Snippets
Path Intellisense
Npm Intellisense
Document this 
Auto Rename Tag
vscode-icon
One Dark Theme
Open-In-Browser
Quokka
HTML Boilerplate
Prettier
Color Info
SVG Viewer
Icon Fonts



```

