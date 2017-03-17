title: 不使用 border-radius 实现圆角
date: 2016-01-08 19:10:23
categories: CSS
tags:
---
CSS3中，利用属性 border-radius 可以很方便的设置出圆角矩形，但是在IE8及以下版本却不支持这个属性，为了在旧版本的浏览器中实现宽高自适应的圆角矩形，可以采用以下三种方法。
<!--more-->

## 一、使用背景图片制作圆角
使用背景图片制作宽、高自适应的圆角矩形，首先需要切出4个圆角图片，然后把4个圆角图片放在矩形的四个角上就大功告成了，这一共需要5个div标签，1个容器div和4个圆角背景div。是不是很好理解。

具体实现来说，需要注意一些细节。对于矩形容器div：
* 上下内边距的大小至少设置为圆角的高度。
* position设置为相对定位。
* 放置内容。

对于4个圆角背景div：
* 分别设置各个圆角背景图片。
* 绝对定位于容器矩形的四个角。
* 需要设置宽高值才能显示背景图片。

假设圆角图片的宽高是5px，具体代码如下：
```HTML
<div class="content">
    <div class="top-left"></div>
    <div class="top-right"></div>
    <div class="btm-left"></div>
	<div class="btm-right"></div>
</div>
```

```CSS
.content {
	position: relative;
	padding: 5px;
}
.top-left,
.top-right,
.btm-left,
.btm-right {
	position: absolute;
	width: 5px;
	height: 5px;
}
.top-left {
	top: 0;
	left: 0;
	background: url(imgs/top-left.png) no-repeat top left;
}
.top-right {
	top: 0;
	right: 0;
	background: url(imgs/top-right.png) no-repeat top left;
}
.btm-left {
	bottom: 0;
	left: 0;
	background: url(imgs/btm-left.png) no-repeat top left;
}
.btm-right {
	bottom: 0;
	right: 0;
	background: url(imgs/btm-right.png) no-repeat top left;
}
```
[点击查看在线demo](http://codepen.io/theqwang/pen/VePrZM)

## 二、纯 CSS+div 制作圆角矩形
这种方法的原理是利用像素点绘制弧线来模拟圆角。简单起见，这里我用圆角半径为3px的例子讲解，其中圆角矩形的背景色为粉色，矩形的边框为黑色。左上角圆角放大后如下图所示：
![圆角放大图](http://7xidwy.com1.z0.glb.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-01-03%2023.06.10.png)
这里矩形上下各需要3个高度为1px、垂直罗在一起的div才能模拟出圆角。具体实现如下：
* 第一个div就是上面图中矩形的黑色上边框，因为模拟的圆角半径为3px，所以该div的左右外边距设为3px，背景色设为黑色。
* 第二个div的左右外边距设为2px，左右边框为1px的黑线，背景色为粉色。
* 第三个div的左右外边距设为1px，左右边框为1px的黑线，背景色为粉色。
* 圆角矩形下面的两个圆角是以上三个div反序排列。
* 矩形内容区域的div只需要设置左右边框和背景色即可。

具体代码如下
```HTML
<div class="wrapper">
    <div class="r1"></div>
	<div class="r2"></div>
	<div class="r3"></div>
	<div class="content">aaaaa</div>
	<div class="r3"></div>
	<div class="r2"></div>
	<div class="r1"></div>
</div>
```

```CSS
.r1 {
	height: 1px;
	margin: 0 3px;
	background-color: #111;
}
.r2 {
	height: 1px;
	margin: 0 2px;
	background-color: #f4b4b4;
	border-left: 1px solid #111;
	border-right: 1px solid #111;
}
.r3 {
	height: 1px;
	margin: 0 1px;
	background-color: #f4b4b4;
	border-left: 1px solid #111;
	border-right: 1px solid #111;
}
.content {
	background-color: #f4b4b4;
	border-left: 1px solid #111;
	border-right: 1px solid #111;
}
```

[点击查看在线demo](http://codepen.io/theqwang/pen/QydqrZ)

此方法优缺点分析：
* 不使用背景图片，减少了HTTP请求数。
* 后期维护性好，但是圆角矩形像素增加，无意义的div代码将成倍增加。
* 实现的圆角矩形有局限性。
* 只能实现纯色圆角。

## 三、使用CSS border 画出梯形模拟圆角
通过设置border属性，可以得到梯形和三角形，效果如下图所示：
![border 画梯形和三角形](http://7xidwy.com1.z0.glb.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-01-04%2009.27.16.png)

* 当给div的四个边框设置比较大的宽度值时，就能得到四个梯形，如上图中的第一个图案；
* 在此基础上，当把div的高度值设为0后，就能得到梯形和等腰三角形了，如上图中的第二个图案；
* 当把div的宽、高都设为0，且只设置上、左的两个边框时，就能得到两个直角三角形，图中的第三个图案就是这种情况；
* 在此基础上，把其中的一个边框的颜色设置为透明，就能像第四个图案一样得到一个直角三角形。

代码如下：

```HTML
<div class="box1"></div>
<div class="box2"></div>
<div class="box3"></div>
<div class="box4"></div>
```

```CSS
.box1 {
	height: 20px;
	width: 20px;
	border-top: 20px solid red;
	border-right: 20px solid green;
	border-bottom: 20px solid blue;
	border-left: 20px solid yellow;
}
.box2 {
	height: 0;
	width: 20px;
	border-top: 20px solid red;
	border-right: 20px solid green;
	border-bottom: 20px solid blue;
	border-left: 20px solid yellow;
}
.box3 {
	height: 0;
	width: 0;
	border-top: 50px solid red;
	/* border-right: 20px solid green; */
	/* border-bottom: 20px solid blue; */
	border-left: 50px solid yellow;
}
.box4 {
	height: 0;
	width: 0;
	border-top: 50px solid red;
	/* border-right: 20px solid green; */
	/* border-bottom: 20px solid blue; */
	border-left: 50px solid transparent;
}
```
[点击查看在线demo](http://codepen.io/theqwang/pen/eJgMxv)

已经知道了如何通过 border 画出梯形了，那又该如何利用梯形模拟圆角矩形呢？很简单，只要在矩形的上面和下面各放上一个梯形，就能得到圆角矩形了。原理嘛就是利用梯形的左右两个斜边模拟圆角啦，直接上效果图：
![效果图](http://7xidwy.com1.z0.glb.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-01-04%2009.51.06.png)

具体代码如下：

```HTML
<div class="wrapper">
    <div class="top"></div>
	<div class="content">
	    <p>利用border画出梯形，模拟圆角</p>
	</div>
	<div class="bottom"></div>
</div>
```

```CSS
div {
	box-sizing: border-box;
}
.top {
	height: 0;
	border-top: 3px solid transparent;
	border-bottom: 3px solid #111;
	border-left: 3px solid transparent;
	border-right: 3px solid transparent;
}
.bottom {
	height: 0;
	border-top: 3px solid #111;
	border-bottom: 3px solid transparent;
	border-left: 3px solid transparent;
	border-right: 3px solid transparent;
}
.content {
	color: #fff;
	background-color: #111;
}
.wrapper {
	width: 200px;
	margin: 0 auto;
}
```

[点击查看在线demo](http://codepen.io/theqwang/pen/BjpwmY)

这种方法与方法二相比，更加简洁易懂，减少了无意义的div标签，在模拟较小的实色圆角时，不失为最佳的方法。
