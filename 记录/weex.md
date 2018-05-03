# weex调试工具
- weexplayground
# 安装
1. npm i -g weex-toolkit
2. weex -v 检测
3. Weex init project
4. npm i
5. npm run serve
6. npm run dev

# 通用样式
- 适配和缩放
    - weex对于长度值目前只支持像素、适配以750px为标准
    - 有些属性不支持简写形式
    - border:1px solid #f00 需要分别设置
    - background:#f00 需要 background-color:#f00
- 定位
    -  position不支持static定位，支持relative,absolute,fixed,sticky,默认是relative
    -  不支持z-index,越靠后的元素层级越高
    -  android默认overflow:hidden
    -  线性渐变仅支持linear-gradient
    -  box-shadown仅支持IOS
    -  默认是flex布局
    -  image组件,border-radius仅对IOS有效,不能单独设置一个或几个的border-radius

# 组件
- <a>
    - a组件定义了指向weex页面打包后的一个js地址
    - 组件内无法直接添加文本,需要嵌套一个text进行文本添加
    - a组件基本可以嵌套任何组件为子组件
    - 支持所有的通用样式
    - 不能为a添加click事件