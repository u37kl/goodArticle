# [axios从无从下手到手到擒来(1)-使用XMLHttpRequest封装简单的ajax请求函数](https://segmentfault.com/a/1190000021944305)

```
1. XHR的MDN文档
```

https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest

------

```
2. XHR的理解
1). 使用XHR对象可以发送ajax请求与服务端进行交互
2). 前端可以获取数据,而无需让整个页面进行刷新
3). 只更新Web页面的局部,而不影响用户的操作
```

------

```
3. 一般http请求与ajax请求的区别
1). ajax请求是一种特别的http请求
2). 对服务器端来说没有任何区别,区别在浏览器端
3). 对浏览器端来说,只有XHR或fetch发出的请求才是ajax请求,其他的都是一般http请求
4). 浏览器端接收到响应:
    a. 一般请求: 浏览器会直接显示响应数据体,也就是刷新/跳转页面
    b. ajax请求: 浏览器不会对页面做任何更新操作,只是调用相应的回调函数并传入响应数据
    
```

------

```
4. XHR的API
1). XMLHttpRequest() 创建XHR对象的构造函数
2). status 响应状态码,比如404,200...
3). statusText 响应状态文本
4). readyState 标识请求状态的只读属性
    0 初始
    1 open()之后
    2 send()之后
    3 请求中
    4 请求完成
5). onreadystatechange 绑定readyState改变的监听
6). responseType 指定响应数据类型,如果是'json',得到响应后自动解析响应数据体
7). response 响应体数据,类型取决于responseType
8). timeout 指定请求超时时间,默认为0没有限制
9). ontimeout 绑定超时的监听
10). onerror 绑定请求错误的监听
11). open(url,method[,async]) 初始化一个请求
12). send(data) 发送请求
13). abort() 中断请求
14). getResponseHeader(name)  获取指定的响应头
15). getAllResponseHeaders() 获取所有的响应头组成的字符串
16). setRequestHeader(name,value) 设置请求头
```

------

```
5. 使用XHR简单封装axios
  封装目的
    a. 函数的返回值类型为Promise,成功的结果为response,失败的结果为error
    b. 能够处理GET/POST/DELETE/PUT等类型的请求
    c. 构造函数的参数为一个配置对象
    d. 响应的json数据自动解析为js对象
(function (window) {
  //参数为配置对象,默认get请求,queryParams和data都为空对象
  function axios({url, method = 'GET', params = {}, data = {}}) {
    //返回一个Promise
    return new Promise((resolve, reject) => {
      //处理method的大小写
      method = method.toUpperCase()

      //处理params, 拼接成queryString
      if (params) {
        let queryString = ''
        Object.keys(params).forEach(key => {
          queryString += `${key}=${params[key]}&`
        })
        //去掉最后一个&
        queryString = queryString.substring(0, queryString.length - 1)
        //把queryString拼到url上
        url += '?' + queryString
      }

      //创建xhr对象
      const request = new XMLHttpRequest()
      //打开连接(还没有请求)
      request.open(method, url)
      //发送请求
      if (method === 'GET' || method === 'DELETE') {
        request.send()
      } else if (method === 'POST' || method === 'PUT') {
        //设置请求头类型
        request.setRequestHeader('ContentType', 'application/json;charset=utf-8')
        //将data从js对象转成json string
        request.send(JSON.stringify(data))
      }

      //绑定状态改变监听
      request.onreadystatechange = () => {
        //请求没有完成
        if (request.readyState !== 4) return
        //只有status在[200,300)之间才代表成功
        const {status, statusText} = request
        //axios的源码中也是这样的简单粗暴
        if (status >= 200 & status < 300) {
          //response对象
          const response = {
            data: JSON.parse(request.response),
            status,
            statusText
          }
          //resolve promise
          resolve(response)
        } else {
          //其他状态都表示失败, reject promise, 给予一个友好的提示
          reject(new Error('request failed! response code = ' + status))
        }
      }
    })
  }

  window.axios = axios
})(window)
```

###### axios系列传送门

[axios从无从下手到手到擒来(1)-使用XMLHttpRequest封装简单的ajax请求函数](https://segmentfault.com/a/1190000021944305)
[axios从无从下手到手到擒来(2)-axios的理解与使用](https://segmentfault.com/a/1190000021944651)
[axios从无从下手到手到擒来(3)-源码分析之整体结构](https://segmentfault.com/a/1190000021948563)
[axios从无从下手到手到擒来(4)-源码分析之axios与Axios的关系](https://segmentfault.com/a/1190000021949448)
[axios从无从下手到手到擒来(5)-源码分析之默认axios与axios.create()的区别](https://segmentfault.com/a/1190000021949834)
[axios从无从下手到手到擒来(6)-源码分析之axios运行的整体流程](https://segmentfault.com/a/1190000021950095)