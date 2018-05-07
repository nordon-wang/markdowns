# 基础语法



## 基础查询

1. 基础的查询

```json

// 基础查询
{
    hero{
        name
    }
}
// 查询结果
{
    "data":{
        "hero":{
            "name":"nordon"
        }
    }
}

```

- 查询和其查询结果几乎是一样的结果

2. 查询是可交互的，可根据自己的需求和喜好修改查询

   1. 在新的查询中的hero增加一个appearsIn字段

   ```json
   // 查询语句
   {
       hero{
       	name
       	# 增加新的修去 增加一个friends字段
       	# friends 返回的是一个数组
           friends{
               name
           }
   	}
   }
   
   // 查询结果
   {
     "data": {
       "hero": {
         "name":"nordon",
         "friends": [
           {
             "name": "Luke Skywalker"
           },
           {
             "name": "Han Solo"
           },
           {
             "name": "Leia Organa"
           }
         ]
       }
     }
   }
   ```

   

   ## 传参

   1. 查询可以传递参数

   ```json
   // 查询语句
   {
       human(id:"1000"){
           name
           height
       }
   }
   
   //查询结果
   {
       "data":{
           "human":{
               "name":"nordon",
               "height":1.83
           }
       }
   }
   ```

   2. GrapgQL中每一个字段和嵌套对象都能有自己的一组参数

   ```json
   // 多个参数的查询语句
   {
       human(id:'1'){
           name
           height(unit:FOOT)
       }
   }
   
   //查询结果
   {
     "data": {
       "human": {
         "name": "Luke Skywalker",
         "height": 5.6430448
       }
     }
   }
   ```

   - 参数可以有多种不同的类型

## 别名

1. 在查询名称相同时会存在冲突，通过别名来进行区分查询

```json
// 使用别名的查询
{
    hero1:hero(id:'1'){
        name
    }
    hero2:hero(id:'2'){
        name
    }
}

// 结果
{
  "data": {
    "hero1": {
      "name": "Luke Skywalker"
    },
    "hero2": {
      "name": "R2-D2"
    }
  }
}
```

## 片段

1. 将重复的代码组装成一个可复用的片段
2. 片段就是一个可复用单元
3. 一次编写，到处可用

```json
// 片段
fragment myFra on Character{
    name
    appearsIn
    friends{
        name
    }
}

// 结合片段进行查询
{
    hero1:hero(id:'1'){
        ...myFra
    }
    hero2:hero(id:'2'){
		...myFra
    }
}

// 结果
{
  "data": {
    "hero1": {
      "name": "Luke Skywalker",
      "appearsIn": [
        "NEWHOPE",
        "EMPIRE",
        "JEDI"
      ],
      "friends": [
        {
          "name": "Han Solo"
        },
        {
          "name": "Leia Organa"
        },
        {
          "name": "C-3PO"
        },
        {
          "name": "R2-D2"
        }
      ]
    },
    "hero2": {
      "name": "R2-D2",
      "appearsIn": [
        "NEWHOPE",
        "EMPIRE",
        "JEDI"
      ],
      "friends": [
        {
          "name": "Luke Skywalker"
        },
        {
          "name": "Han Solo"
        },
        {
          "name": "Leia Organa"
        }
      ]
    }
  }
}
```

## 操作名称

1. 使用操作类型的关键字和操作名称进行查询，若是省略则是简写语法
2. 简写语法会造成代码存在歧义，不能很直观的体现操作对象和方法

```json
//操作类型(query) + 操作名称(HeroNameAndFriends) 进行查询
query HeroNameAndFriends {
    hero{
        name
        friends{
            name
        }
    }
}

//结果
{
  "data": {
    "hero": {
      "name": "R2-D2",
      "friends": [
        {
          "name": "Luke Skywalker"
        },
        {
          "name": "Han Solo"
        },
        {
          "name": "Leia Organa"
        }
      ]
    }
  }
}
```

- 操作类型: query,mutation,subscription
- 操作名称: 操作有意义和明确的名称

## 变量

1. 字段的参数可能是会动态变化的，需要一个变量来存储这个参数
2. 