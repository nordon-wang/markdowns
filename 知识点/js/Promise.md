想优雅地进行异步操作，必须要熟识一个极其重要的概念 —— Promise。它是取代传统回调，实现同步链式写法的解决方案；是理解 generator、async/await 的关键

### 初见雏形

在微信小程序开发过程中，我们使用 wx.request() 在微信小程序环境中发送一个网络请求。参考官方文档，具体用法如下：

```js
wx.request({
  url: 'xxxx', // 仅为示例，并非真实的接口地址
  data: {
    x: '',
    y: ''
  },
  header: {
    'content-type': 'application/json' // 默认值
  },
  success(res) {
    console.log(res.data)
  }
})
```

配置化的 API 风格和早期使用 jQuery 中 Ajax 方法的封装类似。这样的设计有一个小的问题，就是容易出现“回调地狱”问题。如果想先通过 ./userInfo 接口来获取登录用户信息数据，再从登录用户信息数据中，通过请求 `./${id}/friendList` 接口来获取登录用户所有好友列表，就需要：

```js
wx.request({
  url: './userInfo',
  success(res) {
    const id = res.data.id
    wx.request({
      url: `./${id}/friendList`,
      success(res) {
        console.log(res)
      }
    })
  }
})
```

这只是嵌套了一层回调而已，还够不成“地狱”场景，但是足以说明问题。

我们知道解决“回调地狱”问题的一个极佳方式就是 Promise，将微信小程序 wx.request() 方法进行 Promise 化：

```js
const wxRequest = (url, data = {}, method = 'GET') => 
  new Promise((resolve, reject) => {
    wx.request({
      url,
      data,
      method,
      header: {
        //通用化 header 设置
      },
      success: function (res) {
        const code = res.statusCode
        if (code !== 200) {
          reject({ error: 'request fail', code })
          return
        }
        resolve(res.data)
      },
      fail: function (res) {
        reject({ error: 'request fail'})
      },
    })
  })
```

Promise 基本概念不再过多介绍。这是一个典型的 Promise 化案例，当然我们不仅可以对 wx.request() API 进行 Promise 化，更应该做的通用，能够 Promise 化更多类似（通过 success 和 fail 表征状态）的接口：

```js
const promisify = fn => args => 
  new Promise((resolve, reject) => {
    args.success = function(res) {
      return resolve(res)
    }
    args.fail = function(res) {
      return reject(res)
    }
  })

// 使用 
const wxRequest = promisify(wx.request)
```

通过上例，可以知道：

> Promise 其实就是一个构造函数，我们使用这个构造函数创建一个 Promise 实例。该构造函数很简单，它只有一个参数，按照 Promise/A+ 规范的命名，把 Promise 构造函数的参数叫做 executor，executor 类型为函数。这个函数又“自动”具有 resolve、reject 两个方法作为参数。

请仔细体会上述结论，那么可以通过结论，开始实现 Promise 的第一步：

```js
function Promise(executor) {

}
```

在上面的 wx.request() 介绍中，实现了 Promise 化，因此对于嵌套回调场景，可以：

```js
wxRequest('./userInfo')
  .then(
    data => wxRequest(`./${data.id}/friendList`),
    error => {
      console.log(error)
    }
  )
  .then(
    data => {
      console.log(data)
    },
    error => {
      console.log(error)
    }
  )
```

通过观察使用例子, 可以来剖析 Promise 的实质：

> **结论**　Promise 构造函数返回一个 promise 对象实例，这个返回的 promise 对象具有一个 then 方法。then 方法中，调用者可以定义两个参数，分别是 onfulfilled 和 onrejected，它们都是函数类型。其中 onfulfilled 通过参数，可以获取 promise 对象 resolved 的值，onrejected 获得 promise 对象 rejected 的值。通过这个值，我们来处理异步完成后的逻辑。

因此，继续实现 Promise：

```js
function Promise(executor) {

}

Promise.prototype.then = function(onfulfilled, onrejected) {

}
```

继续复习 Promise 的知识，看例子来理解：

```js
let promise1 = new Promise((resolve, reject) => {
  resolve('data')
})

promise1.then(data => {
  console.log(data)
})

let promise2 = new Promise((resolve, reject) => {
  reject('error')
})

promise2.then(data => {
  console.log(data)
}, error => {
  console.log(error)
})
```

> **结论**　在使用 new 关键字调用 Promise 构造函数时，在合适的时机（往往是异步结束时），调用 executor 的参数 resolve 方法，并将 resolved 的值作为 resolve 函数参数执行，这个值便可以后续在 then 方法第一个函数参数（onfulfilled）中拿到；同理，在出现错误时，调用 executor 的参数 reject 方法，并将错误信息作为 reject 函数参数执行，这个错误信息可以在后续的 then 方法第二个函数参数（onrejected）中拿到。

因此，在实现 Promise 时，应该有两个值，分别储存 resolved 的值，以及 rejected 的值（当然，因为 Promise 状态的唯一性，不可能同时出现 resolved 的值和 rejected 的值，因此也可以用一个变量来存储）；同时也需要存在一个状态，这个状态就是 promise 实例的状态（pending，fulfilled，rejected）；同时还要提供 resolve 方法以及 reject 方法，这两个方法需要作为 executor 的参数提供给开发者使用：

```js
function Promise(executor) {
  this.status = 'pending'
  this.value = null
  this.reason = null

  function resolve(value) {
    this.value = value
  }

  function reject(reason) {
    this.reason = reason
  }

  executor(resolve, reject)
}


Promise.prototype.then = function(onfulfilled, onrejected) {
  // 健壮性处理
  onfulfilled = typeof onfulfilled === 'function' ? onfulfilled : data => data
  onrejected = typeof onrejected === 'function' ? onrejected : error => {throw error}
	
  onfulfilled(this.value)

  onrejected(this.reason)
}
```

为了保证 onfulfilled、onrejected 能够强健执行，我们为其设置了默认值

### 状态完善

先来看一到题目，判断输出：

```js
let promise = new Promise((resolve, reject) => {
  resolve('data')
  reject('error')
})

promise.then(data => {
  console.log(data)
}, error => {
  console.log(error)
})
```

**只会**输出：data，因为我们知道 promise 实例状态只能从 pending 改变为 fulfilled，或者从 pending 改变为 rejected。状态一旦变更完毕，就不可再次变化或者逆转。也就是说：如果一旦变到 fulfilled，就不能再 rejected，一旦变到 rejected，就不能 fulfilled。

而我们的代码实现，显然无法满足这一特性。执行上一段代码时，将会输出 data 以及 error。

因此，需要对状态进行判断和完善：

```js
function Promise(executor) {
  this.status = 'pending'
  this.value = null
  this.reason = null

  const resolve = value => {
    if (this.status === 'pending') {
      this.value = value
      this.status = 'fulfilled'
    }
  }

  const reject = reason => {
    if (this.status === 'pending') {
      this.reason = reason
      this.status = 'rejected'
    }
  }

  executor(resolve, reject)
}

Promise.prototype.then = function(onfulfilled, onrejected) {
  // 健壮性处理
  onfulfilled = typeof onfulfilled === 'function' ? onfulfilled : data => data
  onrejected = typeof onrejected === 'function' ? onrejected : error => {throw error}
	
  // 只允许 promise 实例状态从 pending 改变为 fulfilled，或者从 pending 改变为 rejected。
  if (this.status === 'fulfilled') {
    onfulfilled(this.value)
  }
  if (this.status === 'rejected') {
    onrejected(this.reason)
  }
}
```

我们看，在 resolve 和 reject 方法中，我们加入判断，只允许 promise 实例状态从 pending 改变为 fulfilled，或者从 pending 改变为 rejected。

### 异步完善

到目前为止，实现还差了哪些内容呢？别急，再从示例代码分析：

```js
let promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve('data')
  }, 2000)
})

promise.then(data => {
  console.log(data)
})
```

正常来讲，上述代码会在 2 秒之后输出 data，但是目前实现的代码，并没有输入任何信息。这是为什么呢？

原因很简单，因为当前的实现逻辑全是同步的。在上面实例化一个 promise 的构造函数时，我们是在 setTimeout 逻辑里才调用 resolve，也就是说，2 秒之后才会调用 resolve 方法，也才会去更改 promise 实例状态。而结合目前实现，返回实现代码，then 方法中的 onfulfilled 执行是同步的，它在执行时 this.status 仍然为 pending，并没有做到“2 秒中之后再执行 onfulfilled”。

那该怎么办呢？应该在“合适”的时间才去调用 onfulfilled 方法，这个合适的时间就应该是开发者调用 resolve 的时刻，那么先在状态（status）为 pending 时，把开发者传进来的 onfulfilled 方法存起来，在 resolve 方法中再去执行即可：

```js
function Promise(executor) {
  this.status = 'pending'
  this.value = null
  this.reason = null
  this.onFulfilledFunc = Function.prototype
  this.onRejectedFunc = Function.prototype

  const resolve = value => {
    if (this.status === 'pending') {
      this.value = value
      this.status = 'fulfilled'

      this.onFulfilledFunc(this.value)
    }

  }

  const reject = reason => {
    if (this.status === 'pending') {
      this.reason = reason
      this.status = 'rejected'

      this.onRejectedFunc(this.reason)
    }
  }

  executor(resolve, reject)
}

Promise.prototype.then = function(onfulfilled, onrejected) {
  onfulfilled = typeof onfulfilled === 'function' ? onfulfilled : data => data
  onrejected = typeof onrejected === 'function' ? onrejected : error => {throw error}

  if (this.status === 'fulfilled') {
    onfulfilled(this.value)
  }
  if (this.status === 'rejected') {
    onrejected(this.reason)
  }
  if (this.status === 'pending') {
    this.onFulfilledFunc = onfulfilled
    this.onRejectedFunc = onrejected
  }
}
```

测试一下，发现现在的实现也可以支持异步了！

同时，**我们知道 Promise 是异步执行的：**

```js
let promise = new Promise((resolve, reject) => {
   resolve('data')
})

promise.then(data => {
  console.log(data)
})
console.log(1)
```

正常的话，这里会**按照顺序**，输出 1 再输出 data。

而目前的实现，却没有考虑这种情况，先输出 data 再输出 1。因此，需要将 resolve 和 reject 的执行，放到任务队列中。这里姑且先放到 setTimeout 里，保证异步执行（这样的做法并不严谨，为了保证 Promise 属于 microtasks，很多 Promise 的实现库用了 MutationObserver 来模仿 nextTick）。

```js
const resolve = value => {
  if (value instanceof Promise) {
    return value.then(resolve, reject)
  }
  setTimeout(() => {
    if (this.status === 'pending') {
      this.value = value
      this.status = 'fulfilled'

      this.onFulfilledFunc(this.value)
    }
  })
}

const reject = reason => {
  setTimeout(() => {
    if (this.status === 'pending') {
      this.reason = reason
      this.status = 'rejected'

      this.onRejectedFunc(this.reason)
    }
  })
}

executor(resolve, reject)
```

这样一来，在执行到 executor(resolve, reject) 时，也能保证在 nextTick 中才去执行，不会阻塞同步任务。

同时在 resolve 方法中，加入了对 value 值是一个 Promise 实例的判断。看一下到目前为止的实现代码：

```js
function Promise(executor) {
  this.status = 'pending'
  this.value = null
  this.reason = null
  this.onFulfilledFunc = Function.prototype
  this.onRejectedFunc = Function.prototype

  const resolve = value => {
    if (value instanceof Promise) {
      return value.then(resolve, reject)
    }
    setTimeout(() => {
      if (this.status === 'pending') {
        this.value = value
        this.status = 'fulfilled'

        this.onFulfilledFunc(this.value)
      }
    })
  }

  const reject = reason => {
    setTimeout(() => {
      if (this.status === 'pending') {
        this.reason = reason
        this.status = 'rejected'

        this.onRejectedFunc(this.reason)
      }
    })
  }

  executor(resolve, reject)
}

Promise.prototype.then = function(onfulfilled, onrejected) {
  onfulfilled = typeof onfulfilled === 'function' ? onfulfilled : data => data
  onrejected = typeof onrejected === 'function' ? onrejected : error => {throw error}

  if (this.status === 'fulfilled') {
    onfulfilled(this.value)
  }
  if (this.status === 'rejected') {
    onrejected(this.reason)
  }
  if (this.status === 'pending') {
    this.onFulfilledFunc = onfulfilled
    this.onRejectedFunc = onrejected
  }
}
```

### 细节完善

到此为止，似乎目前的 Promise 实现越来越靠谱了，但是还有些细节需要完善。

比如当 promise 实例状态变更之前，添加多个 then 方法：

```js
let promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve('data')
  }, 2000)
})

promise.then(data => {
  console.log(`1: ${data}`)
})
promise.then(data => {
  console.log(`2: ${data}`)
})


// 应该输出 1:data 2:data
```

而目前的实现，只会输出 2: data，这是因为第二个 then 方法中的 onFulfilledFunc 会覆盖第一个 then 方法中的 onFulfilledFunc。

这个问题也好解决，只需要将所有 then 方法中的 onFulfilledFunc 储存为一个数组 onFulfilledArray，在 resolve 时，依次执行即可。对于 onRejectedFunc 同理，改动后的实现为：

```js
function Promise(executor) {
  this.status = 'pending'
  this.value = null
  this.reason = null
  this.onFulfilledArray = [] // modify
  this.onRejectedArray = [] // modify

  const resolve = value => {
    if (value instanceof Promise) {
      return value.then(resolve, reject)
    }
    setTimeout(() => {
      if (this.status === 'pending') {
        this.value = value
        this.status = 'fulfilled'

        // modify
        this.onFulfilledArray.forEach(func => { 
          func(value)
        })
      }
    })
  }

  const reject = reason => {
    setTimeout(() => {
      if (this.status === 'pending') {
        this.reason = reason
        this.status = 'rejected'
				
        // modify
        this.onRejectedArray.forEach(func => {
          func(reason)
        })
      }
    })
  }
	
  // 在构造函数中如果出错，将会自动触发 promise 实例状态为 rejected，我们用 try…catch 块对 executor 进行包裹
  try {
    executor(resolve, reject)
  } catch(e) {
    reject(e)
  }
}

Promise.prototype.then = function(onfulfilled, onrejected) {
  onfulfilled = typeof onfulfilled === 'function' ? onfulfilled : data => data
  onrejected = typeof onrejected === 'function' ? onrejected : error => {throw error}

  if (this.status === 'fulfilled') {
    onfulfilled(this.value)
  }
  if (this.status === 'rejected') {
    onrejected(this.reason)
  }
  if (this.status === 'pending') {
    this.onFulfilledArray.push(onfulfilled)
    this.onRejectedArray.push(onrejected)
  }
}
```

### 链式调用

在完成了`Promise`的基础功能之后，看一道题目

```js
const promise = new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve('Nordon')
  }, 2000)
})

promise.then(data => {
  console.log(data)
  return `${data} next then`
})
.then(data => {
  console.log(data)
})
```

这段代码执行后，将会在 2 秒后输出：Nordon，紧接着输出：Nordon next then。

可以看到，Promise 实例的 then 方法支持链式调用，输出 resolved 值后，如果在 then 方法体 onfulfilled 函数中同步显式返回新的值，将会在新 Promise 实例的 then 方法 onfulfilled 函数中输出新值。

如果在第一个 then 方法体 onfulfilled 函数中返回另一个 Promise 实例

```js
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
      resolve('Nordon')
  }, 2000)
})

promise.then(data => {
  console.log(data)
  return new Promise((resolve, reject) => {
    setTimeout(() => {
        resolve(`${data} next then`)
    }, 4000)
  })
})
.then(data => {
  console.log(data)
})
```

将在 2 秒后输出：Nordon，紧接着再过 4 秒后（第 6 秒）输出：Nordon next then。

由此可知：

> 一个 Promise 实例的 then 方法体 onfulfilled 函数和 onrejected 函数中，是支持再次返回一个 Promise 实例的，也支持返回一个非 Promise 实例的普通值；并且返回的这个 Promise 实例或者这个非 Promise 实例的普通值将会传给下一个 then 方法 onfulfilled 函数或者 onrejected 函数中，这样就支持链式调用了。

该怎么实现这种行为呢？

首先来分析一下：为了能够支持 then 方法的链式调用，那么每一个 then 方法的 onfulfilled 函数和 onrejected 函数都应该返回一个 Promise 实例。

先实现返回一个非`Promise`对象的情况

```js
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
      resolve('Nordon')
  }, 2000)
})

promise.then(data => {
  console.log(data)
  return `${data} next then`
})
.then(data => {
  console.log(data)
})
```

这种 onfulfilled 函数返回一个普通值的场景，这里 onfulfilled 函数指的是：

```js
data => {
  console.log(data)
  return `${data} next then`
}
```

在之前实现的 then 方法中，就可以创建一个新的 promise2 用以返回：

```js
Promise.prototype.then = function(onfulfilled, onrejected) {
  onfulfilled = typeof onfulfilled === 'function' ? onfulfilled : data => data
  onrejected = typeof onrejected === 'function' ? onrejected : error => { throw error }
  // promise2 将作为 then 方法的返回值
  let promse2
  if (this.status === 'fulfilled') {
    return promse2 = new Promise((resolve, reject) => {
      setTimeout(() => {
        try {
          // 这个新的 promse2 resolved 的值为 onfulfilled 的执行结果
          let result = onfulfilled(this.value)
          resolve(result)
        }
        catch(e) {
          reject(e)
        }
      })
    })
  }
  if (this.status === 'rejected') {
    onrejected(this.reason)
  }
  if (this.status === 'pending') {
    this.onFulfilledArray.push(onfulfilled)
    this.onRejectedArray.push(onrejected)
  }
}
```

当然别忘了 this.status === 'rejected' 状态和 this.status === 'pending' 状态也要加入相同的逻辑：

```js
Promise.prototype.then = function(onfulfilled, onrejected) {
  // promise2 将作为 then 方法的返回值
  let promise2
  
  if (this.status === 'fulfilled') {
    return promise2 = new Promise((resolve, reject) => {
            setTimeout(() => {
                try {
                    // 这个新的 promise2 resolved 的值为 onfulfilled 的执行结果
                    let result = onfulfilled(this.value)
                    resolve(result)
                }
                catch(e) {
                    reject(e)
                }
            })
    })
  }
  
  if (this.status === 'rejected') {
    return promise2 = new Promise((resolve, reject) => {
            setTimeout(() => {
                try {
                    // 这个新的 promise2 reject 的值为 onrejected 的执行结果
                    let result = onrejected(this.value)
                    resolve(result)
                }
                catch(e) {
                    reject(e)
                }
            })
    })
  }
  
  if (this.status === 'pending') {
    return promise2 = new Promise((resolve, reject) => {
      this.onFulfilledArray.push(() => {
        try {
          let result = onfulfilled(this.value)
          resolve(result)
        }
        catch(e) {
          reject(e)
        }
      })

      this.onRejectedArray.push(() => {
        try {
          let result = onrejected(this.reason)
          resolve(result)
        }
        catch(e) {
          reject(e)
        }
      })      
    })
  }
}
```

这里要重点理解 this.status === 'pending' 判断分支中的逻辑，这也最难理解的。先想想：当使用 Promise 实例，调用其 then 方法时，应该返回一个 Promise 实例，返回的就是 this.status === 'pending' 判断分支中返回的 promise2。那么这个 promise2 什么时候被 resolve 或者 reject 呢？应该是在异步结束，依次执行 onFulfilledArray 或者 onRejectedArray 数组中的函数时。

再思考，那么 onFulfilledArray 或者 onRejectedArray 数组中的函数应该做些什么呢？很明显，需要将 promise2 的状态切换，并 resolve onfulfilled 函数执行结果或者 reject onrejected 结果。

这也就是目前的改动，将 this.onFulfilledArray.push 的函数由：

```js
this.onFulfilledArray.push(onfulfilled)
```

改为：

```js
() => {
    setTimeout(() => {
        try {
            let result = onfulfilled(this.value)
            resolve(result)
        }
        catch(e) {
            reject(e)
        }
    })
}
```

的原因。 this. onRejectedArray.push 的函数的改动点同理。



继续来实现 then 方法显式返回一个 Promise 实例的情况。对应场景：

```js
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
      resolve('Nordon')
  }, 2000)
})

promise.then(data => {
  console.log(data)
  return new Promise((resolve, reject) => {
    setTimeout(() => {
        resolve(`${data} next then`)
    }, 3000)
  })
})
.then(data => {
  console.log(data)
})
```

对比第一种情况（ onfulfilled 函数和 onrejected 函数返回一个普通值的情况），实现这种 onfulfilled 函数和 onrejected 函数返回一个 Promise 实例也并不困难。但是我们需要小幅度重构一下代码，在上面实现的 let result = onfulfilled(this.value) 语句和 let result = onrejected(this.reason) 语句中，变量 result 由一个普通值会成为一个 Promise 实例。换句话说就是：变量 result 既可以是一个普通值，也可以是一个 Promise 实例，为此抽象出 resolvePromise 方法进行统一处理。改动已有实现为：

```js
const resolvePromise = (promise2, result, resolve, reject) => { // 待完善

}

Promise.prototype.then = function(onfulfilled, onrejected) {
  // promise2 将作为 then 方法的返回值
  let promise2
  if (this.status === 'fulfilled') {
    return promise2 = new Promise((resolve, reject) => {
            setTimeout(() => {
                try {
                    //这个新的 promise2 resolved 的值为 onfulfilled 的执行结果
                    let result = onfulfilled(this.value)
                    resolvePromise(promise2, result, resolve, reject)
                }
                catch(e) {
                    reject(e)
                }
            })
    })
  }
  if (this.status === 'rejected') {
    return promise2 = new Promise((resolve, reject) => {
            setTimeout(() => {
                try {
                    //这个新的 promise2 reject 的值为 onrejected 的执行结果
                    let result = onrejected(this.value)
                 resolvePromise(promise2, result, resolve, reject)
                }
                catch(e) {
                    reject(e)
                }
            })
    })
  }
  if (this.status === 'pending') {
    return promise2 = new Promise((resolve, reject) => {
      this.onFulfilledArray.push(value => {
        try {
          let result = onfulfilled(value)
          resolvePromise(promise2, result, resolve, reject)
        }
        catch(e) {
          reject(e)
        }
      })

      this.onRejectedArray.push(reason => {
        try {
          let result = onrejected(reason)
          resolvePromise(promise2, result, resolve, reject)
        }
        catch(e) {
          reject(e)
        }
      })      
    })
  }
}
```

现在的任务就是完成 resolvePromise 函数，这个函数接受四个参数：

- promise2: 返回的 Promise 实例
- result: onfulfilled 或者 onrejected 函数的返回值
- resolve: promise2 的 resolve 方法
- reject: promise2 的 reject 方法

有了这些参数，就具备了抽象逻辑的必备条件。接下来就是动手实现：

```js
const resolvePromise = (promise2, result, resolve, reject) => {
  // 当 result 和 promise2 相等时，也就是说 onfulfilled 返回 promise2 时，进行 reject
  if (result === promise2) {
    reject(new TypeError('error due to circular reference'))
  }

  // 是否已经执行过 onfulfilled 或者 onrejected
  let consumed = false
  let thenable

  if (result instanceof Promise) {
    if (result.status === 'pending') {
      result.then(function(data) {
        resolvePromise(promise2, data, resolve, reject)
      }, reject)
    } else {
      result.then(resolve, reject)
    }
    return
  }

  let isComplexResult = target => (typeof target === 'function' || typeof target === 'object') && (target !== null)

  // 如果返回的是疑似 Promise 类型
  if (isComplexResult(result)) {
    try {
      thenable = result.then
      // 如果返回的是 Promise 类型，具有 then 方法
      if (typeof thenable === 'function') {
        thenable.call(result, function(data) {
          if (consumed) {
            return
          }
          consumed = true

          return resolvePromise(promise2, data, resolve, reject)
        }, function(error) {
          if (consumed) {
            return
          }
          consumed = true

          return reject(error)
        })
      }
      else {
        resolve(result)
      }

    } catch(e) {
      if (consumed) {
        return
      }
      consumed = true
      return reject(e)
    }
  }
  else {
    resolve(result)
  }
}
```

接着，对于 onfulfilled 函数返回的结果 result：如果 result 非 Promise 实例，非对象，非函数类型，是一个普通值的话（上述代码中 isComplexResult 函数进行判断），直接将 promise2 以该值 resolve 掉。

对于 onfulfilled 函数返回的结果 result：如果 result 含有 then 属性方法，称该属性方法为 thenable，说明 result 是一个 Promise 实例，执行该实例的 then 方法（既 thenable），此时的返回结果有可能又是一个 Promise 实例类型，也可能是一个普通值，因此还要递归调用 resolvePromise。

### 穿透实现

看代码

```js
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
      resolve('Nordon')
  }, 2000)
})


promise.then(null)
.then(data => {
  console.log(data)
})
```

这段代码将会在 2 秒后输出：Nordon。这就是 Promise 穿透现象：

> 给 .then() 函数传递非函数值作为其参数时，实际上会被解析成 .then(null)，这时候的表现应该是：上一个 promise 对象的结果进行“穿透”，如果在后面链式调用仍存在第二个 .then() 函数时，将会获取被穿透下来的结果。

那该如何实现 Promise 穿透呢？

其实很简单，并且我们已经做到了。想想在 then() 方法的实现中：我们已经对 onfulfilled 和 onrejected 函数加上判断：

```js
Promise.prototype.then = function(onfulfilled = Function.prototype, onrejected = Function.prototype) {
  onfulfilled = typeof onfulfilled === 'function' ? onfulfilled : data => data
  onrejected = typeof onrejected === 'function' ? onrejected : error => { throw error }

    // ...
}
```

如果 onfulfilled 不是函数类型，则给一个默认值，该默认值是返回其参数的函数。onrejected 函数同理。这段逻辑，就是起到了实现“穿透”的作用。

### 其他方法&&静态方法

#### catch 实现

Promise.prototype.catch 可以进行异常捕获，它的典型用法：

```js
const promise1 = new Promise((resolve, reject) => {
  setTimeout(() => {
      reject('Nordon error')
  }, 2000)
})

promise1.then(data => {
  console.log(data)
}).catch(error => {
  console.log(error)
})
```

会在 2 秒后输出：Nordon error。

其实在这种场景下，它就相当于：

```js
Promise.prototype.catch = function(catchFunc) {
  return this.then(null, catchFunc)
}
```

因为.then() 方法的第二个参数也是进行异常捕获的，通过这个特性，可以比较简单地实现了 Promise.prototype.catch。

#### resolve 实现

> Promise.resolve(value) 方法返回一个以给定值解析后的 Promise 实例对象。

例子

```js
Promise.resolve('data').then(data => {
  console.log(data)
})
console.log(1)
```

先输出 1 再输出 data。

那么实现 Promise.resolve(value) 也很简单：

```js
Promise.resolve = function(value) {
  return new Promise((resolve, reject) => {
    resolve(value)
  })
}

// 其实 reject 同理
Promise.reject = function(value) {
  return new Promise((resolve, reject) => {
    reject(value)
  })
}
```

#### all 实现

> Promise.all(iterable) 方法返回一个 Promise 实例，此实例在 iterable 参数内所有的 promise 都“完成（resolved）”或参数中不包含 promise 时回调完成（resolve）；如果参数中 promise 有一个失败（rejected），此实例回调失败（reject），失败原因的是第一个失败 promise 的结果。

例子

```js
const promise1 = new Promise((resolve, reject) => {
  setTimeout(() => {
      resolve('Nordon')
  }, 2000)
})

const promise2 = new Promise((resolve, reject) => {
  setTimeout(() => {
      resolve('WY')
  }, 2000)
})

Promise.all([promise1, promise2]).then(data => {
  console.log(data)
})
```

将在 2 秒后输出：["Nordon", "WY"]。

实现思路:

```js
Promise.all = function(promiseArray) {
  if (!Array.isArray(promiseArray)) {
      throw new TypeError('The arguments should be an array!')
  }
  return new Promise((resolve, reject) => {
    try {
      let resultArray = []

      const length = promiseArray.length

      for (let i = 0; i <length; i++) {
        promiseArray[i].then(data => {
          resultArray.push(data)

          if (resultArray.length === length) {
            resolve(resultArray)
          }
        }, reject)
      }
    }
    catch(e) {
      reject(e)
    }
  })
}
```

先进行了对参数 promiseArray 的类型判断，对于非数组类型参数，进行抛错。Promise.all 会返回一个 Promise 实例，这个实例将会在 promiseArray 中的所有 Promise 实例 resolve 后进行 resolve，且 resolve 的值是一个数组，这个数组存有 promiseArray 中的所有 Promise 实例 resolve 的值。

整体思路依赖一个 for 循环对 promiseArray 进行遍历。同样按照这个思路，继续对 Promise.race 进行实现。

#### race 实现

```js
Promise.race = function(promiseArray) {
  if (!Array.isArray(promiseArray)) {
      throw new TypeError('The arguments should be an array!')
  }
  return new Promise((resolve, reject) => {
    try {
          const length = promiseArray.length
      for (let i = 0; i <length; i++) {
        promiseArray[i].then(resolve, reject)
      }
    }
    catch(e) {
      reject(e)
    }
  })
}
```

简单分析一下，这里使用 for 循环同步执行 promiseArray 数组中的所有 promise 实例 then 方法，第一个 resolve 的实例直接会触发新 Promise（代码中新 new 出来的） 实例的 resolve 方法

### 完整代码

截止目前本文已经对根据[Promise/A+](https://promisesaplus.com/)的常用`API`实现了简单版本的`Promise`

整体代码如下

```js
function Promise(executor) {
  this.status = 'pending'
  this.value = null
  this.reason = null
  this.onFulfilledArray = []
  this.onRejectedArray = []

  const resolve = value => {
    if (value instanceof Promise) {
      return value.then(resolve, reject)
    }
    setTimeout(() => {
      if (this.status === 'pending') {
        this.value = value
        this.status = 'fulfilled'

        this.onFulfilledArray.forEach(func => {
          func(value)
        })
      }
    })
  }

  const reject = reason => {
    setTimeout(() => {
      if (this.status === 'pending') {
        this.reason = reason
        this.status = 'rejected'

        this.onRejectedArray.forEach(func => {
          func(reason)
        })
      }
    })
  }


    try {
        executor(resolve, reject)
    } catch(e) {
        reject(e)
    }
}

const resolvePromise = (promise2, result, resolve, reject) => {
  // 当 result 和 promise2 相等时，也就是说 onfulfilled 返回 promise2 时，进行 reject
  if (result === promise2) {
    return reject(new TypeError('error due to circular reference'))
  }

  // 是否已经执行过 onfulfilled 或者 onrejected
  let consumed = false
  let thenable

  if (result instanceof Promise) {
    if (result.status === 'pending') {
      result.then(function(data) {
        resolvePromise(promise2, data, resolve, reject)
      }, reject)
    } else {
      result.then(resolve, reject)
    }
    return
  }

  let isComplexResult = target => (typeof target === 'function' || typeof target === 'object') && (target !== null)
  // 如果返回的是疑似 Promise 类型
  if (isComplexResult(result)) {
    try {
      thenable = result.then
      // 如果返回的是 Promise 类型，具有 then 方法
      if (typeof thenable === 'function') {
        thenable.call(result, function(data) {
          if (consumed) {
            return
          }
          consumed = true

          return resolvePromise(promise2, data, resolve, reject)
        }, function(error) {
          if (consumed) {
            return
          }
          consumed = true

          return reject(error)
        })
      }
      else {
        return resolve(result)
      }

    } catch(e) {
      if (consumed) {
        return
      }
      consumed = true
      return reject(e)
    }
  }
  else {
    return resolve(result)
  }
}

Promise.prototype.then = function(onfulfilled, onrejected) {
  onfulfilled = typeof onfulfilled === 'function' ? onfulfilled : data => data
  onrejected = typeof onrejected === 'function' ? onrejected : error => {throw error}

  // promise2 将作为 then 方法的返回值
  let promise2

  if (this.status === 'fulfilled') {
    return promise2 = new Promise((resolve, reject) => {
      setTimeout(() => {
        try {
          // 这个新的 promise2 resolved 的值为 onfulfilled 的执行结果
          let result = onfulfilled(this.value)
          resolvePromise(promise2, result, resolve, reject)
        }
        catch(e) {
          reject(e)
        }
      })
    })
  }
  if (this.status === 'rejected') {
    return promise2 = new Promise((resolve, reject) => {
      setTimeout(() => {
        try {
          // 这个新的 promise2 reject 的值为 onrejected 的执行结果
         let result = onrejected(this.reason)
         resolvePromise(promise2, result, resolve, reject)
        }
        catch(e) {
          reject(e)
        }
      })
    })
  }
  if (this.status === 'pending') {
    return promise2 = new Promise((resolve, reject) => {
      this.onFulfilledArray.push(value => {
        try {
          let result = onfulfilled(value)
          resolvePromise(promise2, result, resolve, reject)
        }
        catch(e) {
          return reject(e)
        }
      })

      this.onRejectedArray.push(reason => {
        try {
          let result = onrejected(reason)
          resolvePromise(promise2, result, resolve, reject)
        }
        catch(e) {
          return reject(e)
        }
      })      
    })
  }
}

Promise.prototype.catch = function(catchFunc) {
  return this.then(null, catchFunc)
}

Promise.resolve = function(value) {
  return new Promise((resolve, reject) => {
    resolve(value)
  })
}

Promise.reject = function(value) {
  return new Promise((resolve, reject) => {
    reject(value)
  })
}

Promise.race = function(promiseArray) {
  if (!Array.isArray(promiseArray)) {
      throw new TypeError('The arguments should be an array!')
  }
  return new Promise((resolve, reject) => {
    try {
      const length = promiseArray.length
      for (let i = 0; i <length; i++) {
        promiseArray[i].then(resolve, reject)
      }
    }
    catch(e) {
      reject(e)
    }
  })
}

Promise.all = function(promiseArray) {
  if (!Array.isArray(promiseArray)) {
      throw new TypeError('The arguments should be an array!')
  }
  return new Promise((resolve, reject) => {
    try {
      let resultArray = []

      const length = promiseArray.length

      for (let i = 0; i <length; i++) {
        promiseArray[i].then(data => {
          resultArray.push(data)

          if (resultArray.length === length) {
            resolve(resultArray)
          }
        }, reject)
      }
    }
    catch(e) {
      reject(e)
    }
  })
}
```