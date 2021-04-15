# [axios从无从下手到手到擒来(4)-源码分析之axios与Axios的关系](https://segmentfault.com/a/1190000021949448)

关于axios和Axios的关系,之前在没有看源码的时候,我想当然的认为axios肯定是Axios的实例对象啦...直到我看了源码,稀碎稀碎的.
**首先贴出createInstance函数的定义**

```
function createInstance(defaultConfig) {
  var context = new Axios(defaultConfig);
  
  var instance = bind(Axios.prototype.request, context);

  utils.extend(instance, Axios.prototype, context); 

  utils.extend(instance, context);

  return instance;
}
```

**下面进行逐行解释**

```
1.第一行
```

new了一个Axios的实例,起名叫做context,但是最后renturn的却不是它,
所以我们先下一个定论: **axios肯定不是Axios的实例对象**

```
2.第二行调用bind函数返回一个instance变量,
```

bind函数的定义如下

```
module.exports = function bind(fn, thisArg) {
  return function wrap() {
    var args = new Array(arguments.length);
    for (var i = 0; i < args.length; i++) {
      args[i] = arguments[i];
    }
    return fn.apply(thisArg, args);
  };
};
```

结合第二行代码来说,instance是bind返回的函数,调用instance函数,就是调用Axios的原型方法request,并且request方法的this指向为刚刚new出来的context对象.
那Axios.prototype.request是个啥呢? 让我们稍微看一下Axios的内部实现

```
Axios.prototype.request = function request(config) {
  //config是string的话,默认config为请求url,结合下面,就是默认get请求
  if (typeof config === 'string') {
    config = arguments[1] || {};
    config.url = arguments[0];
  } else {
    config = config || {};
  }

  // 合并配置
  config = mergeConfig(this.defaults, config);
  // 添加method配置, 默认为get
  config.method = config.method ? config.method.toLowerCase() : 'get';

  /*
  chain的字面意思是链,axios的请求并非是初始化xhr,然后发送ajax请求,而是需要一个流程,chain就是流程链
  dispatchRequest封装的是请求函数,在发送请求前对诸如url,requestHeader等做了一些处理,
  然后再通过adapter判断是用http还是xhr发送请求,
  如果adapter执行成功,调用adapter.then下的onResolved方法,解析response数据并返回,
  如果执行失败,调用adapter.then下的onRejected方法,判断error类型是不是Cancel,再做相应处理
  那为什么数组中的第二个元素是undefined呢? 别方,往下看~~
  */
  var chain = [dispatchRequest, undefined];
  //创建一个成功的promise,并且把config当做value向后传递
  var promise = Promise.resolve(config);

  // 后添加的请求拦截器保存在数组的前面
  this.interceptors.request.forEach(function unshiftRequestInterceptors(interceptor) {
    //注意这里是unshift方法
    chain.unshift(interceptor.fulfilled, interceptor.rejected);
  });
  // 后添加的响应拦截器保存在数组的后面
  this.interceptors.response.forEach(function pushResponseInterceptors(interceptor) {
    chain.push(interceptor.fulfilled, interceptor.rejected);
  });
  /*所以如果你有多个请求/响应拦截器,经过unshift和push的处理之后,chain就变成了这样
  [请求拦截器n,...,请求拦截器2,请求拦截器1,ajax请求,响应拦截器1,响应拦截器2,...响应拦截器n]
  所以就解释了请求拦截器,先添加后执行,响应拦截器,先添加先执行
  */

  // 通过promise的then()串连起所有的请求拦截器/请求方法/响应拦截器
  //这里都是一对一对的执行,对应(onResolved,onRejected),
  //所以chain定义时的第二个元素为undefined,不然你说放个啥?
  while (chain.length) {
    promise = promise.then(chain.shift(), chain.shift());
  }

  // 返回用来指定我们的onResolved和onRejected的promise
  return promise;
```

**综上,**Axios.prototype.request是Axios库核心的发送请求的函数,所以bind函数返回的instance就是一个请求函数,调用axios(config)就是调用Axios.prototype.request(config).
你又会问,为什么要搞一个非Axios实例的instance,我用context.request(config)一样也可以发送请求.真是一个好问题!!翻来覆去看源码就是想不出答案,我觉得吧,就是一个字:**懒**,就是为了让各位看官写代码时,少写几个字母...

```
3. 第三行和第四行,调用extend函数.
```

extend函数的源码如下

```
function extend(a, b, thisArg) {
  forEach(b, function assignValue(val, key) {
    if (thisArg && typeof val === 'function') {
      a[key] = bind(val, thisArg);
    } else {
      a[key] = val;
    }
  });
  return a;
}
```

内部代码就不解释了,嵌套层级太多,引用函数注释'*Extends object a by mutably adding to it the properties of object b*.'翻译过来就是'*向对象a添加对象b上的属性来扩展对象a*'
所以,第三行和第四行就是将Axios的原型方法和对象属性都扩展到instance上,那来看一看Axios有哪些原型方法和对象属性:

```
// 用来得到带query参数的url
Axios.prototype.getUri = function getUri(config) {
  config = mergeConfig(this.defaults, config);
  return buildURL(config.url, config.params, config.paramsSerializer).replace(/^\?/, '');
};

// delete, get, head, options方法
utils.forEach(['delete', 'get', 'head', 'options'], function forEachMethodNoData(method) {
  /*eslint func-names:0*/
  Axios.prototype[method] = function(url, config) {
    return this.request(utils.merge(config || {}, {
      method: method,
      url: url
    }));
  };
});

// post, put, patch方法
utils.forEach(['post', 'put', 'patch'], function forEachMethodWithData(method) {
  /*eslint func-names:0*/
  Axios.prototype[method] = function(url, data, config) {
    return this.request(utils.merge(config || {}, {
      method: method,
      url: url,
      data: data
    }));
  };
});
还有前面的request方法

Axios的构造函数
// 将指定的config, 保存为defaults属性
this.defaults = instanceConfig;
// 将包含请求/响应拦截器管理器的对象保存为interceptors属性
this.interceptors = {
request: new InterceptorManager(),
response: new InterceptorManager()
};
```

也就是说,经过这四行代码,instance既是函数也是对象,当做函数直接调用时等同于调用原型对象的request函数,当做对象时,它又拥有实例对象的全部属性和方法.
**所以我们可以下一个定论:从语法上来说,axios不是Axios的实例,但是从功能上来说,axios是Axios的实例,axios()调用的是Axios.prototype.request()函数bind()返回的函数**

###### axios系列传送门

[axios从无从下手到手到擒来(1)-使用XMLHttpRequest封装简单的ajax请求函数](https://segmentfault.com/a/1190000021944305)
[axios从无从下手到手到擒来(2)-axios的理解与使用](https://segmentfault.com/a/1190000021944651)
[axios从无从下手到手到擒来(3)-源码分析之整体结构](https://segmentfault.com/a/1190000021948563)
[axios从无从下手到手到擒来(4)-源码分析之axios与Axios的关系](https://segmentfault.com/a/1190000021949448)
[axios从无从下手到手到擒来(5)-源码分析之默认axios与axios.create()的区别](https://segmentfault.com/a/1190000021949834)
[axios从无从下手到手到擒来(6)-源码分析之axios运行的整体流程](https://segmentfault.com/a/1190000021950095)