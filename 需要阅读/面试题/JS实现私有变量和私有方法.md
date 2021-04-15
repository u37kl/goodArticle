# JavaScript如何实现私有变量私有方法

## 前言

大家都知道JavaScript不像Java这类强类型语言，有特定的字面量（private）来定义参数为类的私有参数；那JavaScript如何能实现私有变量呢，下面来介绍几种实现方式。

## 一、函数作用域封装局部变量的方式。

### 1.1 闭包定义局部变量（自执行函数）

```
var PClass = (function(){
    var a = '私有变量';
    var getA = function(){
    	console.log(a, '私有方法')
	}
    var setA = function(val){
            a = val
        }
    function P (){
    	this.b = '变量b，外部可访问'
    }
    P.prototype = {
        getB : function(){
            console.log(this.b,'--- 获取对象公有属性b ---')
        },
        setType: function(a,val){
            if(a == 'a'){
                setA(val);
            }
        },
        getType: function(a){
            if(a == 'a'){
                getA();
            }
        }
    }
    return P
}())

var pclass = new PClass()

pclass.b            //打印    变量b，外部可访问
pclass.getB()       //打印    变量b，外部可访问 --- 获取对象公有属性b ---
console.log(pclass) //打印    P {b: "变量b，外部可访问"}
// 通过对外开放的另一方法我们可以获取到了私有变量a
pclass.getType('a') //打印    私有变量 私有方法
复制代码
```

根据上面的例子我们不难发现，私有变量只能再闭包内部访问，外部无法访问，当时这个方案并不完美,为什么这么说，比如我们再new 一个 pclass1 ， 然后修改a属性的值，你会发现pclass的a值也发生了变化

```
var pclass1 = new PClass();
pclass1.setType('a', '修改私有属性a的值');
pclass1.getType('a');  //打印    修改私有属性a的值  私有方法
pclass.getType('a');  //打印    修改私有属性a的值  私有方法
复制代码
```

那么针对上面的问题我们怎么解决呢；其实很简单，我们只要将这个闭包改成非自执行函数就OK了

### 1.2 闭包定义局部变量（非自执行函数）

```
var pClass = function(){
	var a = '私有变量';
	var getA = function(){
		console.log(a, '私有方法')
	}
	var setA = function(val){
		a = val
	}
    function P (){
		this.b = '变量b，外部可访问'
    }
    P.prototype = {
        getB : function(){
            console.log(this.b,'--- 获取对象公有属性b ---')
        },
        setType: function(a,val){
            if(a == 'a'){
                setA(val);
            }
        },
        getType: function(a){
            if(a == 'a'){
                getA();
            }
        }
    }
    return P
}

var PClass1 = pClass(); //开辟了一个函数作用域
var PClass2 = pClass(); //重新开辟了一个函数作用域
var pclass1 = new PClass1();
var pclass2 = new PClass2();
pclass1.setType('a', '修改私有属性a的值');
pclass1.getType('a');   //打印    修改私有属性a的值  私有方法
pclass2.getType('a');   //打印    私有变量 私有方法
复制代码
```

## 二、ES6 定义私有变量，私有方法

### 2.1 使用ES6扩展的类型symbol类型定义

先来解释下symbol类型；Symbol值通过Symbol函数生成。这就是说，对象的属性名现在可以有两种类型，一种是原来就有的字符串，另一种就是新增的Symbol类型。凡是属性名属于Symbol类型，就都是独一无二的，可以保证不会与其他属性名产生冲突；类似与UUID的用法。

```
Symbol('1') == Symbol('1') //打印  false
var sy = sb = Symbol('a');
sy === sb //打印  true   说明该数据类型以引用的方式传值。
复制代码
```

解释了Symbol 类型数据，那下面来写一个基于Symbol类型的私有变量，私有属性吧。

```
var Pclass = (function(){
    const a = Symbol('a');
    const m = Symbol('m');
    class  Pclass {
        constructor(){
            this[a] = 'a这是私有变量';
            this.b = '变量B-外部可访问';
            this[m] = function(){
                console.log('私有方法');
            }
        }
        getA(){
            console.log(this[a]);
        }
        getM(){
            console.log(this[m]);
        }
    }
    return Pclass
}())

let pc = new Pclass() 
console.log(pc)   //打印 Pclass {b: "变量B-外部可访问", Symbol(a): "这是私有变量", Symbol(m): ƒ}
复制代码
```

由上述代码我们可以发现 只要不把 a = Symbol('a'); m = Symbol('m') 这两个引用对外暴露，外部是无法访问到定义的私有变量a，和私有方法m， 因为他们的真实属性名称是a, m 这两个引用，而且是唯一的。

### 2.1 使用ES6扩展的类型WeakMap类型定义

先来解释下WeakMap类型, 该类型数据是一个键-值（key-val）对的集合，只不过他的键（key）是一个引用，不同于一般的键-值。WeakMap 的使用如下。

```
const wm = new WeakMap();
const a = {}, b = {};
wm.set(a, '这是a对象键的值');
wm.set(b, '这是b对象键的值');
console.log(wm.get(a)) //打印 这是a对象键的值
console.log(wm.get(b)) //打印 这是b对象键的值
复制代码
```

解释了WeakMap类型数据，那下面来写一个基于WeakMap类型的私有变量，私有属性吧。

```
var Pclass = (function(){
    const aa = new WeakMap();
    const mt = new WeakMap();
    class  Pclass {
        constructor(){
            this.b = 'b这是公有变量';
            aa.set(this, '私有变量aa')
            mt.set(this, function(){
                console.log('私有方法mt')
            })
        }
        getA(){
            console.log(aa.get(this));
        }
        getM(){
            console.log(mt.get(this));
        }
    }
    return Pclass
}())
let pc = new Pclass() 
console.log(pc) // Pclass {b: "b这是公有变量"}
复制代码
```

同上述代码我们可以发现 只要不把 aa = new WeakMap(); m = new WeakMap()这两个集合对外开放 外部是无法访问到定义的私有变量a，和私有方法m， 因为他们的真实属性名称是a, m 这两个引用，而且是唯一的。