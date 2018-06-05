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
  let newNum = num + ''

  let result = newNum.split('').reduce( (sum,item) => {
    return sum + Math.pow(item,2)
  },'')
  
  return +result
}
```

- 数字的每一位进行相加

```javascript
/* 
    digital_root(942)
    => 9 + 4 + 2
    => 15 ...
    => 1 + 5
    => 6 */
function digital_root(n) {
  if(n/10 <1){
      // 个位数
      return n;
    }else{
      let tem = n.toString().split('').reduce( (sum,item) => sum + +item,0)
      return digital_root(tem)
    }
}

// 一句话
function digital_root(n) {
	return n/10 <1 ?  n :  digital_root(n.toString().split('').reduce( (sum,item) => sum + +item,0))
}
```

- 将数组中唯一的奇数或者偶数筛选出来 

```javascript
function findOutlier(arr){
  let result = [{
      count:0,
      num:0
    },{
      count:0,
      num:0
    }]

    arr.forEach(item => {
      if(item % 2){
        result[0].count++;
        result[0].num = item;
      }else{
        result[1].count++;
        result[1].num = item;
      }
    })

    let r = result.filter( item => item.count == 1)
    
    return r[0].num
}
```

