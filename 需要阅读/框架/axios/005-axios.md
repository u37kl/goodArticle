# [axios从无从下手到手到擒来(5)-源码分析之默认axios与axios.create()的区别](https://segmentfault.com/a/1190000021949834)

上文说到默认暴露的axios是通过createInstance函数创建的.在开发中,除了使用axios之外,我们还可以使用axios.create()创建新的Axios instance,那么,这两个对象都有啥区别呢?
首先,贴出源码

```
// Create the default instance to be exported
var axios = createInstance(defaults);

// Expose Axios class to allow class inheritance
axios.Axios = Axios;

// Factory for creating new instances
axios.create = function create(instanceConfig) {
  return createInstance(mergeConfig(axios.defaults, instanceConfig));
};

// Expose Cancel & CancelToken
axios.Cancel = require('./cancel/Cancel');
axios.CancelToken = require('./cancel/CancelToken');
axios.isCancel = require('./cancel/isCancel');

// Expose all/spread
axios.all = function all(promises) {
  return Promise.all(promises);
};
axios.spread = require('./helpers/spread');
```

可以确定的是,axios是Axios库调用createInstance()默认帮我创建的一个对象,而instance是我们手动调用create函数创建的Axios对象,create函数中调用的也是createInstance函数,
mergeConfig()的定义:'function mergeConfig(config1, config2) {}',生成一个新的配置对象,如果config1中和config2中的属性冲突,优先config2中的定义
**相同点:**

1. 它俩都是能发送请求的函数,也就是Axios.prototype.request([config])函数
2. 它俩都拥有Axios原型对象上的各种发送特定请求的方法,get/post/put/delete...
3. 它们拥有各自的默认配置和拦截器配置

**不同点:**

1. 默认配置很可能不一样,项目中使用axios.create()创建一个新axios,就是为了区别与默认的axios的不同,如果传入相同的config,是不是显得很多余?
2. 在源码中,create函数定义之后,又对默认的axios添加了很多属性,如Cancel,CancelToken,all等.所以可以肯定的是,create()出来的instance绝对没有这些后添加的属性

###### axios系列传送门

[axios从无从下手到手到擒来(1)-使用XMLHttpRequest封装简单的ajax请求函数](https://segmentfault.com/a/1190000021944305)
[axios从无从下手到手到擒来(2)-axios的理解与使用](https://segmentfault.com/a/1190000021944651)
[axios从无从下手到手到擒来(3)-源码分析之整体结构](https://segmentfault.com/a/1190000021948563)
[axios从无从下手到手到擒来(4)-源码分析之axios与Axios的关系](https://segmentfault.com/a/1190000021949448)
[axios从无从下手到手到擒来(5)-源码分析之默认axios与axios.create()的区别](https://segmentfault.com/a/1190000021949834)
[axios从无从下手到手到擒来(6)-源码分析之axios运行的整体流程](https://segmentfault.com/a/1190000021950095)