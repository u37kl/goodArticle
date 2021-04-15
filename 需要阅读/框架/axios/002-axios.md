# [axios从无从下手到手到擒来(2)-axios的理解与使用](https://segmentfault.com/a/1190000021944651)

```
本文基本不涉及代码(除了API), 各位看官可以在马桶上当做催翔文来看. 
作为目前最流行的ajax请求库, 并且vue/react官方都推荐使用,它到底是个啥,我们来捋一捋,如果有些地方看不太明白,我将会从第三篇文章中对着源码为各位看官解释.
```

[axios的github地址](https://github.com/axios/axios)

------

```
1. axios的特点
  1). 基于promise的异步请求ajax库
  2). 浏览器端/node端都可以使用
  3). 支持请求/相应拦截
  4). 支持请求取消
  5). 请求/响应数据转换
  6). 批量发送多个请求
  
```

------

```
2. axios的常用语法
   axios(config) 通用/最常用/最本质的发送任意类型的请求的调用方式
   axios(url[,config]) 只指定url,发送get请求
   axios.request(config) 等同于axios(config)
   axios.get(url[,config]) 发get请求
   axios.post(url[,data,config]) 发送post请求
   axios.put(url[,data,config]) 发送put请求
   axios.delete(url[,config]) 发delete请求
   
   axios.defaults.xxx 请求的默认全局配置
   axios.interceptors.request.use() 添加请求拦截器
   axios.interceptors.response.user() 添加响应拦截器
   
   axios.create([config]) 创建一个新的axios(通过create创建的axios没有以下功能)
   
   axios.Cancel() 创建取消请求的错误对象
   axios.CancelToken() 创建取消请求的token对象
   axios.isCancel() 是否是一个取消请求的错误
   axios.all(promises) 批量执行多个请求
   axios.spread() 指定接收所有成功数据的回调函数的方法
```

------

##### 难点理解

```
1. axios.create([config])
1). 根据指定配置创建一个新的axios,也就是每个axios都有自己的配置(如果没有自己的配置,你还创建他干啥)       
2). 新axios只是没有取消请求和批量发请求的方法,其他语法都是一样的
3). 为什么要设计这么个语法?
    假设你的项目很大,后端有多个host,使用一个axios显然不能很好的解决这个问题.
    所以,你需要针对每一个axios对象设置一个baseURL和其他配置属性,也就是使用create语法创建多个不同配置的axios

2. 拦截器函数/ajax请求/请求回调函数的调用顺序
1). 调用axios()并不是马上发送ajax请求,需要经历一系列流程
2). 假如你添加了两个请求拦截器和响应拦截器
    流程: 请求拦截器2 => 请求拦截器1 => ajax请求 => 响应拦截器1 => 响应拦截器2
    为啥请求拦截器先添加后执行,而响应拦截器却先添加先执行?这个问题后面看到源码便一目了然,莫方~
3). 此流程是通过promise串联的,请求拦截器传递的是配置config,响应拦截器传递的是response

3. 取消请求的流程
1). 配置cancelToken对象
2). 缓存用于取消请求的cancel函数
3). 在特定的时机下调用cancel函数
4). 在error回调中判断,如果error是cancel,做相应处理
  
```

###### axios系列传送门

[axios从无从下手到手到擒来(1)-使用XMLHttpRequest封装简单的ajax请求函数](https://segmentfault.com/a/1190000021944305)
[axios从无从下手到手到擒来(2)-axios的理解与使用](https://segmentfault.com/a/1190000021944651)
[axios从无从下手到手到擒来(3)-源码分析之整体结构](https://segmentfault.com/a/1190000021948563)
[axios从无从下手到手到擒来(4)-源码分析之axios与Axios的关系](https://segmentfault.com/a/1190000021949448)
[axios从无从下手到手到擒来(5)-源码分析之默认axios与axios.create()的区别](https://segmentfault.com/a/1190000021949834)
[axios从无从下手到手到擒来(6)-源码分析之axios运行的整体流程](https://segmentfault.com/a/1190000021950095)