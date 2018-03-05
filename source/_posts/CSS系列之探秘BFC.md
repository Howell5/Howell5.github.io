---
title: CSS系列之探秘BFC
date: 2017-03-14
---
> BFC很早就知道这个名词，但却没有真正的去了解它。只知道它能解决margin折叠和清除浮动的问题。这篇文章攫取其他文章和官方文档写成。如有错误，欢迎指出。

<!--more-->
## 定义
Google完，我们得到这样一句话。“A block formatting context is a part of a visual CSS rendering of a Web page. It is the region in which the layout of block boxes occurs and in which floats interact with each other”.
大概句义：“块格式化上下文（block formatting context） 是Web页面的可视CSS渲染的一部分。它是块盒子的布局发生及浮动体彼此交互的区域。” 
抓住关键词，它是CSS渲染的一块区域，在并且区域内部与区域外部交互有其的规则。具体查看 [MDN_BFC](https://developer.mozilla.org/en-US/docs/Web/Guide/CSS/Block_formatting_context) 。

## 触发BFC
- 根元素
根元素这个，很好理解。
html文档建立，就会触发根元素的BFC，我们在根元素内写几个div元素，会发现内部div垂直排列，本身没有兄弟元素，自然满足B、D、E、F这几条规则，内部块级盒子来自同一个BFC(html)，所以相邻margin会重叠，行内盒子横向就不会重叠。
- float属性不为none
当一个元素被设置为float:left or right,会触发这个元素，生成BFC区域，使他不仅拥有BFC的渲染规则，而且会给自身带来副作用，display:block,但是这个元素不会像行内元素通过设置display:block变为块级元素那样，宽度充满其父元素，而是表现的更像行内块级元素,只会使得行内元素不会收缩包裹其内容.
- position为absolute或fixed
这个触发器触发生成BFC后，margin重不重叠这个对于他都没有效果，可能定位流和一般的文档流浮动流不属于一个吧，，所以自然会与浮动元素重叠。
- display为inline-block, table-cell
基本上每一条渲染规则都满足
- overflow不为visible
这个对于来自不同BFC margin不会重叠这一条来说，不满足，其他都能满足
overflow:hidden通常是对父元素定义比较有效的。

## BFC的规则
- 内部的Box会在垂直方向，一个接一个地放置。
- Box垂直方向的距离由margin决定。属于同一个BFC的两个相邻Box的margin会发生重叠
- 每个元素的margin box的左边， 与包含块border box的左边相接触(对于从左往右的格式化，否则相反)。即使存在浮动也是如此。
- BFC的区域不会与float box重叠。
- BFC就是页面上的一个隔离的独立容器，容器里面的子元素不会影响到外面的元素。反之也如此。
- 计算BFC的高度时，浮动元素也参与计算

官方文档的说明见 [CSS2.1 spec](https://www.w3.org/TR/CSS2/visuren.html#block-formatting)
## BFC应用及原理

看了上面罗列的规则和触发条件，很难理解。我们拿几个例子来帮助理解。（例子均来自[蓝色天空_BFC](http://www.cnblogs.com/lhb25/p/inside-block-formatting-ontext.html))
### 1、自适应两栏布局
```
<style>
    body {
        width: 300px;
        position: relative;
    }
 
    .aside {
        width: 100px;
        height: 150px;
        float: left;
        background: #f66;
    }
 
    .main {
        height: 200px;
        background: #fcc;
    }
</style>
<body>
    <div class="aside"></div>
    <div class="main"></div>
</body>
```

![未完成效果](http://upload-images.jianshu.io/upload_images/2575359-cad958e92d9e3d9e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
造成这个现象的原因是由于上述第一条触发条件和第三条规则：
> 根元素

>  每个元素的margin box的左边， 与包含块border box的左边相接触(对于从左往右的格式化，否则相反)。即使存在浮动也是如此。

为了达到我们想要的效果，利用第四条规则：
> BFC的区域不会与float box重叠。

```
// 通过overflow: hidden触发BFC, 使main变成一个BFC而应用了第四条规则，达到了两栏自适应布局
.main {
    overflow: hidden;
}
```

![完成效果](http://upload-images.jianshu.io/upload_images/2575359-7f76e9a5b4aa9f58.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 2、防止垂直margin折叠
```
<style>
    p {
        color: #f55;
        background: #fcc;
        width: 200px;
        line-height: 100px;
        text-align:center;
        margin: 100px;
    }
</style>
<body>
    <p>Haha</p>
    <p>Hehe</p>
</body>
```

![margin折叠](http://upload-images.jianshu.io/upload_images/2575359-4c226a88d8588370.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

造成这种结果由于第二条规则：
> Box垂直方向的距离由margin决定。属于同一个BFC的两个相邻Box的margin会发生重叠

为解决，我们只需往任一个P元素套一个父元素，并触发该父元素成为BFC，那么这两个P元素就不属于同一个BFC了。
```
<style>
    .wrap {
        overflow: hidden;
    }
    p {
        color: #f55;
        background: #fcc;
        width: 200px;
        line-height: 100px;
        text-align:center;
        margin: 100px;
    }
</style>
<body>
    <p>Haha</p>
    <div class="wrap">
        <p>Hehe</p>
    </div>
</body>
```

![margin未折叠](http://upload-images.jianshu.io/upload_images/2575359-4ce9373867ad5d99.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 3、清除内部浮动
```
<style>
    .par {
        border: 5px solid #fcc;
        width: 300px;
    }
 
    .child {
        border: 5px solid #f66;
        width:100px;
        height: 100px;
        float: left;
    }
</style>
<body>
    <div class="par">
        <div class="child"></div>
        <div class="child"></div>
    </div>
</body>
```

![未完成的效果](http://upload-images.jianshu.io/upload_images/2575359-9c535e8f5ed91f61.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

根据第六条规则:
> 计算BFC的高度时，浮动元素也参与计算

```
// 触发par生成BFC，那么par在计算高度时，par内部的浮动元素child也会参与计算。
.par {
    overflow: hidden;
}
```


![完成的效果](http://upload-images.jianshu.io/upload_images/2575359-1a9a92c37dd0f492.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 参考文章
[BFC](http://www.jianshu.com/p/e9394291b163)

[BFC 神奇背后的原理](http://www.cnblogs.com/lhb25/p/inside-block-formatting-ontext.html)