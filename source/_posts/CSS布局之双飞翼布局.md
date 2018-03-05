---
title: CSS布局之双飞翼布局
date: 2016-06-11
---
## 概念
理解了上篇文章的圣杯布局，再来学习双飞翼布局就简单许多了。同样引用上次的元素命名。

<!--more-->
**圣杯布局**是给包裹着三栏的`container`加一个`padding`,让`padding-left`和`padding-right`的数值是左栏和右栏的宽度，然后利用相对定位把他们再移动在两旁。

而**双飞翼布局**是在`center`**里面**再添加一个div，然后对这个div进行`margin-left`和`margin-right`，即：
```
<div id="header"></div>
  <div id="container"> 
    <div id="center" class="column"> <div class="wrap"></div>
  </div>
  <div id="left" class="column"></div>
  <div id="right" class="column"></div>
</div>
<div id="footer"></div>
```
wrap的 css 部分：
```
.wrap {
margin-left: 200px;
margin-right: 150px;
}
```
其他部分和圣杯布局都是一样的。它们要实现的样式都是下面的这种形式：
![](https://segmentfault.com/image?src=https://cloud.githubusercontent.com/assets/7554325/13628574/2e28562c-e60e-11e5-9bae-905c4c7bf56a.png&objectId=1190000004579886&token=9bb8b342939860ca25c504a051283e89)