title: SASS学习笔记
date: 2016-02-01 12:01:56
categories: CSS
tags:
---
# 一、安装
sass 是基于 ruby 开发的，因此安装 sass 需要先安装 ruby。    
安装完 ruby，通过 ruby 的 gem 安装 sass，在命令行输入：

```
gem install sass
```

检查 sass 是否安装成功：

```
sass -v
```

<!--more-->

如果能输出 sass 的版本信息，就说明 sass 安装成功了。

# 二、编译
## 1.  手动编译
命令行下，在项目所在目录使用 sass 命令手动编译 sass 文件，sass 文件的扩展名为 .scss。例子：

```
sass input.scss:output.css
```

冒号前面是要编译的文件，冒号后面是要编译生成的 css 文件。

## 2. 自动编译
还可以在命令行下使用 sass --watch 命令自动监视 sass 文件的变化并编译。例子：

```
sass --watch input.css:output.css
```

也可以自动监视整个目录的变化并自动编译：

```
sass --watch .:.
```

默认自动编译生成的 css 文件与 sass 文件同名。

# 三、输出样式
sass 编译输出的 css 文件有四种样式：

* nested（嵌套，默认）
* compact（紧凑）
* expanded（扩展）
* compressed（压缩)

在编译时使用 --style 命令可以修改输出的 css 的样式：

```
sass --watch .:. --style compact
```

# 四、变量
sass 使用 $ 定义一个变量，用 : 定义变量值。

```
$primary-color: #333;
```

变量的值可以用其他变量：

```
$primary-border: 1px solid $primary-color;
```

补充：
> 1. 变量名中的中划线和下划线可以互相替换。
> 2. 变量如果定义在某个 {} 规则块里，那么该变量只能在此规则块内使用，类似作用域。

# 五、嵌套
## 1.选择器嵌套
在 css 里一些选择器是重复出现的，可以使用 sass 的嵌套语法减少重复：

```
.nav {
	height: 70px;
	ul {
		margin: 0;
		li {
			list-style-type: none;
			padding: 5px;
			}
			}
}
```

在嵌套语法中，使用 & 来表示父选择器。当包含父选择器标识符的嵌套规则被打开时，它不会像后代选择器那样进行拼接，而是&被父选择器直接替换：

```
a {
	display: block;
	&:hover {
		background-color: #fff;
		}
}
```
## 2.属性嵌套
嵌套属性的规则是这样的：把属性名从中划线-的地方断开，在根属性后边添加一个冒号:，紧跟一个{ }块，把子属性部分写在这个{ }块中。
例如，下面的 font-size 和 font-weight 属性可以嵌套 font。

```
body {
	  font-size: 15px;
	    font-weight: 600;
}
```

嵌套写法： 

```
body {
	font: {
		size: 15px;
		weight: 600;
		}
}
```

# 六、@mixin
mixin 是一组样式的集合，主要用于大段样式的重用。

 mixin 类似于javascript中的函数，mixin 有名字和参数，参数可以省略，定义 mixin 的格式如下：

 ```
 @mixin 名字 (参数1，参数2...) {......}
 ```

 引用 mixin 时使用如下写法：

 ```
 @include mixin的名字;
 ```

 mixin 中的参数写法与 sass 变量写法一样，变量的赋值既可以按形参顺序赋值，也可以按参数名赋值，sass 允许通过语法 $name: value 的形式指定每个参数的值。。来看一个具体的例子：

 ```
 @mixin alert ($text-color, $background) {
	 color: $text-color;
	 background-color: $background;
 }
 .alert-warning {
	 @include alert(#333, #ffflert {
		 padding: 15px;
	 }
	 .alert a {
		 font-size: 20px;
	 }
	 .alert-info {
		 @extend .alert;
		 background-color: #FFF;
	 }
	 ```

	 生成如下的 css：

	 ```
	 .alert, .alert-info {
		   padding: 15px;
	 }
	 .alert a, .alert-info a {
		   font-size: 20px;
	 }
	 .alert-info {
		   background-color: #FFF;
	 }
	 ```

	 注意，使用 @extend 会继承选择器下所有子样式。上面的例子里，.alert-info 不仅继承了 .alert 的样式，还会继承 .alert a 的样式。

	 如果 .alert extend 了 .alert-info，等效于对应的 html 的 class="alert alert-info"，

## 1.何时使用@extend
混合器主要用于展示性样式的重用，而类名用于语义化样式的重用。因为继承是基于类的（有时是基于其他类型的选择器），所以继承应该是建立在语义化的关系上。当一个元素拥有的类（比如说.seriousError）表明它属于另一个类（比如说.error），这时使用继承再合适不过了。

# 八、@import与partials
css 使用 @import 在一个 css 文件里引入其他 css 文件，然而，只有执行到 @import 时浏览器才会去下载导入的css 文件，使页面加载变慢，因为增加了 HTTP 连接。

sass 扩展了 @import，在一个  sass 文件里也可以使用 @import 导入其他 sass 文件，然后 sass 会把它编译成一个文件，避免了额外的下载请求，另外，所有在被导入文件中定义的变量和 mixin 均可在导入文件中使用。

这样我们可以把 sass 分成不同的部分，即模块化，每个模块在 sass 中叫做 partials（局部文件），partials 的文件名以 _ 开头，这样 sass 就不会单独编译 partials 而只把该文件用来导入。

例如，我们可以创建 _base.scss 的 partials 模块来重置基本样式，@import 导入 partials 时直接用文件名，不需要加 _ 和文件扩展名：

```
@import ”base“；
```

## 1. 局部导入
 ass允许 @import 命令写在 css {} 规则内，这种导入方式下，生成对应的css文件时，局部文件会被直接插入到css规则内导入它的地方。被导入的局部文件中定义的所有变量和混合器，也会在这个规则范围内生效。这些变量和混合器不会全局有效。

 ```
 .nav {
	 @import "base";
	 text-decoration: none;
 }
 ```

## 2.导入原生css
sass兼容原生 css，所以也支持原生的 css@import。在下列三种情况下会生成原生的CSS@import，尽管这会造成浏览器解析css时的额外下载：

* 被导入文件的名字以.css结尾
* 被导入文件的名字是一个URL地址
* 被导入文件的名字是CSS的url()值

## 3.默认变量值
一般情况下，反复声明一个变量值时，只有最后一个声明有效。

假如你写了一个可被他人通过 @import 导入的 sass 库文件，你可能希望导入者可以定制修改 sass 库文件中的某些值。使用 sass 的 !default 标签可以实现这个目的。!default用于变量，含义是：如果这个变量被声明赋值了，那就用它声明的值，否则就用这个默认值。


```
$alert-color: #8d8d8d !default;
```

# 九、注释
sass 包含 3 中注释方式：

* 单行注释： //注释内容
* 多行注释：/* 注释内容 */
* 强制注释：/*! 注释内容 */

单行注释不会出现在编译后的 css 文件里；多行注释会出现在编译后的 css 文件里，但是不会出现在压缩编译的 css 文件里；强制注释会一直出现在 css 文件里。

---
> # sass高级部分

---

# 一、数据类型
css 属性值和 sass 变量可以分成不同的数据类型。

在命令行下使用 sass -i 命令可以启动 sass 的交互界面，在交互界面下使用 type-of() 函数可以判断数据的类型。

数据类型有如下几种：

* number：5、5px、5deg
* string：hello、"world"
* list：1px solid #111、 20px 15px
* color：#ff0000、red、rgba(0, 0, 0, 1)、rgb(1, 1, 1)、hsl(0, 100%, 50%)
* map：(padding: 10px, margin: 10px)
* boolean: true、false

## 1-1.数字
在 sass 里宽度、高度等属性的值的数据类型是数字，数字可以带 px、 rem 等单位。数字类型的值可以使用 +、 -、 *、 /、 % 运算，其中 / 除法是 css 的保留字符，我们可以使用 (a/b) 的形式进行除法运算。

## 1-2.数字函数
sass 提供了一些数字运算的函数，方便计算。下面列举一些常见的数字函数：

* abs()：取绝对值
* round()：四舍五入
* ceil()：舍去小数部位
* floor()：有小数部分就进位
* min()：多个数字里取最小值
* max()：多个数字里取最大值

## 1-3.字符串
css 里 normal、left、right、absolute 等属性值都是字符串类型，用 “” 包裹的属性值也是字符串类型，两者的区别是引号包裹的字符串可以含有空格。

使用 + 可以连接两个字符串。使用 + 也可以连接一个字符串和一个数字。用 -、/连接字符串都只会保留-、/，用 * 连接字符串会报错。

## 1-4.字符串函数
sass 提供了一些字符串函数来处理字符串。以下是一些常用是字符串函数：

*  to-upper-case()：把字符串转换成大写
*  to-lower-case()：把字符串转换成小写
*  str-length()：计算字符串的长度
*  str-index(a, b)：求子串 b 在 a 里出现的起始位置，注意起始位置从1开始
*  str-insert(a, b, num)：把字符串 b 插入到字符串 a 的 num 位置。

## 1-5.颜色
css 里使用HEX、字符串、RGB、RGBA、HSL等多种方式表示颜色。

## 1-6.颜色函数
* RGB(红, 绿, 蓝)：红绿蓝的值可以使用 0~255 或百分比
* RGBA(红，绿，蓝，不透明度)：不透明度的值为 0~1,0 表示完全透明，1 表示完全不透明
* HSL(色相，饱和度，明度)：色相的值是 0~360，饱和度和明度的值是百分比
* HSLA(色相，饱和度，明度，不透明度)：不透明度的值为 0~1,0 表示完全透明，1 表示完全不透明

sass 提供的颜色函数：

* adjust-hue(颜色值，角度值)：调整颜色的色相值。
* lighten(颜色值，百分数)：使颜色变亮，明度的增加值就是百分数
* darken(颜色值，百分数)：使颜色变暗，明度的减少值就是百分数
* saturate(颜色值，百分数)：增加颜色的纯度，即增加颜色的饱和度
* desaturate(颜色值，百分数)：减少颜色的纯度，即降低颜色的饱和度
* opacity(颜色，数字)：增加颜色的不透明度
* transparentize(颜色，数字)：减少颜色的不透明度

## 1-7.列表
sass 用空格或逗号分隔的多个值都是列表，如 border: 1px solid #eee; font-family: courier, monospace;

列表里可以包含其他列表，如下 padding 的值都是两个列表。

> padding: 10px 5px, 0 5px; 

> padding: (10px 5px) (0 5px); 

sass 在编译时会去掉逗号和括号。

## 1-8.列表函数
列表类似于其他语言里的数组。下面是常见的列表函数：

* length()：求表列有多少个项目
* nth(列表，num)：返回列表里第 num 项，num是从1开始计算的
* index(列表, value)：返回 value 项在列表里的索引值
* append(list1, value)：在 list1 后添加 value 项，还可以有第三个参数指定分隔符
* join(list1, list2)：拼接两个列表，还可以有第三个参数指定分隔符

## 1-9.map
map 类似于 python 的字典，是键值对数据类型。 map 类型书写方式如下：(key1: value1, key2: value2...)

列表函数也都可以应用在 map 对象上，map 还有自己独有的函数，如下：

* map-get(map, key)：返回 map 中对应 key 的值
* map-keys(map)：返回 map 里所有的 key
* map-values(map)：返回 map 里所有的 value
* map-has-key(map, key)：判断 map 是否有对应的 key
* map-merge(map1, map2)：合并两个 map
* map-remove(map, key)：从 map 里删除对应的 key 项

## 1-10.boolean
boolean 包含 true、false。

比较运算符：>、<、>=、<=、== ，逻辑运算符：and、or、not 都会返回 boolean 值。

# 二、interpolation
interpolation 可以把变量或者表达式的值插入到选择器、属性名或注释里。 使用 #{ 变量|表达式 } 的形式使用 interpolation。 例子：

```
$version: "0.0.1";
$type: "alert";
$attr: "border";
/* #{$version} */
body {
	padding: $version;
} 
a-#{$type} {
	#{$attr}-type: solid;
}
```

# 三、控制语句
## @if
sass 可以使用 @if 语句实现条件控制，写法如下： @if 条件 {...} @else {...}

例子：

```
$theme: "dark";
body {
	@if $theme == "dark" {
		background-color: #111;
		} @else {
			background-color: #FFF;
			}
}
```

## @for
@for 循环语句的写法如下： @for $var from <开始值> through | to <结束值> {}。循环中 $var 的值每次增加1，through 与 to 的不同之处在于 through 会包括结束值，to 不包括结束值。

例子，一个简单的网格系统：

```
$columns: 4;
@for $i from 1 through $columns {
	.col-#{$i} {
		width: 100% / $columns * $i;
		}
}
```

生成如下 css：

```
.col-1 {
	  width: 25%;
}
.col-2 {
	  width: 50%;
}
.col-3 {
	  width: 75%;
}
.col-4 {
	  width: 100%;
}
```

## @each
如果要操作列表里的每一项，可以使用 @each。写法如下：@each $var in $list {...}。

```
$icons: success warning error;
@each $icon in $icons {
	.icon-#{$icon} {
		background-image: url(../img/icon-#{$icon}.jpg);
		}
}
```

生成如下 css：

```
.icon-success {
	  background-image: url(../img/icon-success.jpg);
}
.icon-warning {
	  background-image: url(../img/icon-warning.jpg);
}
.icon-error {
	  background-image: url(../img/icon-error.jpg);
}
```

## @while
@while 语句也可以实现循环，但是要比 @for 更灵活，要注意 @while 语句里要有结束循环的语句，不然会一直循环下去。写法： @while 条件 {...}。

```
$i: 6;
@while $i > 0 {
	.iterm-#{$i} {
		width: 5px * $i;
		}
		$i: $i - 2;  // 退出循环的语句
}
```

生成的 css：

```
.iterm-6 {
	  width: 30px;
}
.iterm-4 {
	  width: 20px;
}
.iterm-2 {
	  width: 10px;
}
```

# 四、自定义函数
sass 允许用户自定义函数，写法： @function 名称 (参数1，参数2...) {...@return...}。使用时，直接用函数名调用。

例子：


```
$colors: (dark: #000, light: #fff);
@function color ($key) {
	@return map-get($colors, $key);
}
body {
	background-color: color(light);
}
```

生成的 css：

```
body {
	  background-color: #fff;
}
```

## 警告与错误
在自定义函数里，可以通过 @warn 和 @error 语句在命令行里输出用户自定义的警告和错误，提高函数的健壮性。

前面的 function 例子可以该写成：

```
@function color ($key) {
	@if not map-has-key($colors, $key) {
		@warn "没有 #{$key} 这个颜色";
		}
		@return map-get($colors, $key);
}
```
