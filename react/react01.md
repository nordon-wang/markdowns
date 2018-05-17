# 基础认识

- [官网](https://doc.react-china.org/)
- 特点
  - 声明式编程
  - 组件化
  - 支持客户端和服务端渲染
  - 高效
    - 虚拟DOM，不总是直接操作DOM，只是减少DOM的操作,操作虚拟DOM不会对页面进行重绘，当渲染的时候，才会进行渲染
    - DOM Diff 算法，最小化页面重绘，就是当页面变化时，通过计算那部分需要重绘，只重绘当前部分，减少页面的重绘区域
  - 单向数据流
- js文件
  - react.js：react的核心库
  - react-dom.js：提供操作DOM的react扩展
  - babel.js：解析jsx语法

```javascript
	<div id="app"></div>
    <script src="js/react.development.js"></script>
    <script src="js/react-dom.development.js"></script>
    <script src="js/babel.min.js"></script>
    <script type="text/babel">
        // 创建虚拟DOM元素对象
        let vDom = <h1>react hello</h1>
        // 将虚拟DOM渲染到页面真实DOM容器中
        ReactDOM.render(vDom,document.getElementById('app'))
        
    </script>
```

# JSX

- react提供了创建虚拟DOM的方法
- 虚拟DOM对象最终都会被react转换为真实的DOM
- 编码时，只需要操作react的虚拟DOM相关数据，react会转换为真实DOM变化而更新界面

```javascript
// react直接操作和JSX操作对比
<script type="text/javascript">
    const msg = 'MSG content'
const msgId = 'App'

// 创建虚拟DOM
// const VDom1 = React.createElement('标签名',{id:'xxx'},'内容')
const vDom1 = React.createElement('h1', {
    id: msgId.toLowerCase()
}, msg.toLowerCase())

// 渲染虚拟DOM
ReactDOM.render(vDom1, document.getElementById('app1'))
</script>

<script type="text/babel">
    // 创建虚拟DOM 
    const vDom2 = <h2 id={msgId.toUpperCase()}>{msg.toLowerCase()}</h2>;

// 渲染虚拟DOM 
ReactDOM.render(vDom2, document.getElementById('app2'))
</script>
```

- 动态渲染一个列表
  - 使用数组的Map函数返回所需要的列表内容

```javascript
let arrs = [111,222,333,444]
// 创建虚拟DOM 
const ul = (
    <ul>
    {
        arrs.map((name,index) => <li key={index}>{name}</li>)
    }
    </ul>
)

// 渲染虚拟DOM 
ReactDOM.render(ul, document.getElementById('app1'))
```

# 组件

- 工厂函数创建组件
- 使用工厂函数的效率比使用class高，因为工厂函数中不需要创建一系列的对象之类的
- 当组件有状态(state)的时候就不适合使用了

```javascript
// 工厂函数创建组件
function MyCom(){
    return <h2>工厂函数创建组件</h2>			
}
    	
// 渲染组件标签
// <MyCom />必须这么写
// <MyCom>这么写是错误的
ReactDOM.render(<MyCom />,document.getElementById('app'))
```

- class创建组件

```javascript
// class定义组件
class MyCom2 extends React.Component{
    render(){
        return <h3>class定义组件</h3>
    }
}

// 渲染组件标签
// <MyCom2 />必须这么写
// <MyCom2>这么写是错误的
ReactDOM.render(<MyCom2 />,document.getElementById('app'))
```

# 组件三大属性

## state

- state是组件对象的最重要的属性之一，值是一个对象，这个对象可以包含多个数据
- 组件被称为`状态机`，通过更新组件的state来更新对应的页面显示，就是通过state来控制组件的重新渲染(重绘)



- 初始化状态

```javascript
constructor(props){
    super(props)
    this.state = {
        stateProp1:value1,
        stateProp2:value2
    }
}
```



- 读取某个状态值

```javascript
this.state.statePropertyName
```



- 更新状态:组件重新渲染

```javascript
this.setState({
    stateProp1:newValue1,
    stateProp2:newValue2
})
```



- 案例：点击切换内容

```javascript
//定义组件
class Like extends React.Component{
    constructor(props){
        super(props)
        //初始化状态state
        this.state = {
            isShow:false
        }
        
        //将新增方法中的this强制绑定为组件对象
        this.handleClick = this.handleClick.bind(this)
    }
    
    // 新添加的方法:内置的this默认不是组件对象，是一个undefined
    handleClick(){
        //获取原始状态并取反
        let isShow = !this.state.isShow
        // 设置状态
        this.setState({isShow})
    }
    
    // React.Component中本身就具有render函数，在class中只是重写组件类的方法render，所以这里的this指向没有问题
    render(){
        //获取状态
        // const isShow = this.state.isShow
        const {isShow} = this.state
        //onClick是react的，区分原生的onclick
        // this.handleClick.bind(this)也可以，但是这种效率比较低，因为每次render都会执行一次bind进行绑定，而在constructor中只会在初始化的时候绑定一次
        return <h1 onClick={this.handleClick}>{isShow ? '我是帅哥' : '帅哥是我'}</h1>
    }
    
}

//渲染组件
ReactDOM.render(<Like/>,document.getElementById('app'))


```



## props

- 案例：定义一个显示个人信息的组件
  - 姓名必须指定
  - 性别默认为男
  - 年龄默认为18

```javascript
//创建组件
/* function Person(props) {
	return (
    	<ul>
        	<li>姓名:{props.name}</li>
            <li>性别:{props.sex}</li>
            <li>年龄:{props.age}</li>
        </ul>
    )
} */

// class创建
class Person extends React.Component{
    render(){
        return (
            <ul>
            <li>姓名:{this.props.name}</li>
            <li>性别:{this.props.sex}</li>
            <li>年龄:{this.props.age}</li>
            </ul>
        )
    }
}
		
//属性默认值
Person.defaultProps = {
    sex:'man',
    age:18
}

//属性的类型和必填
Person.propTypes = {
    name:PropTypes.string.isRequired,
    age:PropTypes.number
}

const p1 = {
    name:'nordon',
    sex:'man',
    age:22
}
// 渲染组件
// ReactDOM.render(<Person name={p1.name} sex={p1.sex} age={p1.age} />,document.getElementById('app'))
ReactDOM.render(<Person {...p1} />,document.getElementById('app'))

const p2 = {
	name:'lisan'
}	
ReactDOM.render(<Person name={p2.name}  age={19}/>,document.getElementById('app2'))
```



## refs与事件处理

- 案例：获取input中的值

```javascript
//定义组件
class MyInput extends React.Component {
			
    constructor(props){
        super(props)
        this.handleClick = this.handleClick.bind(this)
        this.handleBlur = this.handleBlur.bind(this)
    }

    handleClick(){
        let input1 = this.refs.inputValue
        console.log(input1.value)
        console.log(this.newInput.value)
    }

    handleBlur(event){
        console.log(event.target.value)
    }

    render (){
        return (
            <div>
                <input type="text" ref="inputValue" />&nbsp;&nbsp;
                <input type="text" ref={inputValue => this.newInput = inputValue } />&nbsp;&nbsp;
                <button onClick={this.handleClick}>获取值</button>&nbsp;&nbsp;
                <input type="text" placeholder="失去焦点获取值" onBlur={this.handleBlur} />
            </div>
        )
    }
}
		
//渲染组件
ReactDOM.render(<MyInput/>,document.getElementById('app'))
```



# ToDolist

- 简单的tdolist案例



1. 拆分组件
2. 实现静态组件
   1. 仅仅只是静态页面，没有动态数据和交互
3. 实现动态组件
   1. 实现初始化数据的动态显示
   2. 实现交互功能

```javascript
// 定义组件
// 组件状态的更改必须在当前组件中进行
class App extends React.Component{ 

    constructor(props){
        super(props)

        this.state = {
            todos:['111','222','333']
        }

        this.addTodo = this.addTodo.bind(this)
    }

    addTodo(todo){
        let {todos} = this.state
        todos.unshift(todo)
        this.setState({
            todos
        })
    }

    render(){ 
        const {todos} = this.state
        return (
            <div>
            <h1>this is todolist demo</h1>
            <Add count={todos.length} addTodo={this.addTodo} />
            <List todos={todos} />
            </div>
        ) 
    }
}

class Add extends React.Component{ 
    constructor(props){
        super(props)

        this.handleAdd = this.handleAdd.bind(this)
    }
    handleAdd(){
        // console.log(this.refs.content.value);
        // <input  type="text" ref="content" /> &nbsp;&nbsp;
        // console.log(this.newTodos.value);
        const data = this.newTodos.value
        if(!data){
            return false;
        }
        this.newTodos.value = ''
        this.props.addTodo(data)
    }
    render(){
        return (
            <div>
            <input  type="text" ref={inputValue => this.newTodos = inputValue} /> &nbsp;&nbsp;
            <button onClick={this.handleAdd}>按钮#{this.props.count + 1}</button>
            </div>
        )
    }
}
Add.propTypes = {
    count:PropTypes.number.isRequired,
    addTodo:PropTypes.func.isRequired
}

class List extends React.Component{

    render(){
        let {todos} = this.props
        return (
            <div>
            <ul>
            {todos.map( (item,index) => <li key={index}>{item}</li> )}
            </ul>
            </div>
        )
    }
}
// name:PropTypes.string.isRequired,
List.propTypes = {
    todos:PropTypes.array.isRequired
}

// 渲染组件
ReactDOM.render(<App/>,document.getElementById('app'))
```

