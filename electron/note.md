### electron完成一个简易的桌面应用程序



###### 学习目标

- 什么是electron,它能做什么

- 使用electron展示一个桌面应用程序的界面(简单的窗口展示)

- electron中菜单的添加和配置

- electron中的主进程

- electron中的渲染进程

- electron中主进程和渲染进程之间的通信

- electron中系统托盘的简单实现

- electron应用程序的打包(windows环境下)


#### 1：大前端的发展现状

	现在前端的发展已经到了大前端的时代了。大前端时代是WEB统一的时代，利用html5或者6甚至7，不但可以开发传统的网站，做炫酷的网页动态效果，更可以实现BS架构应用程序、开发手机端web应用，移动端的app应用、移动端Native应用程序、智能设备（比如可穿戴智能手表，可穿戴智能衣服）等，同时使用node.js还可以实现后台开发。甚至还可以实现桌面应用程序的开发。下面我们要介绍的electron就是用来实现跨平台的桌面应用程序开发的。

#### 2：electron介绍

[Electron](https://electronjs.org/)是由Github开发，用HTML，CSS和JavaScript来构建跨平台桌面应用程序的一个开源库。 Electron通过将[Chromium](https://www.chromium.org/Home)和[Node.js](https://nodejs.org/)合并到同一个运行时环境中，并将其打包为Mac，Windows和Linux系统下的应用来实现这一目的。简单说，electron 是一个可以让我们使用js创建桌面应用程序的框架，并且可以很简单的实现跨平台，让我们可以更轻松的书写业务逻辑，而不用担心跨平台的问题

##### 2.1.它能做什么

Electron 可以让我们使用纯 JavaScript 调用丰富的原生(操作系统) API 来创造桌面应用。除了常规的 HTML5 接口，这些应用能够使用所有的 Node.js 模块，以及 Electron 特有的模块

##### 2.2.开发环境

你可以使用原生的Node.js开发环境来开发Electron应用。 为了打造一个Electron桌面程序的开发环境，你只需要安装好的Node.js、npm、一个顺手的代码编辑器,同时了解一些操作系统命令行客户端的常规操作

#### 3：使用electron完成一个简易计算器

##### 3.1.初体验-展示一个桌面应用程序界面

- electron的下载和引入

  ```javascript
  npm install electron --save-dev
  npm install electron -g
  -----------------
  const { app,BrowserWindow } = require('electron'）
  ```

- app的ready事件：main.js

  - electron会自动的触发ready事件

  - 在这个事件中可以创建electron窗口

    ```javascript
    app.on('ready', () => {
        hm_createWindow() //创建窗口的函数调用
    })
    ```

- electron应用程序窗口的创建(hm_createWindow函数)

  - 引入BrowserWindow

  - 通过BrowserWindow函数创建窗口对象

  - 对窗口进行配置

    ```javascript
    win = new BrowserWindow({
        width:390,
        height:670,
        //  设置标题
        title:'传智计算器'
    })
    ```

- electron窗口的常见操作

  - 加载其它指定的页面

    ```javascript
    win.loadURL(path.join(__dirname,'./views/index.html')) //使用了path模块生成路径
    ```

  - 开启调试工具

    ```javascript
    win.webContents.openDevTools()
    ```

  - 添加关闭窗口的事件

    ```javascript
    win.on('close',() => {
        // 实现关窗口之后的操作
        win = null
        // 关闭主窗口的时候同时也关闭整个应用程序
        app.quit()
    })
    ```

- electron应用程序的启动

  - electron main.js
  - electron .
  - 添加package.json的script配置："start": "electron main.js"，之后使用npm start启动electron

##### 3.2.添加应用程序的菜单项

- 添加菜单项的方法：Menu.buildFromTemplate(菜单项模板)

- 创建菜单项模板

  - label:菜单项文本设置

  - click:单击事件，指定事件处理函数

  - role:指定菜单项的角色

  - accelerator：触发当前菜单项的快捷键

  - 设置子菜单项：通过submenu设置

    ```javascript
    let template = [
        {
            label:'传智计算器',
            submenu:[
                {
                    label:'退出',
                    accelerator:'F11',
                    // role String (可选)-内置事件, 定义菜单项的行为, 当指定 click 属性时将被忽略
                    role:'quit',
                    click:() => {
                         app.quit()
                    }
                }
            ]
        }
    ]
    ```

- 添加菜单到应用程序：

  ```javascript
  Menu.setApplicationMenu(menu)
  ```

##### 3.3.主进程和渲染进程
  - 主进程：在Electron中运行package.json和main脚本的进程称为主进程。例如，main.js文件属于主进程，同时在main.js文件中直接引入的js文件也属于主进程(如：require('./mainProcess/menu.js'))
  - 渲染进程：electron使用Chromium的多进程机制来渲染页面，每个页面拥有一个自己的进程，该进程称为渲染进程。如main.js文件中加载了index.html文件，在index.html文件中引入的index.js就属于渲染进程
- 主进程和渲染进行之间通信的常见方式：(注意,这些方法全部需要在主进程已经创建了BrowserWindow之后才有效)
  - **ipcRenderer用于渲染进程**
    - ipcRenderer.on(channel, listener) --> channel为事件频道,字符串.listener为触发回调函数,用于响应webContent.send()
    - ipcRenderer.send(channel, data) --> 概念同上,用于向ipcMain发送异步信息
  - **ipcMain用于主进程,响应从渲染进程中发送的消息**
    - ipcMain.on(channel,listener) --> 响应从ipcRender中相同channel
  - **event.sender**：主进程监听到渲染进程发送的消息，在响应事件回调函数中使用event.sender.send()方法可以向渲染进程发送消息
  - **webContent**：ipcMain本身是无法直接发送事件的,只能通过响应事件回调的event来发送,那如果我们想先让主进程发送消息就可以使用webcontent.这个webcontent是在BrowserWIndow实例中的方法
    - webContent.send(channel,data) --> 主进程向渲染进程发送消息
- 不同渲染进程之间共享数据
  - 我们可以简单的使用HTML5中提供的API来实现，如localStorage和sessionStorage
  - 在electron中，我们可以在主进程中将一个对象存储为全局变量，在渲染进程中通过remote模块进行操作

##### 3.4.完成菜单项功能：颜色设置

- 添加设置颜色的静态页面：color.html

  ![01-setColor](E:\electron\文档资料\images\01-setColor.png)

- 添加设置颜色菜单项

  ```javascript
  {
      label:'颜色',
          click:() => {
              wh_createColorWindow()
          }
  }
  ```

- 添加单击事件-创建设置颜色窗口

  ```javascript
  function wh_createColorWindow(){
      let win = new BrowserWindow({
          width:250,
          height:100,
          title:'颜色'
      })
      // 加载关于页面
      win.loadURL(path.join(__dirname,"../views/color.html"))
      // 设置不再需要菜单块
      win.setMenu(null)
  }
  ```

- 业务实现：主进程中接收渲染进程传递的数据

  - 渲染进程中引入ipcRenderer

    ```javascript
    const {ipcRenderer} = require('electron')
    ```

  - 通过ipcRenderer发射事件让主进程进行监听处理

    ```javascript
    // 渲染进程向主进程发送消息
    ipcRenderer.send('wh_colortomain',currentColor)
    ```

- 业务实现：渲染进程接收主进程传递的数据

  - 在主进程中引入ipcMain:在主进程中使用，通过ipcMain可以接收渲染进程中发送过来的消息

  - 监听渲染进程中发射的事件：

    ```javascript
    // 主进程监听渲染进程发送过来的消息
    ipcMain.on('wh_colortomain',(event,color) => {
        // 设置样式：主进程向渲染发送消息：设置index.html页面中的dom元素的样式
        win.webContents.send('wh_setColor',color)
    })
    ```

  - 业务实现：通过单独模块处理颜色设置

    ```javascript
    // 渲染进程监听主进程发送过来的消息，进行处理
    ipcRenderer.on('wh_setColor',(event,color) => {
        // 设置对应元素的样式
        result_text.style.color = color
        // 将当前用户设置的颜色存储到本地存储中
        localStorage.setItem('userColor',color)
    })
    ```

##### 3.5.完成菜单项功能：字体大小设置

- 添加事件

  ```javascript
  {
      label:'字体减小',
          accelerator:'F12',
              click:(item,cwindow,event) => {
                  cwindow.webContents.send('sub')
              }
  }
  ```

- 完成事件处理

  ```javascript
  ipcRenderer.on('sub',() => {
      // 获取字体大小
      let fontSize = window.getComputedStyle(result_text,null).fontSize
      // 调整字体大小
      let fontNum = fontSize.replace('px','') - 0 - 3
      // 重置
      result_text.style.fontSize =  fontNum + "px"
      localStorage.setItem('userFontSize',fontNum)
  })
  ```

##### 3.6. 实现桌面端隐藏到托盘

- tray模块：通过tray可以实现系统托盘功能

- 我们的需求：在点关闭的时候把window隐藏起来，并且任务栏也一并需要隐藏，在点击系统通知区的图标时再把界面显示出来

- 引入：const { Tray,Menu } = require('electron')

- 代码：minProcess/tray.js

  ```javascript
  // 使用tray可以来实现通知栏图标
  const { Menu, Tray, dialog } = require('electron')
  const path = require('path')
  
  function createTray(win) {
      let appIcon = new Tray(path.join(__dirname, '../images/lar.jpg'))
  
      // 为图标构建菜单项
      const menu = Menu.buildFromTemplate([
          {
              label: '关闭',
              click: () => {
                  dialog.showMessageBox({
                      type: 'warning',
                      title: '退出提示',
                      message: '请问是否真的需要退出？',
                      buttons: ['确定', '取消']
                  }, (index) => {
                      if (index == 0) {
                          appIcon.destroy()
                          app.quit()
                      }
                  })
              }
          }
      ])
  
      appIcon.on('click', () => { //我们这里模拟桌面程序点击通知区图标实现打开关闭应用的功能
          win.isVisible() ? win.hide() : win.show()
          win.isVisible() ?win.setSkipTaskbar(false):win.setSkipTaskbar(true);
      })
  
      // 设置鼠标上移时的文本提示
      appIcon.setToolTip('黑马程序员')
      // 设置菜单项
      appIcon.setContextMenu(menu)
  }
  // 暴露成员
  module.exports = createTray
  ```


##### 3.7.完成简易的计算功能

- 为dom元素绑定事件

  ```html
  <div class="calculate">
      <div class="item" onClick="main.reset()">AC</div>
      <div class="item" onClick="main.clickopt('+/-')">+/-</div>
      <div class="item" onClick="main.clickopt('%')">%</div>
      <div class="item orange" onClick="main.clickopt('/',this)">÷</div>
      <div class="item" onClick="main.clickNum('7')">7</div>
      <div class="item" onClick="main.clickNum('8')">8</div>
      <div class="item" onClick="main.clickNum('9')">9</div>
      <div class="item orange" onClick="main.clickopt('*',this)">×</div>
      <div class="item" onClick="main.clickNum('4')">4</div>
      <div class="item" onClick="main.clickNum('5')">5</div>
      <div class="item" onClick="main.clickNum('6')">6</div>
      <div class="item orange" onClick="main.clickopt('-',this)">-</div>
      <div class="item" onClick="main.clickNum('1')">1</div>
      <div class="item" onClick="main.clickNum('2')">2</div>
      <div class="item" onClick="main.clickNum('3')">3</div>
      <div class="item orange" onClick="main.clickopt('+',this)">+</div>
      <div class="item large" onClick="main.clickNum('0')">0</div>
      <div class="item" onClick="main.clickNum('.')">.</div>
      <div class="item orange" onClick="main.calc()">=</div>
  </div>
  ```

- 单击数字

  ```javascript
  clickNum(num) {
      res = document.querySelector('.result-text'),
      isPoint = num === '.';
      // 数字转为字符串
      result = result.toString();
      // 如果输入小数点并且已经结果中已经有小数点了
      if (result.indexOf('.') !== -1 && isPoint) {
          return;
      }
      result = result + num;
      res.innerHTML = result;
  }
  ```

- 单击运算符

  ```javascript
  clickopt(ope) {
      res = document.querySelector('.result-text');
      switch (ope) {
          case '+/-':
              res.innerHTML = result = math.eval(result + '*-1');
              break;
          case '%':
              res.innerHTML = result = math.format(math.eval(res.innerHTML + '/100'),4);
              break;
          default:
              res.innerHTML = result = result + ope
              break;
      }
  }
  ```

- 重置

  ```javascript
  reset() {
      res = document.querySelector('.result-text');
      res.innerHTML = '0';
      result = ''
  }
  ```

- 计算结果

  ```javascript
  calc(){
      res = document.querySelector('.result-text');
      res.innerHTML = result =  math.eval(result)
  }
  ```


#### 4：应用程序打包

#####4.1 打包为可执行文件

- 安装 `electron-packager`：npm install --save-dev electron-packager
- 正式打包
  - 直接使用命令打包
    - electron-packager <应用目录> <应用名称> <打包平台> --out=<输出目录> <架构> <应用版本> <忽略文件><图标> --overwrite
    - 配置说明:--platform
      - win: 打包平台：--platform=win32  | 架构：--arch=x64  | 图标：--icon=**.ico
      - mac:打包平台：--platform=darwin| 架构：--arch=x64 | 图标：--icon=**.cins

  - 配置package.json命令实现打包

    ```javascript
     "packagewin": "electron-packager ./ heima --platform=win32 --out=./dist --arch=x64 --app-version=1.0.1 --icon=./images/icon.ico --overwrite",
    ```

##### 4.2 打包为安装包

- 下载：npm i electron-builder -g

- 说明：electron-builder可以将应用程序打包为安装文件，如.exe .dmg,对于windows系统打包为.exe,对于mac系统打包为.dmg

- 实现electron-builder的配置

  ```javascript
   "build": {
      "appId": "com.itcast-wu.app",
      "directories": {
        "app": "./"
      },
      "productName": "广州-计算器",
      "dmg": {
        "icon":"./images/mac.icns",
        "window": {
          "x": "200",
          "y": "150",
          "width": 500,
          "height": 400
        }
      }，
      "mac": {"icon":"./images/mac.icns"},
      "win": {"icon":"./images/icon.ico"}
    }
  ```

  添加script脚本配置：

  ```javascript
  "scripts": {
      "test": "echo \"Error: no test specified\" && exit 1",
      "packagewin": "electron-packager ./ heima --platform=win32 --out=./dist --arch=x64 --app-version=1.0.1 --icon=./images/icon.ico --overwrite",
      "buildmac": "electron-builder --platform=mac --arch=x64",
      "buildwin": "electron-builder --platform=win --arch=x64"
    }
  ```

  运行命令：

  ```javascript
  npm run buildwin
  ```

  注意：

  windows下面打包.exe

  mac下面打包 .dmg





