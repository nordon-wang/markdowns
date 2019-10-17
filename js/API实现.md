## 知识点

### JSON.parse

> JSON.parse(JSON.stringify(obj)) 常用来进行深拷贝，使用起来简单便利，但是大部分开发者在使用时往往会忽略其存在的问题

- 问题

1. 他无法实现对函数 、RegExp等特殊对象的克隆 

2. 会抛弃对象的constructor,所有的构造函数会指向Object 

3. 对象有循环引用,会报错 

```javascript
// 构造函数
function person(pname) {
  this.name = pname;
}

const Messi = new person('Messi');

// 函数
function say() {
  console.log('hi');
};

const oldObj = {
  a: say,
  b: new Array(1),
  c: new RegExp('ab+c', 'i'),
  d: Messi
};

const newObj = JSON.parse(JSON.stringify(oldObj));

// 无法复制函数
console.log(newObj.a, oldObj.a); // undefined [Function: say]
// 稀疏数组复制错误
console.log(newObj.b[0], oldObj.b[0]); // null undefined
// 无法复制正则对象
console.log(newObj.c, oldObj.c); // {} /ab+c/i
// 构造函数指向错误
console.log(newObj.d.constructor, oldObj.d.constructor); // [Function: Object] [Function: person]
```

### apply

> apply的作用是修改调用函数的当前执行上下文，在理解其作用的基础上思考模拟实现apply的大概思路，更有助于个人成长，切勿不理解时死记硬背一些代码片段

```javascript
Function.prototype._apply = function (targetObject, argsArray) {
  // 若是没有传递，则置为空数组
  if(typeof argsArray === 'undefined' || argsArray === null) {
    argsArray = []
  }

  // 是否传入执行上下文，若没有指定，则指向 window
  if(typeof targetObject === 'undefined' || targetObject === null){
      targetObject = window
  }

  // 利用Symbol的特性，设置为key
  const targetFnKey = Symbol('key')
  // 将调用_apply的函数赋值
  targetObject[targetFnKey] = this
  // 执行函数，并在删除之后返回
  const result = targetObject[targetFnKey](...argsArray)
  delete targetObject[targetFnKey]
  return result
}
```

> 使用Symbol作为key，是为了防止重复，例如targetFnKey设置为cb，当传入的targetObject自身拥有cb这个方法，就会导致执行之后便呗delete，导致问题

### new

- 执行过程
  1. 创建一个空对象，作为将要返回的对象实例
  2. 将这个空对象的原型指向构造函数的prototype属性
  3. 将这个空对象赋值给函数内部的this
  4. 执行构造函数内部代码

```javascript
function _new(/* 构造函数 */ constructor, /* 构造函数参数 */ params) {
  // 将 arguments 对象转为数组
  var args = [].slice.call(arguments);
  // 取出构造函数
  var constructor = args.shift();
  // 创建一个空对象，继承构造函数的 prototype 属性
  var context = Object.create(constructor.prototype);
  // 执行构造函数
  var result = constructor.apply(context, args);
  // 如果返回结果是对象，就直接返回，否则返回 context 对象
  return (typeof result === 'object' && result != null) ? result : context;
}
```

> 在理解new的原理，需要补充两个小的知识点

1. new.target, 函数内部可以使用它，如果当前函数是使用 new进行调用，那么new.target指向当前函数，否则为undefined

```javascript
function NewTargetTest() {
  console.log(new.target === NewTargetTest)
}

NewTargetTest() // false
new NewTargetTest() // true
```

2. 构造函数隐藏的return

- 构造函数会默认返回构造后的this对象

```javascript
function ReturnTest(name) {
  this.name = name
}

const returnText = new ReturnTest('wy')
console.log(returnText.name) // wy
```

- 将上面的代码稍加改造，显示的返回一个空对象{},此时会覆盖默认返回的this对象

```javascript
function ReturnTest(name) {
  this.name = name
  return {}
}

const returnText = new ReturnTest('wy')
console.log(returnText.name) // undefined
```

- 将上面的代码稍加改造，显示的返回一个基本类型的数据，此时将不会影响构造函数返回的this对象

```javascript
function ReturnTest(name) {
  this.name = name
  return 'test'
}

const returnText = new ReturnTest('wy')
console.log(returnText.name) // wy
```

> 总结：在构造函数中，若显示的返回一个对象，则会覆盖默认的this对象，基本数据类型则不会

### 单例模式

> 单例模式是一种常见的设计模式，单例模式能够保证一个类仅有唯一的实例，并提供一个全局访问点，可以很好节省内存。在js中，可以结合必包实现

```javascript
function Animal(name) {
  this.name = name
}

const AnimalSingle = (function () {
  let animalSingle = null
  
  return function (name) {
    if(animalSingle){
      return animalSingle
    }
    return animalSingle = new Animal(name)
  }
})();

const animal1 = new AnimalSingle('dog')
const animal2 = new AnimalSingle('cat')

console.log(animal1.name); // dog
console.log(animal2.name); // dog
```

> Animal只会被实例化一次，且之后的每次的实例化都会返回第一次的。

### compose

> compose 是函数式编程中很重要的函数之一， 因为其巧妙的设计而被广泛使用。compose函数的作用就是组合函数的，将函数串联起来执行，将多个函数组合起来，一个函数的输出结果是另一个函数的输入参数，一旦第一个函数开始执行，就会像多米诺骨牌一样推导执行了

- lodash 版本

```javascript
var compose = function(funcs) {
    var length = funcs.length
    var index = length
    while (index--) {
        if (typeof funcs[index] !== 'function') {
            throw new TypeError('Expected a function');
        }
    }
    return function(...args) {
        var index = 0
        var result = length ? funcs.reverse()[index].apply(this, args) : args[0]
        while (++index < length) {
            result = funcs[index].call(this, result)
        }
        return result
    }
}
```

- Redux 版本

```javascript
function compose(...funcs) {
    if (funcs.length === 0) {
        return arg => arg
    }

    if (funcs.length === 1) {
        return funcs[0]
    }

    return funcs.reduce((a, b) => (...args) => a(b(...args)))
}
```

- 测试代码

```javascript
function composeReduce1(arg) {
  console.log('composeReduce1', arg);
  return 'composeReduce1'
}
function composeReduce2(arg) {
  console.log('composeReduce2', arg);
  return 'composeReduce2'
}
function composeReduce3(arg) {
  console.log('composeReduce3',arg);
}


// lodash版本的 compose参数是一个数组 [composeReduce1, composeReduce2, composeReduce3]
let composeChild = compose(composeReduce1, composeReduce2, composeReduce3)
composeChild('init')

// 输出
// composeReduce3 init
// composeReduce2 composeReduce3
// composeReduce1 composeReduce2
```

### 属性读取

> cannot read property of undefined 是一个常见的错误，如果意外的得到了一个空对象或者空值，便会报错

- 数据结构

```javascript
const obj = {
  user: {
      posts: [
          { title: 'Foo', comments: [ 'Good one!', 'Interesting...' ] },
          { title: 'Bar', comments: [ 'Ok' ] },
          { title: 'Baz', comments: []}
      ],
      comments: []
  }
}
```

- && 短路运算符进行可访问性嗅探

```
obj.user && obj.user.posts
```

- try...catch

```javascript
let result
try {
    result = obj.user.posts[0].comments
}
catch {
    result = null
}
```

- 提取方法 - reduce

```javascript
const getByReduce = (attrArr, resObj) =>{
  return  attrArr.reduce((res, key) => {
    return (res && res[key]) ? res[key] : null
  }, resObj)
}

console.log(getByReduce(['user', 'posts', 0, 'comments'], obj)) // [ 'Good one!', 'Interesting...' ]
console.log(getByReduce(['user', 'post', 0, 'comments'], obj)) // null
```

- 提取方法 - 柯理化

```javascript
const getByCurry = attrArr => {
  return resObj => {
    return attrArr.reduce((res, key) => {
      return res && res[key] ? res[key] : null;
    }, resObj);
  };
};

const getUserComments = getByCurry(['user', 'posts', 0, 'comments']);

console.log(getUserComments(obj)); // [ 'Good one!', 'Interesting...' ]
console.log(getUserComments({ user: { posts: [] } })); // null
```

### 原型污染

> 通过原型可以将原型链上面的方式和属性进行污染

```js
let person = {name: 'lucas'}

console.log(person.name)

person.__proto__.toString = () => {alert('evil')}

console.log(person.name)

let person2 = {}

console.log(person2.toString())
```

- 解决
  - 冻结 `Object.prototype`，使原型不能扩充属性
  - 遇见 `constructor` 以及 `__proto__` 敏感属性，阻止其操作

### JSONP

> 实现一个jsonp，虽然现在jsonp还是存在部分的使用场景的，即使其只能支持get类型的请求等缺陷，但是还是需要掌握其整个流程是怎么样的
>
> 以百度搜索为例，当在百度搜索时，当输入框的内容变化便会去搜素关键字，就是通过jsonp实现的

```js
function jsonp(url, callback, successCallback) {
  let script = document.createElement('script');
  script.src = url;
  script.async = true;
  script.type = 'text/javascript';

  window[callback] = function(data) {
    successCallback && successCallback(data);
  };

  document.body.appendChild(script);
}

jsonp(
  'https://www.baidu.com/sugrec?pre=1&p=3&ie=utf-8&json=1&prod=pc&from=pc_web&sugsid=1461,21119,18559,29522,29720,29567,29221&wd=%E5%88%98%E5%BE%B7%E5%8D%8E%20&bs=%E5%88%98%E5%BE%B7%E5%8D%8E&pbs=%E5%88%98%E5%BE%B7%E5%8D%8E&csor=4&pwd=%E5%88%98%E5%BE%B7%E5%8D%8E&cb=jQuery1102024053669643223596_1570162206732&_=1570162206766',
  'jQuery1102024053669643223596_1570162206732',
  function(data) {
    console.log(data);
  }
);
```

返回内容

```js
{
  q: '刘德华 ',
  p: false,
  g: [
    { type: 'sug', sa: 's_1', q: '刘德华 文慧' },
    { type: 'sug', sa: 's_2', q: '刘德华生日是几月几日' },
    { type: 'sug', sa: 's_3', q: '刘德华电影全集国语 大全在线观看' },
    { type: 'sug', sa: 's_4', q: '刘德华 十七岁' },
    { type: 'sug', sa: 's_5', q: '刘德华老婆是哪个' },
    { type: 'sug', sa: 's_6', q: '刘德华 生日' },
    { type: 'sug', sa: 's_7', q: '刘德华 今天' },
    { type: 'sug', sa: 's_8', q: '刘德华 歌曲' },
    { type: 'sug', sa: 's_9', q: '刘德华 古天乐' },
    { type: 'sug', sa: 's_10', q: '刘德华多少岁了今年' }
  ],
  slid: '9198684244459417731'
}
```







