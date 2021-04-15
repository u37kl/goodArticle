# [babel-polyfill的几种使用方式](https://www.cnblogs.com/Jeely/p/11231530.html)

##### 为什么要使用babel-polyfill？

​    Babel是一个广泛使用的转码器，可以将ES6代码转为ES5代码，从而可以在现有环境执行，所以我们可以用ES6编写，而不用考虑环境支持的问题；
​    有些浏览器版本的发布早于ES6的定稿和发布，因此如果在编程中使用了ES6的新特性，而浏览器没有更新版本，或者新版本中没有对ES6的特性进行兼容，那么浏览器就会无法识别ES6代码，例如IE9根本看不懂代码写的let和const是什么东西？只能选择报错，这就是浏览器对ES6的兼容性问题；





​    Babel默认只转换新的JavaScript语法（syntax），如箭头函数等，而不转换新的API，比如Iterator、Generator、Set、Maps、Proxy、Reflect、Symbol、Promise等全局对象，以及一些定义在全局对象上的方法（比如Object.assign）都不会转码；因此我们需要polyfill；
​    因为这是一个 polyfill （它需要在源代码之前运行），我们需要让它成为一个 dependency（上线时的依赖）,而不是一个 devDependency（开发时的依赖）；





# 1. 使用转换插件: babel-plugin-transform-xxx

- 使用方法
  - 缺啥补啥,在`package.json`添加所需的依赖`babel-plugin-transform-xxx`
  - 在`.babelrc`中的`plugins`项指定使用`babel-plugin-transform-xxx`插件
  - 代码中不需要显式`import/require`, `.babelrc`中不需要指定`useBuiltIns`, `webpack.config.js`中不需要做额外处理,一切由babel插件完成转换
- 优点
  - 作用域是模块,避免全局冲突
  - 是**按需**引入,避免不必要引入造成及代码臃肿
- 缺点
  - 每个模块内单独引用和定义polyfill函数,造成了重复定义,使代码产生冗余

> 配置完一个转换插件后, 代码中每个使用这个API的地方的代码都会被转换成使用`polyfill`中实现的代码

# 2. 使用插件 babel-runtime 与 babel-plugin-tranform-runtime

相比方法1, 相当于抽离了公共模块, 避免了重复引入, 从一个叫`core.js`的库中引入所需polyfill(一个国外大神用ES3写的ES5+ polyfill)

- 使用方法
  - `package.json`中添加依赖`babel-plugin-tranform-runtime`以及 `babel-runtime`
  - `.babelrc`中配置插件:`"plugins": ["transform-runtime"]`
  - 接下来, 代码中可以直接使用ES6+的新特性,无需`import/require`额外东西, `webpack`也不需要做额外配置
- 优点
  - 无全局污染
  - 依赖统一按需引入(polyfill是各个模块共享的), 无重复引入, 无多余引入
  - 适合用来编写lib(第三方库)类型的代码
- 缺点
  - 被`polyfill`的对象是临时构造并被`import/require`的,并不是真正挂载到全局
  - 由于不是全局生效, 对于实例化对象的方法,如`[].include(x)`, 依赖于`Array.prototype.include`仍无法使用

# 3. 全局babel-polyfill(不使用useBuiltIns)

- 使用方法
  - 法3.1: (浏览器环境)单独在html的`<head>`标签中引入`babel-polyfill.js`(CDN或本地文件均可)
  - 法3.2: 在`package.json`中添加`babel-polyfill`依赖, 在`webpack`配置文件增加入口: 如`entry: ["babel-polyfill",'./src/app.js']`, polyfill将会被打包进这个入口文件中, 而且是放在文件最开始的地方
  - 法3.3: 在`package.json`中添加`babel-polyfill`依赖, 在`webpack`入口文件顶部使用`import/require`引入,如`import 'babel-polyfill'``
- 优点
  - 一次性解决所有兼容性问题,而且是全局的,浏览器的`console`也可以使用
- 缺点
  - 一次性引入了ES6+的所有polyfill, 打包后的js文件体积会偏大
  - 对于现代的浏览器,有些不需要polyfill,造成流量浪费
  - 污染了全局对象
  - 不适合框架或库的开发

# 4. 全局babel-polyfill(使用babel-preset-env插件和useBuiltIns属性)

- 使用方法
  - `packge.json`引入依赖`babel-preset-env`
  - `.babelrc`中使用配置`preset-env`
  - 指定`useBuiltins`选项为`true`
  - 指定**浏览器环境或node环境**, 配置需要兼容的浏览器列表
  - 在`webpack`入口文件中使用`import/require`引入`polyfill`, 如`import 'babel-polyfill'`
  - 以上配置完成之后, `babe`l会根据指定的浏览器兼容列表自动引入所有所需的`polyfill`, 不管你代码中有没有使用
- .babelrc示例

```
{
  "presets": [
    ["env", {
      "modules": false,
      "targets": {
        "browsers": ["ie >=9"]
      },
      "useBuiltIns": true,
      "debug": true
    }]
  ]
}
```

- 优点
  - 按需(按照**指定的浏览器环境**所需)引入`polyfill`, 一定程度上减少了不必要`polyfill`的引入
  - 配置简单, 无需对`webpack`做额外的配置
  - 注意:
    - 不可与方法3混用,否则会引起冲突
    - 全局方式要保证`polyfill`在所有其它脚本之前被执行(首行`import`或者设置为html的第一个`<head>`标签)

## 5. polyfill.io

- 一个`CDN`方式提供的`polyfill`, 可根据浏览器`UserAgent`自动返回合适的`polyfill`, 详细内容自行google



作者：荣儿飞
链接：https://www.jianshu.com/p/3b27dfc6785c
来源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。