## 背景

- 一些营销页面在微信中分裂传播时，容易被封杀
- `web view`嵌套`h5`页面和原生小程序页面存在一些差异
- 营销页面使用`taro`编写，但是与载体解耦，方便嵌套在多个项目中

## 前期梳理

核心需求是将现有`h5`页面转为`taro`页面且作为独立项目存在

使用`taro`将营销原有功能和页面重新编写，需多加留意业务逻辑在`h5`和`小程序`间的差异

作为单独项目存在可以方便快速复用在不同的项目中，可以将页面作为路由级别的组件进行封装处理，在宿主容器中只需要提供一个载体使用组件即可

- 如何保持营销项目与宿主项目的开发、打包一致性

  `taro`是支持多端的，宿主项目在开发时可以通过指令启动项目，而项目可以为各种`小程序`和`h5`页面，此时将营销项目作为一个第三方多端组件库使用，会在宿主容器编译时将营销项目一起编译，可以保证编译目标一致

## 第三方库开发

通过taro指令创建一个基础项目，因为第三方的`UI`库和`taro`项目目录结构基本保持一致

创建完成项目之后便需要进行改造

1. 创建`src/componenets`文件夹，里面存放的组件即为作为路由页面使用的组件
2. 在`src`目录下增加一个入口文件`index.js`, 主要是用于输出`UI`组件，打包之后供宿主项目导入使用

```javascript
export { default as Test } from './components/test/test'
export { default as Foo } from './components/foo/foo'
```

3. 配置文件改造

   为了打包出可以在 H5 端使用的组件库，需要在 `config/index.js` 文件中增加一些配置

   ```javascript
   if (process.env.TARO_BUILD_TYPE === 'ui') {
     Object.assign(config.h5, {
       enableSourceMap: false,
       enableExtract: false,
       enableDll: false
     })
     config.h5.webpackChain = chain => {
       chain.plugins.delete('htmlWebpackPlugin')
       chain.plugins.delete('addAssetHtmlWebpackPlugin')
       chain.merge({
         output: {
           path: path.join(process.cwd(), 'dist', 'h5'),
           filename: 'index.js',
           libraryTarget: 'umd',
           library: 'taro-ui-sample'
         },
         externals: {
           nervjs: 'commonjs2 nervjs',
           classnames: 'commonjs2 classnames',
           '@tarojs/components': 'commonjs2 @tarojs/components',
           '@tarojs/taro-h5': 'commonjs2 @tarojs/taro-h5',
           'weui': 'commonjs2 weui'
         }
       })
     }
   }
   ```

4. 打包命令

   通过打包指令便可以将其打包

   ```shell
   $ TARO_BUILD_TYPE=ui taro build --ui
   ```

至此基础的第三方UI库准备工作已完成

此项目可以通过打包指令作为`npm`包使用，也可以作为独立的项目使用

当作为单独的项目使用时，将`components`中的路由页面引入放置在`pages`目录下，作为项目的组件进行使用即可

## 发布 npm

已经准备好第三方库，接下来便是将其作为`npm`包进行发版，在发布`npm`时除了常规的`npm publish`等常规操作之外，需要注意两点

1. `.gitignore`不能包含`dist`目录

   执行`TARO_BUILD_TYPE=ui taro build --ui`便会将组件库打包，需要将打包生成的`dist`目录发布且上传

2. 修改`package.json`的`main`属性、

   当作为第三方库被下载引入使用时，需要指定入口

```json
{
  "main": "dist/index.js",
  "main:h5": "dist/h5/index.js",
}
```

## 使用

`npm`包发版之后，便可以通过`npm i xxx`下载并在项目中使用

注意点

​	配置需要额外编译的源码模块在`config/index.js`增加`esnextModules`配置

​	由于引用 `node_modules` 的模块，默认不会编译，所以需要额外给 H5 配置 `esnextModules`，在 taro 项目的 `config/index.js` 中新增如下配置项

```javascript
const condif = {
  h5: {
    esnextModules: ['xxx'] // npm包名
  }
}
```

在项目中使用

```react
import { Foo } from 'hb-market'

class Index extends Component{
  
  render() {
    return (
    	<View>
      	<Foo></Foo>
      </View>
    )
  }
}
```



