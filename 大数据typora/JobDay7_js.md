## javascript

#### **JavaScript**是什么？有什么作用？

HTML：就是用来写网页的。   骨架

CSS：  就是用来美化页面的。 服饰

JavaScript：前端大脑、灵魂。行为



JavaScript是WEB上强大的**脚本语言**。

脚本语言：

​		无法独立执行。必须嵌入到其他语言中，结合使用。

​		直接被浏览器解析执行。

作用：控制页面特效展示行为。



------

#### **JavaScript**的语言特征及编程注意事项

特征：

- JavaScript无需编译，直接被浏览器解释并执行
- JavaScript无法单独运行，必须嵌入到HTML代码中运行
- JavaScript的执行过程由上到下依次执行

注意：

- JavaScript没有访问系统文件的权限（安全）
- 由于JavaScript无需编译，是由上到下依次解释执行，所以在保证可读性的情况下，允许使用链式编程
- JavaScript和java没有任何直接关系

#### **JavaScript**的组成

![img](file:///C:\Users\pinkill\AppData\Local\Temp\ksohtml\wps5FC5.tmp.png) 

1. ECMAScript	(核心)：规定了JS的**语法和基本对象**。

	. DOM	文档对象模型：**处理网页内容**的方法和接口

   标记型文档。HTML

	. BOM	浏览器对象模型：**与浏览器交互**的方法和接口

#### **JavaScript**的引入方式

##### **1、内部脚本**

在当前页面内部写script标签，script内部即可书写JavaScript代码

格式：

```html
<script type=”text/javascript”>
	JavaScript的代码
</script>
```

script标签理论上可以书写在HTML文件的任意位置

##### **2**、外部引入

在HTML文档中，通过`<script src="">`标签引入.js文件

格式：

```html
<script  type="text/javascript"  src="javascript文件路径"></script>
```

外部引用时script标签内不能有script代码，即使写了也不会执行。

##### **script标签规范化的放置位置（了解）**

开发规范规定，script标签的放置位置为：

​	Body结束标签前。

如图所示：

```html
<body>
	...
	<script></script>
</body>
```

优点：

 保证html展示内容优先加载，最后加载脚本。 增强用户体验性