# 001 谈谈XSS攻击

http://47.98.159.95/my_blog/blogs/browser/browser-security/001.html#%E4%BB%80%E4%B9%88%E6%98%AF-xss-%E6%94%BB%E5%87%BB

##  什么是 XSS 攻击？

`XSS` 全称是 `Cross Site Scripting`(即`跨站脚本`)，为了和 CSS 区分，故叫它`XSS`。XSS 攻击是指浏览器中执行恶意脚本(无论是跨域还是同域)，从而拿到用户的信息并进行操作。

这些操作一般可以完成下面这些事情:

1. 窃取`Cookie`。
2. 监听用户行为，比如输入账号密码后直接发送到黑客服务器。
3. 修改 DOM 伪造登录表单。
4. 在页面中生成浮窗广告。

通常情况，XSS 攻击的实现有三种方式——**存储型**、**反射型**和**文档型**。原理都比较简单，先来一一介绍一下。

### [#](http://47.98.159.95/my_blog/blogs/browser/browser-security/001.html#存储型)存储型

`存储型`，顾名思义就是将恶意脚本存储了起来，确实，存储型的 XSS 将脚本存储到了服务端的数据库，然后在客户端执行这些脚本，从而达到攻击的效果。

常见的场景是留言评论区提交一段脚本代码，如果前后端没有做好转义的工作，那评论内容存到了数据库，在页面渲染过程中`直接执行`, 相当于执行一段未知逻辑的 JS 代码，是非常恐怖的。这就是存储型的 XSS 攻击。

### [#](http://47.98.159.95/my_blog/blogs/browser/browser-security/001.html#反射型)反射型

`反射型XSS`指的是恶意脚本作为**网络请求的一部分**。

比如我输入:

```text
http://sanyuan.com?q=<script>alert("你完蛋了")</script>
```

1

这样，在服务器端会拿到`q`参数,然后将内容返回给浏览器端，浏览器将这些内容作为HTML的一部分解析，发现是一个脚本，直接执行，这样就被攻击了。

之所以叫它`反射型`, 是因为恶意脚本是通过作为网络请求的参数，经过服务器，然后再反射到HTML文档中，执行解析。和`存储型`不一样的是，服务器并不会存储这些恶意脚本。

### [#](http://47.98.159.95/my_blog/blogs/browser/browser-security/001.html#文档型)文档型

文档型的 XSS 攻击并不会经过服务端，而是作为中间人的角色，在数据传输过程劫持到网络数据包，然后**修改里面的 html 文档**！

这样的劫持方式包括`WIFI路由器劫持`或者`本地恶意软件`等。

## [#](http://47.98.159.95/my_blog/blogs/browser/browser-security/001.html#防范措施)防范措施

明白了三种`XSS`攻击的原理，我们能发现一个共同点: 都是让恶意脚本直接能在浏览器中执行。

那么要防范它，就是要避免这些脚本代码的执行。

为了完成这一点，必须做到**一个信念，两个利用**。

### [#](http://47.98.159.95/my_blog/blogs/browser/browser-security/001.html#一个信念)一个信念

千万不要相信任何用户的输入！

无论是在前端和服务端，都要对用户的输入进行**转码**或者**过滤**。

如:

```js
<script>alert('你完蛋了')</script>
```

1

转码后变为:

```js
&lt;script&gt;alert(&#39;你完蛋了&#39;)&lt;/script&gt;
```

1

这样的代码在 html 解析的过程中是无法执行的。

当然也可以利用关键词过滤的方式，将 script 标签给删除。那么现在的内容只剩下:

什么也没有了:）

### [#](http://47.98.159.95/my_blog/blogs/browser/browser-security/001.html#利用-csp)利用 CSP

CSP，即浏览器中的内容安全策略，它的核心思想就是服务器决定浏览器加载哪些资源，具体来说可以完成以下功能:

1. 限制其他域下的资源加载。
2. 禁止向其它域提交数据。
3. 提供上报机制，能帮助我们及时发现 XSS 攻击。

### [#](http://47.98.159.95/my_blog/blogs/browser/browser-security/001.html#利用-httponly)利用 HttpOnly

很多 XSS 攻击脚本都是用来窃取Cookie, 而设置 Cookie 的 HttpOnly 属性后，JavaScript 便无法读取 Cookie 的值。这样也能很好的防范 XSS 攻击。

## [#](http://47.98.159.95/my_blog/blogs/browser/browser-security/001.html#总结)总结

`XSS` 攻击是指浏览器中执行恶意脚本, 然后拿到用户的信息进行操作。主要分为`存储型`、`反射型`和`文档型`。防范的措施包括:

- 一个信念: 不要相信用户的输入，对输入内容转码或者过滤，让其不可执行。
- 两个利用: 利用 CSP，利用 Cookie 的 HttpOnly 属性。



# 002 谈谈 CSRF 攻击

http://47.98.159.95/my_blog/blogs/browser/browser-security/002.html#%E6%80%BB%E7%BB%93

## 什么是CSRF攻击？

CSRF(Cross-site request forgery), 即跨站请求伪造，指的是黑客诱导用户点击链接，打开黑客的网站，然后黑客利用用户**目前的登录状态**发起跨站请求。

举个例子, 你在某个论坛点击了黑客精心挑选的小姐姐图片，你点击后，进入了一个新的页面。

那么恭喜你，被攻击了:）

你可能会比较好奇，怎么突然就被攻击了呢？接下来我们就来拆解一下当你点击了链接之后，黑客在背后做了哪些事情。

可能会做三样事情。列举如下：

### [#](http://47.98.159.95/my_blog/blogs/browser/browser-security/002.html#_1-自动发-get-请求)1. 自动发 GET 请求

黑客网页里面可能有一段这样的代码:

```js
<img src="https://xxx.com/info?user=hhh&count=100"></img>
```

1

进入页面后自动发送 get 请求，值得注意的是，这个请求会自动带上关于 xxx.com 的 cookie 信息(这里是假定你已经在 xxx.com 中登录过)。

假如服务器端没有相应的验证机制，它可能认为发请求的是一个正常的用户，因为携带了相应的 cookie，然后进行相应的各种操作，可以是转账汇款以及其他的恶意操作。

### [#](http://47.98.159.95/my_blog/blogs/browser/browser-security/002.html#_2-自动发-post-请求)2. 自动发 POST 请求

黑客可能自己填了一个表单，写了一段自动提交的脚本。

```js
<form id='hacker-form' action="https://xxx.com/info" method="POST">
  <input type="hidden" name="user" value="hhh" />
  <input type="hidden" name="count" value="100" />
</form>
<script>document.getElementById('hacker-form').submit();</script>
```

同样也会携带相应的用户 cookie 信息，让服务器误以为是一个正常的用户在操作，让各种恶意的操作变为可能。

### [#](http://47.98.159.95/my_blog/blogs/browser/browser-security/002.html#_3-诱导点击发送-get-请求)3. 诱导点击发送 GET 请求

在黑客的网站上，可能会放上一个链接，驱使你来点击:

```js
<a href="https://xxx/info?user=hhh&count=100" taget="_blank">点击进入修仙世界</a>
```

1

点击后，自动发送 get 请求，接下来和`自动发 GET 请求`部分同理。

这就是`CSRF`攻击的原理。和`XSS`攻击对比，CSRF 攻击并不需要将恶意代码注入用户当前页面的`html`文档中，而是跳转到新的页面，利用服务器的**验证漏洞**和**用户之前的登录状态**来模拟用户进行操作。

## [#](http://47.98.159.95/my_blog/blogs/browser/browser-security/002.html#防范措施)防范措施

### [#](http://47.98.159.95/my_blog/blogs/browser/browser-security/002.html#_1-利用cookie的samesite属性)1. 利用Cookie的SameSite属性

`CSRF攻击`中重要的一环就是自动发送目标站点下的 `Cookie`,然后就是这一份 Cookie 模拟了用户的身份。因此在`Cookie`上面下文章是防范的不二之选。

恰好，在 Cookie 当中有一个关键的字段，可以对请求中 Cookie 的携带作一些限制，这个字段就是`SameSite`。

`SameSite`可以设置为三个值，`Strict`、`Lax`和`None`。

**a.** 在`Strict`模式下，浏览器完全禁止第三方请求携带Cookie。比如请求`sanyuan.com`网站只能在`sanyuan.com`域名当中请求才能携带 Cookie，在其他网站请求都不能。

**b.** 在`Lax`模式，就宽松一点了，但是只能在 `get 方法提交表单`况或者`a 标签发送 get 请求`的情况下可以携带 Cookie，其他情况均不能。

**c.** 在`None`模式下，也就是默认模式，请求会自动携带上 Cookie。

### [#](http://47.98.159.95/my_blog/blogs/browser/browser-security/002.html#_2-验证来源站点)2. 验证来源站点

这就需要要用到请求头中的两个字段: **Origin**和**Referer**。

其中，**Origin**只包含域名信息，而**Referer**包含了`具体`的 URL 路径。

当然，这两者都是可以伪造的，通过 Ajax 中自定义请求头即可，安全性略差。

### [#](http://47.98.159.95/my_blog/blogs/browser/browser-security/002.html#_3-csrf-token)3. CSRF Token

`Django`作为 Python 的一门后端框架，用它开发过的同学就知道，在它的模板(template)中, 开发表单时，经常会附上这样一行代码:

```text
{% csrf_token %}
```

1

这就是`CSRF Token`的典型应用。那它的原理是怎样的呢？

首先，浏览器向服务器发送请求时，服务器生成一个字符串，将其植入到返回的页面中。

然后浏览器如果要发送请求，就必须带上这个字符串，然后服务器来验证是否合法，如果不合法则不予响应。这个字符串也就是`CSRF Token`，通常第三方站点无法拿到这个 token, 因此也就是被服务器给拒绝。

## [#](http://47.98.159.95/my_blog/blogs/browser/browser-security/002.html#总结)总结

CSRF(Cross-site request forgery), 即跨站请求伪造，指的是黑客诱导用户点击链接，打开黑客的网站，然后黑客利用用户目前的登录状态发起跨站请求。

`CSRF`攻击一般会有三种方式:

- 自动 GET 请求
- 自动 POST 请求
- 诱导点击发送 GET 请求。

防范措施: `利用 Cookie 的 SameSite 属性`、`验证来源站点`和`CSRF Token`。



# 003 (传统RSA版本)HTTPS为什么让数据传输更安全？

http://47.98.159.95/my_blog/blogs/browser/browser-security/003.html#%E5%AF%B9%E7%A7%B0%E5%8A%A0%E5%AF%86%E5%92%8C%E9%9D%9E%E5%AF%B9%E7%A7%B0%E5%8A%A0%E5%AF%86



谈到`HTTPS`, 就不得不谈到与之相对的`HTTP`。`HTTP`的特性是明文传输，因此在传输的每一个环节，数据都有可能被第三方窃取或者篡改，具体来说，HTTP 数据经过 TCP 层，然后经过`WIFI路由器`、`运营商`和`目标服务器`，这些环节中都可能被中间人拿到数据并进行篡改，也就是我们常说的**中间人攻击**。

为了防范这样一类攻击，我们不得已要引入新的加密方案，即 HTTPS。

`HTTPS`并不是一个新的协议, 而是一个加强版的`HTTP`。其原理是在`HTTP`和`TCP`之间建立了一个中间层，当`HTTP`和`TCP`通信时并不是像以前那样直接通信，直接经过了一个中间层进行加密，将加密后的数据包传给`TCP`, 响应的，`TCP`必须将数据包解密，才能传给上面的`HTTP`。这个中间层也叫`安全层`。`安全层`的核心就是对数据`加解密`。

接下来我们就来剖析一下`HTTPS`的加解密是如何实现的。

## [#](http://47.98.159.95/my_blog/blogs/browser/browser-security/003.html#对称加密和非对称加密)对称加密和非对称加密

### [#](http://47.98.159.95/my_blog/blogs/browser/browser-security/003.html#概念)概念

首先需要理解`对称加密`和`非对称加密`的概念，然后讨论两者应用后的效果如何。

`对称加密`是最简单的方式，指的是`加密`和`解密`用的是**同样的密钥**。

而对于`非对称加密`，如果有 A、 B 两把密钥，如果用 A 加密过的数据包只能用 B 解密，反之，如果用 B 加密过的数据包只能用 A 解密。

### [#](http://47.98.159.95/my_blog/blogs/browser/browser-security/003.html#加解密过程)加解密过程

接着我们来谈谈`浏览器`和`服务器`进行协商加解密的过程。

首先，浏览器会给服务器发送一个随机数`client_random` 和一个加密的方法列表。

服务器接收后给浏览器返回另一个随机数`server_random`和加密方法。

现在，两者拥有三样相同的凭证: `client_random`、`server_random`和加密方法。

接着用这个加密方法将两个随机数混合起来生成密钥，这个密钥就是浏览器和服务端通信的`暗号`。

### [#](http://47.98.159.95/my_blog/blogs/browser/browser-security/003.html#各自应用的效果)各自应用的效果

如果用`对称加密`的方式，那么第三方可以在中间获取到`client_random`、`server_random`和加密方法，由于这个加密方法同时可以解密，所以中间人可以成功对暗号进行解密，拿到数据，很容易就将这种加密方式破解了。

那能不能只用`非对称加密`呢？理论上是可以的，但实际上非对称加密需要的计算量非常大，对于稍微大一点的数据即使用最快的处理器也非常耗时。后面有机会给大家分享一下 RSA 非对称加密算法的原理，大家就会有更加直观的认识，这里我们先不深究。

## [#](http://47.98.159.95/my_blog/blogs/browser/browser-security/003.html#对称加密和非对称加密的结合)对称加密和非对称加密的结合

可以发现，对称加密和非对称加密，只用前者会有安全隐患，只用后者性能消耗又太大。那我们能不能把两者结合，保证性能的同时又能保证安全呢？

其实是可以的，演示一下整个流程：

1. 浏览器向服务器发送`client_random`和加密方法列表。
2. 服务器接收到，返回`server_random`、加密方法以及公钥。
3. 浏览器接收，接着生成另一个随机数`pre_random`, 并且用公钥加密，传给服务器。(敲黑板！重点操作！)
4. 服务器用公钥解密这个被加密后的`pre_random`。

现在浏览器和服务器有三样相同的凭证:`client_random`、`server_random`和`pre_random`。然后两者用相同的加密方法混合这三个随机数，生成最终的`密钥`。

然后浏览器和服务器尽管用一样的密钥进行通信，即使用`对称加密`。

这个最终的密钥是很难被中间人拿到的，为什么呢? 因为中间人没有私钥，从而**拿不到pre_random**，也就无法生成最终的密钥了。

回头比较一下和单纯的使用**非对称加密**, 这种方式做了什么改进呢？本质上是**防止了私钥加密的数据外传**。单独使用**非对称加密**，最大的漏洞在于服务器传数据给浏览器只能用`私钥`加密，这是危险产生的根源。利用`对称和非对称`加密结合的方式，就防止了这一点，从而保证了安全。

## [#](http://47.98.159.95/my_blog/blogs/browser/browser-security/003.html#添加数字证书)添加数字证书

尽管通过两者加密方式的结合，能够很好地实现加密传输，但实际上还是存在一些问题。黑客如果采用 DNS 劫持，将目标地址替换成黑客服务器的地址，然后黑客自己造一份公钥和私钥，照样能进行数据传输。而对于浏览器用户而言，他是不知道自己正在访问一个危险的服务器的。

事实上`HTTPS`在上述`结合对称和非对称加密`的基础上，又添加了`数字证书认证`的步骤。其目的就是让服务器证明自己的身份。

### [#](http://47.98.159.95/my_blog/blogs/browser/browser-security/003.html#传输过程)传输过程

为了获取这个证书，服务器运营者需要向第三方认证机构获取授权，这个第三方机构也叫`CA`(`Certificate Authority`), 认证通过后 CA 会给服务器颁发**数字证书**。

这个数字证书有两个作用:

1. 服务器向浏览器证明自己的身份。
2. 把公钥传给浏览器。

这个验证的过程发生在什么时候呢？

当服务器传送`server_random`、加密方法的时候，顺便会带上`数字证书`(包含了`公钥`), 接着浏览器接收之后就会开始验证数字证书。如果验证通过，那么后面的过程照常进行，否则拒绝执行。

现在我们来梳理一下`HTTPS`最终的加解密过程: ![project](media/http知识巩固准备/1.jpg)

### [#](http://47.98.159.95/my_blog/blogs/browser/browser-security/003.html#认证过程)认证过程

浏览器拿到数字证书后，如何来对证书进行认证呢？

首先，会读取证书中的明文内容。CA 进行数字证书的签名时会保存一个 Hash 函数，来这个函数来计算明文内容得到`信息A`，然后用公钥解密明文内容得到`信息B`，两份信息做比对，一致则表示认证合法。

当然有时候对于浏览器而言，它不知道哪些 CA 是值得信任的，因此会继续查找 CA 的上级 CA，以同样的信息比对方式验证上级 CA 的合法性。一般根级的 CA 会内置在操作系统当中，当然如果向上找没有找到根级的 CA，那么将被视为不合法。

## [#](http://47.98.159.95/my_blog/blogs/browser/browser-security/003.html#总结)总结

HTTPS并不是一个新的协议, 它在`HTTP`和`TCP`的传输中建立了一个安全层，利用`对称加密`和`非对称机密`结合数字证书认证的方式，让传输过程的安全性大大提高。