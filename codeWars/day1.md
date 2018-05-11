- 反向重置输入框值

```javascript
// 反向重置输入框值
let num = 123;
console.log(+num.toString().split('').reverse().join(''))
```

- 截取字符串
  - 若是偶数则截取中间两个字符串
  - 若是奇数则截取中间一个字符串 

```javascript
function getMiddle(n)
{
  //Code goes here!
  return Boolean(n.length % 2) ? n.substr(n.length / 2,1) : n.substr(n.length / 2 - 1,2)
}
```

- 过滤数组中的数字

```javascript
function filter_list(arr) {
    return arr.filter( item => {
      return typeof item == 'number'
    })
}
```

- 数字的每一位进行平方

```javascript
function squareDigits(num){
  //may the code be with you

  let newNum = num + ''

  let result = newNum.split('').reduce( (sum,item) => {
    return sum + Math.pow(item,2)
  },'')
  
  return +result
}
```

