# 网络爬虫

![20181017105838](../图片/20181017105838.png)

## 1.网络爬虫是什么？

网络爬虫是一种运行在互联网上用来数据的自动化程序和脚本

- 分解出三个点：
  - 互联网上都有哪些数据？
    - 形形色色的网站组成（新闻|贴吧|知道|音乐|图片|视频|地图|文库）
    - 电商网站（用户|商品|订单|支付|物流|评论|分享）
    - 微博（发送信息|转发|点赞|关注|被关注）
    - ……
  - 怎么去获取？通过什么样的技术手段去获取
    - 网站的本质就是服务端程序，通过客户端（浏览器）和服务端尽心进行交互。
    - HTTP协议（HTTP GET 或者Post ）
    - 开发网络爬虫，就是使用HTTP协议模拟浏览器进行网络请求
    - request 就是提交请求信息
    - response 获得数据
  - 自动化程序或脚本 使用什么语言进行开发
    - 语言是没有任何限制（Python、Java、.net、任何一种）



## 2.网络爬虫作用域分类

1）爬虫爬取的数据可以用来做数据分析

2）爬虫爬去的数据可以给搜索系统使用

3）竞品分析

4）基于数据的商业模式



网络爬虫按照系统结构和实现技术，大致可以分为以下几种类型：

- 通用网络爬虫（General Purpose Web Crawler）

- 聚焦网络爬虫（Focused Web Crawler）

- 增量式网络爬虫（Incremental Web Crawler）

- 深层网络爬虫（Deep Web Crawler）



## 3.爬虫过程

1.请求获取数据

2.处理解析数据

3.存储数据

### 3.1 请求获取数据

#### 3.1.1 JDK URL

```java
package getinfo;

import java.io.*;
import java.net.URL;
import java.net.URLConnection;
import java.nio.charset.Charset;

public class MyJDKHTTPPost {

    public static void main(String[] args) throws IOException {
        //请求路径 请求方式 请求参数 响应数据

        //1.创建url对象，指定请求路径
        URL url = new URL("https://www.baidu.com/");
        //2.开启连接,post请求数据在请求体，要开启output进行传输
        URLConnection conn = url.openConnection();
        conn.setDoOutput(true);

        OutputStream os = conn.getOutputStream();
        os.write("username=aa&password=aa".getBytes());
        os.flush();
        os.close();
        //3.发送请求，获取响应数据
        InputStream is = conn.getInputStream();
        //4.包装打印数据
        BufferedReader reader = new BufferedReader(new InputStreamReader(is,Charset.forName("utf-8")));

        String line = null;
        while ((line = reader.readLine())!= null ){
            System.out.println(line);
        }

        reader.close();
        is.close();
    }
}
```

#### 3.1.2 HttpClient

```java
package getinfo;

import org.apache.http.Header;
import org.apache.http.HttpEntity;
import org.apache.http.HttpResponse;
import org.apache.http.NameValuePair;
import org.apache.http.client.HttpClient;
import org.apache.http.client.entity.UrlEncodedFormEntity;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.message.BasicNameValuePair;
import org.apache.http.util.EntityUtils;

import java.util.ArrayList;
import java.util.List;

public class MyHttpClient {
    public static void main(String[] args) throws Exception {
        //1.获取httpClient对象
        HttpClient httpClient = HttpClients.createDefault();
        //2.设置http 请求方式 请求路径
        HttpGet httpget = new HttpGet("https://www.baidu.com/");
        //3.通过httpClient对象执行步骤2定义的请求内容，得到返回对象
        HttpResponse response = httpClient.execute(httpget);
        //4.由返回对象执行处理
        HttpEntity entity = response.getEntity();//entity是获取的响应内容

        if (entity != null) {
            System.out.println("Login form get: " + response.getStatusLine());
            //打印响应内容
            System.out.println(EntityUtils.toString(entity,"utf-8"));
        }

        System.out.println("------------------");
        
        HttpPost httpost = new HttpPost("https://www.baidu.com/");
		//post请求需要传入请求体请求参数
        List <NameValuePair> nvps = new ArrayList<>();
        nvps.add(new BasicNameValuePair("username", "username"));
        nvps.add(new BasicNameValuePair("password", "password"));

        httpost.setEntity(new UrlEncodedFormEntity(nvps, "utf-8"));

        response = httpClient.execute(httpost);
        entity = response.getEntity();

        System.out.println("Login form post: " + response.getStatusLine());
        Header[] headers = response.getAllHeaders();
        if (entity != null) {
            for (int i = 0; i < headers.length; i++) {
                System.out.println(headers[i].getName()+" "+headers[i].getValue());
            }
        }
    }
}
```



### 3.2 处理数据

#### 3.2.1 document对象(DOM)xml

了解

#### 3.2.2 jsoup

maven依赖

```xml
<dependency>
    <groupId>org.jsoup</groupId>
    <artifactId>jsoup</artifactId>
    <version>1.10.3</version>
</dependency>
```

jsoup参考文档

> http://www.open-open.com/jsoup/selector-syntax.htm

jsoup使用实例

```java
package parsedata;

import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;
import org.jsoup.select.Elements;

import java.io.IOException;
import java.net.URL;

public class MyJsoup {
    public static void main(String[] args) throws IOException {
        //jsoup获取docuemnt
        Document doc = Jsoup.parse(new URL("http://www.itcast.cn"),2000);
        //获取 标签
        Elements aTag = doc.select("a");
        //获取 id
        Elements webimID = doc.select("#webim");
        //获取 class
        Elements  footerClass = doc.select(".footer");
        //获取某个属性的
        Elements srcAttr = doc.select("[src]");
        //获取属性以什么开头的
        Elements sStartAttr = doc.select("[^s]");
        // 获取某个属性值等于什么的
        Elements jslist = doc.select("[src=/js/index.js]");
        // [attr^=value] 开始，[attr$=value] 结束，[attr*=value] 包含
    }
}
```

