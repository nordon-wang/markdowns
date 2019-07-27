- 学习目标
  - 了解webpack打包原理
  - 了解webpack的loader原理
  - 了解webpack的plugin原理
  - 了解ast抽象语法树的应用
  - 了解tapable的原理
  - 手写一个简单的webpack

## 1.准备工作

1. 新建一个项目 webpack-theory

2. 新建 bin 目录，创建``oyo-pack.js`文件， 将打包工具主程序放入其中

   主程序的顶部应当有: `#!/usr/bin/env node` 标识，指定程序执行环境为 node

   ```
   #!/usr/bin/env node
   // log的内容修改直接，可以直接生效
   console.log('当通过npm link链接之后，通过oyo-pack指令可以直接打出');
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