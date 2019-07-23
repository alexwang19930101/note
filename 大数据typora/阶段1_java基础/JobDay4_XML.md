## XMl

### 文档声明

在编写XML文档时，需要先使用文档声明来声明XML文档。且必须出现在文档的第一行。

最简单的语法:`<?xml version="1.0"?>`

用encoding属性说明文档所使用的字符编码，默认为UTF-8。保存在磁盘上的文件编码要与声明的编码一致。	如：`<?xml version="1.0" encoding="GB2312"?>`用standalone属性说明文档是否独立，即是否依赖其他文档。

如：`<?xml version="1.0" standalone="yes"?>`

### 元素

XML元素指XML文件中出现的标签。

一个标签分为开始和结束标签(不能省略)。一个标签有如下几种书写形式：

包含标签体：

```xml
<age>18</age>
<Student><name></name><age>18</age></Student>
```

不含标签体，属性需要用双引号或者单引号包含：

```xml
<Student name="zhangsan" age="18"/>
```

一个标签中可以嵌套若干子标签，但所有标签必须合理的嵌套，不允许有交叉嵌套。

```xml
<mytag1><mytag2></mytag1></mytag2>  WRONG不允许交叉
```

一个XML文档必须有且仅有一个根标签，其他标签都是这个根标签的子标签或孙标签。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<student>
</student>

<student>
</student>

*要改成下面这样*

<?xml version="1.0" encoding="UTF-8"?>
<student>
	<student>
	</student>
</student>
```

元素(标签)的名称可以包含字母、数字、减号、下划线和英文句点，但必须遵守下面的一些规范：

1. 严格区分大小写；<p><P>
2. 只能以字母或下划线开头；
3. abc _abc不能以xml(或XML、Xml等)开头----W3C保留日后使用；
4. 名称字符之间不能有空格或制表符；
5. 名称字符之间不能使用冒号； (有特殊用途)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- xml标签严格区分大小写 -->
<!-- 
	xml只能有一个根标签
	可以有重复的子标签 
-->
<China name="ali">
	<!-- 没有标签体，所以没有子标签。双引号中可以有单引号，不可以再出现双引号 -->
	<beijing name="北京"/>
	
	<hubei>
		<name>湖北</name>
	</hubei>
	
	<huhan>
		<changsha>
			<name>长沙</name>
		</changsha>
	</huhan>
</China>
```

### 注释

XML中的注释语法为：注意：

<!--这是注释-->

注释不能嵌套

### CDATA区

CDATA是Character Data的缩写

作用：把标签当做普通文本内容；

语法：<![CDATA[内容]]>

```
<?xml version="1.0" encoding="UTF-8"?>
<China name="ali">
	<name>
		<![CDATA[
			<age>58</age>
		]]>
	</name>
</China>
```

可以用替代字符：

|      |      |
| ---- | ---- |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |

### 指令

处理指令，简称PI(Processing Instruction)。

作用：用来指挥软件如何解析XML文档。

语法：必须以"<?"作为开头，以"?>"作为结尾。常用处理指令：

XML声明:`<?xml version="1.0" encoding="GB2312"?>`

xml-stylesheet指令:`<?xml-stylesheet type="text/css" href="some.css"?>`

### DTD（schema替代了）

XML都是用户自定义的标签，若出现小小的错误，软件程序将不能正确地获取文件中的内容而报错。约束文档定义了在XML中允许出现的标签名称、属性及标签出现的顺序等等。

DTD(Document Type Definition)：文档类型定义。

作用：约束XML的书写规范

XML使用DOCTYPE声明语句来指明它所遵循的DTD文档，有两种形式：

1. 当引用的DTD文档在本地时，采用如下方式：`<!DOCTYPE 根元素 SYSTEM "DTD文档路径">`如：`<!DOCTYPE 书架 SYSTEM "book.dtd">`
2. 当引用的DTD文档在公共网络上时，采用如下方式：`<!DOCTYPE 根元素 PUBLIC "DTD名称" "DTD文档的URL">`

语法：<!ELEMENT 元素名称 使用规则>

使用规则：

- (#PCDATA):指示元素的主体内容只能是普通的文本.(Parsed Character Data)
- EMPTY：用于指示元素的主体为空。比如<br/>
- ANY:用于指示元素的主体内容为任意类型。
- (子元素)：指示元素中包含的子元素.有点像正则表达式

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!ELEMENT 书架 (书+)>
	<!ELEMENT 书 (书名,作者,售价)>
	<!ELEMENT 书名 (#PCDATA)>
	<!ELEMENT 作者 (#PCDATA)>
	<!ELEMENT 售价 (#PCDATA)>
	
	
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE 书架 SYSTEM "book.dtd" >
<书架>
  <书>
    <书名>书名</书名>
    <作者>作者</作者>
    <售价>售价</售价>
  </书>
</书架>
```

也可以直接定义

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<!DOCTYPE 书架 [
	<!ELEMENT 书架 (书+)>
	<!ELEMENT 书 (书名,作者,售价)>
	<!ELEMENT 书名 (#PCDATA)>
	<!ELEMENT 作者 (#PCDATA)>
	<!ELEMENT 售价 (#PCDATA)>
]>
<书架>
	<书>
		<书名>Java就业培训教程</书名>
		<作者>张孝祥</作者>
		<售价>39.00元</售价>
	</书>
	<书>
		<书名>疯狂Java</书名>
		<作者>李刚</作者>
		<售价>69.00元</售价>
	</书>
</书架>
```

在DTD文档中使用ATTLIST关键字来为一个元素声明属性。

语法：		

<!ATTLIST 元素名

​	属性名1 属性值类型 设置说明			

​	属性名2 属性值类型 设置说明>

```
#REQUIRED：表示该属性必须出现
#IMPLIED：表示该属性可有可无
#FIXED:表示属性的取值为一个固定值。语法：#FIXED "固定值"
直接值：表示属性的取值为该默认值
```

### Schema

XML Schema 文件自身就是一个XML文件，但它的扩展名通常为.xsd.取代了DTD

一个XML Schema文档通常称之为模式文档(约束文档)，遵循这个文档书写的xml文件称之为实例文档。

编写了一个XML Schema约束文档后，通常需要把这个文件中声明的元素绑定到一个ＵＲＩ地址上，在XML Schema技术中有一个专业术语来描述这个过程，即把XML Schema文档声明的元素绑定到一个名称空间上，以后XML文件就可以通过这个URI（即名称空间）来告诉解析引擎，xml文档中编写的元素来自哪里，被谁约束。

```xml
<?xml version='1.0' encoding='UTF-8' ?>
<xs:schema xmlns:xs='http://www.w3.org/2001/XMLSchema'
	targetNamespace='http://www.itheima.com'>
	<xs:element name='书架'>
		<xs:complexType>
			<xs:sequence maxOccurs='unbounded'>
				<xs:element name='书'>
					<xs:complexType>
						<xs:sequence>
							<xs:element name='书名' type='xs:string' />
							<xs:element name='作者' type='xs:string' />
							<xs:element name='售价' type='xs:string' />
						</xs:sequence>
					</xs:complexType>
				</xs:element>
			</xs:sequence>
		</xs:complexType>
	</xs:element>
</xs:schema>
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<p:书架 xmlns:p="http://www.itheima.com" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.itheima.com book.xsd ">
	<书>
		<书名>书名</书名>
		<作者>作者</作者>
		<售价>售价</售价>
	</书>
    <书>
		<书名>书名</书名>
		<作者>作者</作者>
		<售价>售价</售价>
	</书>
</p:书架>
```

### java解析XML

XML解析方式分为两种：DOM方式和SAX方式

- DOM：Document Object Model，文档对象模型。这种方式是W3C推荐的处理XML的一种方式。DOM会一次性加载整个数的内容。
- SAX：Simple API for XML。这种方式不是官方标准，属于开源社区XML-DEV，几乎所有的XML解析器都支持它。逐个节点进行解析。

XML解析开发包：

- JAXP：是SUN公司推出的解析标准实现。

- Dom4J：是开源组织推出的解析开发包。(牛，大家都在用，包括SUN公司的一些技术的实现都在用。)

  Dom for java four  也有Log4j 都是four --> for

### Dom4J

```java
/*
 * Dom4J的常用方法：
 * 		Document
 * 			 Element getRootElement() :获取根元素对象（根标签）
 * 		Element
 * 			 List elements() ：获取所有的子元素
 * 			 List elements(String name)：根据指定的元素名称来获取相应的所有的子元素
 * 			 Element element(String name)：根据指定的元素名称来获取子元素对象,如果元素名称重复，则获取第一个元素 
 * 			 String	elementText(String name) ：根据指定的子元素名称，来获取子元素中的文本
 * 			 String	getText() ：获取当前元素对象的文本
 * 			 void setText(String text)：设置当前元素对象的文本
 * 			 String	attributeValue(String name)：根据指定的属性名称获取其对应的值
 * 			 public Element addAttribute(String name,String value)：根据指定的属性名称和值进行添加或者修改
 * 			 addElement
 * 		DocumentHelper
 * 			static Element	createElement(String name) 
 */
```



|      |      |
| ---- | ---- |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |

