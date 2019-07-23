## ajax

Asynchronous JavaScript and xml	是	异步的js和xml

实现页面的部分更新

#### 1.XMLHttpRequest

##### XMLHttpRequest 对象

## 向服务器发送请求

如需将请求发送到服务器，我们使用 XMLHttpRequest 对象的 open() 和 send() 方法：

open() 方法需要三个参数。

- 第一个参数定义发送请求所使用的方法（GET 还是 POST）。
- 第二个参数规定服务器端脚本的  URL。
- 第三个方法规定应当对请求进行异步地处理。

send() 方法可将请求送往服务器。

```
//xmlhttp是已经创建的XMLHttpRequest的对象实例
xmlhttp.open("GET","test1.txt",true);
xmlhttp.send(null);
```

| 方法                         | 描述                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| open(*method*,*url*,*async*) | 规定请求的类型、URL 以及是否异步处理请求。  *method*：请求的类型；GET 或 POST  *url*：文件在服务器上的位置  *async*：true（异步）或 false（同步） |
| send(*string*)               | 将请求发送到服务器。  *string*：仅用于 POST 请求，设置请求体 |

如果请求方式是`get`，那么我们使用send(null) 原因:get请求方式是没有http请求体

只有`post`请求方式时，才有请求体,所以send的参数只是在post请求时使用

如果需要像 HTML 表单那样 POST 数据，用 setRequestHeader() 来添加 HTTP 头。然后在 send() 
方法中规定希望发送的数据

例如**`Xmlhttp.send(“username=xxx&password=xxx”);`**

```
xmlhttp.open("POST","ajax_test.asp",true);
xmlhttp.setRequestHeader("Content-type","application/x-www-form-urlencoded");
xmlhttp.send("fname=Bill&lname=Gates");
```

| 方法                               | 描述                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| setRequestHeader(*header*,*value*) | 向请求添加 HTTP 头。  *header*: 规定头的名称  *value*: 规定头的值 |

**1. onreadystatechange 属性**

onreadystatechange 属性存有处理服务器响应的函数。下面的代码定义一个空的函数，可同时对 onreadystatechange  属性进行设置：

```
xmlHttp.onreadystatechange=function()
  {
  // 我们需要在这里写一些代码
  }
```

**2. readyState 属性**

readyState 属性存有服务器响应的状态信息。每当 readyState 改变时，onreadystatechange 函数就会被执行。

这是 readyState 属性可能的值：

| 状态 | 描述                                                         |
| ---- | ------------------------------------------------------------ |
| 0    | 请求未初始化（在调用 open() 之前）                           |
| 1    | 请求已提出（调用 send() 之前）                               |
| 2    | 请求已发送（这里通常可以从响应得到内容头部）                 |
| 3    | 请求处理中（响应中通常有部分数据可用，但是服务器还没有完成响应） |
| 4    | 请求已完成（可以访问服务器响应并使用它）                     |

我们要向这个 onreadystatechange 函数添加一条 If 语句，来测试我们的响应是否已完成（意味着可获得数据）：

```
xmlHttp.onreadystatechange=function()
  {
  if(xmlHttp.readyState==4 && xmlHttp.status==200)
    {
    // 从服务器的response获得数据
    }
  }
```

**3. responseText 属性**

可以通过 responseText 属性来取回由服务器返回的数据。

在我们的代码中，我们将把时间文本框的值设置为等于 responseText：

```
xmlHttp.onreadystatechange=function()
  {
  if(xmlHttp.readyState==4 && xmlHttp.status==200)
    {
    document.myForm.time.value=xmlHttp.responseText;
    }
  }
```

**4.status**

|        |                           |
| ------ | ------------------------- |
| status | 200: "OK" 404: 未找到页面 |

#### 使用js来完成ajax

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
<script type="text/javascript">
    //1.创建XMLHttpRequest核心对象
	var xmlHttp

	function showHint(str) {

		if (str.length == 0) {
			document.getElementById("txtHint").innerHTML = "";
			return;
		}
		//获取XMLHttpRequest对象
		xmlHttp = GetXmlHttpObject()

		if (xmlHttp == null) {
			alert("您的浏览器不支持AJAX！");
			return;
		}

		var url = "gethint.asp";
		url = url + "?q=" + str;
		url = url + "&sid=" + Math.random();
        //2.设置回调函数
		xmlHttp.onreadystatechange = stateChanged;
        //3.设置请求方式，请求路径，是否异步
		xmlHttp.open("GET", url, true);
        //4.发送请求
		xmlHttp.send(null);
	}

	function stateChanged() {
		if (xmlHttp.readyState == 4) {
			document.getElementById("txtHint").innerHTML = xmlHttp.responseText;
		}
	}

	function GetXmlHttpObject() {
		var xmlHttp = null;
		try {
			// Firefox, Opera 8.0+, Safari
			xmlHttp = new XMLHttpRequest();
		} catch (e) {
			// Internet Explorer
			try {
				xmlHttp = new ActiveXObject("Msxml2.XMLHTTP");
			} catch (e) {
				xmlHttp = new ActiveXObject("Microsoft.XMLHTTP");
			}
		}
		return xmlHttp;
	}
</script>
</head>
<body>
</body>
</html>
```

#### 使用jquery来完成ajax

- **ajax 请求**

  - $.ajax(url,[settings\])
    - jQuery 底层 AJAX 实现。简单易用的高层实现见 `$.get`, `$.post` 等。`$.ajax() `返回其创建的 XMLHttpRequest 对象。大多数情况下你无需直接操作该函数，除非你需要操作不常用的选项，以获得更多的灵活性。
  - load(url,[data\],[callback])
  - $.get(url,[data\],[fn],[type])
    - 这是一个简单的 GET 请求功能以取代复杂 `$.ajax `。请求成功时可调用回调函数。如果需要在出错时执行函数，请使用 `$.ajax`。
  - $.getJSON(url,[data\],[fn])
  - $.getScript(url,[callback\])
  - $.post(url,[data\],[fn],[type])
    - 这是一个简单的 POST 请求功能以取代复杂 `$.ajax` 。请求成功时可调用回调函数。如果需要在出错时执行函数，请使用 `$.ajax`。

  **url**:待载入页面的URL地址

  **data**:待发送 Key/value 参数。

  **callback/fn**:载入成功时回调函数。

  **type**:返回内容格式，xml, html, script, json, text, _default。

- **ajax 事件**

  - ajaxComplete(callback)
  - ajaxError(callback)
  - ajaxSend(callback)
  - ajaxStart(callback)
  - ajaxStop(callback)
  - ajaxSuccess(callback)

```js
$.get("/day17-jquery/RegionServlet",{method2do:"getCity",code:$(obj).prop("name")},function(data){
			$("#bootstrapCity").html("");
			for(var i=0;i<data.length;i++){
				$("#bootstrapCity").append("<li><a href='#' name='"+data[i].code+"'  onclick=changeb(this)>"+data[i].fullName+"</a></li>");
			}
		},"json");
```

