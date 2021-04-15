# [奇思妙想 CSS 文字动画](https://juejin.cn/post/6937102296442470413#heading-13)

之前有些过两篇关于字体的文章，是关于如何定义字体的：

- [你该知道的字体 font-family](https://github.com/chokcoco/iCSS/issues/6)
- [Web 字体 font-family 再探秘](https://github.com/chokcoco/iCSS/issues/69)

本文将会和这篇 -- [CSS 奇思妙想边框动画](https://github.com/chokcoco/iCSS/issues/92)类似，讲一些**文字效果**，利用不同的属性搭配，实现各式各样的文字动效。

## Google Font

在写各种 DEMO 的时候，有的时候一些特殊的字体能更好的体现动画的效果。这里讲一个快速引入不同格式字体的小技巧。

就是 [Google Font](https://fonts.google.com/) 这个网站，上面有非常多的不同的开源字体：

![image](media/Untitled 2/5779503f90c34de18becefa90c1b3b5a~tplv-k3u1fbpfcp-zoom-1.image)

当我们相中了一个我们喜欢的字体，它也提供了非常快速的便捷的引入方式。选中对应的字体，选择 `+Select this style`，便可以通过 `link` 和 `@import` 两种方式引入：

![image](media/Untitled 2/3f74286c64344d53937cbf86c1323085~tplv-k3u1fbpfcp-zoom-1.image)

使用 `link` 标签引入：

```HTML
<link rel="preconnect" href="https://fonts.gstatic.com">
<link href="https://fonts.googleapis.com/css2?family=Source+Code+Pro:wght@200&display=swap" rel="stylesheet">
复制代码
```

OR，在 CSS 代码中，使用 `@import` 引入：

```CSS
<style>
@import url('https://fonts.googleapis.com/css2?family=Source+Code+Pro:wght@200&display=swap');
</style>
复制代码
```

上述两种方式内部其实都是使用的 `@font-face` 进行了字体的定义。

我们可以通过 [@font-face](https://developer.mozilla.org/zh-CN/docs/Web/CSS/@font-face) 快速声明指定一个自定义字体。类似这样：

```CSS
@font-face {
  font-family: "Open Sans";
  src: url("/fonts/OpenSans-Regular-webfont.woff2") format("woff2"),
       url("/fonts/OpenSans-Regular-webfont.woff") format("woff");
}
复制代码
```

这样，利用 Google Font，我们就可以便捷的享受各种字体了。

我在我的个人项目或者一些 DEMO 中，需要一些艺术字体或者特殊字体展示不一样的效果时，经常会使用这种方式。而至于在业务中，是否需要引入一些特殊字体，就见仁见智了。

------

接下来，就会分门别类的看看，字体在 CSS 中，和不同是属性相结合，能够鼓捣出什么样的效果。

## 文字与阴影

通过将字体与字体阴影 `text-shadow`，相结合，阴影的不同运用方式，产生不一样的效果。

我们通过一系列 Demo 来看看。

### 长阴影文字效果

通过多层次，颜色逐渐变化（透明）的阴影变化，可以生成长阴影：

```CSS
div {
  text-shadow: 0px 0px #992400, 1px 1px rgba(152, 36, 1, 0.98), 2px 2px rgba(151, 37, 2, 0.96), 3px 3px rgba(151, 37, 2, 0.94), 4px 4px rgba(150, 37, 3, 0.92), 5px 5px rgba(149, 38, 4, 0.9), 6px 6px rgba(148, 38, 5, 0.88), 7px 7px rgba(148, 39, 5, 0.86), 8px 8px rgba(147, 39, 6, 0.84), 9px 9px rgba(146, 39, 7, 0.82), 10px 10px rgba(145, 40, 8, 0.8), 11px 11px rgba(145, 40, 8, 0.78), 12px 12px rgba(144, 41, 9, 0.76), 13px 13px rgba(143, 41, 10, 0.74), 14px 14px rgba(142, 41, 11, 0.72), 15px 15px rgba(142, 42, 11, 0.7), 16px 16px rgba(141, 42, 12, 0.68), 17px 17px rgba(140, 43, 13, 0.66), 18px 18px rgba(139, 43, 14, 0.64), 19px 19px rgba(138, 43, 15, 0.62), 20px 20px rgba(138, 44, 15, 0.6), 21px 21px rgba(137, 44, 16, 0.58), 22px 22px rgba(136, 45, 17, 0.56), 23px 23px rgba(135, 45, 18, 0.54), 24px 24px rgba(135, 45, 18, 0.52), 25px 25px rgba(134, 46, 19, 0.5), 26px 26px rgba(133, 46, 20, 0.48), 27px 27px rgba(132, 47, 21, 0.46), 28px 28px rgba(132, 47, 21, 0.44), 29px 29px rgba(131, 48, 22, 0.42), 30px 30px rgba(130, 48, 23, 0.4), 31px 31px rgba(129, 48, 24, 0.38), 32px 32px rgba(129, 49, 24, 0.36), 33px 33px rgba(128, 49, 25, 0.34), 34px 34px rgba(127, 50, 26, 0.32), 35px 35px rgba(126, 50, 27, 0.3), 36px 36px rgba(125, 50, 28, 0.28), 37px 37px rgba(125, 51, 28, 0.26), 38px 38px rgba(124, 51, 29, 0.24), 39px 39px rgba(123, 52, 30, 0.22), 40px 40px rgba(122, 52, 31, 0.2), 41px 41px rgba(122, 52, 31, 0.18), 42px 42px rgba(121, 53, 32, 0.16), 43px 43px rgba(120, 53, 33, 0.14), 44px 44px rgba(119, 54, 34, 0.12), 45px 45px rgba(119, 54, 34, 0.1), 46px 46px rgba(118, 54, 35, 0.08), 47px 47px rgba(117, 55, 36, 0.06), 48px 48px rgba(116, 55, 37, 0.04), 49px 49px rgba(116, 56, 37, 0.02), 50px 50px rgba(115, 56, 38, 0);
}
复制代码
```

![image](media/Untitled 2/f7e1bced5ba746ae8f37b7eddc69bbe6~tplv-k3u1fbpfcp-zoom-1.image)

当然，多重阴影以及每重的颜色我们很难一个一个手动去写，在写长阴影的时候通常需要借助 `SASS`、`LESS` 去帮助节省时间：

```CSS
@function makelongrightshadow($color) {
    $val: 0px 0px $color;

    @for $i from 1 through 50 {
        $color: fade-out(desaturate($color, 1%), .02);
        $val: #{$val}, #{$i}px #{$i}px #{$color};
    }

    @return $val;
}
div {
    text-shadow: makeLongShadow(hsl(14, 100%, 30%));
}
复制代码
```

### 立体阴影文字效果

如果多层阴影，但是颜色变化没那么强烈，能够塑造一种立体的效果。

```CSS
div {
  text-shadow: 0 -1px 0 #ffffff, 0 1px 0 #2e2e2e, 0 2px 0 #2c2c2c, 0 3px 0 #2a2a2a, 0 4px 0 #282828, 0 5px 0 #262626, 0 6px 0 #242424, 0 7px 0 #222222, 0 8px 0 #202020, 0 9px 0 #1e1e1e, 0 10px 0 #1c1c1c, 0 11px 0 #1a1a1a, 0 12px 0 #181818, 0 13px 0 #161616, 0 14px 0 #141414, 0 15px 0 #121212);
}
复制代码
```

![image](media/Untitled 2/2ddb1073b8f948cd9a7e163023252838~tplv-k3u1fbpfcp-zoom-1.image)

### 内嵌阴影文字效果

合理的阴影颜色和背景底色搭配，搭配，可以实现类似内嵌效果的阴影。

```CSS
div {
  color: #202020;
  background-color: #2d2d2d;
  letter-spacing: .1em;
  text-shadow: -1px -1px 1px #111111, 2px 2px 1px #363636;
}
复制代码
```

![image](media/Untitled 2/c6bc32ba43424dd4bc536db36c305897~tplv-k3u1fbpfcp-zoom-1.image)

[CodePen Demo -- 5 text shadow effects in css3](https://codepen.io/Chokcoco/pen/MWbEBee)

### 氖光效果（Neon）

氖光效果，英文名叫 Neon，是我在 Codepen 上看到的最多的效果之一。它的原理非常简单，却可以产生非常酷炫的效果。

我们只需要设置 3~n 层阴影效果，每一层的模糊半径（文字阴影的第三个参数）间隔较大，并且每一层的阴影颜色相同即可。

```CSS
p {
    color: #fff;
    text-shadow: 
        0 0 10px #0ebeff,
        0 0 20px #0ebeff,
        0 0 50px #0ebeff,
        0 0 100px #0ebeff,
        0 0 200px #0ebeff
}
复制代码
```

当然，通常运用 Neon 效果时，背景底色都是偏黑色。

![img](media/Untitled 2/1f41cacbb48e4cdbb7e53de6dccf3e37~tplv-k3u1fbpfcp-zoom-1.image)

合理运用 Neon 效果，就可以制作非常多有意思的动效。譬如作用于鼠标 hover 上去的效果：

```CSS
p {
    transition: .2s;
 
    &:hover {
        text-shadow: 0 0 10px #0ebeff, 0 0 20px #0ebeff, 0 0 50px #0ebeff, 0 0 100px #0ebeff, 0 0 200px #0ebeff;
    }
}
复制代码
```

![img](media/Untitled 2/0e04e32b1b8d47d9a88811e42c0a7f46~tplv-k3u1fbpfcp-zoom-1.image)

[CodePen Demo -- Neon Demo](https://codepen.io/Chokcoco/pen/ZEBaJer)

CodePen 上有一个 2K+ 赞的 DEMO，实现了非常赞的 Neon 效果，可以戳进去看看 [CodePen -- Neon Glow](https://codepen.io/FelixRilling/pen/qzfoc)。

也可以选取适当合适的字体，配合动画效果，实现一种渐进的亮灯效果：

```HTML
<p>
  <span id="n">n</span>
  <span id="e">e</span>
  <span id="o">o</span>
  <span id="n2">n</span>
</p>
复制代码
p:hover span {
  animation: flicker 1s linear forwards;
}
p:hover #e {
  animation-delay: .2s;
}
p:hover #o {
  animation-delay: .5s;
}
p:hover #n2 {
  animation-delay: .6s;
}

@keyframes flicker {
  0% {
    color: #333;
  }
  5%, 15%, 25%, 30%, 100% {
    color: #fff;
    text-shadow: 
      0px 0px 5px var(--color),
      0px 0px 10px var(--color),
      0px 0px 20px var(--color),
      0px 0px 50px var(--color);
      
  }
  10%, 20% {
    color: #333;
    text-shadow: none;
  }
}
复制代码
```

![img](media/Untitled 2/e600eccb8b554f87ace76985aebc3f00~tplv-k3u1fbpfcp-zoom-1.image)

截 GIF 图的帧率不太够，看着效果不太好，可以点进下面的 DEMO 感受下：

[CodePen Demo -- Neon Demo](https://codepen.io/Chokcoco/pen/zYoEaaq)

## 文字与背景

CSS 中的背景 background，也提供了一些属性用于增强文字的效果。

### background-clip 与文字

背景中有个属性为 `background-clip`， 其作用就是**设置元素的背景（背景图片或颜色）的填充规则**。

与 `box-sizing` 的取值非常类似，通常而言，它有 3 个取值，`border-box`，`padding-box`，`content-box`，后面规范新增了一个 `background-clip`。时至今日，部分浏览器仍需要添加前缀 webkit 进行使用 `-webkit-background-clip`。

使用了这个属性的意思是，以区块内的文字作为裁剪区域向外裁剪，文字的背景即为区块的背景，文字之外的区域都将被裁剪掉。

看个最简单的 Demo ，没有使用 `background-clip:text` :

```HTML
<div>Clip</div>

<style>
div {
  font-size: 180px;
  font-weight: bold;
  color: deeppink;
  background: url($img) no-repeat center center;
  background-size: cover;
}
</style>
复制代码
```

效果如下：

![image](media/Untitled 2/c3ddca9f36a3454a9309d8b4f1ddef42~tplv-k3u1fbpfcp-zoom-1.image)

[CodePen Demo](https://codepen.io/Chokcoco/pen/WjOBzB)

#### 使用 `background-clip:text`

我们稍微改造下上面的代码，添加 `-webkit-background-clip:text`：

```CSS
div {
  font-size: 180px;
  font-weight: bold;
  color: deeppink;
  background: url($img) no-repeat center center;
  background-size: cover;
  background-clip: text;
}
复制代码
```

效果如下：

![image](media/Untitled 2/e1e163f0fd7d4d089bc803477606f6f7~tplv-k3u1fbpfcp-zoom-1.image)

[CodePen Demo](https://codepen.io/Chokcoco/pen/eWRaVJ)

看到这里，可能有人就纳闷了，这不就是文字设置 `color` 属性嘛。

别急，由于文字设置了颜色，挡住了 div 块的背景，如果将文字设置为透明呢？文字是可以设置为透明的 `color: transparent` 。

```CSS
div {
  color: transparent;
  background-clip: text;
}
复制代码
```

效果如下：

![image](media/Untitled 2/e14df22f432848f6b37aa87679a9763e~tplv-k3u1fbpfcp-zoom-1.image)

[CodePen Demo](https://codepen.io/Chokcoco/pen/oWwRmE)

通过将文字设置为透明，原本 div 的背景就显现出来了，而文字以外的区域全部被裁剪了，这就是 `background-clip:text` 的作用。

### 利用 `background-clip` 图文搭配

这样，我们可以选取合适的图片合适的字体，实现任意风格的文字效果。

![image](media/Untitled 2/05c8e8ccc7be4cb088e66297c1a3b4b2~tplv-k3u1fbpfcp-zoom-1.image)

[CodePen Demo -- background-clip: text & Image text](https://codepen.io/Chokcoco/pen/wvopKEL)

又或者，利用这个效果实现一张创意海报：

### 利用 `background-clip` 实现渐变文字

再者，利用这个属性，也可以轻松的实现渐变色的文字：

```CSS
{
    background: linear-gradient(45deg, #009688, yellowgreen, pink, #03a9f4, #9c27b0, #8bc34a);
    background-clip: text;
}
复制代码
```

![img](media/Untitled 2/59c0e8a8a6dc47fe88b609bf1dadc399~tplv-k3u1fbpfcp-zoom-1.image)

配合 `background-position` 或者 `filter: hue-rotate()`，让渐变动起来：

```CSS
{
    background: linear-gradient(45deg, #009688, yellowgreen, pink, #03a9f4, #9c27b0, #8bc34a);
    background-clip: text;
    animation: huerotate 5s infinite;
}

@keyframes huerotate {
    100% {
        filter: hue-rotate(360deg);
    }
}
复制代码
```

![img](media/Untitled 2/48ec2dcb9e8d49c18917fa455af36fba~tplv-k3u1fbpfcp-zoom-1.image)

[CodePen Demo -- background-clip: text 文字渐变色](https://codepen.io/Chokcoco/pen/PmjMwz)

### 利用 `background-clip` 给文字增加高光动画

利用 `background-clip`， 我们还可以轻松的给文字增加高光动画。

譬如这样：

![img](media/Untitled 2/37bdbdf3faa04fcda2c892e4117e046f~tplv-k3u1fbpfcp-zoom-1.image)

其本质也是利用了 `background-clip`，伪代码如下：

```HTML
<p data-text="Lorem ipsum dolor"> Lorem ipsum dolor </p>
复制代码
p {
    position: relative;
    color: transparent;
    background-color: #E8A95B;
    background-clip: text;
}
p::after {
    content: attr(data-text);
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    background-image: linear-gradient(120deg, transparent 0%, transparent 6rem, white 11rem, transparent 11.15rem, transparent 15rem, rgba(255, 255, 255, 0.3) 20rem, transparent 25rem, transparent 27rem, rgba(255, 255, 255, 0.6) 32rem, white 33rem, rgba(255, 255, 255, 0.3) 33.15rem, transparent 38rem, transparent 40rem, rgba(255, 255, 255, 0.3) 45rem, transparent 50rem, transparent 100%);
    background-clip: text;
    background-size: 150% 100%;
    background-repeat: no-repeat;
    animation: shine 5s infinite linear;
}
@keyframes shine {
	0% {
		background-position: 50% 0;
	}
	100% {
		background-position: -190% 0;
	}
}
复制代码
```

去掉伪元素的 `background-clip: text`，就能看懂原理：

![img](media/Untitled 2/7d530349ec3c4ef9acf8cbf8cf33aa1e~tplv-k3u1fbpfcp-watermark.image)

[CodePen Demo -- shine Text && background-clip](https://codepen.io/Chokcoco/pen/OJbEOmb)

### `mask` 与文字

还有一个与背景相关的属性 -- `mask` 。

之前在多篇文章都提到了 `mask`，比较详细的一篇是 -- [奇妙的 CSS MASK](https://github.com/chokcoco/iCSS/issues/80)，本文不对 mask 的基本概念做过多讲解，向下阅读时，如果对一些 mask 的用法感到疑惑，可以再去看看。

只需要记住核心的，使用 `mask` 最重要结论就是：**添加了 mask 属性的元素，其内容会与 mask 表示的渐变的 transparent 的重叠部分，并且重叠部分将会变得透明。**

利用 `mask`，我们可以实现各种文字的出场特效：

```HTML
<div>
    <p>Hello MASK</p>
</div>
复制代码
```

核心的 CSS 代码：

```CSS
div {
    mask: radial-gradient(circle at 50% 0%, #000, transparent 30%);
    animation: scale 6s infinite;
}
@keyframes scale {
    0% {
        mask-size: 100% 100%;
    }
    60%,
    100% {
        mask-size: 150% 800%;
    }
}
复制代码
```

![img](media/Untitled 2/c3d14532a7e94903af6b77a07830ef5a~tplv-k3u1fbpfcp-zoom-1.image)

改变不同的方向，又或者是这样：

![img](media/Untitled 2/2b6c310ad968423699cef8d3b460e29c~tplv-k3u1fbpfcp-zoom-1.image)

[CodePen Demo -- MASK Text Effect](https://codepen.io/Chokcoco/pen/OJbxZLM)

## 文字与混合模式(mix-blend-mode)及滤镜(filter)

接下来，就到了非常有意思的混合模式及滤镜了。这两个属性给 CSS 世界增添了非常多的趣味性，活灵活用，会感叹 CSS 居然如此的强大美妙。

之前有多非常多篇关于**混合模式**及**滤镜**的文章，一些基础的用法就不再赘述。

### 给文字添加边框，生成镂空文字

在 CSS 中，我们可以利用 `-webkit-text-stroke`，给文字快速的添加边框，利用这个，可以快速生成镂空型的文字：

```CSS
p {
    -webkit-text-stroke: 3px #373750;
}
复制代码
```

![image](media/Untitled 2/c803c2792a064732aa9dc7d530c47b04~tplv-k3u1fbpfcp-zoom-1.image)

当然，我们看到，用到的属性 `-webkit-text-stroke` 带了 `webkit` 前缀，存在一定的兼容性问题。

所以，在更早的时候，我们还会使用 `text-shadow`，生成镂空文字。

```CSS
p {
    text-shadow: 0 0 5px #fff;
}
复制代码
```

![img](media/Untitled 2/e54e338e644e4e06a51ceab30c4321c1~tplv-k3u1fbpfcp-zoom-1.image)

可以看到，因为使用的是阴影，所以有很明显的虚化的感觉，存在一定的瑕疵。

还有一种非常绕的方法，利用混合模式加上滤镜，也能生成镂空文字。

```CSS
p {
    position: relative;
    color: #fff;

    &::after {
        content: 'Magic Text';
        position: absolute;
        left: 0;
        top: 0;
        color: #fff;
        mix-blend-mode: difference;
        filter: blur(1px);
    }
}
复制代码
```

![img](media/Untitled 2/ae1ee722a08a409087dd71d252c59978~tplv-k3u1fbpfcp-zoom-1.image)

这里利用 `filter: blur(1px)` 生成了一个比原字体稍微大一点点的字体覆盖在原字体之上，再利用 `mix-blend-mode: difference` 消除掉了同色的部分，只留下了利用模糊滤镜多出来的那一部分。

> `mix-blend-mode: difference`: 差值模式（Difference），作用是查看每个通道中的颜色信息，比较底色和绘图色，用较亮的像素点的像素值减去较暗的像素点的像素值。与白色混合将使底色反相；与黑色混合则不产生变化。

示意动图如下：

![img](media/Untitled 2/ba3c51c30036410a94beb630b285e7b1~tplv-k3u1fbpfcp-watermark.image)

[CodePen Demo -- Hollow TEXT EFFECT](https://codepen.io/Chokcoco/pen/ZEBoEzM?editors=1100)

### 利用混合模式，生成渐变色镂空文字

好，回到上面的 `-webkit-text-stroke`，拿到了镂空文字后，我们还可以利用混合模式 `mix-blend-mode: multiply` 生成渐变色的文字。

> `mix-blend-mode: multiply`: 正片叠底（multiply），将两个颜色的像素值相乘，然后除以255得到的结果就是最终色的像素值。通常执行正片叠底模式后的颜色比原来两种颜色都深。任何颜色和黑色正片叠底得到的仍然是黑色，任何颜色和白色执行正片叠底则保持原来的颜色不变，而与其他颜色执行此模式会产生暗室中以此种颜色照明的效果。

```CSS
p {
    position: relative;
    -webkit-text-stroke: 3px #9a9acc;

    &::before{
		content: ' ';
		width: 100%;
		height: 100%;
		position: absolute;
		left: 0;
		top: 0;
		background-image: linear-gradient(45deg, #ff269b, #2ab5f5, #ffbf00);
		mix-blend-mode: multiply;
	}
}
复制代码
```

![img](media/Untitled 2/b6cfef1703104771ab07fa4fbdb8a2a3~tplv-k3u1fbpfcp-zoom-1.image)

在这里，`mix-blend-mode: multiply` 发挥的作用和 mask 非常的类似，我们其实是生成了一幅渐变图案，但是只有在文字轮廓内，渐变颜色才会显现。

当然，上述效果和整体的黑色底色也是有关系的。

示意图如下：

![img](media/Untitled 2/34e134561692470b936af3ecf15e4288~tplv-k3u1fbpfcp-zoom-1.image)

[CodePen Demo -- Hollow TEXT EFFECT](https://codepen.io/Chokcoco/pen/Rwoybzr)

### 利用混合模式，生成光影效果文字

OK，在上述的基础上，我们可以继续叠加混合模式，这次我们利用剩余的一个 `::after` 伪类，再添加一个 `mix-blend-mode: color-dodge` 混合模式，给文字加上最后的点缀，实现美妙的光影效果。

> `mix-blend-mode: color-dodge`: 颜色减淡模式（Color Dodge），查看每个通道的颜色信息，通过降低“对比度”使底色的颜色变亮来反映绘图色，和黑色混合没变化。。

核心的伪代码：

```CSS
p {
    position: relative;
    -webkit-text-stroke: 3px #7272a5;

    &::before {
		content: ' ';
		background-image: linear-gradient(45deg, #ff269b, #2ab5f5, #ffbf00);
		mix-blend-mode: multiply;
    }

    &::after {
        content: "";
        position: absolute;
        background: radial-gradient(circle, #fff, #000 50%);
        background-size: 25% 25%;
        mix-blend-mode: color-dodge;
        animation: mix 8s linear infinite;
    }
}

@keyframes mix {
    to {
        transform: translate(50%, 50%);
    }
}
复制代码
```

看看效果：

![img](media/Untitled 2/c0c1146b931f455f85eb9b05222e1034~tplv-k3u1fbpfcp-zoom-1.image)

这里就要感叹 `mix-blend-mode: color-dodge` 的神奇了，去掉这个混合模式，其实是这样的：

![img](media/Untitled 2/a24af4e77b1a4bf69f279c59bb7faca1~tplv-k3u1fbpfcp-watermark.image)

[CodePen -- Hollow TEXT EFFECT](https://codepen.io/Chokcoco/pen/Rwoybzr)

好，就上面这个效果，还可以继续吗？答案是可以的。限于篇幅，本文不再继续在这个效果上深入，感兴趣的可以拿着上面的 DEMO 自己再捣鼓捣鼓。

### 利用混合模式实现文字与底色反色的效果

这里还是利用 `mix-blend-mode: difference` 差值模式，实现一种文字与底色反色的 Title 效果。

> `mix-blend-mode: difference`: 差值模式（Difference），作用是查看每个通道中的颜色信息，比较底色和绘图色，用较亮的像素点的像素值减去较暗的像素点的像素值。与白色混合将使底色反相；与黑色混合则不产生变化。

代码非常的简单，我们实现一个黑白相间的背景，文本的颜色为白色，配合上差值模式，即可实现黑底上的文字为白色，白底上的文字为黑色的效果。

```CSS
p  {
    background: repeating-radial-gradient(circle at 200% 200%, #000 0, #000 150px, #fff 150px, #fff 300px);

    &::before {
        content: "LOREM IPSUM";
        color: #fff;
        mix-blend-mode: difference;
    }
}
复制代码
```

可以用于一些标题效果：

![image](media/Untitled 2/fb9a2ad31b5c44b9be15db9c5fab71db~tplv-k3u1fbpfcp-zoom-1.image)

[CodePen Demo -- Radial-gradient + Mix-blend-mode](https://codepen.io/Chokcoco/pen/GRpzYxE)

### 利用混合模式实现动态类抖音风格 Glitch 效果

OK，接下来，我们再尝试下其他混合模式的搭配。在 [CSS 故障艺术](https://github.com/chokcoco/iCSS/issues/78) 一文中，提到了一种故障艺术。

什么是故障艺术？我们熟知的抖音的 LOGO 正是故障艺术其中一种表现形式。它有一种魔幻的感觉，看起来具有闪烁、震动的效果，很吸引人眼球。

#### 关键点

- 利用 mix-blend-mode: lighten 混合模式实现两段文字结构重叠部分为白色
- 利用元素位移完成错位移动动画，形成视觉上的冲击效果

看看效果：

![img](media/Untitled 2/10f4f62d4d424a4ba3c3d601324113c5~tplv-k3u1fbpfcp-watermark.image)

本文篇幅有点长，代码就不上了，完整 DEMO 在这里：

[类抖音 LOGO 文字故障效果](https://chokcoco.github.io/CSS-Inspiration/#/./blendmode/blend-text-glitch.md)

当然，我们也不是一定要使用混合模式去使得融合部分为白色，可以仅仅是使用这个配色效果，基于上面效果的另外一个版本，没有使用混合模式。

#### 关键点

- 利用了伪元素生成了文字的两个副本
- 视觉效果由位移、遮罩、混合模式完成
- 配色借鉴了抖音 LOGO 的风格

![img](media/Untitled 2/ca38248a6a4d423e919d6bbcb8a07412~tplv-k3u1fbpfcp-zoom-1.image)

完整 DEMO 在这里：

[CSS文字故障效果](https://chokcoco.github.io/CSS-Inspiration/#/./others/word-glitch.md)

仅仅使用配色没有使用混合模式的好处在于，对于每一个文字的副本，有了更大的移动距离和可以处理的空间。

### Glitch Art 风格的 404 效果

稍微替换一下文本文案为 404，再添加上一些滤镜效果（`hue-rotate()`、`blur()`）嘿嘿，找到了一个可能实际可用的场景：

效果一：

![404](media/Untitled 2/e2367737b2dc417ab58342f9898636e1~tplv-k3u1fbpfcp-zoom-1.image)

效果二：

![404](media/Untitled 2/42d32d06ddce4170a9efc0f7ffa8da44~tplv-k3u1fbpfcp-zoom-1.image)

两个 404 效果的 Demo 如下：

- [CodePen -- CSS 404故障效果](https://codepen.io/Chokcoco/pen/OJPexEm)
- [CodePen -- 404故障效果](https://codepen.io/Chokcoco/pen/QWwXqra)

> 小技巧，在使用混合模式的时，有的时候，效果不希望和背景混合在一起，可以使用 `isolation: isolate` 进行隔离。

### 使用滤镜生成文字融合效果

在 [你所不知道的 CSS 滤镜技巧与细节](https://github.com/chokcoco/iCSS/issues/30) 一文中，介绍了利用滤镜实现的一种融合效果。

利用了**模糊滤镜叠加对比度滤镜产生的融合效果**。

单独将两个滤镜拿出来，它们的作用分别是：

1. `filter: blur()`： 给图像设置高斯模糊效果。
2. `filter: contrast()`： 调整图像的对比度。

但是，当他们“合体”的时候，产生了奇妙的融合现象。简单的例子：

![img](media/Untitled 2/cfb9df41bb2e44429a391546899195a1~tplv-k3u1fbpfcp-zoom-1.image)

[CodePen Demo -- filter mix between blur and contrast](https://codepen.io/Chokcoco/pen/QqWBqV)

利用这个特性，可以实现一些文字融合的效果：

利用这个方法，我们还可以设计一些文字融合的效果：

![img](media/Untitled 2/6e9ae9ce7b9843f089f9960d4bb8ffad~tplv-k3u1fbpfcp-zoom-1.image)

![img](media/Untitled 2/cb2de2337d704204a19bdae51f366339~tplv-k3u1fbpfcp-zoom-1.image)

具体实现你可以看这里：

[CodePen Demo -- word animation | word filter](https://codepen.io/Chokcoco/pen/jLjNRj)

## 文字与 SVG

最后，我们再来看看文字与 SVG。

在 SVG 与 CSS 的搭配中，有一类非常适合拿来做动画的属性，也就是 `stroke-*` 相关的几个属性，利用它们，我们只需要掌握简单的 SVG 语法，就可以快速制作相关的线条动画。

我们利用 SVG 中几个和边框、线条相关的属性，来实现文字的线条动画，下面罗列一下，其实大部分和 CSS 对比一下非常好理解，只是换了个名字：

- stroke-width：类比 css 中的 border-width，给 svg 图形设定边框宽度；
- stroke：类比 css 中的 border-color，给 svg 图形设定边框颜色；
- stroke-linejoin | stroke-linecap：设定线段连接处的样式；
- stroke-dasharray：值是一组数组，没数量上限，每个数字交替表示划线与间隔的宽度；
- stroke-dashoffset：则是虚线的偏移量

> 具体的更深入的介绍，可以看看这篇：[【Web动画】SVG 线条动画入门](https://www.cnblogs.com/coco1s/p/6225973.html)

### 线条文字动画

接下来，我们利用 `stroke-*` 相关属性，实现一个简单的线条文字动画。

```HTML
<svg viewBox="0 0 400 200">
	<text x="0" y="70%"> Lorem </text>
</svg>	
复制代码
svg text {
	animation: stroke 5s infinite alternate;
	letter-spacing: 10px;
	font-size: 150px;
}
@keyframes stroke {
	0% {
		fill: rgba(72, 138, 20, 0);
		stroke: rgba(54, 95, 160, 1);
		stroke-dashoffset: 25%;
		stroke-dasharray: 0 50%;
		stroke-width: 1;
	}
	70% {
		fill: rgba(72, 138, 20, 0);
		stroke: rgba(54, 95, 160, 1);
		stroke-width: 3;
	}
	90%,
	100% {
		fill: rgba(72, 138, 204, 1);
		stroke: rgba(54, 95, 160, 0);
		stroke-dashoffset: -25%;
		stroke-dasharray: 50% 0;
		stroke-width: 0;
	}
}
复制代码
```

动画的核心就是，利用动态变化文字的 `stroke-dasharray` 和 `stroke-dashoffset` 形成视觉上的线条变换，动画的最后再给文字上色。看看效果：

![img](media/Untitled 2/7d5f2cb8e9ac4c0695597b386014412b~tplv-k3u1fbpfcp-watermark.image)

[CodePen Demo -- SVG Text Line Effect](https://codepen.io/Chokcoco/pen/dyOjMMd)

## 最后

本文介绍了一些我认为比较有意思的文字动画小技巧，当然 CSS 中还有非常多有意思的文字效果，限于篇幅，不一一展开。

本文到此结束，希望对你有帮助 :)，想 Get 到最有意思的 CSS 资讯，千万不要错过我的公众号 -- **iCSS前端趣闻** 😄

更多精彩 CSS 技术文章汇总在我的 [Github -- iCSS](https://github.com/chokcoco/iCSS) ，持续更新，欢迎点个 star 订阅收藏。

如果还有什么疑问或者建议，可以多多交流，原创文章，文笔有限，才疏学浅，文中若有不正之处，万望告知。


作者：chokcoco
链接：https://juejin.cn/post/6937102296442470413
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。