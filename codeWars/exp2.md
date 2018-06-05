- 数组的偶数、0位置不变，奇数升序 

  ```javascript
  test(arr) {
      if(!arr.length) return [];
      // 先将数组奇数过滤 然后进行升序排序
      let newArr = arr.filter(item => item % 2 != 0).sort((num1,num2) => num1 - num2)      
      // 将偶数插入至 newArr
      arr.forEach((item,index) => {
          if(item % 2 == 0){
              newArr.splice(index,0,item)
          }
      })
      return newArr
  }
  ```

  

- 装换成2进制，然后将2进制每位进行累加

  ```javascript
  test(num){
      return num.toString(2).split('').reduce((sum,item) => sum += +item,0)
  }
  ```

- 将数字按规定返回

  ```javascript
  /**
  Test.assertEquals(expandedForm(12), '10 + 2');
  Test.assertEquals(expandedForm(42), '40 + 2');
  Test.assertEquals(expandedForm(70304), '70000 + 300 + 4');*/
  function expandedForm(num) {
      if(num < 10){
          return num + ''
      }
  
      let length = Math.pow(10,num.toString().split('').length - 1)
  
      if(num % length){
          return Math.floor(num / length) * length + ' + ' + expandedForm(num % length)
      }else{
          return Math.floor(num / length) * length + ''
      }
  }
  ```



































```javascript
view_module
```





