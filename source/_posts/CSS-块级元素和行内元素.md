title: CSS块级元素和行内元素
date: 2015-12-23 19:52:03
categories: CSS
tags:
---
## 块级元素block
在视觉上显示为一个块的元素，最明显的特征就是它默认横向充满父元素的内容区域，即默认独占一行。典型的块元素有div、p、h1~h6等。

特点：
1. 每个块级元素独占一行。
2. 块级元素的height、width、padding、border、margin都可以设置。
3. 块级元素在默认情况下，width是父元素的100%。
4. 可以容纳其他块级元素和内联元素。

<!--more-->

## 行内元素inline
行内元素不独占一行，其左右可以有其他行内元素。例如a、span、strong等。

特点：
1. 和相邻行内元素都在一行上。直到一行排不下才会换行。
2. 行内元素的宽度和高度就是其容纳的内容的宽高，在行内元素上设置width、height无效。
3. 垂直方向上的padding、border、margin不会产生边距效果，水平方向上的padding、border、margin可以影响元素的水平间距(布局)。
4. 设置line-height 可以改变行内元素的高度。

**注意：**
行内元素设置padding，border在应用背景图片的时候会有显示，但是不改变垂直方向的布局，对于设置了background-color背景和padding的行内元素，背景可以向元素上面和下面延伸，但是不会改变行高，结果会出现设置后padding，border的背景会覆盖上面的元素的内容，下部被当做背景，被下级元素覆盖。(《CSS权威指南》P249)

margin水平方向起作用，垂直方向不起作用，原因在于：行内元素的外边距不会改变元素的行高。（《CSS权威指南》P227）

可以点击如下demo查看：[demo](http://codepen.io/theqwang/pen/obLKxg)
