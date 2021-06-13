
##  一.使用Rollup搭建开发环境


### 1.什么是Rollup?

 *Rollup* 是一个 JavaScript 模块打包器,可以将小块代码编译成大块复杂的代码， rollup.js更专注于Javascript类库打包 （开发应用时使用Wwebpack，开发库时使用Rollup）

### 2.环境搭建

**安装rollup环境**

```bash
npm install @babel/preset-env @babel/core rollup rollup-plugin-babel rollup-plugin-serve cross-env -D
```

**rollup.config.js文件编写**

```js
import babel from 'rollup-plugin-babel';
import serve from 'rollup-plugin-serve';
export default {
    input: './src/index.js',
    output: {
        format: 'umd', // 模块化类型
        file: 'dist/umd/vue.js', 
        name: 'Vue', // 打包后的全局变量的名字
        sourcemap: true
    },
    plugins: [
        babel({
            exclude: 'node_modules/**'
        }),
        process.env.ENV === 'development'?serve({
            open: true,
            openPage: '/public/index.html',
            port: 3000,
            contentBase: ''
        }):null
    ]
}
```

**配置.babelrc文件**

```json
{
    "presets": [
        "@babel/preset-env"
    ]
} 
```

**执行脚本配置**

```json
"scripts": {
    "build:dev": "rollup -c",
    "serve": "cross-env ENV=development rollup -c -w"
}
```

## 二.Vue响应式原理

导出`vue`构造函数

```js
import {initMixin} from './init';

function Vue(options) {
    this._init(options);
}
initMixin(Vue); // 给原型上新增_init方法
export default Vue;
```

`init`方法中初始化`vue`状态

```js
import {initState} from './state';
export function initMixin(Vue){
    Vue.prototype._init = function (options) {
        const vm  = this;
        vm.$options = options
        // 初始化状态
        initState(vm)
    }
}
```

根据不同属性进行初始化操作

```js
export function initState(vm){
    const opts = vm.$options;
    if(opts.props){
        initProps(vm);
    }
    if(opts.methods){
        initMethod(vm);
    }
    if(opts.data){
        // 初始化data
        initData(vm);
    }
    if(opts.computed){
        initComputed(vm);
    }
    if(opts.watch){
        initWatch(vm);
    }
}
function initProps(){}
function initMethod(){}
function initData(){}
function initComputed(){}
function initWatch(){}
```

### 1.初始化数据

```js
import {observe} from './observer/index.js'
function initData(vm){
    let data = vm.$options.data;
    data = vm._data = typeof data === 'function' ? data.call(vm) : data;
    observe(data);
}
```

### 2.递归属性劫持

```js
class Observer { // 观测值
    constructor(value){
        this.walk(value);
    }
    walk(data){ // 让对象上的所有属性依次进行观测
        let keys = Object.keys(data);
        for(let i = 0; i < keys.length; i++){
            let key = keys[i];
            let value = data[key];
            defineReactive(data,key,value);
        }
    }
}
function defineReactive(data,key,value){
    observe(value);
    Object.defineProperty(data,key,{
        get(){
            return value
        },
        set(newValue){
            if(newValue == value) return;
            observe(newValue);
            value = newValue
        }
    })
}
export function observe(data) {
    if(typeof data !== 'object' && data != null){
        return;
    }
    return new Observer(data);
}
```

### 3.数组方法的劫持

```js
import {arrayMethods} from './array';
class Observer { // 观测值
    constructor(value){
        if(Array.isArray(value)){
            value.__proto__ = arrayMethods; // 重写数组原型方法
            this.observeArray(value);
        }else{
            this.walk(value);
        }
    }
    observeArray(value){
        for(let i = 0 ; i < value.length ;i ++){
            observe(value[i]);
        }
    }
}
```

**重写数组原型方法**

```js
let oldArrayProtoMethods = Array.prototype;
export let arrayMethods = Object.create(oldArrayProtoMethods);
let methods = [
    'push',
    'pop',
    'shift',
    'unshift',
    'reverse',
    'sort',
    'splice'
];
methods.forEach(method => {
    arrayMethods[method] = function (...args) {
        const result = oldArrayProtoMethods[method].apply(this, args);
        const ob = this.__ob__;
        let inserted;
        switch (method) {
            case 'push':
            case 'unshift':
                inserted = args;
                break;
            case 'splice':
                inserted = args.slice(2)
            default:
                break;
        }
        if (inserted) ob.observeArray(inserted); // 对新增的每一项进行观测
        return result
    }
})
```

**增加\_\_ob\_\_属性**

```js
class Observer { 
    constructor(value){
        Object.defineProperty(value,'__ob__',{
            enumerable:false,
            configurable:false,
            value:this
        });
        // ...
    }
 }
```

> 给所有响应式数据增加标识，并且可以在响应式上获取`Observer`实例上的方法

### 4.数据代理

```js
function proxy(vm,source,key){
    Object.defineProperty(vm,key,{
        get(){
            return vm[source][key];
        },
        set(newValue){
            vm[source][key] = newValue;
        }
    });
}
function initData(vm){
    let data = vm.$options.data;
    data = vm._data = typeof data === 'function' ? data.call(vm) : data;
    for(let key in data){ // 将_data上的属性全部代理给vm实例
        proxy(vm,'_data',key)
    }
    observe(data);
}
```

## 三.模板编译

```js
Vue.prototype._init = function (options) {
    const vm = this;
    vm.$options = options;
    // 初始化状态
    initState(vm);
    // 页面挂载
    if (vm.$options.el) {
    	vm.$mount(vm.$options.el);
    }
}
Vue.prototype.$mount = function (el) {
    const vm = this;
    const options = vm.$options;
    el = document.querySelector(el);

    // 如果没有render方法
    if (!options.render) {
        let template = options.template;
        // 如果没有模板但是有el
        if (!template && el) {
        	template = el.outerHTML;
        }
        const render= compileToFunctions(template);
        options.render = render;
    }
}
```

将`template`编译成`render函数`

### 1.解析标签和内容

```js
const ncname = `[a-zA-Z_][\\-\\.0-9_a-zA-Z]*`;  
const qnameCapture = `((?:${ncname}\\:)?${ncname})`;
const startTagOpen = new RegExp(`^<${qnameCapture}`); // 标签开头的正则 捕获的内容是标签名
const endTag = new RegExp(`^<\\/${qnameCapture}[^>]*>`); // 匹配标签结尾的 </div>
const attribute = /^\s*([^\s"'<>\/=]+)(?:\s*(=)\s*(?:"([^"]*)"+|'([^']*)'+|([^\s"'=<>`]+)))?/; // 匹配属性的
const startTagClose = /^\s*(\/?)>/; // 匹配标签结束的 >
const defaultTagRE = /\{\{((?:.|\r?\n)+?)\}\}/g
function start(tagName,attrs){
    console.log(tagName,attrs)
}
function end(tagName){
    console.log(tagName)
}
function chars(text){
    console.log(text);
}
function parseHTML(html){
    while(html){
        let textEnd = html.indexOf('<');
        if(textEnd == 0){
            const startTagMatch = parseStartTag();
            if(startTagMatch){
                start(startTagMatch.tagName,startTagMatch.attrs);
                continue;
            }
            const endTagMatch = html.match(endTag);
            if(endTagMatch){
                advance(endTagMatch[0].length);
                end(endTagMatch[1]);
                continue;
            }
        }
        let text;
        if(textEnd >= 0){
            text = html.substring(0,textEnd);
        }
        if(text){
            advance(text.length);
            chars(text);
        }
    }
    function advance(n){
        html = html.substring(n);
    }
    function parseStartTag(){
        const start = html.match(startTagOpen);
        if(start){
            const match = {
                tagName:start[1],
                attrs:[]
            }
            advance(start[0].length);
            let attr,end;
            while(!(end = html.match(startTagClose)) && (attr = html.match(attribute))){
                advance(attr[0].length);
                match.attrs.push({name:attr[1],value:attr[3]});
            }
            if(end){
                advance(end[0].length);
                return match
            }
        }
    }
}
export function compileToFunctions(template){
    parseHTML(template);
    return function(){}
}
```

### 2.生成ast语法树

语法树就是用对象描述`js`语法

```js
{
    tag:'div',
    type:1,
    children:[{tag:'span',type:1,attrs:[],parent:'div对象'}],
    attrs:[{name:'zf',age:10}],
    parent:null
}
```

```js
let root;
let currentParent;
let stack = [];
const ELEMENT_TYPE = 1;
const TEXT_TYPE = 3;

function createASTElement(tagName,attrs){
    return {
        tag:tagName,
        type:ELEMENT_TYPE,
        children:[],
        attrs,
        parent:null
    }
}
function start(tagName, attrs) {
    let element = createASTElement(tagName,attrs);
    if(!root){
        root = element;
    }
    currentParent = element;
    stack.push(element);
}
function end(tagName) {
    let element = stack.pop();
    currentParent = stack[stack.length-1];
    if(currentParent){
        element.parent = currentParent;
        currentParent.children.push(element);
    }
}
function chars(text) {
    text = text.replace(/\s/g,'');
    if(text){
        currentParent.children.push({
            type:TEXT_TYPE,
            text
        })
    }
}
```

### 3.生成代码

`template`转化成render函数的结果

```html
<div style="color:red">hello {{name}} <span></span></div>
render(){
   return _c('div',{style:{color:'red'}},_v('hello'+_s(name)),_c('span',undefined,''))
}
```

实现代码生成

```js
function gen(node) {
    if (node.type == 1) {
        return generate(node);
    } else {
        let text = node.text
        if(!defaultTagRE.test(text)){
            return `_v(${JSON.stringify(text)})`
        }
        let lastIndex = defaultTagRE.lastIndex = 0
        let tokens = [];
        let match,index;
        
        while (match = defaultTagRE.exec(text)) {
            index = match.index;
            if(index > lastIndex){
                tokens.push(JSON.stringify(text.slice(lastIndex,index)));
            }
            tokens.push(`_s(${match[1].trim()})`)
            lastIndex = index + match[0].length;
        }
        if(lastIndex < text.length){
            tokens.push(JSON.stringify(text.slice(lastIndex)))
        }
        return `_v(${tokens.join('+')})`;
    }
}
function getChildren(el) { // 生成儿子节点
    const children = el.children;
    if (children) {
        return `${children.map(c=>gen(c)).join(',')}`
    } else {
        return false;
    }
}
function genProps(attrs){ // 生成属性
    let str = '';
    for(let i = 0; i<attrs.length; i++){
        let attr = attrs[i];
        if(attr.name === 'style'){
            let obj = {}
            attr.value.split(';').forEach(item=>{
                let [key,value] = item.split(':');
                obj[key] = value;
            })
            attr.value = obj;
        }
        str += `${attr.name}:${JSON.stringify(attr.value)},`;
    }
    return `{${str.slice(0,-1)}}`;
}
function generate(el) {
    let children = getChildren(el);
    let code = `_c('${el.tag}',${
        el.attrs.length?`${genProps(el.attrs)}`:'undefined'
    }${
        children? `,${children}`:''
    })`;
    return code;
}
let code = generate(root);
```

### 4.生成`render`函数

```js
export function compileToFunctions(template) {
    parseHTML(template);
    let code = generate(root);
    let render = `with(this){return ${code}}`;
    let renderFn = new Function(render);
    return renderFn
}
```

## 四.创建渲染`watcher`

### 1.初始化渲染Watcher

```js
import {mountComponent} from './lifecycle'
Vue.prototype.$mount = function (el) {
    const vm = this;
    const options = vm.$options;
    el = document.querySelector(el);

    // 如果没有render方法
    if (!options.render) {
        let template = options.template;
        // 如果没有模板但是有el
        if (!template && el) {
            template = el.outerHTML;
        }

        const render= compileToFunctions(template);
        options.render = render;
    }
    mountComponent(vm,el);
}
```

**lifecycle.js**

```js
export function lifecycleMixin() {
    Vue.prototype._update = function (vnode) {}
}
export function mountComponent(vm, el) {
    vm.$el = el;
    let updateComponent = () => {
        // 将虚拟节点 渲染到页面上
        vm._update(vm._render());
    }
    new Watcher(vm, updateComponent, () => {}, true);
}
```

**render.js**

```js
export function renderMixin(Vue){
    Vue.prototype._render = function () {}
}
```

**watcher.js**

```js
let id = 0;
class Watcher {
    constructor(vm, exprOrFn, cb, options) {
        this.vm = vm;
        this.exprOrFn = exprOrFn;
        if (typeof exprOrFn == 'function') {
            this.getter = exprOrFn;
        }
        this.cb = cb;
        this.options = options;
        this.id = id++;
        this.get();
    }
    get() {
        this.getter();
    }
}

export default Watcher;
```

先调用`_render`方法生成虚拟`dom`,通过`_update`方法将虚拟`dom`创建成真实的`dom`

### 2.生成虚拟`dom`

```js
import {createTextNode,createElement} from './vdom/create-element'
export function renderMixin(Vue){
    Vue.prototype._v = function (text) { // 创建文本
        return createTextNode(text);
    }
    Vue.prototype._c = function () { // 创建元素
        return createElement(...arguments);
    }
    Vue.prototype._s = function (val) {
        return val == null? '' : (typeof val === 'object'?JSON.stringify(val):val);
    }
    Vue.prototype._render = function () {
        const vm = this;
        const {render} = vm.$options;
        let vnode = render.call(vm);
        return vnode;
    }
}
```

创建虚拟节点

```js
export function createTextNode(text) {
    return vnode(undefined,undefined,undefined,undefined,text)
}
export function createElement(tag,data={},...children){
    let key = data.key;
    if(key){
        delete data.key;
    }
    return vnode(tag,data,key,children);
}
function vnode(tag,data,key,children,text){
    return {
        tag,
        data,
        key,
        children,
        text
    }
}
```

### 3.生成真实`DOM`元素

将虚拟节点渲染成真实节点

```js
import {patch} './observer/patch'
export function lifecycleMixin(Vue){
    Vue.prototype._update = function (vnode) {
        const vm = this;
        vm.$el = patch(vm.$el,vnode);
    }
}
```

```js
export function patch(oldVnode,vnode){
    const isRealElement = oldVnode.nodeType;
    if(isRealElement){
        const oldElm = oldVnode;
        const parentElm = oldElm.parentNode;
        
        let el = createElm(vnode);
        parentElm.insertBefore(el,oldElm.nextSibling);
        parentElm.removeChild(oldVnode)
   		return el;
    } 
}
function createElm(vnode){
    let {tag,children,key,data,text} = vnode;
    if(typeof tag === 'string'){
        vnode.el = document.createElement(tag);
        updateProperties(vnode);
        children.forEach(child => { 
            return vnode.el.appendChild(createElm(child));
        });
    }else{
        vnode.el = document.createTextNode(text);
    }
    return vnode.el
}
function updateProperties(vnode){
    let newProps = vnode.data || {}; // 获取当前老节点中的属性 
    let el = vnode.el; // 当前的真实节点
    for(let key in newProps){
        if(key === 'style'){ 
            for(let styleName in newProps.style){
                el.style[styleName] = newProps.style[styleName]
            }
        }else if(key === 'class'){
            el.className= newProps.class
        }else{ // 给这个元素添加属性 值就是对应的值
            el.setAttribute(key,newProps[key]);
        }
    }
}
```

## 五.生命周期的合并

### 1.Mixin原理

```js
import {mergeOptions} from '../util/index.js'
export function initGlobalAPI(Vue){
    Vue.options = {};

    Vue.mixin = function (mixin) {
        // 将属性合并到Vue.options上
        this.options = mergeOptions(this.options,mixin);
        return this;
    }
}
```

### 2.合并生命周期

```js
export const LIFECYCLE_HOOKS = [
    'beforeCreate',
    'created',
    'beforeMount',
    'mounted',
    'beforeUpdate',
    'updated',
    'beforeDestroy',
    'destroyed',
]
const strats = {};
function mergeHook(parentVal, childValue) {
    if (childValue) {
        if (parentVal) {
            return parentVal.concat(childValue);
        } else {
            return [childValue]
        }
    } else {
        return parentVal;
    }
}
LIFECYCLE_HOOKS.forEach(hook => {
    strats[hook] = mergeHook
})
export function mergeOptions(parent, child) {
    const options = {}
    for (let key in parent) {
        mergeField(key)
    }
    for (let key in child) {
        if (!parent.hasOwnProperty(key)) {
            mergeField(key);
        }
    }
    function mergeField(key) {
        if (strats[key]) {
            options[key] = strats[key](parent[key], child[key]);
        } else {
            if (typeof parent[key] == 'object' && typeof child[key] == 'object') {
                options[key] = {
                    ...parent[key],
                    ...child[key]
                }
            }else{
                options[key] = child[key];
            }
        }
    }
    return options
}
```

### 4.调用生命周期

```js
export function callHook(vm, hook) {
    const handlers = vm.$options[hook];
    if (handlers) {
        for (let i = 0; i < handlers.length; i++) {
            handlers[i].call(vm);
        }
    }
}
```



### 5.初始化流程中调用生命周期

```js
Vue.prototype._init = function (options) {
    const vm = this;
    vm.$options = mergeOptions(vm.constructor.options,options);
    // 初始化状态
    callHook(vm,'beforeCreate');
    initState(vm);
    callHook(vm,'created');
    if (vm.$options.el) {
    	vm.$mount(vm.$options.el);
    }
}
```



## 六.依赖收集

每个属性都要有一个`dep`,每个`dep`中存放着`watcher`,同一个`watcher`会被多个`dep`所记录。

### 1.在渲染时存储watcher

```js
class Watcher{
    // ...
    get(){
        pushTarget(this);
        this.getter();
        popTarget();
    }
}
```

```js
let id = 0;
class Dep{
    constructor(){
        this.id = id++;
    }
}
let stack = [];
export function pushTarget(watcher){
    Dep.target = watcher;
    stack.push(watcher);
}
export function popTarget(){
    stack.pop();
    Dep.target = stack[stack.length-1];
}
export default Dep;
```

### 2.对象依赖收集

```js
let dep = new Dep();
Object.defineProperty(data, key, {
    get() {
        if(Dep.target){ // 如果取值时有watcher
            dep.depend(); // 让watcher保存dep，并且让dep 保存watcher
        }
        return value
    },
    set(newValue) {
        if (newValue == value) return;
        observe(newValue);
        value = newValue;
        dep.notify(); // 通知渲染watcher去更新
    }
});
```

**Dep实现**

```js
class Dep{
    constructor(){
        this.id = id++;
        this.subs = [];
    }
    depend(){
        if(Dep.target){
            Dep.target.addDep(this);// 让watcher,去存放dep
        }
    }
    notify(){
        this.subs.forEach(watcher=>watcher.update());
    }
    addSub(watcher){
        this.subs.push(watcher);
    }
}
```

**watcher**

```js
constructor(){
	this.deps = [];
	this.depsId = new Set();
}
addDep(dep){
    let id = dep.id;
    if(!this.depsId.has(id)){
        this.depsId.add(id);
        this.deps.push(dep);
        dep.addSub(this);
    }
}
update(){
    this.get();
}
```

### 3.数组的依赖收集

```js
this.dep = new Dep(); // 专门为数组设计的
if (Array.isArray(value)) {
	value.__proto__ = arrayMethods;
	this.observeArray(value);
} else {
	this.walk(value);
}	

function defineReactive(data, key, value) {
    let childOb = observe(value);
    let dep = new Dep();
    Object.defineProperty(data, key, {
        get() {
            if(Dep.target){
                dep.depend();
                if(childOb){ 
                    childOb.dep.depend(); // 收集数组依赖
                }
            }
            return value
        },
        set(newValue) {
            if (newValue == value) return;
            observe(newValue);
            value = newValue;
            dep.notify();
        }
    })
}


arrayMethods[method] = function (...args) {
    	// ...
        ob.dep.notify()
        return result;
}
```

**递归收集数组依赖**

```js
if(Dep.target){
    dep.depend();
    if(childOb){
        childOb.dep.depend(); // 收集数组依赖
        if(Array.isArray(value)){ // 如果内部还是数组
            dependArray(value);// 不停的进行依赖收集
        }
    }
}
function dependArray(value) {
    for (let i = 0; i < value.length; i++) {
        let current = value[i];
        current.__ob__ && current.__ob__.dep.depend();
        if (Array.isArray(current)) {
            dependArray(current)
        }
    }
}
```

## 七.实现Vue异步更新之nextTick

### 1.实现队列机制

```
update(){
    queueWatcher(this);
}
```

**scheduler**

```js
import {
    nextTick
} from '../util/next-tick'
let has = {};
let queue = [];

function flushSchedulerQueue() {
    for (let i = 0; i < queue.length; i++) {
        let watcher = queue[i];
        watcher.run()
    }
    queue = [];
    has = {}
}
export function queueWatcher(watcher) {
    const id = watcher.id;
    if (has[id] == null) {
        has[id] = true;
        queue.push(watcher);
        nextTick(flushSchedulerQueue)
    }
}
```

### 2.nextTick实现原理

**util/next-tick.js**

```js
let callbacks = [];
function flushCallbacks() {
    callbacks.forEach(cb => cb());
}
let timerFunc;
if (Promise) { // then方法是异步的
    timerFunc = () => {
        Promise.resolve().then(flushCallbacks)
    }
}else if (MutationObserver) { // MutationObserver 也是一个异步方法
    let observe = new MutationObserver(flushCallbacks); // H5的api
    let textNode = document.createTextNode(1);
    observe.observe(textNode, {
        characterData: true
    });
    timerFunc = () => {
        textNode.textContent = 2;
    }
}else if (setImmediate) {
    timerFunc = () => {
        setImmediate(flushCallbacks)
    }
}else{
    timerFunc = () => {
        setTimeout(flushCallbacks, 0);
    }
}
export function nextTick(cb) {
    callbacks.push(cb);
    timerFunc();
}
```

