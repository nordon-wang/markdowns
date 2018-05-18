# 表单

- 通过三种当时获取表单的数据
- 包含表单的组件分类
  - 受控组件：表单项输入数据能够自动收集成状态，案例中的age字段
  - 非受控组件：需要时才手动读取表单输入框中的数据，案例中的username和password字段
- 大部分推荐使用`受控组件`，因为其更符合react的思想，不需要进行DOM的操作，而且react也不推荐过多的使用refs

```javascript
class Login extends React.Component {
    constructor(props){
        super(props)

        this.state = {
            age:18
        }

        this.handleSubmit = this.handleSubmit.bind(this)
        this.handleChange = this.handleChange.bind(this)
    }

    handleSubmit(event){
        event.preventDefault();
        // 通过旧的refs获取username
        const username = this.refs.username.value
        // 通过新的refs获取username
        const pwd = this.pwd.value
        // 通过状态获取age
        const {age} = this.state

        console.log(username,pwd,age);
    }

    handleChange(event){
        // 由于原生的onchange事件并不是真的在change时触发事件，而是在失去焦点的时候才会触发change事件
        // react在onChange事件做了优化，会在change的时候就触发事件
        const age = event.target.value
        this.setState({
            age
        })

    }

    render(){
        return (
            <form action="" method="get" onSubmit={this.handleSubmit}>
            <p>
            username: <input type="text" ref="username" />
            </p>
            <p>
            password: <input type="password" ref={input => this.pwd = input} />
            </p>
            <p>
            age: <input type="number" value={this.state.age} onChange={this.handleChange} />
            </p>
            <p>
            <input type="submit" value="login" />
            </p>
            </form>
        )
    }
}


ReactDOM.render(<Login/>,document.getElementById('app'))
```

# 组件生命周期

- 生命周期理解
  - 就是一个组件对象从创建到结束的一个过程，在这个过程组件对象会经历特定的阶段，每个特定的阶段都会对应一个相应的勾子函数
  - 勾子函数本质就是生命周期的回调函数，在组件的生命周期特定时刻进行回调
  - React.Component已经定义好了一系列的勾子函数，若是需要在特定的时间节点做一些事情，可以重写特定的勾子函数，在勾子中实现自己的逻辑功能
- 生命周期
  - 组件有三个生命周期状态
    - Mount：插入真实DOM，其对应的勾子函数为：componentWillMount()和componentDidMount()
    - Update：重新渲染，其对应的勾子函数为：componentWillUpdate()和componentDidMount()
    - Unmount：销毁真实DOM，其对应的勾子函数为：componentWillUnmount()
- 生命周期流程
  - 首次初始化渲染显示：ReactDOM.render()
    - constructor()：创建对象初始化state
    - componentWillMount()：将要插入回调
    - render()：插入虚拟DOM回调
    - componentDidMount()：插入完成回调
  - 每次更新state：this.setState()
    - componentWillUpdate()：将要更新回调
    - render()：更新、重新渲染
    - componentDidUpdate()：更新完成回调
  - 移除组件:ReactDOM.unmountComponentAtNode()
    - componentWillUnmount()：组件将要销毁时的回调
  - 常用勾子
    - render()：初始化渲染和更新都会调用
    - componentDidMount()：开启监听、ajax请求等
    - componentWillUnmount()：做一些收尾的工作，清除定时器等
- 案例
- 实现一个组件
  - 让文本实现显示/隐藏动画
  - 切换时间2s
  - 点击按钮 从界面中移除

```javascript
// 定义组件 
class Life extends React.Component{
    constructor(props){
        super(props)

        this.state = {
            opacity:1,
            color:`#f0f`
        }

        this.cancelTime = this.cancelTime.bind(this)
    }

    componentDidMount(){
        // 定时器作用域问题
        // 1. 通过bind解决
        // 2. 箭头函数
        this.timer = setInterval( function() {
            let {opacity} = this.state
            opacity -= 0.1

            //不能使用opacity === 0
            // 因为js的计算存在误差
            if(opacity <= 0){
                opacity = 1
            }
            this.setState({
                opacity
            })

        }.bind(this),200)

    }

    cancelTime(){
        // 移除组件
        ReactDOM.unmountComponentAtNode(document.getElementById('app'))

    }

    componentWillUnmount(){
        // 销毁组件之前的勾子
        // 定时器必须清除，不然会造成内存泄露的问题
        clearInterval(this.timer)

    }

    render(){
        const {msg} = this.props
        const {...style} = this.state

        return (
            <div>
            <h1 style={style} >{msg}</h1>
            <button onClick={this.cancelTime}>取消定时器</button>
            </div>
        )
    }
}

// 渲染组件
ReactDOM.render(<Life msg="生命周期演示"/>,document.getElementById('app'))
```

- 问题

  - 样式绑定，首先将样式存在state中，在render中将样式帮上

    ```javascript
    //方式2：绑定样式
    // 因为绑定样式是在js中使用，所以样式是使用对象的方式传入
    const {opacity,color} = this.state
    //这里需要使用双花括号，因为是在js中绑定样式，样式在一个对象中以键值对的形式存在
    <h1 style={{opacity,color}} >{msg}</h1>
    
    //方式2：绑定样式
    // 使用...将样式以对象的方式取出，可以直接绑定至样式上 
    const {...style} = this.state
    <h1 style={style} >{msg}</h1>
    ```

    