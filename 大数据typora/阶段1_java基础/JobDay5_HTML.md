## HTML

本章节必须记忆层次关系，标签内容信息

------

### 1.介绍

HTML：(Hyper Text Markup Language) 超文本标记语言

- 文本：相当于记事本里写的文字。 展示信息
- 超文本：超越了文本仅仅展示信息的功能范畴。泛指：图片、有样式的文字、点击跳转页面的文字
- 语言：工具
- 标记：标签

HTML是由标签所组成的语言，能展示超文本效果

HTML是用来`写网页`的，是设计页面的基础。

### 1.1特征

HTML文件的扩展名为html或者htm。Htm是老的命名规范，html的新的。

HTML文件由浏览器直接解析执行，无需编译，直接由上到下依次解析执行。(java编译 生成字节码文件 + 查错 )

HTML标签通常由开始标签和结束标签组成。例如:`<font>`内容体`</font>`。

开始标签和结束标签之间的内容叫做内容体。

 

HTML没有内容体的标签叫做空标签。仅由一个标签组成。例如：`<br/>`自关闭

HTML标签不区分大小写，为了方便阅读，建议使用小写。

 

HTML标签是有属性的，格式为：属性名=”属性值”，属性值用引号引起。引号包含单引号和双引号

HTML标签建议包裹嵌套，不建议交叉嵌套。

```html
<html>
	<head>
		<title></title>
	</head>
	<body>
		<font color = "red">hello world</font>
		<font color = "blue">hello world</font>
		<font color = "yellow">hello world</font>
		<font color = "#0094ff">hello world</font>
	</body>
</html>
```

------

### 1.2 Hbuilder

一款开发工具，用！！！

------

### 1.3 HTML的基本标签

### ***字体标签和格式化标签***

#### 1.3.1 `<font></font>`标签

***字体标签***，用于展示效果中修饰文字样式

```html
<font 属性名=”属性值”>文字</font>
```

- size:	控制字体大小.最小1 ~ 最大7。 如果设置范围不在1~7之间，设置无效
- color：   控制字体颜色. 使用英文设置（例如：red,blue…） 
- face：     控制字体类型。只能设置系统字库中存在的字体类型

#### 1.3.2`<br/>`标签

HTML源码中换行，浏览器解析时会自动忽略。

***换行标签***，用于展示效果中换行

#### 1.3.3`<p></p>`标签

***段落标签***，用于展示效果中划分段落。并且自动在段前和段后自动加空白行

align:段落内容的对齐方式

- left(默认)， 内容居左
- Right  右
- Center 居中

#### 1.3.4`<h1><\h1>`标签

***标题标签***，用于展示效果中划分标题

其中`<h1>`最大，`<h6>`最小

#### 1.3.5`&nbsp;`符号

HTML源码中的多个空格，效果中最终会合并成一个。

***空格符号***，用于展示效果中显示一个空白的位置



### ***图片标签***

#### 1.3.6`<img/>`标签

用于在页面效果中展示一张图片。

- src：指明图片的路径。（必有属性）

  图片路径的写法：

  内网路径：

  ​	绝对路径：文件在硬盘上的具体位置。【不建议使用】

  ​		例如：C:\ JavaWeb001_html\img\c_1.jpg

​	       相对路径：从引入者所在目录出发。【建议使用相对路径】

​			例如：../img/c_1.jpg

​			../表示上一层目录

​			./表示当前目录

​      互联网路径：

​		必须前面加上http://

​			例如：`http://www.baidu.com/xxx.jpg`

- width：图片宽度

- height：图片的高度

  宽度和高度的设置：

  `默认单位是px，像素`。例如：width=”400”   其实设置的是 width=”400px”。固定设置方式

   百分比设置。例如：width=”50%”。  是`父标签`的百分比。 动态改变的。



### ***列表标签***

#### 1.3.7`<ul></ul>`

无序列表标签，用于在效果中定义一个无序列表，使用最多

#### **1.3.8`<li></li>`**

列表条目项标签，用于在效果中定义一个列表的条目

#### **1.3.9`<ol></ol>`**

有序列表标签，用于在效果中定义一个有序列表。在列表前面有序号，不推荐

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>列表标签</title>
	</head>
	<body>
		<ul type="disc">
			<li><font size="6" color="aquamarine" face="微软雅黑">列表第1行</font></li>
			<li><font size="6" color="azure" face="微软雅黑">列表第2行</font></li>
			<li><font size="6" color="#00ff94" face="微软雅黑">列表第3行</font></li>
		</ul>
		<ul type="square">
			<li><font size="6" color="aquamarine" face="微软雅黑">列表第1行</font></li>
			<li><font size="6" color="azure" face="微软雅黑">列表第2行</font></li>
			<li><font size="6" color="00ff94" face="微软雅黑">列表第3行</font></li>
		</ul>
		<ul type="circle">
			<li><font size="6" color="aquamarine" face="微软雅黑">列表第1行</font></li>
			<li><font size="6" color="azure" face="微软雅黑">列表第2行</font></li>
			<li><font size="6" color="00ff94" face="微软雅黑">列表第3行</font></li>
		</ul>
	</body>
</html>
```



### ***超链接标签***

#### **1.3.10`<a></a>`**

`超链接标签`，用于在效果中定义一个可以点击跳转的链接

href：超链接跳转的路径 (必有属性)

- ​	内网本机路径：相对路径和绝对路径
- ​	互联网路径：http://地址
- ​	本页：默认跳转到本页

 

超链接正常工作：

1. a标签中必须有内容
2. a标签必须有href属性

注意：

1. a标签内容体，不仅仅是文字，也可以是其他内容，例如图片
2. a标签的href属性，不仅仅可以链接到html上，也可以链接到其他文件上，例如图

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>超级链接</title>
	</head>
	<body>
		<a href="图片标签.html">图片标签.html</a><br />
		<a href="http://www.baidu.com">baidu</a><br />
		<!-- 描述：把图片链接到html -->
		<a href="图片标签.html">
			<img src="../img/joke.jpg" />
		</a><br />
		<!-- 描述：超链连到图片 -->
        <a href="../img/c_2.jpg">链接到图</a>
	</body>
</html>
```



### ***表格标签***

#### 1.3.11`<table></table>`

表格标签，用于在效果中定义一个表格

- border：设置表格的边框粗细
- width：设置表格的宽度

#### 1.3.12`<tr></tr>`

表格的行标签，用于在效果中定义一个表格行

#### 1.3.13`<td></td>`

表格的单元格标签，用于在效果中定义一个表格行中的单元格

`<thead>`, `<tbody>` 和 `<tfoot>`很少被用到，这是由于浏览器对它们的支持不太好。

#### **1.3.14`<th></th>`**

表格的表头单元格标签，用于在效果中定义一个表格行中的表头单元格

 `<th>`和`<td>`唯一区别：`<th>`内容 居中加粗

##### **单元格合并**

`<td>`或者`<th>`都有两个单元格合并属性：

-  colspan：跨列合并单元格

- ------

  rowspan：跨行合并单元格



### ***块标签***

#### **1.3.15`<span></span>`**

行级的块标签，用于在效果中 一行上定义一个块，进行内容显示。

- span有多少内容，就会占用多大空间。
- Span不会自动换行

适用于少量数据展示



#### 1.3.16`<div></div>`

块级的块标签，用于在效果中 定义一块，默认占满一行，进行内容的显示

- 默认占满一行
- 会自动换行

适用于大量数据展示

------

### 1.4 HTML的表单标签

#### 1.4.1 `<form></form>`标签

表单是一个包含表单元素的区域。

表单元素是允许用户在表单中（比如：文本域、下拉列表、单选框、复选框等等）输入信息的元素。

`action`:将数据提交到何处。

​    默认提交到本页。

​	1. 本机内网路径：

​		相对路径：

​		绝对路径：

​	2. 互联网路径：

​		http://www.baidu.com/xxx

 

 `method`:将数据以何种方式提交

​	默认为：get

​	提交方式可定义：get     或者     post

​		Get提交方式特点：把数据拼接到地址栏上

​		Post提交方式特点：没有把提交数据拼接到地址栏上。请求体

​	-`Get`和`post`提交方式区别：

​		1. get提交的参数列表拼接到了地址栏后面,post方式不会 拼接地址栏

​		2. get方式提交的数据 敏感信息不安全,Post方式提交的数据  相对安全

​		3. get方式提交的数据量 有限的,Post方式从理论上提交的数据量 无限大

 

尽量使用post方式提交表单

### ***输入项标签***

#### 1.4.2 `<input/>`标签

表单输入项标签之一，用户可以在该标签上 通过填写和选择 进行数据的输入。

1. `type`:设置该标签的种类

- 以下是种类：
  -  text:文本框。 默认为空，默认值可以**用`value`设置**
  -  password:密码框。  内容为非明文，默认值可以**用`value`设置**
  -  radio:单选框。   在同一组内有单选效果,**需要使用`name`属性放入同一组**，**需要value属性的值说明选择的哪一个**，可以用`checked="checked"`默认选中 
  -  checkbox:复选框。  在同一组内有复选效果，**需要name来表明提交数据，如提交后bike=on**
  -  submit:提交按钮。用于控制表单提交数据
  -  reset:重置按钮。 用于将表单输入项恢复到默认状态
  -  file:附件框。用于文件上传。
  -  hidden:隐藏域。一般用作提交服务器需要拿到，但用户不需要看到的数据。
  -  button:普通按钮。需要和JS事件一起用

2. `name`: 单选框、复选框进行数据的分组。/    设置该标签对应的参数名

   ​	某个表单输入项需要通过参数列表提交，就必须设置name属性

3. `value`:设置该标签对应的参数值。   /   作为按钮的名字

- value属性的设置策略：
  - 文本框、密码框这样的表单输入项，可以不强制指定value，因为用户可以自由输入
  - 单选框、复选框这样的表单输入项，必须强制指定value，因为用户无法输入，只能选择，如果不指定value，那么提交上去的只有on

 

4. checked:设置单选框/复选框的默认选中状态
5. readonly:设置该标签的参数值只读，用户无法手动更改。数据是可以正常提交
6. disabled:设置该标签不可用，参数值无法更改，且参数值也无法提交

 

 参数列表的格式：

参数名1=参数值1&参数名2=参数值2&参数名3=参数值3…….

例如：username=zhangsan&password=123&sex=man



### ***选择框标签***

#### 1.4.3 `<select></select>`标签  

定义一个选择框

-  name: 设置该标签对应的参数名
-  multiple：设置该标签选项全部显示，并且可以进行多选提交。默认为单选。

 

#### 1.4.4 `<option></option>`标签

选项标签，用于为一个选择框添加一个选项

-  value:设置需要提交的参数值。
-  selected:设置选项的`默认选中`状态

注意事项：

​	Option的内容体一般是用来进行展示

​	参数值 应该是option的value属性值

![表单](C:\Users\pinkill\Desktop\大数据typora\表单.PNG)

```html
<!DOCTYPE html>
<html>

	<head>
		<meta charset="UTF-8">
		<title></title>
	</head>

	<body>
		<fieldset>
			<legend>Information:</legend>
			<form action="html_form_action.asp" method="get">
				First name:
				<input type="text" name="firstname">
				<br /> Last name:
				<input type="text" name="lastname">
				<br /> gender:
				<input type="radio" name="sex" value="male"> Male
				<input type="radio" name="sex" value="female"> Female<br> vehicle:&nbsp;
				<input type="checkbox" name="bike"> I have a bike
				<input type="checkbox" name="car"> I have a car<br /> select car
				<select name="cars">
					<option value="volvo">Volvo</option>
					<option value="saab">Saab</option>
					<option value="fiat">Fiat</option>
					<option value="audi">Audi</option>
				</select><br/>
				<select name="cars2">
					<option value="volvo">Volvo</option>
					<option value="saab">Saab</option>
					<option value="fiat" selected="selected">Fiat</option>
					<option value="audi">Audi</option>
				</select><br/>
				<textarea rows="10" cols="40">The cat was playing in the garden.
			</textarea><br/>
				<input type="button" value="Hello world!"><br /> Type your first name:
				<input type="text" name="FirstName" value="Mickey" size="20">
				<br>Type your last name:
				<input type="text" name="LastName" value="Mouse" size="20">
				<br>
				<input type="submit" value="Submit">
			</form>
		</fieldset>
	</body>
</html>
```

提交到本地时：`http://127.0.0.1:8020/Day01_HTML/html/07%e8%a1%a8%e5%8d%95.html?firstname=&lastname=&cars=volvo&cars2=fiat&FirstName=Mickey&LastName=Mouse`



样式可以交由bootstrap