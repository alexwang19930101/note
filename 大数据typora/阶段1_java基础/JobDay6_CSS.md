## css

#### 1.1 CSS是什么

HTML --> 骨架

CSS --> 装饰

ccs：cascading style sheets 层叠样式表

- 层叠：一层层叠加
- 样式表：存储样式

> CSS 通常称为CSS样式或层叠样式表，主要用于设置HTML页面中的文本内容（字体、大小、对其方式等）、图片的外形（高宽、边框样式、边距等）以及版面的布局等外观显示样式。
>
> CSS可以使HTML页面更好看，CSS色系的搭配可以让用户更舒服，CSS+DIV布局更佳灵活，更容易绘制出用户需要的结构。

#### 1.2 示例代码

```html
<!DOCTYPE html>
<html>java
	<head>
		<meta charset="UTF-8">
		<title></title>
	</head>
	<body>
		<font size="7">ok</font><br />
		<font style="font-size: 200px;">ok</font>
	</body>
</html>
```



#### 1.3 为什么使用CSS样式替代HTML属性

因为HTML属性在单独使用时有一定的局限性，所以要配合CSS样式代码才可以展示更为丰富的效果。

#### 1.4 CSS代码放在哪

css代码最好放在`<style></style>`标签内

`<style></style>`标签最好放在`<head></head>`标签中

css的选择器：

```
选择器name｛
	/*CSS代码*/
	属性名：属性值;
	属性名：属性值1 属性值2 属性值3；
｝
```

1.当把html代码中的标签名称当css的`元素选择器`

​	html中所有的该元素都是这个样式

​	元素选择器用的比较少最常用的是

```
标签名｛/*CSS样式代码*/｝

eg：
body｛
	padding：0px；//内边距
	margin：0px；//外边距
｝ 
```

2.`类选择器`：对HTML代码的标签分类，以HTML的类名(class属性)作为选择器名称

```
.类名｛/*CSS代码*/｝
```

3.`id选择器`通过`id`标签属性进行

```
#id｛/*CSS代码*/｝
```

保证id在整个html代码中的唯一性！！

​	命名：

​		一般以标签的首字母开头 div --> div   span --> span1

​		以标签的功能命名 login



------

### 边框属性

所有HTML标签都是有边框，默认不可见

#### border

格式：宽度 样式 颜色

border：1px solid red，1像素 实线 红色边框

#### width

设置变标签宽

#### height

用于设置标签的高度

#### background-color

用于设置标签的背景颜色

1. 英文单词
2. 颜色代码 #xxxxxx，一个颜色两位

#### background-image

用于设置标签的背景颜色

------

### 布局

float 悬浮，不同图层

- none不浮动	
	 left左	
- right右

因为元素设置浮动属性后，会脱离原有文档流（会脱离原有的板式），从而会影响其他元素的样式，所以设置浮动以后，页面样式需要重新调整

------

### **转换**

#### display

HTML提供丰富的标签，这些标签被定义成了不同的类型，一般分为：块级元素和行内元素。

- 块元素：以区域块方式出现。每个块标签独自占据一整行或多整行。块结束会自动换行
  - 常见的块元素：<table>、<h1>、<p>、<div>、<ul>等
- 行内元素：根据内容多少来占用行内空间，不会自动换行
  - 常见的行内元素：<span>、<a>、<font> 等

display属性可以使得元素 在行内元素和块元素之间相互转换。

格式：

​	选择器{display:属性值}

常用的属性值：

- block：此元素将显为块元素（块元素默认的display属性值）
- inline：此元素将显示为行内元素（行内元素默认的display属性值）
- none：此元素将被隐藏，不显示，也不占用页面空间。

------

### **字体**

#### **3.4.1 font-size**

​	用于设置字体的大小。

- 10`px`
- ​10`%`

#### **3.4.2 color**

​	用于设置字体的颜色。

------

###  **css** 盒子模型

#### 什么是盒子模型

​	所有的HTML元素，我们都可以看成一个四边形，即一个盒子。

​	用CSS来设置元素盒子的 内边距、边框 和 外边距 的样式的方式，

​	相当于设置盒子的样式，所以我们将其称之为 盒子模型

##### border	边框

HTML元素盒子的框体

- border-top		上

- border-bottom     下

- border-left             左

- border-right          右

通用性设置：

  一次性设置上下左右边框样式 为1像素的 实体 红色线

  border:1px solid red;



##### padding 内边距

内边距：HTML元素里的内容体  到  HTML元素边框 的距离

边框有四个属性可以设置：

- padding-top:上边距
- padding-right:右边距
- padding-bottom:下边距
- padding-left:左边距

通用性设置：

一次性设置上下左右内边距距离为10PX

padding:10px;



##### margin 外边距

HTML元素边框 到 其他HTML元素边框的距离

- margin-top:上边距
- margin -right:右边距
- margin -bottom:下边距
- margin -left:左边距

通用性设置：

一次性设置上下左右外边距距离为10PX

margin:10px;



------

### **内部样式**

##### 行内样式

行内样式，是通过标签的style属性来设置元素的样式。

格式：

`<html标签 style=”css样式代码” />`



##### `<style></style>`标签方式

当某些样式在页面中被多个标签重复使用，为了编码更加灵活，避免书写重复代码，

我们将样式代码从标签style属性中抽取出来，统一写入到style标签中。

格式：

```
<style>
	css样式代码
</style>
```

### **外部样式**

#### **5.2.1 `<link/>`标签方式**

`<link/>`又称为链入式，是将所有的样式放在一个或多个以.css为扩展名的外部样式表文件中，通过`<link>`标签将样式连接到HTML文档中。

格式：

`<link rel="stylesheet" type="text/css" href="css文件路径"/>`

- rel="stylesheet" ,固定值，表示样式表
- type="text/css"，固定值，表示css类型
- href ，表示css文件位置