# 你不知道的 Proxy

在 [从观察者模式到响应式的设计原理](https://mp.weixin.qq.com/s/QLGNzK2-HcKWuavg0LQqpg) 这篇文章中，阿宝哥介绍了 [observer-util](https://github.com/nx-js/observer-util) 这个库如何使用 Proxy 来实现响应式。而对于 [vue-next](https://github.com/vuejs/vue-next) 项目中的 `@vue/reactivity` 模块，也是利用 Proxy 来实现响应式。因此，如果你要学习 `@vue/reactivity` 模块的话，就需要先掌握 Proxy。

接下来，阿宝哥将从 6 个方面入手，带你一步一步揭开 Proxy 对象的神秘面纱。阅读完本文，你将了解以下内容：

- 代理的作用；
- Proxy 对象与 Reflect 对象的相关知识；
- Proxy 对象的 6 个使用场景；
- 使用 Proxy 对象时的一些注意事项；
- Proxy 在开源项目中的应用。

### 一、聊一聊代理

在日常工作中，相信挺多小伙伴都用过 Web 调试代理工具，比如 [Fiddler](https://www.telerik.com/fiddler) 或 [Charles](https://www.charlesproxy.com/)。通过使用 Web 调试代理工具，我们可以抓取 HTTP/HTTPS 协议请求，还可以手动修改请求参数和响应结果。不仅如此，在调试线上问题时，利用 Web 调试代理工具，你还可以把线上 **压缩混淆过** 的 JS 文件映射成本地 **未压缩混淆过** 的 JS 文件。

在简单介绍了 Web 调试代理工具的基本功能之后，我们来看一下使用 Web 调试代理工具的 HTTP 请求流程：

![img](media/Untitled 1/ea2ce1c0e3714aada52901d0ade05b7f~tplv-k3u1fbpfcp-zoom-1.image)

通过上图可知，在引入 Web 调试代理工具之后，我们发起的 HTTP 请求都会通过 Web Proxy 进行转发和处理。增加了 Web Proxy 代理层，让我们能够更好地控制 HTTP 请求的流程。对于单页应用程序来说，当从服务器获取数据之后，我们就会读取相应的数据在页面上显示出来：

![img](media/Untitled 1/16e2bbfabf564b4db22e5989f665e8e1~tplv-k3u1fbpfcp-zoom-1.image)

以上流程与浏览器直接从服务器获取数据类似：

![img](media/Untitled 1/67cbbf72ca0d425098c1ec751706cfea~tplv-k3u1fbpfcp-zoom-1.image)

为了能够灵活控制 HTTP 请求的流程，我们增加了的 Web Proxy 层。那么我们能否控制数据对象的读取流程呢？答案是可以的，我们可以利用 Web API，比如 [Object.defineProperty](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty) 或 [Proxy](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy) API。在引入 Web API 之后，数据的访问流程如下图所示：

![img](media/Untitled 1/f178ffbcd6db4963841b889b163cdbea~tplv-k3u1fbpfcp-zoom-1.image)

接下来，阿宝哥将重点介绍 [Proxy](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy) API，它可是 Vue3 实现数据响应式幕后的 “功臣” 哟。对它感兴趣的小伙伴，跟阿宝哥一起学起来吧。

> 关注「全栈修仙之路」阅读阿宝哥原创的 4 本免费电子书（累计下载2.7万+）及 10 篇源码分析系列教程。

### 二、Proxy 对象简介

[Proxy](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy) 对象用于创建一个对象的代理，从而实现基本操作的拦截和自定义（如属性查找、赋值、枚举、函数调用等）。Proxy 的构造函数语法为：

> ```javascript
> const p = new Proxy(target, handler)
> 复制代码
> ```

相关的参数说明如下：

- target：要使用 `Proxy` 包装的目标对象（可以是任何类型的对象，包括原生数组，函数，甚至另一个代理）。
- handler：一个通常以函数作为属性的对象，各属性中的函数分别定义了在执行各种操作时代理 `p` 的行为。

在介绍 Proxy 对象的使用示例前，我们先来了解一下它的兼容性：

![img](media/Untitled 1/d27c32e90588448f8cd5974ac47f21d4~tplv-k3u1fbpfcp-zoom-1.image)

（图片来源：[caniuse.com/?search=Pro…](https://caniuse.com/?search=Proxy）)

由上图可知，Proxy API 的兼容性并不是很好，所以大家在使用的时候要注意其兼容性问题。

#### 2.1 Proxy 对象使用示例

了解完 Proxy 构造函数，我们来看一个简单的例子：

```javascript
const man = {
  name: "阿宝哥",
};

const proxy = new Proxy(man, {
  get(target, property, receiver) {
    console.log(`正在访问${property}属性`);
    return target[property];
  },
});

console.log(proxy.name);
console.log(proxy.age);
复制代码
```

在以上示例中，我们使用了 Proxy 构造函数为 `man` 对象，创建了一个代理对象。**在创建代理对象时，我们定义了一个 get 捕获器，用于捕获属性读取的操作。** 捕获器的作用就是用于拦截用户对目标对象的相关操作，在这些操作传播到目标对象之前，会先调用对应的捕获器函数，从而拦截并修改相应的行为。

在设置了 get 捕获器之后，当成功运行以上的示例代码，控制台会输出以下结果：

```shell
正在访问name属性
阿宝哥
正在访问age属性
undefined
复制代码
```

通过观察以上输出结果，我们可以发现 **get 捕获器** 不仅可以拦截已知属性的读取操作，也可以拦截未知属性的读取操作。在创建 Proxy 对象时，除了定义 **get 捕获器** 之外，我们还可以定义其他的捕获器，比如 has、set、delete、apply 或 ownKeys 等。

#### 2.2 handler 对象支持的捕获器

handler 对象支持 13 种捕获器，这里阿宝哥只列举以下 5 种常用的捕获器：

- handler.get()：属性读取操作的捕获器。
- handler.set()：属性设置操作的捕获器。
- handler.deleteProperty()：delete 操作符的捕获器。
- handler.ownKeys()：Object.getOwnPropertyNames 方法和 Object.getOwnPropertySymbols 方法的捕获器。
- handler.has()：in 操作符的捕获器。

**需要注意的是，所有的捕获器是可选的。如果没有定义某个捕获器，那么就会保留源对象的默认行为。** 看完上面的捕获器介绍，是不是觉得 Proxy 对象很强大。

### 三、Reflect 对象简介

[Reflect](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Reflect) 是一个内置的对象，它提供拦截 JavaScript 操作的方法。这些方法与 proxy handlers 的方法相同。Reflect 不是一个函数对象，因此它是不可构造的。

在介绍 Reflect 对象的使用示例前，我们先来了解一下它的兼容性：

![img](media/Untitled 1/83f213dd8710424eb375ec185e942a65~tplv-k3u1fbpfcp-zoom-1.image)

（图片来源：[caniuse.com/?search=Ref…](https://caniuse.com/?search=Reflect）)

#### 3.1 Reflect 对象使用示例

```javascript
const man = {
  name: "阿宝哥",
  city: "Xiamen",
};

console.log(Reflect.set(man, "sex", 1)); // true
console.log(Reflect.has(man, "name")); // true
console.log(Reflect.has(man, "age")); // false
console.log(Reflect.ownKeys(man)); // [ 'name', 'city', 'sex' ]
复制代码
```

除了示例中介绍的 `set`、`has` 和 `ownKeys` 方法之外，Reflect 对象还支持 `get`、`defineProperty` 和 `deleteProperty` 等方法。下面阿宝哥将简单介绍 `Reflect` 对象所支持的一些静态方法。

#### 3.2 Reflect 对象支持的静态方法

Reflect 的所有属性和方法都是静态的，该对象提供了与 Proxy handler 对象相关的 13 个方法。同样，这里阿宝哥只列举以下 5 个常用的方法：

- Reflect.get(target, propertyKey[, receiver])：获取对象身上某个属性的值，类似于 target[name]。
- Reflect.set(target, propertyKey, value[, receiver])：将值赋值给属性的函数。返回一个布尔值，如果更新成功，则返回 true。
- Reflect.deleteProperty(target, propertyKey)：删除 target 对象的指定属性，相当于执行 delete target[name]。
- Reflect.has(target, propertyKey)：判断一个对象是否存在某个属性，和 in 运算符的功能完全相同。
- Reflect.ownKeys(target)：返回一个包含所有自身属性（不包含继承属性）的数组。

在实际的 Proxy 使用场景中，我们往往会结合 Reflect 对象提供的静态方法来实现某些特定的功能。为了让大家能够更好地理解并掌握 Proxy 对象，接下来的环节，阿宝哥将列举 Proxy 对象的 **6** 个使用场景。

### 四、Proxy 使用场景

这里我们先来介绍 Proxy 对象的第一个使用场景 —— **增强型数组**。

#### 4.1 增强型数组

##### 定义 enhancedArray 函数

```javascript
function enhancedArray(arr) {
  return new Proxy(arr, {
    get(target, property, receiver) {
      const range = getRange(property);
      const indices = range ? range : getIndices(property);
      const values = indices.map(function (index) {
        const key = index < 0 ? String(target.length + index) : index;
        return Reflect.get(target, key, receiver);
      });
      return values.length === 1 ? values[0] : values;
    },
  });

  function getRange(str) {
    var [start, end] = str.split(":").map(Number);
    if (typeof end === "undefined") return false;

    let range = [];
    for (let i = start; i < end; i++) {
      range = range.concat(i);
    }
    return range;
  }

  function getIndices(str) {
    return str.split(",").map(Number);
  }
}
复制代码
```

##### 使用 enhancedArray 函数

```javascript
const arr = enhancedArray([1, 2, 3, 4, 5]);

console.log(arr[-1]); //=> 5
console.log(arr[[2, 4]]); //=> [ 3, 5 ]
console.log(arr[[2, -2, 1]]); //=> [ 3, 4, 2 ]
console.log(arr["2:4"]); //=> [ 3, 4]
console.log(arr["-2:3"]); //=> [ 4, 5, 1, 2, 3 ]
复制代码
```

由以上的输出结果可知，增强后的数组对象，就可以支持负数索引、分片索引等功能。除了可以增强数组之外，我们也可以使用 Proxy API 来增强普通对象。

#### 4.2 增强型对象

##### 创建 enhancedObject 函数

```javascript
const enhancedObject = (target) =>
  new Proxy(target, {
    get(target, property) {
      if (property in target) {
        return target[property];
      } else {
        return searchFor(property, target);
      }
    },
  });

let value = null;
function searchFor(property, target) {
  for (const key of Object.keys(target)) {
    if (typeof target[key] === "object") {
      searchFor(property, target[key]);
    } else if (typeof target[property] !== "undefined") {
      value = target[property];
      break;
    }
  }
  return value;
}
复制代码
```

##### 使用 enhancedObject 函数

```javascript
const data = enhancedObject({
  user: {
    name: "阿宝哥",
    settings: {
      theme: "dark",
    },
  },
});

console.log(data.user.settings.theme); // dark
console.log(data.theme); // dark
复制代码
```

以上代码运行后，控制台会输出以下代码：

```shell
dark
dark
复制代码
```

通过观察以上的输出结果可知，使用 `enhancedObject` 函数处理过的对象，我们就可以方便地访问普通对象内部的深层属性。

#### 4.3 创建只读的对象

##### 创建 Proxy 对象

```javascript
const man = {
  name: "semlinker",
};

const handler = {
  set: "Read-Only",
  defineProperty: "Read-Only",
  deleteProperty: "Read-Only",
  preventExtensions: "Read-Only",
  setPrototypeOf: "Read-Only",
};

const proxy = new Proxy(man, handler);
复制代码
```

##### 使用 proxy 对象

```javascript
console.log(proxy.name);
proxy.name = "kakuqo";
复制代码
```

以上代码运行后，控制台会输出以下代码：

```javascript
semlinker
proxy.name = "kakuqo";
           ^
TypeError: 'Read-Only' returned for property 'set' of object '#<Object>' is not a function
复制代码
```

观察以上的异常信息可知，导致异常的原因是因为 `handler` 对象的 `set` 属性值不是一个函数。如果不希望抛出运行时异常，我们可以定义一个 `freeze` 函数：

```javascript
function freeze (obj) {
  return new Proxy(obj, {
    set () { return true; },
    deleteProperty () { return false; },
    defineProperty () { return true; },
    setPrototypeOf () { return true; }
  });
}
复制代码
```

定义好 `freeze` 函数，我们使用数组对象来测试一下它的功能：

```javascript
let frozen = freeze([1, 2, 3]);
frozen[0] = 6;
delete frozen[0];
frozen = Object.defineProperty(frozen, 0, { value: 66 });
console.log(frozen); // [ 1, 2, 3 ]
复制代码
```

上述代码成功执行后，控制台会输出 `[ 1, 2, 3 ]`，很明显经过 `freeze` 函数处理过的数组对象，已经被 “冻结” 了。

#### 4.4 拦截方法调用

##### 定义 traceMethodCalls 函数

```javascript
function traceMethodCalls(obj) {
  const handler = {
    get(target, propKey, receiver) {
      const origMethod = target[propKey]; // 获取原始方法
      return function (...args) {
        const result = origMethod.apply(this, args);
        console.log(
          propKey + JSON.stringify(args) + " -> " + JSON.stringify(result)
        );
        return result;
      };
    },
  };
  return new Proxy(obj, handler);
}
复制代码
```

##### 使用 traceMethodCalls 函数

```javascript
const obj = {
  multiply(x, y) {
    return x * y;
  },
};

const tracedObj = traceMethodCalls(obj);
tracedObj.multiply(2, 5); // multiply[2,5] -> 10
复制代码
```

上述代码成功执行后，控制台会输出 `multiply[2,5] -> 10`，即我们能够成功跟踪 `obj` 对象中方法的调用过程。其实，除了能够跟踪方法的调用，我们也可以跟踪对象中属性的访问，具体示例如下：

```javascript
function tracePropAccess(obj, propKeys) {
  const propKeySet = new Set(propKeys);
  return new Proxy(obj, {
    get(target, propKey, receiver) {
      if (propKeySet.has(propKey)) {
        console.log("GET " + propKey);
      }
      return Reflect.get(target, propKey, receiver);
    },
    set(target, propKey, value, receiver) {
      if (propKeySet.has(propKey)) {
        console.log("SET " + propKey + "=" + value);
      }
      return Reflect.set(target, propKey, value, receiver);
    },
  });
}

const man = {
  name: "semlinker",
};
const tracedMan = tracePropAccess(man, ["name"]);

console.log(tracedMan.name); // GET name; semlinker
console.log(tracedMan.age); // undefined
tracedMan.name = "kakuqo"; // SET name=kakuqo
复制代码
```

在以上示例中，我们定义了一个 `tracePropAccess` 函数，该函数接收两个参数：obj 和 propKeys，它们分别表示需跟踪的目标和需跟踪的属性列表。调用 `tracePropAccess` 函数后，会返回一个代理对象，当我们访问被跟踪的属性时，控制台就会输出相应的访问日志。

#### 4.5 隐藏属性

##### 创建 hideProperty 函数

```javascript
const hideProperty = (target, prefix = "_") =>
  new Proxy(target, {
    has: (obj, prop) => !prop.startsWith(prefix) && prop in obj,
    ownKeys: (obj) =>
      Reflect.ownKeys(obj).filter(
        (prop) => typeof prop !== "string" || !prop.startsWith(prefix)
      ),
    get: (obj, prop, rec) => (prop in rec ? obj[prop] : undefined),
  });
复制代码
```

##### 使用 hideProperty 函数

```javascript
const man = hideProperty({
  name: "阿宝哥",
  _pwd: "www.semlinker.com",
});

console.log(man._pwd); // undefined
console.log('_pwd' in man); // false
console.log(Object.keys(man)); // [ 'name' ]
复制代码
```

通过观察以上的输出结果，我们可以知道，利用 Proxy API，我们实现了指定前缀属性的隐藏。除了能实现隐藏属性之外，利用 Proxy API，我们还可以实现验证属性值的功能。

#### 4.6 验证属性值

##### 创建 validatedUser 函数

```javascript
const validatedUser = (target) =>
  new Proxy(target, {
    set(target, property, value) {
      switch (property) {
        case "email":
          const regex = /^(([^<>()\[\]\\.,;:\s@"]+(\.[^<>()\[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/;
          if (!regex.test(value)) {
            console.error("The user must have a valid email");
            return false;
          }
          break;
        case "age":
          if (value < 20 || value > 80) {
            console.error("A user's age must be between 20 and 80");
            return false;
          }
          break;
      }

      return Reflect.set(...arguments);
    },
  });
复制代码
```

##### 使用 validatedUser 函数

```javascript
let user = {
  email: "",
  age: 0,
};

user = validatedUser(user);
user.email = "semlinker.com"; // The user must have a valid email
user.age = 100; // A user's age must be between 20 and 80
复制代码
```

上述代码成功执行后，控制台会输出以下结果：

```shell
The user must have a valid email
A user's age must be between 20 and 80
复制代码
```

介绍完 Proxy 对象的使用场景之后，我们来继续介绍与 Proxy 对象相关的一些问题。

### 五、Proxy 相关问题

#### 5.1 this 的指向问题

```javascript
const target = {
  foo() {
    return {
      thisIsTarget: this === target,
      thisIsProxy: this === proxy,
    };
  },
};

const handler = {};
const proxy = new Proxy(target, handler);
console.log(target.foo()); // { thisIsTarget: true, thisIsProxy: false }
console.log(proxy.foo()); // { thisIsTarget: false, thisIsProxy: true }
复制代码
```

上述代码成功执行后，控制台会输出以下结果：

```shell
{ thisIsTarget: true, thisIsProxy: false }
{ thisIsTarget: false, thisIsProxy: true }
复制代码
```

通过以上输出的结果，`foo` 方法中的 this 指向与当前的调用者有关。看起来挺简单的，但在一些场景下如果稍不注意的话，就会出现问题，比如以下这个示例：

```javascript
const _name = new WeakMap();

class Person {
  constructor(name) {
    _name.set(this, name);
  }
  
  get name() {
    return _name.get(this);
  }
}
复制代码
```

在以上示例中，我们使用 `WeakMap` 对象来存储 Person 对象的私有信息。定义完 `Person` 类，我们就可以通过以下方式来使用它：

```javascript
const man = new Person("阿宝哥");
console.log(man.name); // 阿宝哥

const proxy = new Proxy(man, {});
console.log(proxy.name); // undefined
复制代码
```

对于以上的代码，当我们通过 `proxy` 对象来访问 `name` 属性时，你会发现输出的结果是 `undefined`。这是因为当使用 `proxy.name` 的方式访问 `name` 属性时，`this` 指向的是 `proxy` 对象，而 `_name` WeakMap 对象中存储的是 `man` 对象，所以输出的结果是 `undefined`。

然而，对于以上的问题，如果我们按照以下方式定义 `Person` 类，就不会出现以上问题：

```javascript
class Person {
  constructor(name) {
    this._name = name;
  }
  get name() {
    return this._name;
  }
}

const man = new Person("阿宝哥");
console.log(man.name); // 阿宝哥

const proxy = new Proxy(man, {});
console.log(proxy.name); // 阿宝哥
复制代码
```

另外，如果你对 [WeakMap](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/WeakMap) 感兴趣的话，可以阅读 [你不知道的 WeakMap](https://mp.weixin.qq.com/s/9Mscr5f4V5lXtAqdGorkmA) 这篇文章。

#### 5.2 get 捕获器 receiver 参数是什么

```javascript
const p = new Proxy(target, {
  get: function(target, property, receiver) {
    // receiver
  }
});
复制代码
```

get 捕获器用于拦截对象的读取属性操作，该捕获器含有三个参数：

- target：目标对象。
- property：被读取的属性名。
- receiver：指向当前的 Proxy 对象或者继承于当前 Proxy 的对象。

为了更好地了解 `receiver` 参数的描述信息，我们来举个具体的示例：

```javascript
const proxy = new Proxy({},
  {
    get: function (target, property, receiver) {
      return receiver;
    },
  }
);

console.dir(proxy.getReceiver === proxy); // true
var inherits = Object.create(proxy);
console.dir(inherits.getReceiver === inherits); // true
复制代码
```

那么我们能否改变 `receiver` 指向的对象呢？答案是可以的，通过 Reflect 对象提供的 `get` 方法，我们可以动态设置 `receiver` 对象的值，具体使用方式如下所示：

```javascript
console.dir(Reflect.get(proxy, "getReceiver", "阿宝哥"));
复制代码
```

其实 `receiver` 的名称是来源于 ECMAScript 规范：

- [[Get]] (propertyKey, Receiver) → any

  Return the value of the property whose key is propertyKey from this object. If any ECMAScript code must be executed to retrieve the property value, *Receiver* is used as the **this** value when evaluating the code.

- [[Set]]   (propertyKey, value, Receiver) → Boolean

  Set the value of the property whose key is propertyKey to value. If any ECMAScript code must be executed to set the property value, *Receiver* is used as the **this** value when evaluating the code. Returns **true** if the property value was set or **false** if it could not be set.

以上的 `[[Get]]` 和 `[[Set]] `被称为内部方法，ECMAScript 引擎中的每个对象都与一组内部方法相关联，这些内部方法定义了其运行时行为。需要注意的是，这些内部方法不是 ECMAScript 语言的一部分。对于对象的访问器属性来说，在执行内部代码时，`Receiver` 将被作为 `this` 的值，同样使用 Reflect 对象提供的 API，我们也可以通过设置 `receiver` 参数的值来改变 `this` 的值：

```javascript
const obj = {
  get foo() {
    return this.bar;
  },
};

console.log(Reflect.get(obj, "foo")); // undefined
console.log(Reflect.get(obj, "foo", { bar: 2021 })); // 2021
复制代码
```

#### 5.3 包装内置构造函数的实例

当使用 Proxy 包装内置构造函数实例的时候，可能会出现一些问题。比如使用 Proxy 代理 Date 构造函数的实例：

```javascript
const target = new Date();
const handler = {};
const proxy = new Proxy(target, handler);

proxy.getDate(); // Error
复制代码
```

当以上代码运行后，控制台会输出以下异常信息：

```shell
proxy.getDate();
      ^
TypeError: this is not a Date object.
复制代码
```

出现以上问题的原因是因为有些原生对象的内部属性，只有通过正确的 `this` 才能拿到，所以 Proxy 无法代理这些原生对象的属性。那么如何解决这个问题呢？要解决这个问题，我们可以为 `getDate` 方法绑定正确的 `this`：

```javascript
const target = new Date();
const handler = {
  get(target, property, receiver) {
    if (property === "getDate") {
      return target.getDate.bind(target);
    }
    return Reflect.get(target, property, receiver);
  },
};

const proxy = new Proxy(target, handler);
console.log(proxy.getDate());
复制代码
```

#### 5.4 创建可撤销的代理对象

通过 `Proxy.revocable()` 方法可以用来创建一个可撤销的代理对象，该方法的签名为：

> ```
> Proxy.revocable(target, handler);
> 复制代码
> ```

相关的参数说明如下：

- target：将用 `Proxy` 封装的目标对象。可以是任何类型的对象，包括原生数组，函数，甚至可以是另外一个代理对象。
- handler：一个对象，其属性是一批可选的函数，这些函数定义了对应的操作被执行时代理的行为。

调用 `Proxy.revocable` 方法之后，其返回值是一个对象，其结构为： `{"proxy": proxy, "revoke": revoke}`，其中：

- proxy：表示新生成的代理对象本身，和用一般方式 `new Proxy(target, handler)` 创建的代理对象没什么不同，只是它可以被撤销掉。
- revoke：撤销方法，调用的时候不需要加任何参数，就可以撤销掉和它一起生成的那个代理对象。

了解完 `revocable` 方法之后，我们来举一个具体的示例：

```javascript
const target = {}; 
const handler = {};
const { proxy, revoke } = Proxy.revocable(target, handler);

proxy.name = "阿宝哥";
console.log(proxy.name); // 阿宝哥

revoke();
console.log(proxy.name); // TypeError: Revoked
复制代码
```

当以上代码成功运行之后，控制台会输出以下内容：

```shell
阿宝哥
Uncaught TypeError: Cannot perform 'get' on a proxy that has been revoked
  at <anonymous>
复制代码
```

通过观察以上的结果，我们可知当 `proxy` 对象被撤销之后，我们就没有办法对已撤销的 `proxy` 对象执行任何操作。

### 六、Proxy 在开源项目中的应用

因为 Proxy 对象能够提供强大的拦截能力，所以它被应用在一些成熟的开源项目中，用于实现响应式的功能，比如 [vue-next](https://github.com/vuejs/vue-next) 和 [observer-util](https://github.com/nx-js/observer-util) 项目。对于 [observer-util](https://github.com/nx-js/observer-util) 这个项目，阿宝哥已经写了一篇 [从观察者模式到响应式的设计原理](https://mp.weixin.qq.com/s/QLGNzK2-HcKWuavg0LQqpg) 的文章来介绍该项目，感兴趣的小伙伴可以自行阅读。

而对于 [vue-next](https://github.com/vuejs/vue-next) 项目来说，响应式的功能被封装在 `@vue/reactivity` 模块中，该模块为我们提供了一个 `reactive` 函数来创建响应式对象。下面我们来简单了解一下 `reactive` 函数的实现：

```typescript
// packages/reactivity/src/reactive.ts
export function reactive(target: object) {
  if (target && (target as Target)[ReactiveFlags.IS_READONLY]) {
    return target
  }
  return createReactiveObject(
    target,
    false,
    mutableHandlers,
    mutableCollectionHandlers
  )
}
复制代码
```

在 `reactive` 函数内部，会继续调用 `createReactiveObject` 函数来创建响应式对象，该函数也是被定义在 `reactive.ts` 文件中，该函数的的具体实现如下：

```typescript
// packages/reactivity/src/reactive.ts
function createReactiveObject(
  target: Target,
  isReadonly: boolean,
  baseHandlers: ProxyHandler<any>,
  collectionHandlers: ProxyHandler<any>
) {
  // 省略部分代码  
  const proxyMap = isReadonly ? readonlyMap : reactiveMap
  const existingProxy = proxyMap.get(target)
  if (existingProxy) {
    return existingProxy
  }
  const proxy = new Proxy(
    target,
    targetType === TargetType.COLLECTION ? collectionHandlers : baseHandlers
  )
  proxyMap.set(target, proxy)
  return proxy
}
复制代码
```

在 `createReactiveObject` 函数内部，我们终于见到了期待已久的 `Proxy` 对象。当 `target` 对象不是集合类型的对象，比如 Map、Set、WeakMap 和 WeakSet 时，在创建 Proxy 对象时，使用的是 `baseHandlers`，该 `handler` 对象定义了以下 5 种捕获器：

```typescript
export const mutableHandlers: ProxyHandler<object> = {
  get,
  set,
  deleteProperty,
  has,
  ownKeys
}
复制代码
```

其中 `get` 和 `set` 捕获器是分别用于收集 effect 函数和触发 effect 函数的执行。好了，这里阿宝哥只是介绍一下 `@vue/reactivity` 中的 `reactive` 函数，关于该模块是如何实现响应式的细节，这里就不展开介绍了，阿宝哥后续会单独写一篇文章来详细分析该模块的功能。

> 关注「全栈修仙之路」阅读阿宝哥原创的 4 本免费电子书（累计下载 2.7万+）及 50 几篇 “重学TS” 教程。

### 七、参考资源

- [MDN - Proxy](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy)
- [MDN - Reflect](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Reflect)
- [exploringjs - proxies](https://exploringjs.com/es6/ch_proxies.html)
- [stackoverflow - what-is-a-receiver-in-javascript](https://stackoverflow.com/questions/37563495/what-is-a-receiver-in-javascript)