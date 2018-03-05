---
title: 简单移动web学习笔记
date: 2016-04-17
---
近期学了较多移动web开发的相关知识，所以在这里写点东西，也算坐个简单总结。

<!--more-->
[](http://blog.josephong.me/2016/08/10/%E7%A7%BB%E5%8A%A8web%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/#pixel像素基础)
## pixel像素基础
- px：CSS pixels 逻辑像素，浏览器使用的抽象单位
- dp，pt：device independent pixels 设备独立像素
- dpr：devicePixelRatio 设备像素缩放比

> 计算公式：1px=(dpr)² * dp
在JavaScript中，可以通过window.devicePixelRatio
获取当前设备的dpr

- DPI:打印机每英寸可以喷的墨汁点（印刷行业）
- PPI:屏幕每英寸的像素数量，即单位英寸内的像素密度(ppi越高，像素越高，图像越清晰)

**这几个属性之间的关系，以iPhone5为例**：iPhone5设备分辨率1136*640dp –>根号[(1136²+640²)]/4 = 326ppi –> 326ppi属于Retina屏幕,所以根据规定dpr=2 –>1px = dpr² *dp –>iphone5的屏幕宽高为320*568px。

例如这样，设计师可以根据320px为基准设计视觉稿。

[](http://blog.josephong.me/2016/08/10/%E7%A7%BB%E5%8A%A8web%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/#Viewport视图)
### Viewport视图
为了让手机能获得良好的网页浏览体验，Apple 找到了一个办法：在移动版的 Safari（iOS平台）中定义了 viewport meta 标签，它的作用就是创建一个虚拟的窗口（viewport），而且这个虚拟窗口的分辨率接近于桌面显示器，Apple 将其定位为 980px.手机浏览器默认为我们做了两件事情

一、页面渲染在一个980px（ios）的Viewport
二、缩放
- visual viewport（度量/视口）= 窗口缩放 scalce，让手机页面可以看到多少
- layout viewport（布局）负责渲染底层页面

**最常用移动web viewport设置**
【布局viewport】=【设备宽度】=【度量viewport】
```
//常见viewport设置
<meta name="viewport"
content="width=device-width, //根据设备调整width
initial-scale=1, //比例为1
user-scalable=no" //不允许用户缩放

//在开发调试过程中，可以通过JS代码获得当前layout和visual的值
document.body.clientWidth //获得layout viewport值
window.innerWidth //获得visual viewport值
```

### 设计移动Web
方案一：根据设备的实际宽度来设计（常用）手机宽320 px，我们就拿320 px设计
方案二：1px=1dp缩放0.5（initial-scale=0.5）。根据设备的物理像素 dp 等于抽象像素 px 来设计。1像素边框和高清图片都不需要额外处理。

## 移动web特别样式
### 一像素边框
比如想让border的线更细些，我们虽然会设置成border：1px；但在是retina屏幕下，1px使用2dp渲染。使之看起来有2px，那我们就用border：0.5px咯。但很可惜，这种语法只在ios8以上的情况下有用，所以不是好方案。
这时我们就常用到scaleY(0.5),大致代码如下：
```
.scale{
position: relative;
}
.scale:after{
content:"";
position: absolute;
bottom:0px;
left:0px;
right:0px;
border-bottom:1px solid #ddd;
-webkit-transform:scaleY(.5);
-webkit-transform-origin:0 0;
}
```
### 相对单位 rem
- 为了适应各大屏幕的手机，px略显固定,不能根据尺寸的大小而改变，使用相对单位更能体验页面兼容性
  - em：是根据父节点的font-size为相对单位但em在多层嵌套下，变得非常难以操作

  - rem：是根据html的font-size为相对单位rem更加能作为全局统一设置的度量（但在实际产品交互体验上还需考量是否使用）

- 引出另一个问题rem的基值设置多少好？回归我们的目的：为了适应各大手机屏幕rem = screen.width/20有时我们也会看到在业界中会用如下方案：
> 将scale设置为1/dpr, 而<html>的font-size计算为screen.width * dpr / >10,然后再以less将UE图上得到的尺寸透明转换为对应的rem值。

关于这一方案的实现
- iphone5上的rem基值为32px，渲染一张64 *64px的div，则用2rem *2rem去渲染。换算公式非常简单：
width：px/rem基值
height：px/rem基值

- 不使用rem的情况：font-size
一般来说font-size是不应该使用rem等相对单位的。因为字体的大小是趋向于阅读的实用性，并不适合与排版布局。
同理趋向于一些固定的元素的特性。我们不使用rem而改为使用px去确保在不同屏幕上表现一致（和rem的目的相反）。

## 多行文本溢出···
单行文本溢出，对title类的使用非常多，而多行文本类，在详情介绍则用的比较多。
```
//单行文本溢出...
.inaline {
overflow: hidden;
white-space: nowrap;
text-flow: ellipsis;
}

//多行文本溢出...
.intwoline {
display: -webkit-box !important;
overflow: hidden;

text-overflow: ellipsis;
word-break: break-all;

-webkit-box-orient: vertical;
-webkit-line-clamp: 2; //关键属性，限制行数
}
```
大概就是这些了，写的比较乱。如有错误欢迎指正。