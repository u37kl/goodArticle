# 从Chrome源码看浏览器如何layout布局

[2017年2月26日](https://www.rrfed.com/2017/02/26/chrome-layout/)[会编程的银猪](https://www.rrfed.com/author/yincheng/)

假设有以下html/css：

| 1    | **<div** style="border:1px solid #000; width:50%; height: 100px; margin: 0 auto"**>****</div>** |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

这在浏览器上面将显示一个框：

为了画出这个框，首先要知道从哪里开始画、画多大，其次是边缘stroke的颜色，就可以把它画出来了：

| 1234567 | **void** draw(SkCanvas* canvas) {  SkPaint paint;  paint.setStrokeWidth(1);  *//从位置为(200, 200)的地方开始画，宽度为400，高度为100*  SkRect rect = SkRect::MakeXYWH(200, 200, 400, 100);  canvas->drawRect(rect, paint);} |
| ------- | ------------------------------------------------------------ |
|         |                                                              |

上面是用Skia画的代码，[Skia](https://skia.org/)是一个跨平台的开源2D图形库，是Chrome/firefox/android采用的底层Paint引擎。

为了能够获取到具体的值，就得进行layout。什么叫layout？把css转化成维度位置等可直接用来描绘的信息的过程就叫layout，如下Chrome源码对layout的解释：

| 123  | *// The purpose of the layout tree is to do layout (aka reflow) and store its**// results for painting and hit-testing. Layout is the process of sizing and**// positioning Nodes on the page.* |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

 

《[从Chrome源码看浏览器如何计算CSS](http://www.rrfed.com/2017/02/22/chrome-css/)》这篇文章介绍了怎么把css转化成ComputedStyle，上面的div，它被转化后的style如下所示：

![img](media/Untitled/1-20210120004323462.png)

width的大小是50，类型是百分比，而margin值是0，类型是auto，这两种都不能直接用来画的。所以需要通过layout计算出具体的数字。

##  1. 建立layout树

《[从Chrome源码看浏览器如何构建DOM树](http://www.rrfed.com/2017/01/30/chrome-build-dom/)》这篇文章介绍了如何html文本的过程。当解析完收到的html片段后，会触发Layout Tree的构建：

| 123  | **void** Document::finishedParsing() {   updateStyleAndLayoutTree();} |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

每个非display:none/content的Node结点都会相应地创建一个LayoutObject，如下blink源码的注释：

| 12   | *// Also some Node don't have an associated LayoutObjects e.g. if display: none**// or display: contents is set.* |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

并建立起它们的父子兄弟关系：

| 12   | LayoutObject* newLayoutObject = m_node->createLayoutObject(style);parentLayoutObject->addChild(newLayoutObject, nextLayoutObject); |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

形成一棵独立的layout树。

当layout树建立好之后，紧接着用style计算layout的值。

## 2. 计算layout值

以上面的div为例，它需要计算它的宽度和margin。

### （1）计算宽度

宽度的计算是根据数值的类型：

| 123456789 | **switch** (length.type()) { **case** Fixed:  **return** LayoutUnit(length.**value**()); **case** Percent:  *// Don't remove the extra cast to float. It is needed for rounding on*  *// 32-bit Intel machines that use the FPU stack.*  **return** LayoutUnit(    **static_cast**<**float**>(maximumValue * length.percent() / 100.0f));} |
| --------- | ------------------------------------------------------------ |
|           |                                                              |

如上所示，如果是Fixed，则直接返回一个LayoutUnit封装的数据，1px = `1 << 6` = 64 unit，这也是Blink存储的精度。从这里可以看到，设置小数的px其实是有用的。

如果是Percent百分比，则用百分比乘以最大值，而这个最大值是用容器传进来的宽度。

### （2）计算margin值

上面的div的margin给它设置了margin: 0 auto，需要计算实际的数字。blink会检测两边是不是都为auto，如果是的话就认为是居中：

| 123456789101112 | *// CSS 2.1: "If both 'margin-left' and 'margin-right' are 'auto', their used**// values are equal. This horizontally centers the element with respect to**// the edges of the containing block."*const ComputedStyle& containingBlockStyle = containingBlock->styleRef();**if** (marginStartLength.isAuto() && marginEndLength.isAuto()) { LayoutUnit centeredMarginBoxStart = std::max(   LayoutUnit(),   (availableWidth - childWidth) / 2);  marginStart = centeredMarginBoxStart; marginEnd = availableWidth - childWidth - marginStart; **return**;} |
| --------------- | ------------------------------------------------------------ |
|                 |                                                              |

上面第8行用容器的宽度减掉本身的宽度，然后除以2就得到margin-left，接着用容器的宽度减掉本身的宽度和margin-left就得到margin-right。为什么margin-right还要再算一下，因为上面的代码是删减版的，它还有另外一种情况要处理，这里不是很重要，被我省掉了。

margin和width算好了，便把它放到layoutObject结点的盒模型数据结构里面：

| 12   | m_frameRect.setWidth(width);m_marginBoxOutsets.setStart(marginLeft); |
| ---- | ------------------------------------------------------------ |
|      |                                                              |



### （3）盒模型数据结构

在blink的源码注释里面，很形象地画出了盒模型图：

```
// ***** THE BOX MODEL *****
// The CSS box model is based on a series of nested boxes:
// http://www.w3.org/TR/CSS21/box.html
//
//       |----------------------------------------------------|
//       |                                                    |
//       |                   margin-top                       |
//       |                                                    |
//       |     |-----------------------------------------|    |
//       |     |                                         |    |
//       |     |             border-top                  |    |
//       |     |                                         |    |
//       |     |    |--------------------------|----|    |    |
//       |     |    |                          |    |    |    |
//       |     |    |       padding-top        |####|    |    |
//       |     |    |                          |####|    |    |
//       |     |    |    |----------------|    |####|    |    |
//       |     |    |    |                |    |    |    |    |
//       | ML  | BL | PL |  content box   | PR | SW | BR | MR |
//       |     |    |    |                |    |    |    |    |
//       |     |    |    |----------------|    |    |    |    |
//       |     |    |                          |    |    |    |
//       |     |    |      padding-bottom      |    |    |    |
//       |     |    |--------------------------|----|    |    |
//       |     |    |                      ####|    |    |    |
//       |     |    |     scrollbar height ####| SC |    |    |
//       |     |    |                      ####|    |    |    |
//       |     |    |-------------------------------|    |    |
//       |     |                                         |    |
//       |     |           border-bottom                 |    |
//       |     |                                         |    |
//       |     |-----------------------------------------|    |
//       |                                                    |
//       |                 margin-bottom                      |
//       |                                                    |
//       |----------------------------------------------------|
//
// BL = border-left
// BR = border-right
// ML = margin-left
// MR = margin-right
// PL = padding-left
// PR = padding-right
// SC = scroll corner (contains UI for resizing (see the 'resize' property)
// SW = scrollbar width
```



上面的盒模型耳熟能详，不太一样的是，它还把滚动条给画出来了。

这个盒模型border及其以内区域是用一个LayoutRect m_frameRect对象表示的：

```
*// The CSS border box rect for this box.**//**// The rectangle is in this box's physical coordinates.**// The location is the distance from this**// object's border edge to the container's border edge (which is not**// always the parent). Thus it includes any logical top/left along**// with this box's margins.*LayoutRect m_frameRect;
```



上面源码注释说得很明白，意思是说这个LayoutRect的位置是从它本身的边到容器的边的距离，因此它的距离/位置包含了margin值和left/top的位移偏差。LayoutRect记录了一个盒子的位置和大小：

| 12   | LayoutPoint m_location; LayoutSize m_size; |
| ---- | ------------------------------------------ |
|      |                                            |

上面（1）和（2）计算好宽度后就去设置这个大小，保存起来。

可以在源码里面看到用这个对象对处理的一些获取宽度的方式，如clientWidth：

| 123456 | *// More IE extensions. clientWidth and clientHeight represent the interior of**// an object excluding border and scrollbar.*LayoutUnit LayoutBox::clientWidth() const { **return** m_frameRect.width() - borderLeft() - borderRight() -     verticalScrollbarWidth();} |
| ------ | ------------------------------------------------------------ |
|        |                                                              |

clientWidth是除去border和scrollbar的宽度。

而offsetWidth是frameRect的宽度——算上border和scrollbar：

| 123  | *// IE extensions. Used to calculate offsetWidth/Height.*LayoutUnit offsetWidth() const override { **return** m_frameRect.width(); }LayoutUnit offsetHeight() const override { **return** m_frameRect.height(); } |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

 

Margin区域是用一个LayoutRectOutsets表示的，这个对象记录了margin的上下左右值：

| 1234 | LayoutUnit m_top; LayoutUnit m_right; LayoutUnit m_bottom; LayoutUnit m_left; |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

 

上面已经分析宽高的计算，还差位置的计算。

### （4）位置计算

位置计算就是要算出x和y或者说left和top的值，这两个值分别在下面两个函数计算得到：

| 1234567 | *// Now determine the correct ypos based off examination of collapsing margin**// values.*LayoutUnit logicalTopBeforeClear =  collapseMargins(**child**, layoutInfo, childIsSelfCollapsing,          childDiscardMarginBefore, childDiscardMarginAfter);*// Now place the child in the correct left position*determineLogicalLeftPositionForChild(**child**); |
| ------- | ------------------------------------------------------------ |
|         |                                                              |

用以下html做为例子：

| 12345678910 | *<!DOCType html>***<html>****<head>****</head>****<body>**  **<div** id="div-1" style="border:5px solid #000; width:50%; height: 100px; margin: 0 auto;"**>****</div>**  **<div** id="div-2" style="margin: 50px; padding:80px; border: 20px solid"**>**    **<div** id="div-3" style="margin:15px"**>**hello, world**</div>**  **</div>****</body>****</html>** |
| ----------- | ------------------------------------------------------------ |
|             |                                                              |

我先把计算出来的结果打印出来，如下所示：

> [LayoutBlockFlow.cpp(925)] location is: “190.25”, “0” size is “400.5”, “110”  (div-1)
>
> [LayoutBlockFlow.cpp(925)] location is: “115”, “115” size is “451”, “18”      (div-3)
>
> [LayoutBlockFlow.cpp(925)] location is: “50”, “160” size is “681”, “248”     (div-2)
>
> [LayoutBlockFlow.cpp(925)] location is: “8”, “8” size is “781”, “408”        (body)
>
> [LayoutBlockFlow.cpp(925)] location is: “0”, “0” size is “797”, “466”        (html)

由于它是一个递归的过程，所以上面打印的顺序是由子元素到父元素的。以div-2为例算一下，它的x = 50, y = 160：因为div-1占据的空间为h = border * 2 + height = 5 * 2 + 100 = 110，并且div-2有一个margin-top = 50，所以div-2的y = 110 + 50 = 160.

对于div-3，由于div-2有一个80px的padding和20px的border，同时它自己本身有一个15px的margin，所以div-3的y = 50 + 20 + 15 = 115.

如果把行内元素也打印出来，那么结果是这样的：

> [LayoutBlockFlowLine.cpp(1997)] inline location is: “0”, “0” size is “400.5”, “10”  (div-1 content)
>
> [LayoutBlockFlow.cpp(925)] location is: “190.25”, “0” size is “400.5”, “110”
>
> [LayoutBlockFlowLine.cpp(1997)] inline location is: “0”, “115” size is “451”, “18”   (div-3 text)
>
> [LayoutBlockFlow.cpp(925)] location is: “115”, “115” size is “451”, “18”
>
> …（后面一样）

第三行是div-3的文本节点创建的layoutObject，它的行高是18px，所以它的size高度是18px。

这里可以看到块级元素间的空白节点不会产生layoutObject，这在代码里面可以找到佐证：



| 1234567891011 | **bool** Text::textLayoutObjectIsNeeded(const ComputedStyle& style,                  const LayoutObject& **parent**) const { **if** (!length())  **return** **false**; **if** (style.display() == EDisplay::None)  **return** **false**; **if** (!containsOnlyWhitespace())  **return** **true**;  *//其它判断*} |
| ------------- | ------------------------------------------------------------ |
|               |                                                              |

上面代码第7行，如果Text结点含有非空白字符，则马上返回true，否则的话继续判断：

| 123  | **if** (**parent**.isLayoutBlock() && !**parent**.childrenInline() &&    (!prev \|\| !prev->isInline()))   **return** **false**; |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

第二行——如果存在上一个相邻结点，并且这个结点不是行内元素则返回false，不创建layout对象。

所以在块级元素后面的空白文本结点将不会参与渲染，这个就解释了为什么块**级元素后的换行不会被转换成一个空格**。在源码里面还可以看到，块级元素内的开头空白字符将会被忽略：

| 12   | *// Whitespace at the start of a block just goes away. Don't even**// make a layout object for this text.* |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

 

这里有个问题，为什么它要递归地算，即先算子元素的再回过头来算父元素呢？因为有些属性必须得先知道子元素的才能知道父元素，例如父元素的高度是子元素撑起的，但是有些属性要先知道父元素的才能算子元素的，例如子元素的宽度是父元素的50%。所以在计算子元素之前会先把当前元素的layout计算一下，然后再传给子元素，子元素计算好之后会返回父元素是否需要重新layout，如下：

| 123456 | *// Use the estimated block position and lay out the child if needed. After* *// child layout, when we have enough information to perform proper margin* *// collapsing, float clearing and pagination, we may have to reposition and* *// lay out again if the estimate was wrong.* **bool** childNeededLayout =   positionAndLayoutOnceIfNeeded(**child**, logicalTopEstimate, layoutInfo); |
| ------ | ------------------------------------------------------------ |
|        |                                                              |

具体的计算过程，这里举一两个例子，例如计算left值时，会先取父元素的border-left和padding-left作为起始位置，然后再加上它自己的margin-left就得到它的x/left值。

| 12345678 | **void** LayoutBlockFlow::determineLogicalLeftPositionForChild(LayoutBox& **child**) { LayoutUnit startPosition = borderStart() + paddingStart(); LayoutUnit initialStartPosition = startPosition;  LayoutUnit childMarginStart = marginStartForChild(**child**); LayoutUnit newPosition = startPosition + childMarginStart; *//other code*} |
| -------- | ------------------------------------------------------------ |
|          |                                                              |

我们知道浮动的规则比较复杂，所以相应的计算也比较复杂，我们简单研究一下。

### （5）浮动

用以下三栏布局作为说明：

| 12345 | **<div>**  **<div** style="float:left"**>**hello, world**</div>**  **<div** style="float:right"**>****<p** style="width:100px"**>****</p>****</div>**  **<div** style="margin:0 100px;"**>****</div>****</div>** |
| ----- | ------------------------------------------------------------ |
|       |                                                              |

先来看宽度的计算，对于第一个float: left的div，首先它会先判断一下宽度是否需要fit content：

| 123456 | **bool** LayoutBox::sizesLogicalWidthToFitContent(  const Length& logicalWidth) const { **if** (isFloating() \|\| isInlineBlockOrInlineTable())  **return** **true**; *//other code*} |
| ------ | ------------------------------------------------------------ |
|        |                                                              |

如果它是浮动的或者是inlne-block，则需要宽度适应内容。由于子元素是一个行内文本，它需要计算这个行内元素的宽度，计算的规则非常复杂，这里我把部分注释说明贴出来：

| 12345678910 | *// (3) A text object. Text runs can have breakable characters at the**//   start, the middle or the end. They may also lose whitespace off the**//   front if we're already ignoring whitespace. In order to compute**//   accurate min-width information, we need three pieces of**//   information.**//   (a) the min-width of the first non-breakable run. Should be 0 if**//     the text string starts with whitespace.**//   (b) the min-width of the last non-breakable run. Should be 0 if the**//     text string ends with whitespace.**//   (c) the min/max width of the string (trimmed for whitespace).* |
| ----------- | ------------------------------------------------------------ |
|             |                                                              |

第二个浮动的div，它的子元素是一个p标签，并且它已经指定了宽度。它会去算子元素的宽度加上margin值的宽度，还要判断是否为浮动，循环所有子元素处理，取一个最大值。

 

再来看位置的计算，计算位置的代码还是能够稍微看出点苗头，例如对float: left的计算：

| 12345678910111213141516 | *//如果当前行的剩余空间小于float的宽度，则循环条件成立***while** (logicalRightOffsetForPositioningFloat(      logicalTopOffset, logicalRightOffset, &heightRemainingRight) -      floatLogicalLeft <    floatLogicalWidth) { *//往下挪* logicalTopOffset +=   std::min<LayoutUnit>(heightRemainingLeft, heightRemainingRight); *//计算新的float left位置* floatLogicalLeft = logicalLeftOffsetForPositioningFloat(   logicalTopOffset, logicalLeftOffset, &heightRemainingLeft); }}*//循环结束，找到位置*floatLogicalLeft = std::max(  logicalLeftOffset - borderAndPaddingLogicalLeft(), floatLogicalLeft); |
| ----------------------- | ------------------------------------------------------------ |
|                         |                                                              |

上面它会先判断当前行剩余空间是否小于浮动元素的宽度，如果是的话就一直往下挪。

 

通过上面的层层计算，就可以拿到位置坐标和具体大小，上面两个浮动的div最后计算的结果是：

> [LayoutBlockFlow.cpp(1475)] location is: “0”, “0” size is “77.3281”, “18”
>
> [LayoutBlockFlow.cpp(1475)] location is: “681”, “0” size is “100”, “16”

有了这些信息，结合颜色等style，就可进行Paint了。

 

## 3. Paint

Paint又是一块很复杂的东西，试图在一篇文章里面讲明layout都已经是一件不太可能的事情。

Paint的初始化会使用layout的数据，如下面的BoxPainter的构造函数：

| 1    | BoxPainter(const LayoutBox& layoutBox) : m_layoutBox(layoutBox) {} |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

Paint会调用最上面说的Skia的SkCanvas画：

| 1    | SkCanvas* canvas() { **return** m_canvas; } |
| ---- | ------------------------------------------- |
|      |                                             |

 

这个SkCanvas和JS里面的canvas有什么联系和区别？

Blink JS里的canvas就是这个canvas，当在js里面获取canvas对象进行描绘时：

| 1234 | **var** canvas = document.getElementById("canvas"); **var** ctx = canvas.getContext("2d");ctx.fillStyle = "green";ctx.fillRect(10, 10, 100, 100) |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

就会去获取SkCanvas实例。

| 123  | SkCanvas* HTMLCanvasElement::drawingCanvas() const { **return** buffer() ? m_imageBuffer->canvas() : **nullptr**;} |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

所以不管是用html/css画，还是用canvas画，它们都是同宗同源的，区别就在于借助html/css比较直观简单，浏览器帮你进行layout。而直接用canvas就得从点线面一点一点地去画，但同时它的灵活度就比较大。

 

## 4. 触发layout

什么时候会触发layout，上文的分析已经提及当html片段解析完会触发layout。在上一篇也有提及加载完css后也会触发layout，同时resize页面的时候也会触发layout。因为layout的计算是比较复杂的，所以应减少layout的次数。例如CSS要写在head标签里面，不然写在body里面，一旦遇到新的CSS又会重新layout。

第二个是获取clientWidth/scrollTop等维度信息时，普遍的说法是会触发layout用于获取值，但是在笔者的观察下并没有触发layout：
如下使用getComputedStyle或者获取clientWidth应该会触发layout：

| 123  | **var** style = window.getComputedStyle(document.getElementById("body"));**var** width = style.width;console.log(document.getElementById("canvas").clientWidth) |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

但是无论是我打断点还是打log，都无法观察到layout的触发，而是直接去获取clientWidth的值：

| 1234 | LayoutUnit LayoutBox::clientWidth() const { **return** m_frameRect.width() - borderLeft() - borderRight() -     verticalScrollbarWidth();} |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

不过，改变它的clientWidth的时候是一定会触发layout的：

| 1    | document.getElementById("canvas").style.width = "500px"; |
| ---- | -------------------------------------------------------- |
|      |                                                          |

在layout的函数里面打印的Log：

![img](media/Untitled/3.png)

上面几行是重新计算CSS，下面几行是进行layout。

 

另外需要注意的是尽可能地减少layout的范围，如下的demo——当点击菜单按钮的时候把菜单给放出来：

| 123456789101112131415161718192021 | <style>  #menu,   #show-btn{    display: none;  }  .show-menu #menu,  .show-menu #show-btn{    display: block;  }</style>**<body>****<span** id="show-btn"**>**Menu**</span>****<nav** id="menu"**>**  **<ul>****<li>****</li>****</ul>****</nav>****</body>**<script>  document.getElementById("menu").onclick = **function**(){    document.body.addClass("show-menu");  };</script> |
| --------------------------------- | ------------------------------------------------------------ |
|                                   |                                                              |

上面为了图方便，给body添加了一个类，用这个类控制菜单的状态。但是这样会有很大的问题，因为给body添加了一个类导致它要重新计算style和layout，它一旦layout了，它的子元素也要跟着layout，也就是说整个页面都要重新layout。所以这个代价就很高了，我们应该缩小影响范围。因此，把show-menu的class加到直接相关的元素上面就好了。

 

至此，整一个页面渲染过程就介绍完毕了。我们从html -> CSS -> layout -> paint一步步分析了其中的过程，虽然介绍得不是很全面，但已经把核心的过程剖析了一遍。由于写html/css很多东西都是不透明，完全不知道背后是怎么工作的，只能是看文档说这个标签是怎么用的，那个属性会有什么效果，然后在浏览器上面看效果，有点任浏览器宰割的感觉。所以这个源码解读就是为了能够窥探浏览器背后工作原理，这样对写代码会有帮助，能够做到心中有数。当遇到一些比较困难的问题时，能够很快的找到解决方案或者解决的方向。

例如笔者就遇到一个奇芭的问题，就是使用`height: calc(100% - 80px)`的时候，在手机Safari上面展开某个子菜单时，偶现菜单滑不动的情况。当时就想很可能是在Safari在展开菜单时高度算错了，导致overflow: auto不管用。所以在展开菜单后再手手动计算和设置height，然后就解决问题了。

 

相关阅读：

1. [从Chrome源码看浏览器如何构建DOM树](http://www.rrfed.com/2017/01/30/chrome-build-dom/)
2. [从Chrome源码看浏览器的事件机制](http://www.rrfed.com/2017/02/05/browser-event/)
3. [从Chrome源码看浏览器如何计算CSS](http://www.rrfed.com/2017/02/22/chrome-css/)