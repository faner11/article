VSCode 是目前最受前端工程师喜爱的编辑器,轻量且有强大的插件，我们经常用的一些插件比如 ESLint，Prettier 等因为一些历史原因我们做了很多兼容的配置，现在随着版本的更新，之前做到配置已经影响到了我们正常的工作，比如我们的保存 vue 文件自动格式化时，经常会出现格式错乱需要保存两次，单引号双引号重复修改等问题,这些原因是 ESLint,TSLint,Prettier,Vetur 冲突了，今天贴出我的 VSCode 的 settings.json，供不会配置的同学参考。先推荐些我日常使用的插件。

## 日常使用的插件

- [ESLint](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint) JS 格式化插件

- [TSLint](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-typescript-tslint-plugin) TS 格式化插件

- [Prettier - Code formatter](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode) 代码格式化插件可以配合**ESLint**、**TSLint**一起使用。

- [Vetur](https://marketplace.visualstudio.com/items?itemName=octref.vetur) VUE 强大的开发工具

- [Angular Language Service](https://marketplace.visualstudio.com/items?itemName=Angular.ng-template) Angular 开发神器

- [Angular 8 Snippets](https://marketplace.visualstudio.com/items?itemName=Mikael.Angular-BeastCode) Angular 智能提示，个人感觉比 vscode 官方推荐的好用

- [Debugger for Chrome](https://marketplace.visualstudio.com/items?itemName=msjsdiag.debugger-for-chrome) 好用的 debug 工具

- [Paste JSON as Code](https://marketplace.visualstudio.com/items?itemName=quicktype.quicktype) json 一键转接口，谁用谁知道

- [GitLens](https://marketplace.visualstudio.com/items?itemName=eamodio.gitlens) 增强 VSCode 内置的 Git 功能,超好用

- [Chinese (Simplified) Language Pack](https://marketplace.visualstudio.com/items?itemName=MS-CEINTL.vscode-language-pack-zh-hans) 应该没有人不知道 VSCode 是有官方汉化包吧

### settings.json 配置

主要解决 ESLint,TSLint,Prettier,Vetur 保存时自动格式化而造成格式冲突的问题。

```json
{
  "workbench.colorTheme": "Monokai",
  "window.zoomLevel": 0,
  "editor.fontSize": 14,
  "editor.formatOnSave": true,
  "eslint.autoFixOnSave": true,
  "eslint.validate": [
    "javascript",
    "javascriptreact",
    {
      "language": "vue",
      "autoFix": true
    }
  ],
  "editor.codeActionsOnSave": {
    "source.fixAll.tslint": true
  },
  "prettier.singleQuote": true,
  "prettier.eslintIntegration": true,
  "prettier.tslintIntegration": true,
  "vetur.format.defaultFormatter.js": "none",
  "vetur.format.defaultFormatter.html": "js-beautify-html",
  "[javascript]": {
    "editor.defaultFormatter": "vscode.typescript-language-features"
  },
  "git.enableSmartCommit": true
}
```

这样配置，以上插件是不会互相冲突的，使你更优雅的去工作。
