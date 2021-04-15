# 你能手写一个Promise吗？Yes I promise。

## 前言

[实践系列] 主要是让我们通过实践去加深对一些原理的理解。

[[实践系列\]前端路由](https://juejin.im/post/6844903759458336776)

[[实践系列\]Babel原理](https://juejin.im/post/6844903760603398151)

[[实践系列\]浏览器缓存](https://juejin.im/post/6844903764566999054)

有兴趣的同学可以关注 [[实践系列\]](https://github.com/webfansplz/article) 。 求star求follow~

## 什么是Promise ?

> Promise是JS异步编程中的重要概念，异步抽象处理对象，是目前比较流行Javascript异步编程解决方案之一

## Promises/A+ 规范

> 为实现者提供一个健全的、可互操作的 JavaScript promise 的开放标准。

### 术语

- **解决 (fulfill)** : 指一个 promise 成功时进行的一系列操作，如状态的改变、回调的执行。虽然规范中用 fulfill 来表示解决，但在后世的 promise 实现多以 resolve 来指代之。
- **拒绝（reject)** : 指一个 promise 失败时进行的一系列操作。
- **拒因 (reason)** : 也就是拒绝原因，指在 promise 被拒绝时传递给拒绝回调的值。
- **终值（eventual value）** : 所谓终值，指的是 promise 被解决时传递给解决回调的值，由于 promise 有一次性的特征，因此当这个值被传递时，标志着 promise 等待态的结束，故称之终值，有时也直接简称为值（value）。
- **Promise** : promise 是一个拥有 then 方法的对象或函数，其行为符合本规范。
- **thenable** : 是一个定义了 then 方法的对象或函数，文中译作“拥有 then 方法”。
- **异常（exception）** : 是使用 throw 语句抛出的一个值。

### 基本要求

> 下面我们先来讲述Promise/A+ 规范的几个基本要求。

#### 1. Promise的状态

一个Promise的当前状态必须是以下三种状态中的一种: **等待状态（Pending）** **执行状态（Fulfilled）** 和 **拒绝状态（Rejected）。**

```
const PENDING = 'pending';

const FULFILLED = 'fulfilled';

const REJECTED = 'rejected';

复制代码
```

**等待状态 (Pending)**

处于等待态时，promise 需满足以下条件：

- 可以迁移至执行态或拒绝态

```
 if (this.state === PENDING) {
     this.state = FULFILLED || REJECTED ；
 }
复制代码
```

**执行状态 (Fulfilled)**

处于执行态时，promise 需满足以下条件：

- 不能迁移至其他任何状态
- 必须拥有一个不可变的终值

```
 this.value = value;
复制代码
```

**拒绝状态 (Rejected)**

处于拒绝态时，promise 需满足以下条件：

- 不能迁移至其他任何状态
- 必须拥有一个不可变的据因

```
 this.reason = reason;
复制代码
```

这里的不可变指的是恒等（即可用 === 判断相等），而不是意味着更深层次的不可变（译者注：盖指当 value 或 reason 不是基本值时，只要求其引用地址相等，但属性值可被更改）

#### 2. Then 方法

一个 promise 必须提供一个 then 方法以访问其当前值、终值和据因。

promise 的 then 方法接受两个参数：

```
promise.then(onFulfilled, onRejected)
复制代码
```

**参数可选**

onFulfilled 和 onRejected 都是可选参数。

- 如果 onFulfilled 不是函数，其必须被忽略
- 如果 onRejected 不是函数，其必须被忽略

**onFulfilled 特性**

如果 onFulfilled 是函数：

- 当 promise 执行结束后其必须被调用，其第一个参数为 promise 的终值
- 在 promise 执行结束前其不可被调用
- 其调用次数不可超过一次

**onRejected 特性**

如果 onRejected 是函数：

- 当 promise 被拒绝执行后其必须被调用，其第一个参数为 promise 的据因
- 在 promise 被拒绝执行前其不可被调用
- 其调用次数不可超过一次

**调用时机**

onFulfilled 和 onRejected 只有在执行环境堆栈仅包含平台代码时才可被调用 **注1**

**注1** 这里的平台代码指的是引擎、环境以及 promise 的实施代码。实践中要确保 onFulfilled 和 onRejected 方法异步执行，且应该在 then 方法被调用的那一轮事件循环之后的新执行栈中执行。

这个事件队列可以采用“宏任务（macro - task）”机制或者“微任务（micro - task）”机制来实现。

由于 promise 的实施代码本身就是平台代码（译者注：即都是 JavaScript），故代码自身在处理在处理程序时可能已经包含一个任务调度队列。

**调用要求**

onFulfilled 和 onRejected 必须被作为函数调用（即没有 this 值）

**多次调用**

then 方法可以被同一个 promise 调用多次

- 当 promise 成功执行时，所有 onFulfilled 需按照其注册顺序依次回调
- 当 promise 被拒绝执行时，所有的 onRejected 需按照其注册顺序依次回调

### 简易版实践

> 我们先通过实践一个简易版的Promise来消化一下上面Promises/A+规范的基本要求。

首先

```
npm init 

// 测试实现是否符合 promises/A+ 规范

npm install promises-aplus-tests -D 

复制代码
```

**package.json**

```
{
  "name": "ajpromise",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "promises-aplus-tests ./simple.js"
  },
  "author": "webfansplz",
  "license": "MIT",
  "devDependencies": {
    "promises-aplus-tests": "^2.1.2"
  }
}
    
复制代码
```

**simple.js**

```
//Promise 的三种状态  (满足要求 -> Promise的状态)
const PENDING = 'pending';
const FULFILLED = 'fulfilled';
const REJECTED = 'rejected';

class AjPromise {
  constructor(fn) {
    //当前状态
    this.state = PENDING;
    //终值
    this.value = null;
    //拒因
    this.reason = null;
    //成功态回调队列
    this.onFulfilledCallbacks = [];
    //拒绝态回调队列
    this.onRejectedCallbacks = [];

    //成功态回调
    const resolve = value => {
      // 使用macro-task机制(setTimeout),确保onFulfilled异步执行,且在 then 方法被调用的那一轮事件循环之后的新执行栈中执行。
      setTimeout(() => {
        if (this.state === PENDING) {
          // pending(等待态)迁移至 fulfilled(执行态),保证调用次数不超过一次。
          this.state = FULFILLED;
          // 终值
          this.value = value;
          this.onFulfilledCallbacks.map(cb => {
            this.value = cb(this.value);
          });
        }
      });
    };
    //拒绝态回调
    const reject = reason => {
      // 使用macro-task机制(setTimeout),确保onRejected异步执行,且在 then 方法被调用的那一轮事件循环之后的新执行栈中执行。 (满足要求 -> 调用时机)
      setTimeout(() => {
        if (this.state === PENDING) {
          // pending(等待态)迁移至 fulfilled(拒绝态),保证调用次数不超过一次。
          this.state = REJECTED;
          //拒因
          this.reason = reason;
          this.onRejectedCallbacks.map(cb => {
            this.reason = cb(this.reason);
          });
        }
      });
    };
    try {
      //执行promise
      fn(resolve, reject);
    } catch (e) {
      reject(e);
    }
  }
  then(onFulfilled, onRejected) {
    typeof onFulfilled === 'function' && this.onFulfilledCallbacks.push(onFulfilled);
    typeof onRejected === 'function' && this.onRejectedCallbacks.push(onRejected);
    // 返回this支持then 方法可以被同一个 promise 调用多次
    return this;
  }
}

复制代码
```

就这样,一个简单的promise就完成了.

```
new AjPromise((resolve, reject) => {
  setTimeout(() => {
    resolve(2);
  }, 2000);
})
  .then(res => {
    console.log(res);
    return res + 1;
  })
  .then(res => {
    console.log(res);
  });

//output  

// delay 2s..
//  2 
//  3 
复制代码
```

接下来，我们来看看我们的实现是否完全符合promises/A+规范~

```
npm run test
复制代码
```

GG,测试用例只过了一小部分,大部分飘红~

OK,接下来,我们来继续了解promises/A+ 进一步的规范要求~

### 进一步要求

**由于接下来的要求比较抽象和难理解,所以我们将一步一步实践来加深理解。**

#### 1. 返回

- 1.then方法必须返回一个promise对象
- 2.如果 onFulfilled 或者 onRejected 返回一个值 x ，则运行下面的 Promise 解决过程：[[Resolve]](promise2, x)
- 3.如果 onFulfilled 或者 onRejected 抛出一个异常 e ，则 promise2 必须拒绝执行，并返回拒因 e。
- 4.如果 onFulfilled 不是函数且 promise1 成功执行， promise2 必须成功执行并返回相同的值。
- 5.如果 onRejected 不是函数且 promise1 拒绝执行， promise2 必须拒绝执行并返回相同的据因。
- 6.不论 promise1 被 reject 还是被 resolve 时 promise2 都会被 resolve，只有出现异常时才会被 rejected。

**我们通过以上要求来一步一步完善then方法**
1.

```
// 1.首先,then方法必须返回一个promise对象
  then(onFulfilled, onRejected) {
    let newPromise;
    return (newPromise = new AjPromise((resolve, reject) => {}));
  }
复制代码
```

1. 

```
  then(onFulfilled, onRejected) {
    let newPromise;
    return (newPromise = new AjPromise((resolve, reject) => {
      // 2.如果 onFulfilled 或者 onRejected 返回一个值 x ，则运行下面的 Promise 解决过程：[[Resolve]](promise2, x)
      this.onFulfilledCallbacks.push(value => {
        let x = onFulfilled(value);
        //解决过程 resolvePromise
        resolvePromise(newPromise, x);
      });
      this.onRejectedCallbacks.push(reason => {
        let x = onRejected(reason);
        //解决过程 resolvePromise
        resolvePromise(newPromise, x);
      });
    }));
  }
  // 解决过程
  function resolvePromise() {
  //...
  }

复制代码
```

1. 

```
  then(onFulfilled, onRejected) {
    let newPromise;
    return (newPromise = new AjPromise((resolve, reject) => {
      //  3.如果 onFulfilled 或者 onRejected 抛出一个异常 e ，则 promise2 必须拒绝执行，并返回拒因 e。
      this.onFulfilledCallbacks.push(value => {
        try {
          let x = onFulfilled(value);
          resolvePromise(newPromise, x);
        } catch (e) {
          reject(e);
        }
      });
      this.onRejectedCallbacks.push(reason => {
        try {
          let x = onRejected(reason);
          resolvePromise(newPromise, x);
        } catch (e) {
          reject(e);
        }
      });
    }));
  }
复制代码
```

4,5.

```
  then(onFulfilled, onRejected) {  
    let newPromise;
    // 4.如果 onFulfilled 不是函数且 promise1 成功执行， promise2 必须成功执行并返回相同的值。
    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value;
    // 5.如果 onRejected 不是函数且 promise1 拒绝执行， promise2 必须拒绝执行并返回相同的据因。
    onRejected =
      typeof onRejected === 'function'
        ? onRejected
        : reason => {
            throw reason;
          };
    return (newPromise = new AjPromise((resolve, reject) => {
      this.onFulfilledCallbacks.push(value => {
        try {
          let x = onFulfilled(value);
          resolvePromise(newPromise, x);
        } catch (e) {
          reject(e);
        }
      });
      this.onRejectedCallbacks.push(reason => {
        try {
          let x = onRejected(reason);
          resolvePromise(newPromise, x);
        } catch (e) {
          reject(e);
        }
      });
    }));
  }
复制代码
```

1. 

```
  then(onFulfilled, onRejected) {
    let newPromise;

    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value;
    onRejected =
      typeof onRejected === 'function'
        ? onRejected
        : reason => {
            throw reason;
          };
    // 2.2.6规范 对于一个promise，它的then方法可以调用多次.
    // 当在其他程序中多次调用同一个promise的then时 由于之前状态已经为FULFILLED / REJECTED状态，则会走以下逻辑,
    // 所以要确保为FULFILLED / REJECTED状态后 也要异步执行onFulfilled / onRejected ,这里使用setTimeout

    // 6.不论 promise1 被 reject 还是被 resolve 时 promise2 都会被 resolve，只有出现异常时才会被 rejected。
    // 由于在接下来的解决过程中需要调用resolve,reject进行处理,处理我们在调用处理过程时,传入参数
    if (this.state == FULFILLED) {  
      return (newPromise = new AjPromise((resolve, reject) => {
        setTimeout(() => {
          try {
            let x = onFulfilled(this.value);
            resolvePromise(newPromise, x, resolve, reject);
          } catch (e) {
            reject(e);
          }
        });
      }));
    }
    if (this.state == REJECTED) {
      return (newPromise = new AjPromise((resolve, reject) => {
        setTimeout(() => {
          try {
            let x = onRejected(this.reason);
            resolvePromise(newPromise, x, resolve, reject);
          } catch (e) {
            reject(e);
          }
        });
      }));
    }
    if (this.state === PENDING) {
      return (newPromise = new AjPromise((resolve, reject) => {
        this.onFulfilledCallbacks.push(value => {
          try {
            let x = onFulfilled(value);
            resolvePromise(newPromise, x, resolve, reject);
          } catch (e) {
            reject(e);
          }
        });
        this.onRejectedCallbacks.push(reason => {
          try {
            let x = onRejected(reason);
            resolvePromise(newPromise, x, resolve, reject);
          } catch (e) {
            reject(e);
          }
        });
      }));
    }
  }
复制代码
```

ok,完整的then方法搞定了。相信通过以上实践,你对返回要求已经有了更深的理解。

#### 2. Promise 解决过程

> Promise 解决过程是一个抽象的操作，其需输入一个 promise 和一个值，我们表示为 [[Resolve]](promise, x)，如果 x 有 then 方法且看上去像一个 Promise ，解决程序即尝试使 promise 接受 x 的状态；否则其用 x 的值来执行 promise 。

> 这种 thenable 的特性使得 Promise 的实现更具有通用性：只要其暴露出一个遵循 Promise/A+ 协议的 then 方法即可；这同时也使遵循 Promise/A+ 规范的实现可以与那些不太规范但可用的实现能良好共存。

运行 [[Resolve]](promise, x) 需遵循以下步骤：

**1。x 与 promise 相等**

如果 promise 和 x 指向同一对象，以 TypeError 为据因拒绝执行 promise。

**2。x 为 Promise**

- 如果 x 为 Promise ，则使 promise 接受 x 的状态。
- 如果 x 处于等待态， promise 需保持为等待态直至 x 被执行或拒绝。
- 如果 x 处于执行态，用相同的值执行 promise。
- 如果 x 处于拒绝态，用相同的据因拒绝 promise。

**3。x 为对象或函数**

如果 x 为对象或者函数：

- 把 x.then 赋值给 then。
- 如果取 x.then 的值时抛出错误 e ，则以 e 为据因拒绝 promise。
- 如果 then 是函数，将 x 作为函数的作用域 this 调用之。传递两个回调函数作为参数，第一个参数叫做 resolvePromise ，第二个参数叫做 rejectPromise:
  - 如果 resolvePromise 以值 y 为参数被调用，则运行 [[Resolve]](promise, y)
  - 如果 rejectPromise 以据因 r 为参数被调用，则以据因 r 拒绝 promise
  - 如果 resolvePromise 和 rejectPromise 均被调用，或者被同一参数调用了多次，则优先采用首次调用并忽略剩下的调用
  - 如果调用 then 方法抛出了异常 e：
    - 如果 resolvePromise 或 rejectPromise 已经被调用，则忽略之
    - 否则以 e 为据因拒绝 promise
  - 如果 then 不是函数，以 x 为参数执行 promise
- 如果 x 不为对象或者函数，以 x 为参数执行 promise

如果一个 promise 被一个循环的 thenable 链中的对象解决，而 [[Resolve]](promise, thenable) 的递归性质又使得其被再次调用，根据上述的算法将会陷入无限递归之中。算法虽不强制要求，但也鼓励施者检测这样的递归是否存在，若检测到存在则以一个可识别的 TypeError 为据因来拒绝 promise 。

1.x 与 promise 相等

```
function resolvePromise(promise2, x, resolve, reject) {
  //x 与 promise 相等 
  //如果从onFulfilled中返回的x 就是promise2 就会导致循环引用报错
  
  //如果 promise 和 x 指向同一对象，以 TypeError 为据因拒绝执行 promise
  if (x === promise2) {
    reject(new TypeError('循环引用'));
  }
}
复制代码
```

2.x 为 Promise。

```
function resolvePromise(promise2, x, resolve, reject) {
  if (x === promise2) {
    reject(new TypeError('循环引用'));
  }
  // x 为 Promise
  else if (x instanceof AjPromise) {
    // 如果 x 为 Promise ，则使 promise 接受 x 的状态
    // 如果 x 处于等待态， promise 需保持为等待态直至 x 被执行或拒绝
    if (x.state === PENDING) {
      x.then(
        y => {
          resolvePromise(promise2, y, resolve, reject);
        },
        reason => {
          reject(reason);
        }
      );
    } else {
      // 如果 x 处于执行态，用相同的值执行 promise
      // 如果 x 处于拒绝态，用相同的据因拒绝 promise
      x.then(resolve, reject);
    }
  }
}
复制代码
```

3.x 为对象或函数

```
function resolvePromise(promise2, x, resolve, reject) {
  if (x === promise2) {
    reject(new TypeError('循环引用'));
  }
  if (x instanceof AjPromise) {
    if (x.state === PENDING) {
      x.then(
        y => {
          resolvePromise(promise2, y, resolve, reject);
        },
        reason => {
          reject(reason);
        }
      );
    } else {
      x.then(resolve, reject);
    }
  } else if (x && (typeof x === 'function' || typeof x === 'object')) {
    // 避免多次调用
    let called = false;
    try {
      //把 x.then 赋值给 then
      let then = x.then;
      if (typeof then === 'function') {
        // 如果 then 是函数，将 x 作为函数的作用域 this 调用之。
        // 传递两个回调函数作为参数，第一个参数叫做 resolvePromise ，第二个参数叫做 rejectPromise
        // 如果 resolvePromise 和 rejectPromise 均被调用，或者被同一参数调用了多次，则优先采用首次调用并忽略剩下的调用
        then.call(
          x,
          // 如果 resolvePromise 以值 y 为参数被调用，则运行[[Resolve]](promise, y)
          y => {
            if (called) return;
            called = true;
            resolvePromise(promise2, y, resolve, reject);
          },
          // 如果 rejectPromise 以据因 r 为参数被调用，则以据因 r 拒绝 promise
          r => {
            if (called) return;
            called = true;
            reject(r);
          }
        );
      }else {
        // 如果 then 不是函数，以 x 为参数执行 promise
        resolve(x);
      }  
    } catch (e) {
      // 如果取 x.then 的值时抛出错误 e ，则以 e 为据因拒绝 promise
      // 如果调用 then 方法抛出了异常 e：
      // 如果 resolvePromise 或 rejectPromise 已经被调用，则忽略之
      // 否则以 e 为据因拒绝 promise
      if (called) return;
      called = true;
      reject(e);
    }
  } else {
    // 如果 x 不为对象或者函数，以 x 为参数执行 promise
    resolve(x);
  }
}

复制代码
```

Ok~比较复杂的解决过程也让我们搞定了.接下来我们整合下代码

### Promises/A+ 规范 完整代码

```
const PENDING = 'pending';
const FULFILLED = 'fulfilled';
const REJECTED = 'rejected';

class AjPromise {
  constructor(fn) {
    this.state = PENDING;
    this.value = null;
    this.reason = null;
    this.onFulfilledCallbacks = [];
    this.onRejectedCallbacks = [];
    const resolve = value => {
      if (value instanceof Promise) {
        return value.then(resolve, reject);
      }
      setTimeout(() => {
        if (this.state === PENDING) {
          this.state = FULFILLED;
          this.value = value;
          this.onFulfilledCallbacks.map(cb => {
            cb = cb(this.value);
          });
        }
      });
    };
    const reject = reason => {
      setTimeout(() => {
        if (this.state === PENDING) {
          this.state = REJECTED;
          this.reason = reason;
          this.onRejectedCallbacks.map(cb => {
            cb = cb(this.reason);
          });
        }
      });
    };
    try {
      fn(resolve, reject);
    } catch (e) {
      reject(e);
    }
  }
  then(onFulfilled, onRejected) {
    let newPromise;

    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value;
    onRejected =
      typeof onRejected === 'function'
        ? onRejected
        : reason => {
            throw reason;
          };
    if (this.state === FULFILLED) {
      return (newPromise = new AjPromise((resolve, reject) => {
        setTimeout(() => {
          try {
            let x = onFulfilled(this.value);
            resolvePromise(newPromise, x, resolve, reject);
          } catch (e) {
            reject(e);
          }
        });
      }));
    }
    if (this.state === REJECTED) {
      return (newPromise = new AjPromise((resolve, reject) => {
        setTimeout(() => {
          try {
            let x = onRejected(this.reason);
            resolvePromise(newPromise, x, resolve, reject);
          } catch (e) {
            reject(e);
          }
        });
      }));
    }
    if (this.state === PENDING) {
      return (newPromise = new AjPromise((resolve, reject) => {
        this.onFulfilledCallbacks.push(value => {
          try {
            let x = onFulfilled(value);
            resolvePromise(newPromise, x, resolve, reject);
          } catch (e) {
            reject(e);
          }
        });
        this.onRejectedCallbacks.push(reason => {
          try {
            let x = onRejected(reason);
            resolvePromise(newPromise, x, resolve, reject);
          } catch (e) {
            reject(e);
          }
        });
      }));
    }
  }
}
function resolvePromise(promise2, x, resolve, reject) {
  if (x === promise2) {
    reject(new TypeError('循环引用'));
  }
  if (x instanceof AjPromise) {
    if (x.state === PENDING) {
      x.then(
        y => {
          resolvePromise(promise2, y, resolve, reject);
        },
        reason => {
          reject(reason);
        }
      );
    } else {
      x.then(resolve, reject);
    }
  } else if (x && (typeof x === 'function' || typeof x === 'object')) {
    let called = false;
    try {
      let then = x.then;
      if (typeof then === 'function') {
        then.call(
          x,
          y => {
            if (called) return;
            called = true;
            resolvePromise(promise2, y, resolve, reject);
          },
          r => {
            if (called) return;
            called = true;
            reject(r);
          }
        );
      } else {
        resolve(x);
      }
    } catch (e) {
      if (called) return;
      called = true;
      reject(e);
    }
  } else {
    resolve(x);
  }
}

AjPromise.deferred = function() {
  let defer = {};
  defer.promise = new AjPromise((resolve, reject) => {
    defer.resolve = resolve;
    defer.reject = reject;
  });
  return defer;
};

module.exports = AjPromise;

复制代码
```

再来看看我们的实现是否符合Promises/A+规范

```
npm run test
复制代码
```

nice,测试用例全部通过！

### 源码地址

[传送门](https://github.com/webfansplz/article/tree/master/ajPromise)

如果觉得有帮助到你,请给个star支持下作者~

### 参考文献

[Promises/A+规范译文](http://www.ituring.com.cn/article/66566)

[Promise详解与实现](https://www.jianshu.com/p/459a856c476f)