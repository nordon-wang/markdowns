# get
    -  获得GET请求的方式有两种，一种是从request中获得，一种是一直从上下文中获得。获得的格式也有两种：query和querystring
```
- 请求地址
http://localhost:8080/?name=nordon&age=21

- 接受处理 从request中取
let url = ctx.url
let query = ctx.request.query
let querystring = ctx.request.querystring

- 从上下文中直接取
let query = ctx.query
let querystring = ctx.querystring

- 结果
{ url: '/?name=nordon&age=21',
  query: { name: 'nordon', age: '21' },
  querystring: 'name=nordon&age=21' }
```

# post

- 获取post请求步骤
    - 解析上下ctx中的原声node.js对象req
    - 将post表单数据解析成query string字符串
    - 将字符串转换成JSON
- ctx.request 和 ctx.req 区别
    - ctx.request是koa2中context经过封装的请求对象，用起来更直观、简单
    - ctx.req是context提供的node原生HTTP请求对象，可以得到更多的内容，适合深度编程

- ctx.method
    - koa2中提供ctx.method属性，得到请求的类型
    - 
    - 
- 封装promise进行数据解析,进行post数据处理
```
function myPromise(ctx) {
	return new Promise((resolve,reject) => {
		try{
			let postdata = '';
			ctx.req.on('data',(data) => {
				postdata += data;
			})

			ctx.req.on('end',() => {
				resolve(postdata);
			})
		}catch(err){
			reject(err);
			console.log(err);
		}
	})
}
获取post数据
let data = await myPromise(ctx);
```

# 原生路由
- 封装fs读取文件方法为一个promise对象
```
function myPromise(url) {
	return new Promise((resolve,reject) => {
		fs.readFile(`./template${url}.html`,'utf-8',(err,data)=> {
			if(err){
				reject(err)
			}
			resolve(data)
		})
	})
}
```
- 根据不同的路径 进行文件读取和渲染
```
let data
try {
    let url = ctx.request.url;
    data = await myPromise(url)
} catch (err) {
    data = await myPromise('/404')
}
ctx.body = data;
```
- 设置前缀
1. 全局设置、每个路由的访问都需要增加这个前缀，但是这种设置前缀并不能实现路由的层级  
```
let router = new Router({
	prefix:'/wy'
});

http://localhost:8080/wy/news
```
2. 使用koa-router分别为每个路由设置层级
```
//定义每个路由
let home = new Router()
let page = new Router()

//路由挂载
let router = new Router()
router.use('/home',home.routes(),home.allowedMethods())
router.use('/page',page.routes(),page.allowedMethods())

//使用中间件
server.use(router.routes()).use(router.allowedMethods())
```

# cookie
- 设置cookie
```
ctx.cookies.set('username','nordon',{
	domain:'127.0.0.1',
	path:'/',
	maxAge:1000 * 3600 * 24,
	expires:new Date('2018-01-01'),
	httpOnly:false, // 是否只用于http请求中获取
	overwrite:false // 是否允许重写
})

```
- 获取cookie
```
ctx.cookies.get('username')
```


# 中间件
- koa-bodyparser 处理post数据
```
使用
const bodyParser = require('koa-bodyparser')
server.use(bodyParser())
处理数据
let data = ctx.request.body
```
- koa-router
1. 单个路由
```
const Router = require('koa-router')
let router = new Router()
router.get('/',(ctx,next) => {
    ....
})
server.use(router.routes()).use(allowedMethods())
```
2. 增加路由前缀
```
// home路由
let home = new Router()
home.get('/h1',(ctx,next) => {
	ctx.body = 'h1....'
}).get('/h2',(ctx,next) => {
	ctx.body = 'h2....'
});

// page路由
let page = new Router()
page.get('/p1',(ctx,next) => {
	ctx.body = 'p1...'
}).get('/p2',(ctx,next) => {
	ctx.body = 'p2....'
})

// 装载所有子路由
let router = new Router()
router.use('/h',home.routes(),home.allowedMethods())
router.use('/p',page.routes(),page.allowedMethods())

server.use(router.routes()).use(router.allowedMethods())

```

3. ejs
    1. 在koa2中使用模板机制必须依靠中间件(koa-views)
    2. ejs模板引擎
    3. cnpm install --save koa-views
    4. npm install --save ejs
```
// 加载模板引擎
app.use(views(path.join(__dirname, './view'), {
  extension: 'ejs'
}))
// 渲染
await ctx.render('index', {
title
})
```

4. 静态资源管理
    1. 使用koa-static
```
const staticPath = './static'
 
app.use(static(
  path.join( __dirname,  staticPath)
))
```

5. koa-compose
 1. 组成给定的中间件和返回中间件