目前来看，异步操作的未来非async/await(ES7)莫属。但是大多数项目中，还不能立刻扔掉历史包袱，而且Promise其实也是实现async/await的基础，在ES6中Promise也被写入了规范中，所以深入学习一下Promise还是很有必要的。

首先抛开Promise，**了解一下异步操作的流程**。

假设有一个异步任务的模板，我们使用setTimeout模拟异步，在每个异步任务中先打印下这个参数arg，然后以2*arg作为参数传入回调函数中。下面我们分别以串行、并行方式执行几个异步任务。



```jsx
asyncFn=(arg,cb)=>{
    setTimeout(function(){
        console.log(`参数为${arg}`)
        cb(arg*2)
    },1000);
}
//这是要执行异步任务的参数队列
let items=[1,2,3,4,5,6];
```

*串行任务*:在每个异步任务的回调方法中通过shift()方法，每次从任务队列中取出一个值，并且更新剩余任务的数组，实现任务的接力进行。



```jsx
let results=[];
final=(value)=>{
    console.log(`完成：${value}`);
}

series=(item)=>{
    if (item) {
        asyncFn(item,function(res){
            results.push(res);;
            return series(items.shift());
        })
    }else{
        final(results);
        console.timeEnd('sync');
    }
}
console.time('sync');
series(items.shift());
//串行执行6.10s
```

*并行任务*：一开始就将所有任务都执行，然后监测负责保存异步任务执行结果的数组的长度，若等于任务队列长度时，则是所有异步任务都执行完毕。



```jsx
let len=items.length;

console.time('asyncFn');
items.forEach((item)=>{
    asyncFn(item,function(res){
        results.push(res);
        if (results.length===len) {
            final(results);
            console.timeEnd('asyncFn');
        }
    })
})
//并行执行1.01s
```

*对并行任务的数量进行控制*：增加一个参数记录正在执行的任务的个数，每开始执行一个任务加1，每到回调函数即将结束时减1。



```jsx
//并行与串行的配合，即设定每次最多能并行n个异步任务
let running=0;
let limit=2;
console.time('control');
launcher=()=>{
    while(running < limit && items.length>0){
        let item = items.shift();
        running++;
        asyncFn(item,function(res){
            results.push(res);
            running--;
            if (items.length>0) {
                launcher();
            }else if(running==0){
                final();
                console.timeEnd('control');
            }
        });
    }
}
launcher();
//3.01s
```

#### Promise基础回顾

then方法可以链式调用



```css
(new Promise(fn1))
.then(step1)
.then(step2)
.then(step3)
.then(
    console.log
    console.error
    )
```

错误具有传递性。console.error可以显示之前任一步发生的错误，而且该步之后的任务不会继续执行。但是console.log只能显示step3的返回值。

**新建一个promise对象**



```jsx
var promise=new Promise((resolve,reject){})
```

**实例方法**



```css
promise.then(onFullfilled,onRejected)
```

**静态方法**



```css
Promise.resolve()
Promise.reject()
```

- Promise.resolve

  将传递给他的参数填充到Promise对象并返回这个Promise对象。

  `Promise.resolve(42)`   可以被认为是

  

  ```jsx
   new Promise(function(resolve){
    resolve(42)
  })
  ```

  的语法糖。

  `Promise.resolve()` 方法还能将 `thenable` 对象转换为ES6中定义的promise对象。

  `thenable`  对象就是具有then方法但不是promise对象的对象，比如jQuery.ajax()的返回对象

  即使一个对象具有 .then 方法，也不一定就能作为ES6 Promises对象使用

  

  ```jsx
  var promise=Promise.resolve($.ajax('/json/comment.json'));
  promise.then(function(value){
    console.log(value);
  })
  ```

- Promise.reject

  与Promise.resolve类似的静态方法

  

  ```jsx
  Promise.reject(new Error('err'))；
  ```

  等同于



```jsx
 new Promise(function(resolve,reject){
   reject(new Error('err'))
})
```

**常见应用**：



```jsx
//使用Promise封装一个ajax请求：
function getURL(url){
    return new Promise(function(resolve,reject){
        var req=new XMLHttpRequest();
        req.open('GET',url,true);
        req.onload=function(){
            if (req.status==200) {
                resolve(req.responseText);
            }else{
                reject(new Error(req.statusText))
            }
        };
        req.onerror=function(){
            reject(new Error(req.statusText));
        };
        res.send();
    })
}

//异步加载图片
let preloadImage=(path)=>{
    return new Promise(function(resolve,reject){
        let img=new Image();
        img.onload=resolve;
        img.onerror=reject;
        img.src=path;
    })
}
```

#### *错误捕获*： catch与then

catch方法只是then(undefined,onReject)的封装，实质是一样的。



```jsx
promise.then(undefined,function(err){
  console.error(err);
})
```

- 使用promise.then(onFulfilled, onRejected) 的话onFulFilled中发生错误无法捕获
- 使用.catch链式在then后调用可以捕获then中的错误
- 本质上一样，区别使用场合

##### 错误捕获在IE8的问题

catch是ES3中的保留字，所以在IE8以下不能作为对象的属性名使用，会出现identifier not found错误。

- 点标记法要求对象的属性必须是有效的标识符
- 中括号标记法可以将非法标识符作为对象的属性名使用



```jsx
var promise=Promise.reject(new Error('msg'));
promise["catch"](function(error){
    console.error(error);
})
```

或者使用then方法中添加第二个参数来避免这个问题
 `then(undefined,onReject)`

### Promise的同步异步

Promise只能使用异步调用方式。



```jsx
//Promise在定义时就会调用
var promise=new Promise(function(resolve){
    resolve(2);//异步调用回调函数
    console.log('innner')
})
promise.then(function(value){
    console.log(value);
})
console.log('outer');
```

会依次打印inner,outer,2

- 决不能对异步函数进行同步调用，处理顺序可能会与语气不符，可能带来意料之外的后果
- 还可能导致栈溢出或者异常处理错误等。

Promise保证了每次调用都是异步的，所以在实际编码中不需要使用setTimeout自己实现异步。

##### 有多个Promise实例时：

- promise.all() 所有异步任务并行执行

  接受promise对象组成的数组作为参数。输出的每个promise的结果和参数数组的顺序一致。

- promise.race 有一个异步任务完成则返回结果

  promise.race()同样接受多个promise对象组成的数组作为参数，但是只要有一个promise对象变为fulFilled或者rejected状态，就会继续后面的处理

#### 基于promise.race()实现超时处理



```jsx
function delayPromise(ms) {
    return new Promise(function(resolve) {
        setTimeout(resolve, ms);
    })
}

function timeoutPromise(promise, ms) {
  //用以提供超时基准的promise实例
    var timeout = delayPromise(ms).then(function() {
        throw new Error(`operation timed out after ${ms} ms`);
    })
    return Promise.race([promise, timeout]);
}

//新的task
var taskPromise = new Promise(function(resolve) {
    var delay = Math.random() * 2000;
    setTimeout(function() {
        resolve(`${dealy} ms`);
    }, dealy)
});

timeoutPromise(taskPromise, 1000)
    .then(function(value) {
        console.log(`task在规定时间内结束${value}`)
    })
    .catch(function(err) {
        console.log(`发生超时：${err}`);
    })
```

但是不能区分这个异常是普通错误还是超时错误。需要定制一个error对象。



```jsx
function copyOwnFrom(target,source){
    Object.getOwnPropertyNames(source).forEach(function(propName){
        Object.defineProperty(target,propName,Object.getOwnPropertyDescriptor(source,propName));
    })
    return target
}
//通ECMAScript提供的内建对象Error实现继承
function TimeoutError(){
    var superInstance=Error.apply(null,arguments);
    copyOwnFrom(this,superInstance);
}

TimeoutError.prototype=Object.create(Error.prototype);
TimeoutError.prototype.constructor=TimeoutError;
```

用于提供超时基准的promise实例改为



```jsx
var timeout = delayPromise(ms).then(function() {
        throw new TimeoutError(`operation timed out after ${ms} ms`);
    })
```

在错误捕获中可修改为：



```jsx
timeoutPromise(taskPromise, 1000)
    .then(function(value) {
        console.log(`task在规定时间内结束${value}`)
    })
    .catch(function(err) {
        if(err instanceof TimeoutError){
             console.log(`发生超时：${err}`);
        }else{
             console.log(`错误：${err}`);
        }
    })
```

#### *超时取消XHR请求*



```jsx
//通过cancelableXHR 方法取得包装了XHR的promise对象和取消该XHR请求的方法
//
function cancelableXHR(url){
    var req=new XMLHttpRequest();
    var promise=new Promise(function(resolve,reject){
        req.open('GET',url,true);
        req.onload=function(){
            if (req.status===200) {
                resolve(req.responseText);
            }else{
                reject(new Error(req.statusText))
            }
        }
        req.onerror=function(){
            reject(new Error(req.responseText))
        }
        req.onabort=function(){
            reject(new Error('abort this request'))
        }
        res.send();
    })
    var abort=function(){
        if (req.readyState!==XMLHttpRequest.UNSENT) {
            req.abort();
        }
    }

    return {
        promise:promise,
        abort:abort
    }
}

var object=cancelableXHR('http://www.sqqs.com/index')

timeoutPromise(object.promise,1000).then(function(content){
    console.log(`content:${content}`);
}).catch(function(error){
    if (error instanceof TimeoutError) {
        object.abort();
        return console.log(error)
    }
    console.log(`XHR Error:${error}`);
})
```

### promise 顺序处理sequence

promise.all()是多个promise对象同时执行，没有api直接支持多个任务线性执行。

我们需要在上一个任务执行结果的promise对象的基础上执行下一个promise任务。



```jsx
var promiseA = function() {
    return new Promise(function(resolve) {
        setTimeout(function() {
            resolve(111);
        }, 200)

    })
}

var promiseB = function(args) {
    return new Promise(function(resolve) {
        setTimeout(function() {
            resolve(2222);
            console.timeEnd('sync');
        }, 200);
    })
}
console.time('sync');
var result = Promise.resolve();
[promiseA, promiseB].forEach(function(promise) {
    result = result.then(promise)
})
//print
//sync:408ms
```

通过这个名为result的promise对象来不断更新保存新返回的promise对象，从而实现一种链式调用。

也可以使用reduce重写循环，使得代码更加美观一些：



```jsx
console.time('sync');
tasks=[promiseA, promiseB];
tasks.reduce(function(result,promise){
    return result.then(promise)
},Promise.resolve())
```

其中Promise.resolve()作为reduce方法的初始值赋值给result。

#### promise穿透--永远往then中传递函数

如下例子，在then中传递了一个promise实例



```jsx
Promise.resolve('foo').then(Promise.resolve('bar')).then(function(result){
    console.log(result)
})
```

打印结果为foo,像then 中传递的并非一个函数，实际上会将其解释为then(null)。若想要得到bar,需要将then中传递一个函数



```jsx
Promise.resolve('foo').then(function() {
    return Promise.resolve('bar')
}).then(function(result) {
    console.log(result)
})
//print result:
//bar
```

如果在then中的函数没有对promise对象使用return返回呢，又是什么结果？



```jsx
Promise.resolve('foo').then(function() {
    Promise.resolve('bar')
}).then(function(result) {
    console.log(result)
})
```

会返回一个undefined。

*抛砖引玉，我们再总结一下向then中传递函数的情况*：



```jsx
var doSomething=function(){
    return Promise.resolve('bar')
}
var printResult=function(result){
    console.log(`result:${result}`)
}
//试想一下，以下几个例子输出的结果分别是什么
Promise.resolve('foo').then(function(value){
    return doSomething();
}).then(printResult)

Promise.resolve('foo').then(function(){
    doSomething();
}).then(printResult)

Promise.resolve('foo').then(doSomething()).then(printResult)

Promise.resolve('foo').then(doSomething).then(printResult)
```

#### 【参考资料】

[promise中需要注意的问题](https://link.jianshu.com?t=http://fex.baidu.com/blog/2015/07/we-have-a-problem-with-promises/)

[promise 迷你书](https://link.jianshu.com?t=http://liubin.org/promises-book/)

[js原生promise](https://link.jianshu.com?t=http://www.cnblogs.com/shytong/p/4985014.html)



作者：RichardBillion
链接：https://www.jianshu.com/p/58a6bb46240e
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。