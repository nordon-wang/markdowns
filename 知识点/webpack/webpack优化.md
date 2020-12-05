### webpack相关优化

#### JS压缩

- webpack4后引入 uglifyjs-webpack-plugin
- 支持es6替换 terser-webpack-plugin, production 打包时 默认继承



#### babel 配置

```js
module.exports = {
    presets: [
        [
            '@babel/preset-env', // preset将常用的babel插件做了一个集合
            {
                modules: false, // 不需要将es6语法转成其他模块语法, 否则 tree-shaking、scoped hosting 是基于es module的, 就会导致其失效
                "targets": { // 设置打包的目标浏览器
                    "browsers": [">0.25%"] // 对于市场份额超过0.25%的浏览器都支持
                },
                "useBuiltIns": "usage", // 使用自定义的方式打补丁 polyfill, 若是不设置,可能会导致全部打补丁导致文件太大
                "bugfixes": true // 
            }
        ],
        '@babel/preset-react'
    ],
    plugins: [
        '@babel/plugin-proposal-class-properties',
        "@babel/plugin-transform-runtime", // 提取辅助函数的
    ]
};
```

- 辅助函数

  比如使用es6语法编写一个类`class Demo{}`, 会导致每次都会生成一个辅助函数,而这些辅助函数基本都是可以复用的

