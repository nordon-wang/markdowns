# JS使用

- 使用bower下载

  ```javascript
  bower i codemirror
  ```

- 引入样式文件

  ```html
  <link rel="stylesheet" type="text/css" href="bower_components/codemirror/lib/codemirror.css">
  ```

- 引入js文件

  ```html
  <script type="text/javascript" src="bower_components/codemirror/lib/codemirror.js"></script>
  <script type="text/javascript" src="bower_components/codemirror/mode/javascript/javascript.js"></script>
  <script type="text/javascript" src="bower_components/codemirror/mode/xml/xml.js"></script>
  <script type="text/javascript" src="bower_components/codemirror/mode/htmlmixed/htmlmixed.js"></script>
  <script type="text/javascript" src="bower_components/codemirror/mode/css/css.js"></script>
  ```

  

- 文档结构

  ```html
  <div class="container">
      <textarea class="marpad " tyle="width:70%;height:300px;" id="editor" >
          <!-- 放入编辑对应的文本 -->
      </textarea>
  </div> 
  ```

  

- 初始化

  ```javascript
  // mode:  "text/javascript",
  // mode: "text/css"
  window.onload = function(){
      var myCodeMirror = CodeMirror.fromTextArea(document.getElementById("editor"), {
          lineNumbers: true,
          mode: "text/html"
      });
  }
  ```

  - mode需要注意的是,编辑器解析某种文本,必须引入mode下面对应的js文件,否则还是以纯文本的格式显示或者default
  - 若是mode: "text/html",则必须在htmlmixed.js文件引入之上引入xml.js，否则html是不能被正确解析显示的



# vue-cli中使用

- 下载

  ```javascript
  npm i -S vue-codemirror
  ```

- main.js中使用

  ```javascript
  // codemirror 引入和使用
  import VueCodemirror from 'vue-codemirror'
  import 'codemirror/lib/codemirror.css'
  Vue.use(VueCodemirror)
  ```

- .vue文件中使用

  - template部分，使用双向绑定的写法

    ```html
    //v-model="infor.codeCss" 双向绑定的数据源
    //:options="cssOptions" codemirror的配置项
    <codemirror v-model="infor.codeCss" :options="cssOptions"></codemirror>
    ```

  - script部分

    ```javascript
    //引入xml，html，css，js对应的js解析文件
    import 'codemirror/mode/javascript/javascript.js'
    import 'codemirror/mode/css/css.js'
    import 'codemirror/mode/xml/xml.js'
    import 'codemirror/mode/htmlmixed/htmlmixed.js'
    
    // 引入主题样式文件
    import 'codemirror/theme/monokai.css'
    ```

    ```javascript
    //Vue实例中设置配置项
    data(){
        return {
            infor：{
                codeCss：''
            }，
            cssOptions: {
                tabSize: 4,
                mode: 'text/css',
                theme: 'monokai',
                lineNumbers: true,
                line: true,
              }
        }
    }
    ```

    