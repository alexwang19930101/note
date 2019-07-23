## 1.solr配置安装

流程见阶段3笔记

本次在项目中将solr和redis都安装在虚拟机128上，减少需要手动启动的东西

## 2.spring-data-solr

使用spring-data-solr的注解与配置完成数据库表items信息的导入

完成solr导入工具

配置如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:solr="http://www.springframework.org/schema/data/solr"
	xsi:schemaLocation="http://www.springframework.org/schema/data/solr 
  		http://www.springframework.org/schema/data/solr/spring-solr-1.0.xsd
		http://www.springframework.org/schema/beans 
		http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context 
		http://www.springframework.org/schema/context/spring-context.xsd">
	
	<!-- solr服务器地址 -->
	<solr:solr-server id="solrServer" url="http://192.168.25.128:8080/solr/MyCore1" />

	<!-- solr模板，使用solr模板可对索引库进行CRUD的操作 -->
	<bean id="solrTemplate" class="org.springframework.data.solr.core.SolrTemplate">
		<constructor-arg ref="solrServer" />
	</bean>
</beans>
```

http://192.168.25.128:8080/solr/MyCore1最后的core指定是为了让程序连接指定的core

使用：

```java
@Autowired
private SolrTemplate solrTemplate;

solrTemplate.saveBeans(itemList);
solrTemplate.commit();
```

将solr包放在pojo下，而dao依赖pojo，后面的依赖dao的包都可以使用到solr

pojo的配置：

```java
@Field
private Long id;

@Field("item_title")
private String title;

@Dynamic
@Field("item_spec_*")
private Map<String,String> specMap;
```

以上3中很好理解，不再赘述



## 搜索解决方案

​	大多数搜索引擎应用都必须具有某种搜索功能，问题是搜索功能往往是巨大的资源消耗并且它们由于沉重的数据库加载而拖垮你的应用的性能。
​	这就是为什么转移负载到一个外部的搜索服务器是一个不错的主意，Apache Solr 是一个流行的开源搜索服务器，它通过使用类似 REST 的 HTTP API，这就确保你能从几乎任何编程语言来使用 solr。

​	Solr 是一个开源搜索平台，用于构建搜索应用程序。 它建立在 Lucene(全文搜索引擎)之上。 Solr 是企业级的，快速的和高度可扩展的。 使用 Solr 构建的应用程序非常复杂，可提供高性能。

