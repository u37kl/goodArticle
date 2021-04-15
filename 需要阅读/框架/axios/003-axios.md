# [axios从无从下手到手到擒来(3)-源码分析之整体结构](https://segmentfault.com/a/1190000021948563)

1. **axios的整体目录结构**

   ```
      ├── /dist/ # 项目输出目录 
      ├── /lib/ # 项目源码目录
      │ ├── /adapters/ # 定义请求的适配器 xhr、http
      │ │ ├── http.js # 实现 http 适配器(包装 http 包)
      │ │ └── xhr.js # 实现 xhr 适配器(包装 xhr 对象) 
      │ ├── /cancel/ # 定义取消功能
      │ ├── /core/ # 一些核心功能
      │ │ ├── Axios.js # axios 的核心主类
      │ │ ├── dispatchRequest.js # 用来调用 http 请求适配器方法发送请求的函数
      │ │ ├── InterceptorManager.js   # 拦截器的管理器
      │ │ └── settle.js # 根据 http 响应状态，改变 Promise 的状态
      │ ├── /helpers/  # 一些辅助方法
      │ ├── axios.js # 对外暴露接口
      │ ├── defaults.js  # axios 的默认配置
      │ └── utils.js  # 公用工具
      ├── package.json # 项目信息
      ├── index.d.ts # 配置 TypeScript 的声明文件
      └── index.js # 入口文件
   ```

2. **关于入口文件**

   ```
   入口文件index.js只有这一行代码,所以我们把注意力移步到./lib/axios上
   module.exports = require('./lib/axios');
   ```

3. **默认暴露的axios**

   ```
   axios.js暴露了axios,所以我们重点要搞明白createInstance这个方法.
   到底axios是不是Axios的实例对象,之后下定论
   ...codes...
   // Create the default instance to be exported
   var axios = createInstance(defaults);
   ...codes...
   module.exports = axios;
   ```

4. **关于adapters的选择**

   ```
      adapters下有http.js和xhr.js两个方法,
      很明显一个是用在node端的http请求,一个是客户端的ajax请求,所以我们只关心xhr.js这个方法
      
   ```

5. **简要说一下Cancel类型**
   上一篇文章说过,'*在error回调中判断,如果error是cancel,做相应处理*',那到底Cancel是个啥类型呢?

   ```
      Cancel.js定义了Cancel类型,并且用一个原型属性__CALCEL__做了标识
      function Cancel(message) {
        this.message = message;
      }
   
      Cancel.prototype.toString = function toString() {
        return 'Cancel' + (this.message ? ': ' + this.message : '');
      };
   
      // 用于标识是一个取消的error
      Cancel.prototype.__CANCEL__ = true;
      module.exports = Cancel;
      
      isCancel.js文件中,定义了错误类型的判断方法
      module.exports = function isCancel(value) {
        return !!(value && value.__CANCEL__);}
      };
      
   ```

###### axios系列传送门

[axios从无从下手到手到擒来(1)-使用XMLHttpRequest封装简单的ajax请求函数](https://segmentfault.com/a/1190000021944305)
[axios从无从下手到手到擒来(2)-axios的理解与使用](https://segmentfault.com/a/1190000021944651)
[axios从无从下手到手到擒来(3)-源码分析之整体结构](https://segmentfault.com/a/1190000021948563)
[axios从无从下手到手到擒来(4)-源码分析之axios与Axios的关系](https://segmentfault.com/a/1190000021949448)
[axios从无从下手到手到擒来(5)-源码分析之默认axios与axios.create()的区别](https://segmentfault.com/a/1190000021949834)
[axios从无从下手到手到擒来(6)-源码分析之axios运行的整体流程](https://segmentfault.com/a/1190000021950095)