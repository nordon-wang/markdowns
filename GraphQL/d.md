## 1.1 接口开发方式

- restful
- GraphQL

## 1.2 restful 接口问题

- 接口粒度比较细、很多场景需要调用多次才能完成一个功能
- 接口扩展、维护成本高
- 接口响应的数据格式无法预知、目前基本都是使用json

## 1.3 GraphQL概述

````
GraphQL 是一种用于 API 的查询语言、也是一个满足你数据查询的运行时
是一种接口开发标准、支持常见的服务器开发语言
````

## 1.4 GrapgQL 优势

- 精确获取需要的信息
- 通过单个请求获取各种资源
- 通过类型系统描述查询
- 强大的调试工具



# 二. 上手玩玩

## 2.1 服务端

- 创建服务端项目目录

- 初始化项目 npm init -y

- 创建入口文件 index.js

- 安装依赖包

  - Apollo-server-express

  - express

  - graphql

    ```
    npm install apollo-server-express body-parser express graphql graphql-tools -S
    ```

- 复制官方案例代码到index.js

  - [地址](<https://github.com/apollographql/apollo-server/tree/master/packages/apollo-server-express>)

- 启动

## 2.2 客户端

- 打开 http://localhost:4000/graphql 进行接口测试

```
{
  hello
}
```



# 三.基于apolloServer开发

> apolloserver 是一个实现了 GraphQL 规范的框架、可以基于apolloserver 快速开发基于 GraphQL 的服务端接口、并且方便客户端进行接口调用


## 3.1 开发

- 基本开发步骤-服务端
  - 初始化服务端项目
    - 创建项目目录
    - 创建入口文件
    - 初始化项目
  - 安装依赖
  - 定义类型
  - 解析数据
  - 实例化apollo对象
  - 整合express并监听端口、提供web服务

## 3.2 对象类型定义规则

### 3.2.1 对象类型定义与字段

- GraphQL提供了一套完善的类型系统、可以约束服务端服务端可以提供哪些数据类型供客户端查询使用
- 通过 type 关键字定义类型、type 之后是类型名称(自定义名称)

```
# 课程类型
type Course {
  cname: String
  score: Float
}

# 学生类型
type Student{
  name: String
  scores:[Course]
}
```

- 注意事项
  - 花括号中是对象的字段信息
  - 属性名称是自定义的
  - 属性名后面的类型为标量类型(内置类型)
  - GraphQL 使用 # 注释

### 3.2.2 基本参数

- 对象类型上的每一个字段都可能有零个或者多个参数

```
type Query {
  hello: String
  # 定义字段参数 可以有默认值
  stu(n: Int = 12): Student
}


# 获取参数(通过 resolver 函数的第二个参数获取客户端传递的参数数据)
const resolvers = {
  Query: {
    stu: (obj, args) => {
      console.log(args)
    }
  }
}

# 客户端字段传递
{
  stu(n: 13){
    name
    age
  }
}
```

- 注意事项
  - 参数可以指定默认值、如果有默认值则参数是可选的
  - 客户端可以通过 resolvers 函数的第二个参数获取传递的数据

### 3.3.3 内置类型

- 查询和变更类型

```
内置类型中有两个特殊的类型：Query(主要用于查询操作) 和 Mutation(主要用于变更操作 -- 增、删、改 )

每一个GraphQL 服务有一个 Query 类型、也可能有一个 mutation 类型、这两个类型本质上也是对象类型、唯一的区别就是他们作为客户端访问的入口
```

- 标量类型 — 用于表示基本的字段数据、表示查询数据的叶子节点
  - Int
  - Float
  - String
  - Booleab
  - ID 唯一标识符，随机生成一个字符串、不需要能认识
- 枚举类型 — 是一种特殊的标量、枚举类型限制在一个特殊的可选值集合内

```
enmu Fovour {
	SWIMING
	GODING
	SINGING
}
```

> 上述定义表示只能获取三种值之一、获取其他类型的值是不可以的

- 列表和非空
  - [] 表示列表
  - ！叹号表示非空

```
type Student{
	name: String!
	scores:[Score]!
}
```

> myField:[String!] 表示数组本身可以为空、但是其不能有任何空值成员
>
> myField:[String]! 表示数组本身不可以为空、但是其可以包含空值成员

### 3.3.4 输入类型

> 参数也可以是复杂类型、主要用于变更 mutation 场景 — 需要客户端传递输入类型

```
  # 输入类型
  input UserInfo{
    name: String
    pwd: String
  }

  #用户类型
  type User {
    id: ID
    name: String
    pwd: String
  }
```

> 定义 mutation、可以用于接收客户端传递的input类型数据

```
# 变更类型
type Mutation{
	addUserByInput(userInput: UserInfo): User
}
```

```
# 处理客户端传递的input参数
const resolvers = {
  Query: {
    hello: () => 'Hello world!'
  },
  Mutation: {
    addUserByInput: (obj, args) => {
      return {
        id: uuid(),
        name: args.userInput.name,
        pwd: args.userInput.pwd
      }
    }
  }
};
```

```
# 客户端查询操作
mutation {
	addUserByInput(userInput: {
		name: 'nordon',
		pwd: '123123'
	}){
		id
		name
	}
}
```

- 注意事项
  - input 类型主要用于变更操作的数据传递

## 3.3 数据解析规则详解

###3.3.1 resolver 函数参数用法

> resolvers 用于给类型字段提供实际数据

- resolver 函数参数
  - `parent` 上一级对象、如果字段属于根节点查询类型通常不会被使用
  - `args` 可以提供在 GrapgQL 查询中传入的参数
  - `context` 会被提供给所有解析器、并且持有重要的上下文信息。比如当前登入的用户、数据库访问对象
  - `info` 一个保存与当前查询相关的字段特定信息以及 schema 详细信息的值

```
  Mutation: {
    addUserByInput: (parent, args, context, info) => {
      // parent 当前字段的父级对象 -- Query 中的 hello 相当于根节点，例如查学生中的 name、age 就是子节点
      // args 客户端传递过来的参数
      // context 可以用于操作数据源
      // info 不怎么用
      return {
        id: uuid(),
        name: args.userInput.name,
        pwd: args.userInput.pwd
      }
    }
  }
```

- Parent 参数用法

```
// 类型定义
type Student {
	name: String
	age: Int
	favour: String
}

// 数据解析
const resolvers = {
  Student: {
    // 提供字段的 resolver 函数、如果不提供会默认生成
    name: (parent) => {
      // return 'wangyao' 会覆盖父节点的 name
      return parent.name
    },
    favour: (parent) => {
      if(parent.favour === 'swimming'){
        return '洗澡'
      }else{
        return parent.favour
      }
    }
  },
  Query: {
    hello: () => 'Hello world!',
    stu: (parent) => {
      return {
        name: 'nordon',
        age: 18,
        favour: 'swimming'
      }
    }
  },
};
```

- 注意事项
  - 通过 resolver 函数的第一个参数 parent 可以获取父级对象
  - 如果字段类型是标量类型、会有一个默认的 resolver 函数
  - 默认的 resolver 函数、返回父级对象的字段数据

### 3.3.2 resolver 函数对接数据源

> 通过 context 函数更加方便的对接数据源(数据库、文件、第三方接口)、包括异步操作

- context 基本用法

```
// context 可以用于获取数据源对象
const context = () => {
	return {
		db: 'dbObj'
	}
}

const server = new ApolloServer({
	typeDefs,
	resolvers,
	context
})
```

```
// 在 resolver 函数中可以通过 context 对象操作数据源
const resolvers = {
	Query: {
		hello: (parent, args, context) => {
			// 通过 context 可以获取数据源操作对象
			let ret = context.db
		}
	}
}
```

- 读取文件数据业务模块

```
// db.js 代码：从文件中读取数据、本质上与从数据库中读取数据类型、多可以异步操作
const path = require('path')
const fs = require('fs')

module.exports.getData = () => {
	const dataPath = path.join(__dirname, 'data.json')
	
	return new Promise((resolve, reject) => {
		fs.readFile(dataPage, (err, data) => {
			if(err){
				reject(err)
			}
			
			resolve(data.toString())
		})
	})
}

// data.json 提供数据源
[{
  "name": "nordon",
  "age": 18
},{
  "name": "wangyao",
  "age": 22
},{
  "name": "bing",
  "age": 23
}]
```

- 通过 context 获取数据源操作的对象

```
// 从文件中读取数据作为数据源、 db.getData() 返回 Promise 实例对象
const db = require('./db.js')

const typeDefs = gql`
  # 学生
  type Student {
    name: String
    age: Int
  }

  # 查询类型
  type Query {
    hello: [Student!]!
  }
`;

const context = ({req}) => {
	return {
		db: db
	}
}

const resolvers = {
	Query: {
		hello: async (parent, args, context, info) => {
			const ret = await context.db.getData()
			return JSON.parse(ret)
		}
	}
}


const server = new ApolloServer({
	typeDefs,
	resolvers,
	context
})
```

- 注意事项
  - context 参数主要用于提供数据源相关的对象、方便进行数据操作(数据获取与变更)
- 客户端开发传送门 [GraphQL 客户端开发](https://juejin.im/post/5cf156fee51d455d850d3a90)

# 参考网站

[grapgQL官网](<http://graphql.cn/>)

[apolloserver官网](<https://www.apollographql.com/docs/apollo-server/>)


