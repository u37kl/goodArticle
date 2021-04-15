# [vue双向绑定原理分析](https://www.cnblogs.com/zhenfei-jiang/p/7542900.html)

当我们学习angular或者vue的时候，其双向绑定为我们开发带来了诸多便捷，今天我们就来分析一下vue双向绑定的原理。

简易vue源码地址：https://github.com/jiangzhenfei/simple-Vue

**1.vue双向绑定原理**

vue.js 则是采用数据劫持结合发布者-订阅者模式的方式，通过`Object.defineProperty()`来劫持各个属性的`setter`，`getter`，在数据变动时发布消息给订阅者，触发相应的监听回调。我们先来看Object.defineProperty()这个方法：

```
var obj  = {};
Object.defineProperty(obj, 'name', {
        get: function() {
            console.log('我被获取了')
            return val;
        },
        set: function (newVal) {
            console.log('我被设置了')
        }
})
obj.name = 'fei';//在给obj设置name属性的时候，触发了set这个方法
var val = obj.name;//在得到obj的name属性，会触发get方法
```

已经了解到vue是通过数据劫持的方式来做数据绑定的，其中最核心的方法便是通过`Object.defineProperty()`来实现对属性的劫持，那么在设置或者获取的时候我们就可以在get或者set方法里假如其他的触发函数，达到监听数据变动的目的，无疑这个方法是本文中最重要、最基础的内容之一。

**2.实现最简单的双向绑定**

我们知道通过Object.defineProperty()可以实现数据劫持，是的属性在赋值的时候触发set方法，

```
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>
<body>
    <div id="demo"></div>
    <input type="text" id="inp">
    <script>
        var obj  = {};
        var demo = document.querySelector('#demo')
        var inp = document.querySelector('#inp')
        Object.defineProperty(obj, 'name', {
            get: function() {
                return val;
            },
            set: function (newVal) {//当该属性被赋值的时候触发
                inp.value = newVal;
                demo.innerHTML = newVal;
            }
        })
        inp.addEventListener('input', function(e) {
            // 给obj的name属性赋值，进而触发该属性的set方法
            obj.name = e.target.value;
        });
        obj.name = 'fei';//在给obj设置name属性的时候，触发了set这个方法
    </script>
</body>
</html>
```

当然要是这么粗暴，肯定不行，性能会出很多的问题。

**3.讲解vue如何实现**

先看原理图


3.1 observer用来实现对每个vue中的data中定义的属性循环用Object.defineProperty()实现数据劫持，以便利用其中的setter和getter，然后通知订阅者，订阅者会触发它的update方法，对视图进行更新。

3.2 我们介绍为什么要订阅者，在vue中v-model，v-name，{{}}等都可以对数据进行显示，也就是说假如一个属性都通过这三个指令了，那么每当这个属性改变的时候，相应的这个三个指令的html视图也必须改变，于是vue中就是每当有这样的可能用到双向绑定的指令，就在一个Dep中增加一个订阅者，其订阅者只是更新自己的指令对应的数据，也就是v-model='name'和{{name}}有两个对应的订阅者，各自管理自己的地方。每当属性的set方法触发，就循环更新Dep中的订阅者。

**4.vue代码实现**

4.1 observer实现，主要是给每个vue的属性用Object.defineProperty()，代码如下：


```
function defineReactive (obj, key, val) {
    var dep = new Dep();
        Object.defineProperty(obj, key, {
             get: function() {
                    //添加订阅者watcher到主题对象Dep
                    if(Dep.target) {
                        // JS的浏览器单线程特性，保证这个全局变量在同一时间内，只会有同一个监听器使用
                        dep.addSub(Dep.target);
                    }
                    return val;
             },
             set: function (newVal) {
                    if(newVal === val) return;
                    val = newVal;
                    console.log(val);
                    // 作为发布者发出通知
                    dep.notify();//通知后dep会循环调用各自的update方法更新视图
             }
       })
}
        function observe(obj, vm) {
            Object.keys(obj).forEach(function(key) {
                defineReactive(vm, key, obj[key]);
            })
        }
```

4.2实现compile：

compile的目的就是解析各种指令称真正的html。


```
function Compile(node, vm) {
    if(node) {
        this.$frag = this.nodeToFragment(node, vm);
        return this.$frag;
    }
}
Compile.prototype = {
    nodeToFragment: function(node, vm) {
        var self = this;
        var frag = document.createDocumentFragment();
        var child;
        while(child = node.firstChild) {
            console.log([child])
            self.compileElement(child, vm);
            frag.append(child); // 将所有子节点添加到fragment中
        }
        return frag;
    },
    compileElement: function(node, vm) {
        var reg = /\{\{(.*)\}\}/;
        //节点类型为元素(input元素这里)
        if(node.nodeType === 1) {
            var attr = node.attributes;
            // 解析属性
            for(var i = 0; i < attr.length; i++ ) {
                if(attr[i].nodeName == 'v-model') {//遍历属性节点找到v-model的属性
                    var name = attr[i].nodeValue; // 获取v-model绑定的属性名
                    node.addEventListener('input', function(e) {
                        // 给相应的data属性赋值，进而触发该属性的set方法
                        vm[name]= e.target.value;
                    });
                    new Watcher(vm, node, name, 'value');//创建新的watcher，会触发函数向对应属性的dep数组中添加订阅者，
                }
            };
        }
        //节点类型为text
        if(node.nodeType === 3) {
            if(reg.test(node.nodeValue)) {
                var name = RegExp.$1; // 获取匹配到的字符串
                name = name.trim();
                new Watcher(vm, node, name, 'nodeValue');
            }
        }
    }
}
```


4.3 watcher实现

```
function Watcher(vm, node, name, type) {
    Dep.target = this;
    this.name = name;
    this.node = node;
    this.vm = vm;
    this.type = type;
    this.update();
    Dep.target = null;
}

Watcher.prototype = {
    update: function() {
        this.get();
        this.node[this.type] = this.value; // 订阅者执行相应操作
    },
    // 获取data的属性值
    get: function() {
        console.log(1)
        this.value = this.vm[this.name]; //触发相应属性的get
    }
}
```

4.4 实现Dep来为每个属性添加订阅者


```
function Dep() {
    this.subs = [];
}
Dep.prototype = {
    addSub: function(sub) {
        this.subs.push(sub);
    },
    notify: function() {
        this.subs.forEach(function(sub) {
        sub.update();
        })
    }
}
```


这样一来整个数据的双向绑定就完成了。

**5.梳理**

首先我们为每个vue属性用Object.defineProperty()实现数据劫持，为每个属性分配一个订阅者集合的管理数组dep；然后在编译的时候在该属性的数组dep中添加订阅者，v-model会添加一个订阅者，{{}}也会，v-bind也会，只要用到该属性的指令理论上都会，接着为input会添加监听事件，修改值就会为该属性赋值，触发该属性的set方法，在set方法内通知订阅者数组dep，订阅者数组循环调用各订阅者的update方法更新视图。