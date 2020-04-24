## code review

- `debounce `使用，例如远程搜索时需要通过接口动态的获取数据，若是每次用户输入都接口请求，是浪费带宽和性能的

```html
<Select :remote-method="remoteMethod">
    <Option v-for="item in temoteList" :value="item.value" :key="item.id">{{item.label}}</Option>
</Select>
```

```js
import {debounce} from 'lodash'

methods：{
    remoteMethod： debounce(function (query) {
        // to do ...
    }, 200),
}
```

- `Select `下拉框遍历时，需要注意`options`标签保持同一行

若是存在换行，会导致选中时的值存在多余的空白

![image-20200407190118125](E:\me\markdowns\code review\img\image-20200407190118125.png)

```html
<!-- bad -->
<Select :remote-method="remoteMethod">
    <Option v-for="item in temoteList" :value="item.value" :key="item.id">
        {{item.label}}
    </Option>
</Select>
```

需要将`Options`和下拉框的值保持在同一行

```html
<!-- good -->
<Select :remote-method="remoteMethod">
    <Option v-for="item in temoteList" :value="item.value" :key="item.id">{{item.label}}</Option>
</Select>
```

- `table`中`columns`数据可以单独提取一个外部`js`文件作为配置文件，也可以在当前`.vue`文件中定义一个常量定义`columns`数据，因为无论如何都是固定且不会修改的数据，应该使用`Object.freeze`进行包裹，既可以提高性能还可以将固定的数据抽离，一些下拉框前端固定的数据也建议此操作

```js
const columnList = Object.freeze([
  { title: '姓名', key: 'name', align: 'center' },
  { title: '性别', key: 'gender', align: 'center' }
])
```

- `data`数据具有数据层级结构，切勿过度扁平化或者嵌套层级过深，若是过度扁平化会导致数据命名空间冲突，参数传递和处理，若是层级嵌套过深也会导致`vue`数据劫持的时候递归层级过深，若是嵌套层级丧心病狂那种的，小心递归爆栈的问题。而且层级过深会导致数据操作和处理不便，获取数据做容错处理也比较繁琐。一般层级保持2-3层最好。

若是只有一层数据，过于扁平

```json
{
    name: '',
    age: '',
    gender: ''
}
```

导致处理不方便

```js
// 作为接口参数传递
ajax({
	this.name, this.age, this.gender
})

// 接口获取数据，批量处理
ajax().then(res => {
	const {name, age, gender} = res.data
    this.name = name
    this.age = age
    this.gender = gender
})
```

适当的层级结构不仅增加代码的维护和阅读性，还可以增加操作和处理的便捷性

```js
{
    person: { // 个人信息
        name: '',
        age: '',
        gender: ''
    }
}
```

可以针对`person`进行操作

```js
// 作为接口参数传递
ajax(this.person)

// 接口获取数据，批量处理
ajax().then(res => {
	const {name, age, gender} = res.data
    this.$set(this, 'person', {name, age, gender})
})
```

- 策略模式的使用，避免过多的`if else`判断，也可以替代简单逻辑的`switch`

```js
const formatDemandItemType = (value) => {
    switch (value) {
        case 1:
            return '基础'
        case 2:
            return '高级'
        case 3:
            return 'VIP'
    }
}

// 策略模式
const formatDemandItemType2 = (value) => {
    const obj = {
        1: '基础',
        2: '高级',
        3: 'VIP',
    }
    
    return obj[value]
}
```

- Modal框的控制

一个页面种通常会存在很多个不同功能的弹框，若是每一个弹框都设置一个对应的变量来控制其显示，则会导致变量数量比较冗余和命名困难，可以使用一个变量来控制同一页面中的所有`Modal`弹框的展示

比如某个页面中存在三个`Modal`弹框

```js
// bad
// 每一个数据控制对应的Modal展示与隐藏
new Vue({
    data() {
        return {
            modal1: false,
            modal2: false,
            modal3: false,
        }
    }
})

// good
// 当modalType为对应的值时 展示其对应的弹框
new Vue({
    data() {
        return {
            modalType: '' // modalType值为 modal1，modal2，modal3
        }
    }
})
```

- 注意组件给出的改变时的值和类型

例如`select`下拉框设置`multiple`时，选中的值发生改变时`on-change`事件触发，此时`on-change`的参数类型是`Array`，若是需要一些逻辑判断需要注意

```js
// bad
{
  onChange(val) {
    if(val !== ''){
      // 切勿认为当清空选择的时候以为val为空字符串''
    }
  }
}
```

- `html`中展示一些如`<`，`>`,`&`等字符时，使用字符实体代替

```html
<!-- bad -->
<div>
  > 1 & < 12
</div>
  
<!-- bad -->
<div>
  &gt; 1 &amp; &lt; 12
</div>
```

- 解构赋值以及默认值，当解构的数量小于多少时适合直接解构并赋值默认值，数据是否进行相关的聚合处理

```js
const {
  naem = '',
  age = 10,
  gender = 'man'
} = res.data

// bad
this.name = name
this.age = age
this.gender = gender

// good
this.person = {
  naem,
  age,
  gender
}
```

- 编写`template`模板时，属性过多时，是否换行

```vue
<template>
<!-- 不换行 -->
<VueButton class="icon-button go-up" icon-left="keyboard_arrow_up" v-tooltip="$t('org.vue.components.folder-explorer.toolbar.tooltips.parent-folder')" @click="openParentFolder" />

<!-- 换行 -->
<VueButton
  class="icon-button go-up"
  icon-left="keyboard_arrow_up"
  v-tooltip="$t('org.vue.components.folder-explorer.toolbar.tooltips.parent-folder')"
  @click="openParentFolder"
/>
</template>
```



