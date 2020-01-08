

## install 

在`vue`项目中，使用`vuex`进行数据管理，首先做的就是将`vuex`引入并`Vue.use(Vuex)`

```js
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)
```

在执行`Vue.use(Vuex)`时，会触发`vuex`中暴露出来的方法`install`进行初始化