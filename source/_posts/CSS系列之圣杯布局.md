---
title: CSS系列之圣杯布局
date: 2016-05-10
---
## 圣杯布局的概念
![](http://alistapart.com/d/_made/d/ALA211_grail_300_348_421_81.jpg)

<!--more-->
即一种轻便，广泛使用的三栏流体布局。它具有以下的优秀特性：
- 两边固定宽度而中间可以流动；
- 允许中间栏代码最先出现；
- 允许任一栏的高度都最高；
- 仅需一个额外的div标签；
- 仅需很少必要css代码及最少兼容性补丁代码

我们先看看实现效果：[Take a look](http://alistapart.com/d/holygrail/example_1.html)

## 实现步骤
### STEP 1：CREAT THE FRAME
html代码:
```
<div id="header"></div>
<div id="container">
<div id="footer"></div>
```
给 container 设置padding，为两栏预留空间
```
#container {
padding-left: 200px; /* LC width */
padding-right: 150px; /* RC width */
}
```
效果像是这样：

![image.png](http://upload-images.jianshu.io/upload_images/2575359-614d835155f171d7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### STEP 2：ADD THE COLUMNS

加上三栏：
```
<div id="center" class="column"></div>
<div id="left" class="column"></div>
<div id="right" class="column"></div>
```
给两边加上适合的宽度，然后让中间栏自适应，让三栏全部浮动，当然也要给底栏 footer 清除浮动：
```
#container .column {
  float: left;
}
#center {
  width: 100%;
}
#left {
  width: 200px;  /* LC width */
}
#right {
  width: 150px;  /* RC width */
}
#footer {
  clear: both;
}
```
这时的效果图是这样的：

![image.png](http://upload-images.jianshu.io/upload_images/2575359-d7f8cd1b7442bc85.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### STEP 3: PULL THE LEFT COLUMN INTO PLACE
这里才开始精妙的部分，我们先看代码：
```
#left {
width: 200px; /* LC width */
margin-left: -100%;
}
```
`margin-left: -100%`这条语句使左栏向左浮动了100%，center和left栏就会重叠，效果是这样的：
![](http://upload-images.jianshu.io/upload_images/2575359-4600ad4238e601d7.gif?imageMogr2/auto-orient/strip)
这时我们再利用relative的特性，使left移动到左栏，即：
```
#container .columns {
float: left;
position: relative;
}
#left {
width: 200px; /* LC width */
margin-left: -100%;
right: 200px; /* LC width */
}
```
左栏想要的效果就达到了：
 
![image.png](http://upload-images.jianshu.io/upload_images/2575359-5547d6bb1c628156.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### STEP 4: PULL THE RIGHT COLUMN INTO PLACE
接下来我们把放置右栏就简单多了，利用margin的负值法：
```
#right {
width: 150px; /* RC width */
margin-right: -150px; /* RC width */
}
```
ok，每栏都到达了正确位置，效果是这样的：
![image.png](http://upload-images.jianshu.io/upload_images/2575359-0a7efedeb3f54424.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**圣杯布局就是这样了。如果想要更多了解，推荐看这篇原文**：[In Search of the Holy Grail](http://alistapart.com/article/holygrail)