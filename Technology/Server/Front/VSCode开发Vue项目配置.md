# VSCode开发Vue项目配置
## 安装插件
1. ESLint
2. Vetur
3. Vue 2 Snippets
4. Vuter
5. Code Runner
6. Debugger for Chrome
7. open in browser
8. npm Intellisense
9. HTML CSS Support
10. Auto Close Tag

## 配置代码格式自动扫描
1. .eslintrc.js
```
// http://eslint.org/docs/user-guide/configuring
module.exports = {
    root: true,
    parser: "vue-eslint-parser",
    parserOptions: {
        parser: "babel-eslint",
        sourceType: "module"
    },
    env: {
        browser: true,
        node: true,
        es6: true,
        commonjs: true,
    }
}
```
2. setting.json
使用:https://code.visualstudio.com/docs/getstarted/settings#_default-settings
