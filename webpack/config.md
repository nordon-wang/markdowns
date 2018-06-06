- 生产环境

  ```javascript
  // 项目的出口,首页的位置,是绝对路径
  index: path.resolve(__dirname, '../../newapi/view/home/index/index.html'),
      // 所有静态资源的根路径,必须是绝对路径
      assetsRoot: path.resolve(__dirname, '../../newapi/view/home/index'),
          // 所有被webpack处理过的资源存放的目录,具体是各种资源对应的loader处理
          assetsSubDirectory: 'static',
              // http服务器运行的根目录
              // 图片地址:http://nordon3.com/newapi/view/home/index/static/image/mbg01.png
              assetsPublicPath: '/newapi/view/home/index/',
  ```

  

```javascript
// url-loader只会编译处理html和css中的图片资源,若是在vue中使用v-for遍历图片的地址,是不能被url-loader所处理的,若是想处理这种情况的图片,
// 需要使用import图片的方式通过一个变量去接受,然后让变量放到v-for中遍历渲染
//rl-loader不能检测到js中的background，所以我们凡是在js中引用的地址，必须在外面先import这张图片，url-loader才会解析并打包
/**
 * import img1 from './img1'
 * import img2 from './img2'
 * 
 * data(){
 *  return {
 *     imgUrls:[{
 *      src:img1
 *     },{
 *      src:img2
 *     }]
 *  }
 * }
 */
```

- externals,可以将首屏用不到的文件使用externals的方式单独拎出来，不需要打包至bundle.js中,加快首屏速度

```javascript
externals: {
    jquery: "jQuery",
        lodash: {
            commonjs: 'lodash',
            commonjs2: 'lodash',
            amd: 'lodash',
            root: '_'
        }
},
```

