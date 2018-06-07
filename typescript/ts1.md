## 安装

```javascript
//安装
npm i -g javascript
//版本检测
tsc --version
//编译ts
tsc demo.ts

```

- 自动化编译工程 [git地址](https://github.com/nordon-wang/ts)

## 自动拆分字符串

```javascript

/**
 * 自动拆分字符串
 */

function test(tem,name,age){
    console.log(tem); // 对模板字符串进行拆分,根据传入的变量位置进行拆分
    console.log(name); // 对应myName的值
    console.log(age); // 对应myAge方法的返回值
}

const myName = 'nordon'

const myAge = function(){
    return 18
}

test`my name is ${myName}, my age is ${myAge()}`//调用直接跟模板字符串,不能跟()

```

## 参数类型

```javascript
/**
 * 参数类型
 *  有五种默认的类型
 *      string,number,boolean,void,any
 */

let str1 : string
// str1 = 1 error 因为类型不匹配

let num  = 12
// 报错,ts会自动进行类型推断
// num = '123' 

let num2 : any = 123
// 不会报错,因为any可以接受任何类型
num2 = '123123'

let type1 :number // 数字
let type2 : boolean // 布尔
let type3 : string // 字符串

// 不需要返回任何值
function type4() :void {
}

// 指定返回值类型为string,返回其他类型会报错
function type5() : string{
    return 'must be string'
}

// 方法的参数声明类型,传入的参数必须是string
function tyep6(str:string){

}

// 报错 number类型
// tyep6(123) 


/**
 * 参数类型
 *     可以通过class,interface声明自定义类型
 */

class Person1 {
    name:string;
    age:number;
}

let person1 : Person1 = new Person1()
// 赋值是根据class的定义类型,若是类型不一致会报错
// person1.name = 123
person1.name = 'nordon'
person1.age  = 12
```

## 默认参数

```javascript
/**
 * 默认参数
 */

function defaultArgs(arg1:number = 18, arg2:string = 'norodn', arg3:boolean = false){
    console.log(`arg1 = ${arg1}; arg2 = ${arg2}; arg3 = ${arg3}`);
}

defaultArgs(11,'22',true) //arg1 = 11; arg2 = 22; arg3 = true
defaultArgs() //arg1 = 18; arg2 = norodn; arg3 = false
```

## 可选参数

```javascript
/**
 * 可选参数
 *      在方法参数声明的后面用问号来标明此参数为可选参数
 *      可选参数可以不传递
 *      必填参数不能放在可选参数的后面
 */
// arg2? :string = 'norodn' 在有默认参数值的情况下,不能使用可选参数
function defaultArgs2(arg1:string = '18', arg2 ? :string, arg3:string = 'false'){
    console.log(`arg1 = ${arg1}; arg2 = ${arg2}; arg3 = ${arg3}`);
}

defaultArgs2() //arg1 = 18; arg2 = undefined; arg3 = false
defaultArgs2('11') //arg1 = 11; arg2 = undefined; arg3 = false
defaultArgs2('1','2') //arg1 = 1; arg2 = 2; arg3 = false
defaultArgs2('1','2','3') //arg1 = 1; arg2 = 2; arg3 = 3
```

## 解构

```javascript
/**
 * 解构多层-对象
 */
function getStock(){
    return {
        names:'nordn',
        count:{
            glary:10000,
            age:18
        }
    }
}

const {names:myNames, count : {glary,age}} = getStock()
console.log(myNames, glary,age); //nordn 10000 18
```

```javascript
/**
 * 解构 - 数组
 */
const myArr1:number[] = [11,22,33,44]
const [myNum1,myNum2,...others] = myArr1
console.log(myNum1,myNum2,others); //11 22

const [mynum1,,,mynum4] = myArr1
console.log(mynum1,mynum4); //11 44
```

## 循环遍历区别

```JavaScript
const myArr1 = [11,22,33,44,'55']
myArr1.desc = 'asd'

// break return 没有效果,foreach是不允许打断循环的,属性值会被忽略
myArr1.forEach(item => {
    // console.log(item);
    if(item == 11){
        // console.log('return');
        return 
    }
})

//循环的是键值对中的键 
// 属性也会被遍历
for(let item in myArr1){
    // console.log(myArr1[item]);
}

// 属性值会被忽略,循环可以被打断
for(let item of myArr1){
    console.log(item);
}
```

## 类

```javascript

/**
 * calss
 */
class Person3 {

    // 类的构造方法,只会在类初始化的时候调用一次
    // address 必须要声明的
    constructor(public address:string){
        this.address = address
    }
    // 访问控制符,可以控制类的属性和方法是否可以在外界使用
    // public 默认值
    // private 私有的
    // protected 受保护的,可以在类的内部和子类可以访问,外界不能访问
    names:string

    private age:number


    say(){
        console.log(`say...${this.names},address...${this.address}`);
    }
}

class Child extends Person3{
   
   constructor(address:string, num:number){
       super(address)
   }

   num : number

   eat(){
       console.log(`the child num is ${this.num}`);
        //使用super关键字 调用父类的方法
       super.say()
       
   }
}

```

## 泛型

```JavaScript

/**
 * 泛型
 *      参数化的类型,一般用来限制集合的内容
 */
// 只能放Person3的实例
let childArr : Array<Person3> = []

childArr[0] = new Person3('安徽省...')
childArr[1] = new Child('安徽省...',11)
// 因为不是Person3的实例,不能存放
// childArr[2] = 22
```

## 接口

```JavaScript
/**
 * 接口
 *      用来建立某种代码约定,使得其它开发者在调用某个方法或者创建新的类时必须遵循接口定义的代码约定 
 */
interface Inters{
    myname : string
    age : number
}

class InterPerson {
    // 接口的用法,作为方法参数的类型声明
    constructor(public config:Inters){

    }
}

// 调用时必须按照interface接口所定义的规范使用
// 传入的对象有且只能包含myname和age,多一个或者少一个都不可以
let interP = new InterPerson({
    myname:'nordon',
    age:18
})
```

```JavaScript

/**
 * 在接口interface中声明一个方法,所有通过implements声明实现的类都必须实现这个方法
 */

interface Animal{
    eat()
}

class Sheep implements Animal{
    eat(){
        console.log('必须实现interface中的方法');
    }
}

class Tiger implements Animal{
    eat(){
        console.log('必须实现interface中的方法');
    }
}
```

