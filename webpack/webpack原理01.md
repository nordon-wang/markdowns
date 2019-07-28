- 学习目标
  - 了解webpack打包原理
  - 了解webpack的loader原理
  - 了解webpack的plugin原理
  - 了解ast抽象语法树的应用
  - 了解tapable的原理
  - 手写一个简单的webpack

## 准备工作

1. 新建一个项目 webpack-theory

2. 新建 bin 目录，创建``webpack-theory.js`文件， 将打包工具主程序放入其中

   主程序的顶部应当有: `#!/usr/bin/env node` 标识，指定程序执行环境为 node

   ```
   #!/usr/bin/env node
   // log的内容修改直接，可以直接生效
   console.log('当通过npm link链接之后，通过webpack-theory指令可以直接打出');
   ```

3. 在package.json中配置 bin 脚本，与scripts平级

   ```
   {
   	"bin": "./bin/oyo-pack.js"
   }
   ```

4. 通过 `npm link` 将本地的项目webpack-theory 链接到全局包中，链接之后便可以直接在本地使用，供本地测试使用,具体参考 [npm link](http://javascript.ruanyifeng.com/nodejs/npm.html#toc18)

   1. 成功之后，可以 `cd /usr/local/lib/node_modules` 查看所有安装的包

   ![WX20190725-235506@2x](/Users/nordon.wang/Desktop/self/markdowns/webpack/imgs/WX20190725-235506@2x.png)

   进入目录后，可以看到`webpack-theory`，webpack-theory就是npm link时，在全局的node_modules中生成一个符号链接，指向模块(webpack-theory)的本地目录，当本地的文件(bin/oyo-pack)修改时会自动链接到全局，因为全局的node_modules只是本地的引用

   2. 在本地执行 oyo-pack， 会直接将 bin/oyo-pack.js 的console.log内容输出

## 自定义loader

- 通过配置loader和手写loader可以发现，其实webpack能支持loader，主要步骤如下
  - 读取webpack.config.js配置文件的module.rules配置项，进行倒序迭代(rules的每项匹配规则按倒序匹配)
  - 根据正则匹配到对应的文件类型，同时再批量导入loader函数
  - 倒序迭代调用所有的loader函数(loader从右到左，从下到上)
  - 最后返回处理后的代码
- 实现个人的loader功能时，同样也可以在加载每个模块时，根据rules的正则来匹配是否满足条件，如果满足条件则加载对应的loader函数并迭代调用

- depAnalyse() 方法中获取到源码后，读取loader

```
// 读取文件内容
let source = this.getSource(modulePath)

// 内部定义一个处理loader的函数
const _handleLoader = (usePath, _this) => {
  const loaderPath = path.join(this.root, usePath)
  const loader = require(loaderPath)
  source = loader.call(_this, source)
}

// 读取 rules 规则, 进行倒序遍历
const rules = this.rules
for (let i = rules.length - 1; i >= 0; i--) {
  const {
    test,
    use
  } = rules[i]

  // 匹配 modulePath 是否符合规则，若是符合规则就需要倒序遍历获取所有的loader
  // 获取每一条规则，和当前的 modulePath 进行匹配
  if (test.test(modulePath)) {
    // use 可能是 数组、对象、字符串
    if (Array.isArray(use)) {
      // array
      for (let j = use.length - 1; j >= 0; j--) {
        // const loaderPath = path.join(this.root, use[j])
        // const loader = require(loaderPath)
        // source = loader(source)
        _handleLoader(use[j])
      }
    } else if (typeof use === 'string') {
      // string
      _handleLoader(use)
    } else if (use instanceof Object) {
      // object
      _handleLoader(use.path, {
        query: use.options
      })
    }
  }
}
```

```
const loaderUtils = require('loader-utils')
// loader 其实就是一个函数
// 将js文件中的 aaa 换成 bbb
module.exports = function (source) {
  // console.log(this.query, loaderUtils.getOptions(this).name);
  // console.log(this.query);
  
  return source.replace(/aaa/g, 'nordon')
}
```



## 自定义plugin

> 插件是 webpack 生态系统的重要组成部分，为社区用户提供了一种强大方式来直接触及 webpack 的编译过程(compilation process)。插件能够 [钩入(hook)](https://www.webpackjs.com/api/compiler-hooks/#hooks) 到在每个编译(compilation)中触发的所有关键事件。在编译的每一步，插件都具备完全访问 `compiler` 对象的能力，如果情况合适，还可以访问当前 `compilation` 对象，即自定义插件就是在webpack的编译过程的生命周期钩子中，进行编码开发实现一些功能

### webpack插件组成

1. 一个JavaScript命名函数
2. 在插件函数的prototype上定义一个apply方法
3. 指定一个绑定到webpack自身的事件钩子
4. 处理webpack内部实例的特定数据
5. 功能完成后调用webpack提供的回调

- webpack的生命周期钩子 [生命周期钩子](https://www.webpackjs.com/api/compiler-hooks/#hooks)

### 实现一个普通插件

```
// 1. 构造函数
// 2. prototype中有一个apply方法
class HelloWordPlugin {
  // apply 中有一个 compiler 形参
  apply(compiler){
    console.log('插件执行了');
    // 通过compiler对象可以注册对应的事件，全部的钩子都可以使用
    compiler.hooks.done.tap('HelloWordPlugin', (stats) => {
      console.log('整个webpack打包结束了');
    })

    compiler.hooks.emit.tap('HelloWordPlugin', (compilation) => {
      console.log('触发emit方法');
    })
  }
}

module.exports = HelloWordPlugin
```

### 实现 HtmlWebpackPlugin

> html-webpack-plugin 可以将制定的html模板复制一份输出到dist目录下，溶蚀会自动引入bundle.js

```
const path = require('path')
const fs = require('fs')
const cheerio = require('cheerio')

class HTMLPlugin {
  constructor(options){
    // 插件的参数，filename、template等
    this.options = options
  }

  apply(compiler){
    // 注册 afterEmit 钩子
    // 如果使用done钩子，则需要使用stats.compilation.assets获取，而且会比 afterEmit 晚一些
    compiler.hooks.afterEmit.tap('HTMLPlugin', (compilation) => {
      // 根据模板读取html文件内容
      const result = fs.readFileSync(this.options.template, 'utf-8')
      // 使用 cheerio 来分析 HTML
      let $ = cheerio.load(result)
      // 创建 script 标签后插入HTML中
      Object.keys(compilation.assets).forEach(item => {
        // $(`<script src="/${item}"></script>`).appendTo('body')
        // 为了方便调试将路径修改为相对路径
        $(`<script src="${item}"></script>`).appendTo('body')
      })
      // 转换成新的HTML并写入到 dist 目录中
      fs.writeFileSync(path.join(process.cwd(), 'dist', this.options.filename), $.html())
    })
  }
}

module.exports = HTMLPlugin
```



- 实现步骤
  - 编写一个自定义插件，注册 afterEmit 钩子
  - 根据创建对象时传入的 template 属性来读取 html 模板
  - 使用工具分析HTML，推荐使用 cheerio，可以直接使用jQuery API
  - 循环遍历webpack打包的资源文件列表，如果有多个bundle就都打包进去
  - 输出新生成的HTML字符串到dist目录中
- Compiler和Compilattion的区别
  - compiler对象表示不变的webpack环境，是针对webpack的
  - compilation对象针对的是随时可变的项目文件，只要文件有改动，compilation就会被重新创建

### 在webpack-theory中添加plugin功能

#### tabable简介

> 在webpack内部实现时间流机制的核心就在于tapable，有了它就可以通过事件流的形式，将各个插件串联起来，tapable类似于node中的events库，核心原理就是一个订阅发布模式

- 基本用法
  - 定义钩子
  - 使用者注册事件
  - 在合适的阶段调用钩子，触发事件

## 插件列表

### 解析AST语法树

> 就是将一行代码解析成对象的格式,可以使用在线工具生成ast语法树 [astexplorer](https://astexplorer.net/) 进行查看

1. 安装@babel/parser插件

```
npm i -S @babel/parser
```

2. 使用

```
const parser = require('@babel/parser')
//source是需要生成ast语法树的代码片段
const ast = parser.parse(source)
```

3. 生成效果

源码

```
const news = require('./news')

console.log(news.content)
```

生成的ast语法树

```
Node {
  type: 'File',
  start: 0,
  end: 57,
  loc:
   SourceLocation {
     start: Position { line: 1, column: 0 },
     end: Position { line: 3, column: 25 } },
  program:
   Node {
     type: 'Program',
     start: 0,
     end: 57,
     loc: SourceLocation { start: [Position], end: [Position] },
     sourceType: 'script',
     interpreter: null,
     body: [ [Node], [Node] ],
     directives: [] },
  comments: [] }
```

### 抽象语法树替换

> 专门用于ast语法树中的替换

1. 安装

```
npm i -S @babel/traverse
```

2. 使用

```
// 由于traverse是es6语法导出，需要添加.default获取
const traverse = require('@babel/traverse').default

traverse(ast, {
  // p 是抽象语法树的节点
  CallExpression(p){
    // 将代码中的 require 替换为 __webpack_require__
    if(p.node.callee.name === 'require'){
      p.node.callee.name = '__webpack_require__'
    }
  }
})
```

### 将ast语法树还原

> 将ast的语法树，根据traverse替换后，再转换成js代码

1. 安装

```
npm i -S @babel/generator 
```

2. 使用

```
const generator = require('@babel/generator').default
const sourceCode = generator(ast).code
```

### ejs

> 使用ejs生成模板代码

1. 在模板中使用ejs

```
return __webpack_require__(__webpack_require__.s = "<%-entry%>");
```

```
({
  <% for (let k in modules) {%>
  "<%-k%>": (function (module, exports, __webpack_require__) {

    eval(`<%-modules[k]%>`);
  }),
  <%}%>
});
```

