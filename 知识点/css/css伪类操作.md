- HTML

  ```html
  <p class="red">Hi, this is a plain-old, sad-looking paragraph tag.</p>
  ```

  

- 默认的css样式

  ```css
  .red::before { content: 'red'; color: red; }
  ```

- 方式一

  - 通过类名控制

    ```css
    /* 预先定义好 */
    .red::before { content: 'red'; color: red; }
    ```

    ```javascript
    $('p').removeClass('red').addClass('green');
    ```

    

- 方式二

```javascript
// 动态控制style的样式
document.styleSheets[0].addRule('.red::before','color: green'); document.styleSheets[0].insertRule('.red::before { color: green }', 0);
```

- 方式三

  - 创建一个style标签插入head中

    ```javascript
    // Create a new style tag var 
    style = document.createElement("style"); 
    // Append the style tag to head 
    document.head.appendChild(style); 
    // Grab the stylesheet object 
    sheet = style.sheet 
    // Use addRule or insertRule to inject styles 
    sheet.addRule('.red::before','color: green'); sheet.insertRule('.red::before { color: green }', 0);
    ```

    

  - 动态增加样式进去

    ```javascript
    $('<style>.red::before{color:green}</style>').appendTo('head');
    ```

- 方式四

  - html使用自定义属性data定义，css使用attr进行调用

    ```html
    <p class="red" data-attr="red">Hi, this is plain-old, sad-looking paragraph tag.</p>
    ```

    ```css
    .red::before { content: attr(data-attr); color: red; } 
    ```

    ```javascript
    $('.red').attr('data-attr', 'green');
    ```

    