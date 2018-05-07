- NuxtJS 基于vue的 也可以做SPA

- SSR 
    - 服务器端 -> vue渲染html -> 浏览器
    - SEO
    - 首屏加载速度比传统SPA快

# 安装
- npm install vue-cli -g
- vue init nuxt/starter
- npm install
- npm run dev


- 在package.json中配置IP和端口
```
    "config":{
        "nuxt":{
            "host":"127.0.0.1",
            "port":"8888"
        }
    }
```

- 配置全局引用的css
    - nuxt.config.js 增加css选项
    ```
        css:['~assets/common.css']
    ```
    
    - 在assets中增加css文件(可在新建的css目录中新建css文件)
    
- webpack 配置
    - nuxt中 在nuxt.config.js中build选项中配置
    - 写法和webpack一样,不需要``下载``直接使用
    ```
    loaders:[
      {
        test:/\.(png|jpg|gif|svg)$/,
        loader:"url-loader",
        query:{
          limit:10000,
          name:'img/[name].[hash].[ext]'
        }
      }
    ],
    ```
    
# 路由
- nuxt 中的路由以及默认配置好了
- 只需要在pages目录中穿件对应的目录和vue文件即可
    - nuxt-link 代替 a, 通过params进行传递参数
    -  $route.params.xxx 进行接受参数
```    
    <nuxt-link :to="{
        name:'news',
        params:{
            newsID:6666
        }
    }"> links </nuxt-link>
    
    {{$route.params.newsID}}
```
    
- 动态路由
    - <a href="/news/123">News-1</a>
    - {{$toute.params.id}} 接收
```
news文件夹下面新建了_id.vue的文件，以下画线为前缀的Vue文件就是动态路由，然后在文件里边有 $route.params.id来接收参数。
```
- 动态路由使用nuxt-link传递
```
name名称需要注意 在路由后加上'-'和动态参数
name:'about-id'
<nuxt-link :to="{
	name:'about-id',
	params:{
		id:3333
	}
}">3333</nuxt-link>
```
 - 动态路由校验
    - 传入的id必须为数字
 ```
    validate({params}){
		return /^\d+$/.test(params.id)
	}
 ```
 
 - 增加动画
```
- 全局增加动画效果
.page-enter-active, .page-leave-active{
	transition: opacity 2s;
}

.page-enter, .page-leave{
	opacity: 0;
}
```
```
- 单个动画效果

- css文件中
.test-enter-active, .test-leave-active{
	transition: all 2s;
	font-size: 12px;
}

.test-enter, .test-leave{
	font-size: 40px;
}

- 需要对应的vue文件中
export default {
	transition:'test'
}

```

- 默认模板
    -  在根目录增加一个app.html
```
<!DOCTYPE html>
<html lang="en">
<head>
	{{HEAD}}
</head>
<body>
	<header>my header ....... app</header>
	{{APP}}
</body>
</html>
```

- 默认布局
   - 在layouts/default.vue中修改
```
<template>
  <div>
    <h2>layout template.........</h2>
    <nuxt/>
  </div>
</template>
```

- meta设置
```
data(){
    return{
      title:this.$route.params.title,
    }
  },
//独立设置head信息
  head(){
      return{
        title:this.title,
        meta:[
          {hid:'description',name:'news',content:'This is news page'}
        ]
      }
    }
```

- asyncData 获取数据
    - this此时不能使用 
```
async asyncData(){
	let {data} = await axios.get('https://api.myjson.com/bins/129udv')
	
	return {infor:data}
}

在tempalte中 直接使用infor进行数据遍历渲染
```