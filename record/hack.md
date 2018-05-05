- 自定执行加载组件
```
定义
import Vue from 'vue'

export default {
    install(Vue,){
        console.log('====================================');
        console.log('vue install ........');
        console.log('====================================');
    }
}

加载
import Vue from 'vue'
import MyD from './d'
Vue.use(MyD);
```

- 加载组件
```
Object.defineProperty(Vue.prototype, '$message', { value: Message });
Object.defineProperty(Vue.prototype, '$notify', { value: Notification });
```

- 加载状态
    - document.onreadystatechange 页面加载状态改变时的状态
    - document.readyState 返回当前文档的状态、服务端返回的状态
        - uninitialized 还未开始载入
        - loading 载入中
        - interactive 已加载、文档与用户可以开始交互
        - complete 载入完成
```
document.onreadystatechange = ()=> {
    // 文档加载的状态
    console.log(document.readyState)
}
```

- vue.$nextTick()
```js
// 之前的vue中nextTick使用，但是为了兼容放弃了
// <h1 class="mos">MutationObserver-obj</h1>
// 被监听的目标对象
let $target =  document.querySelector('h1.mos')

// 实例化 MutationObserver
let observer =  new MutationObserver( (mutations) => {
  mutations.forEach( item => {
    console.log('....',item.type)
  })
})

// 配置
let config = { attributes: true, childList: true, characterData: true }
// 监听开始
observer.observe($target,config)
// 结束监听
observer.disconnect();
```

- 判断类型
```js
console.log([].slice.call([1,2,3,4],2))
console.log(Array.prototype.slice.call([1,2,3,4],2))
console.log([].__proto__ === Array.prototype)
console.log(Object.prototype.toString.call([]))
String.prototype.charAt.call('asd',1)
''.charAt.call('asd',2)

Object.is(NaN,NaN) //true
Object.is(0,-0) //false
```