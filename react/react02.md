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

