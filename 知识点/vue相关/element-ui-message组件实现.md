## Element-UI: Message组件解读

`Message`在开发中的使用频率很高，也算是`Element-UI`组件库中比较简单的，对于感兴趣的朋友可以一起探讨一下`Message`组件的实现

> 组件功能是什么

常用于主动操作后的反馈提示，比如提交表单时成功或者失败的提示信息展示

> 组件使用方法

```js
this.$message('这是一条消息提示');

this.$message({
	message: '恭喜你，这是一条成功消息',
  type: 'success'
});
```

> 组件设计思路

`Message`的调用方式都是通过`this.$message`进行调用，通过传递不同的`options`进行组件样式和内容的控制，展示的`html`是动态的插入到`document`中并在`duration`之后移除，组件的展示通过`vue`实例访问并控制。

组件的整体结构分为展示部分和控制部分

- 展示部分单独抽离出一个组件，将组件的展示逻辑和交互封装集中处理
- 控制部分是承接`vue`实例和组件展示

整体执行过程

`Vue`项目中

```js
// 1.引入组件库
import ElementUI from 'element-ui';

// 2.使用组件库
Vue.use(ElementUI); 
```

`Element-UI`组件库中逻辑：每次当`Vue.use`的时候，在`Element—UI`内部会触发`Element-UI`的`install`方法，然后将组件注册为全局组件，将方法放到`Vue.prototype`上，本次只看Message部分即可

```js
// 1.引入Message对象
import Message from '../packages/message/index.js';

// 2. 定义 install函数，
const install = function(Vue, opts = {}) {
  // 将组件遍历注册为全局组件，例如Button组件
  components.forEach(component => {
    Vue.component(component.name, component);
  });
	// 将方法放到Vue原型上
  Vue.prototype.$message = Message;
};
```

经过上述两步的处理，边可以直接在项目中通过`this.$message`进行组件的展示控制，接下来继续探索`Element-UI`内部如何处理的

## 展示部分

首先看一下删减版本之后展示部分的组件内容，代码将部分逻辑展示删除，紧紧展示了基本功能，显示`icon`等功能可自行查看，比较简单

```vue
<template>
  <transition name="el-message-fade" @after-leave="handleAfterLeave">
    <div
      class="el-message"
      :style="positionStyle"
      v-show="visible">
      <slot>
        <p>{{ message }}</p>
      </slot>
    </div>
  </transition>
</template>

<script type="text/javascript">
export default {
  data() {
    return {
      visible: false,
      message: '',
      duration: 3000,
      onClose: null,
      closed: false,
      verticalOffset: 20,
      timer: null
    };
  },

  computed: {
    positionStyle() { // 控制当前组件的显示位置
      return {
        'top': `${ this.verticalOffset }px`
      };
    }
  },

  watch: {
    // 监听closed的变化，设置为true时，将组件销毁
    closed(newVal) {
      if (newVal) {
        this.visible = false;
      }
    }
  },

  methods: {
    // transtion组件的钩子，触发after-leave时执行
    handleAfterLeave() {
      this.$destroy(true); // 销毁组件
      this.$el.parentNode.removeChild(this.$el); // 将组件的DOM移除
    },

    close() {
      this.closed = true; // 组件隐藏
      
      if (typeof this.onClose === 'function') {
        this.onClose(this);
      }
    },
    // 每次手动启动编译之后 设置其展示时间duration之后关闭
    startTimer() {
      if (this.duration > 0) {
        this.timer = setTimeout(() => {
          if (!this.closed) {
            this.close();
          }
        }, this.duration);
      }
    }
  },
  mounted() {
    this.startTimer();
  }
};
</script>

<style>
.el-message{
  min-width: 280px;
  height: 42px;
  box-sizing: border-box;
  border-radius: 4px;
  border: 1px solid #ebeef5;
  position: fixed;
  left: 50%;
  top: 20px;
  transform: translateX(-50%);
  background-color: #edf2fc;
  transition: opacity .3s,transform .4s,top .4s;
  overflow: hidden;
  padding: 15px 15px 15px 20px;
  display: flex;
  align-items: center;
  background-color: #f0f9eb;
  border-color: #e1f3d8;
}
</style>
```

> `template`部分

使用了`Vue`官方封装的`transition`组件，不仅提供了良好的过渡效果，还提供了合适的钩子便于开发者控制，组件中使用`after-leave`钩子，当组件销毁时进行组件的销毁和`DOM`的移除，`visible`用于控制组件的展示与销毁，计算属性`positionStyle`用于设置组件的展示位置，`message`为组件展示的内容数据，搞明白这些变量、计算属性和方法的作用便可以

> `script`部分可参考注释进行理解，需要注意两个地方

首先需要注意生命周期钩子`mount`时做的事情，为何如此做？因为不存在el选项，实例不会立即进入编译阶段，需要显示调用$mount 手动开启编译

还需要注意的时`close`函数中做了两件事，设置`closed`的值触发对应的`watch`，关闭组件，若是存在`onClose`方法则调用，注意这个`onClose`函数的定义是在控制部分定义，稍后会说明

## 控制部分

至此已经清楚`Vue`中是通过`this.$message`触发组件的展示，而展示部分的组件内容也已完成，现在就需要通过控制部分将两者链接，达到期望的功能

与`Vue`关联比较简单，仅仅是定义一个方法并将其导出即可

```js
const Message = options => {
  // 逻辑编写....
}

export default Message;
```

将其引入并绑到`Vue`原型上，省略无关代码

```js
// 引入Message对象
import Message from '../packages/message/index.js';

// 将方法放到Vue原型上
Vue.prototype.$message = Message;
```

这个时候通过`this.$message`即可调用，接下来便是将`Message`函数与组件关联，并控制展示部分

`Message`核心需要做那些事情

- 编译组件，使用渲染并插入到`body`中

- 控制组件内的`visible`变量，触发组件的展示
- 控制组件内的`verticalOffset`变量，决定组件展示时的位置

> 手动开启组件编译，获取其实例访问内部`data`和渲染到页面上

```js
// 1. 使用基础 Vue 构造器，创建一个“子类”
let MessageConstructor = Vue.extend(Main);
// 2. 组件实例, 可以通过instance访问 visible和verticalOffset
instance = new MessageConstructor({
  data: options
});
```

整个`Message`方法其余部分就是在做容错和健壮处理，整体简洁版代码如下

```js
let MessageConstructor = Vue.extend(Main);

let instance; // 当前组件
let instances = []; // 将所有的组件收集，用于位置的判断和销毁等
let seed = 1;

const Message = options => {
  // 健壮性处理
  if (Vue.prototype.$isServer) return;
  options = options || {
    message: 'content' + Date.now(),
    onClose(message){
      console.log('关闭时的回调函数, 参数为被关闭的 message 实例',message);
    }
  };

  if (typeof options === 'string') {
    options = {
      message: options
    };
  }

  // 关闭时的回调函数, 参数为被关闭的 message 实例
  let userOnClose = options.onClose;
  let id = 'message_' + seed++;
  
  // 增加 onClose 方法，组件销毁时，在组件内部调用
  options.onClose = function() { 
    Message.close(id, userOnClose);
  };

  // 组件实例
  instance = new MessageConstructor({
    data: options
  });
  instance.id = id; // 设置ID
  
  instance.$mount(); // 因为不存在el选项，实例不会立即进入编译阶段，需要显示调用$mount 手动开启编译
  document.body.appendChild(instance.$el); // 将Message 组件插入到body中

  // 设置组件距离顶部的距离
  let verticalOffset = options.offset || 20;
  instances.forEach(item => {
    verticalOffset += item.$el.offsetHeight + 16;
  });
  instance.verticalOffset = verticalOffset;

  instance.visible = true; // 控制展示
  instance.$el.style.zIndex = 99; // 控制层级
  instances.push(instance);
  return instance;
};
```

`Message`组件支持`this.$message.error('错了哦，这是一条错误消息');`调用使用，到目前为止还不支持，代码比较简单直接上代码

```js
// 为每个 type 定义了各自的方法，如 Message.success(options)，可以直接调用
['success', 'warning', 'info', 'error'].forEach(type => {
  Message[type] = options => {
    if (typeof options === 'string') {
      options = {
        message: options
      };
    }
    options.type = type;
    return Message(options);
  };
});
```

展示组件内部会调用`this.onClose(this)`，组件内部设置`this.visible=false`关闭弹框，并且移除其对应的`DOM`结构，但是页面展示多个组件时需要修改其余组件的位置

 `onClose`函数是在`Message`函数中定义

```js
// 关闭时的回调函数, 参数为被关闭的 message 实例
let userOnClose = options.onClose;

// 增加 onClose 方法，组件销毁时，在组件内部调用
options.onClose = function() { 
  Message.close(id, userOnClose);
};
```

`onClose`函数最终调用的是`Message`上的静态方法`close`

函数`Message.close`内部主要做了几件事情

- 在页面显示的组件数组中找到需要关闭的组件，将其移除
- 重新计算剩余组件的位置

```js
Message.close = function(id, userOnClose) {
  let len = instances.length;
  let index = -1;

  for (let i = 0; i < len; i++) {
    // 若是匹配到 则介绍此for循环
    if (id === instances[i].id) {
      index = i;
      // 执行初始化组件时传入的onClose回调函数, 参数为被关闭的 message 实例
      if (typeof userOnClose === 'function') { 
        userOnClose(instances[i]);
      }
      instances.splice(i, 1);
      break;
    }
  }
  
  console.log(`每个close ${len} -- ${index}`);
  // 当只存在一个时组件展示或者没有匹配到需要销毁的组件时，不需要继续处理页面已有组件的展示位置
  if (len <= 1 || index === -1 || index > instances.length - 1) return;

  // 每一次销毁一个有效组件时，而页面还存在超过一个的组件，需要将页面展示的组件进行位置调整
  const removedHeight = instances[index].$el.offsetHeight;
  for (let i = index; i < len - 1 ; i++) {
    let dom = instances[i].$el;
    dom.style['top'] =
      parseInt(dom.style['top'], 10) - removedHeight - 16 + 'px';
  }
};
```

至此基础版本的`Message`已经完成，组件的代码不到200行，通过源码的简单阅读和分析，知识点并不是很多，但是优秀组件封装的思路还是值得学习和借鉴

