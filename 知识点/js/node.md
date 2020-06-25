# 优化

1.给出一个`20180408000000`字符串，formatDate函数要处理并返回`2018-04-08 00:00:00` 

```javascript
let _dete='20180408000000'
function formatStr(str){
    return str.replace(/(\d{4})(\d{2})(\d{2})(\d{2})(\d{2})(\d{2})/, "$1-$2-$3 $4:$5:$6")
}
formatStr(_dete);
//"2018-04-08 00:00:00"
```

- 根据x的位置进行替换填充数据 

```javascript
let _dete='20180408000000'
function formatStr(str,type){
    let _type=type||"xxxx-xx-xx xx:xx:xx";
    for(let i = 0; i < str.length; i++){
        _type = _type.replace('x', str[i]);
    }
    return _type;
}
formatStr(_dete);
result:"2018-04-08 00:00:00"
```

- 改进

```javascript
let _dete='20180408000000'
function formatStr(str,type){
    let i = 0,_type = type||"xxxx-xx-xx xx:xx:xx";
    return _type .replace(/x/g, () => str[i++])
}
formatStr(_dete);
result:"2018-04-08 00:00:00"
```

# 克隆

- 浅克隆1

```javascript
function shallowClone(o) {
    const obj = {};
    for ( let i in o) {
        obj[i] = o[i];
    }
    return obj;
}
// 被克隆对象
const oldObj = {
    a: 1,
    b: [ 'e', 'f', 'g' ],
    c: { h: { i: 2 } }
};

const newObj = shallowClone(oldObj);
console.log(newObj.c.h, oldObj.c.h); // { i: 2 } { i: 2 }
console.log(oldObj.c.h === newObj.c.h); // true
```

- 浅克隆1

```javascript
//使用Object.assign()
 function shallowClone(o) {
     const obj = {};
     for ( let i in o) {
         obj[i] = o[i];
     }
     return obj;
 }
// 被克隆对象
const oldObj = {
    a: 1,
    b: [ 'e', 'f', 'g' ],
    c: { h: { i: 2 } }
};

const newObj = Object.assign({},oldObj)
console.log(newObj.c.h, oldObj.c.h); // { i: 2 } { i: 2 }
console.log(oldObj.c.h === newObj.c.h); // true
```

- 深克隆JSON.parse() 
  - 他无法实现对函数 、RegExp等特殊对象的克隆 
  - 会抛弃对象的constructor,所有的构造函数会指向Object 
  - 对象有循环引用,会报错 

```javascript
const oldObj = {
  a: 1,
  b: [ 'e', 'f', 'g' ],
  c: { h: { i: 2 } }
};

const newObj = JSON.parse(JSON.stringify(oldObj));
console.log(newObj.c.h, oldObj.c.h); // { i: 2 } { i: 2 }
console.log(oldObj.c.h === newObj.c.h); // false
newObj.c.h.i = 'change';
console.log(newObj.c.h, oldObj.c.h); // { i: 'change' } { i: 2 }
```

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

