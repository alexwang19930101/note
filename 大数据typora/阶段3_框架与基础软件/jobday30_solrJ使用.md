# SolrJ使用

## 1. 介绍

SolrJ是Apache官方提供的一套Java开发的，访问Solr服务的API
通过这套API可以让我们的程序与Solr服务产生交互，让我们的程序可以实现对Solr索引库的增删改查！
SolrJ的官方wiki地址：`https://wiki.apache.org/solr/Solrj`

maven依赖：

- 注意Solr会使用到HttpClient有可能会产生包冲突，所以要`<exclusions>`解决一下

- 下面的`<build>`模块`<resources>`是为了让**idea**加载到配置文件

```xml
<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>org.apache.solr</groupId>
        <artifactId>solr-solrj</artifactId>
        <version>4.10.2</version>
        <exclusions>
            <exclusion>
                <artifactId>httpclient</artifactId>
                <groupId>org.apache.httpcomponents</groupId>
            </exclusion>
            <exclusion>
                <artifactId>httpcore</artifactId>
                <groupId>org.apache.httpcomponents</groupId>
            </exclusion>
        </exclusions>
    </dependency>

    <!-- Solr底层会使用到slf4j日志系统 -->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-log4j12</artifactId>
        <version>1.7.22</version>
    </dependency>
    <dependency>
        <groupId>commons-logging</groupId>
        <artifactId>commons-logging</artifactId>
        <version>1.2</version>
    </dependency>
</dependencies>

<build>
    <resources>
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.xml</include>
                <include>**/*.properties</include>
            </includes>
        </resource>
        <resource>
            <directory>src/main/resources</directory>
            <includes>
                <include>**/*</include>
            </includes>
        </resource>
    </resources>
    <pluginManagement><!-- lock down plugins versions to avoid using Maven defaults (may be moved to parent pom) -->
        <plugins>
            <plugin>
                <artifactId>maven-clean-plugin</artifactId>
                <version>3.0.0</version>
            </plugin>
            <!-- see http://maven.apache.org/ref/current/maven-core/default-bindings.html#Plugin_bindings_for_jar_packaging -->
            <plugin>
                <artifactId>maven-resources-plugin</artifactId>
                <version>3.0.2</version>
            </plugin>
            <plugin>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.7.0</version>
            </plugin>
            <plugin>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.20.1</version>
            </plugin>
            <plugin>
                <artifactId>maven-jar-plugin</artifactId>
                <version>3.0.2</version>
            </plugin>
            <plugin>
                <artifactId>maven-install-plugin</artifactId>
                <version>2.5.2</version>
            </plugin>
            <plugin>
                <artifactId>maven-deploy-plugin</artifactId>
                <version>2.8.2</version>
            </plugin>
        </plugins>
    </pluginManagement>
</build>
```



## 2. update索引(添加/修改)

### 2.1 Document方式

```java
/**
javajava
```

这种方式已经比lucene的实现方式更加简单，但是依然存在可优化的空间，比如：本地数据库中的数据查询出以后被包装成javaBean，如果采用此方式，必须把javaBean拆解到documen对象中。那么solr有没有可解决的方案呢？答案是肯定的，solrServer除了提供add方法外，还提供了addBean方法。

### 2.2 JavaBean + 注解

数据的类型最好完全对应，不然会在查询时向JavaBean封装产生错误

如果要将从数据库查出的JavaBean直接插入Solr索引库

```java
 @Test
public void addJavaBeanToCore() throws IOException, SolrServerException{
    String url = "http://localhost:8080/solr/MyCore1";
    HttpSolrServer server = new HttpSolrServer(url);
    //同时新增多条数据
    List<ProductCopy> list = new ArrayList<>();
    for(int i = 6;i<9;i++){
        ProductCopy productCopy = new ProductCopy();
        productCopy.setId(""+i);
        productCopy.setName("产品"+i+"号");
        productCopy.setPrice(Float.valueOf(""+i));
        ArrayList<String> titleList = new ArrayList<>();
        productCopy.setTitle(titleList);
        list.add(productCopy);
    }
    //直接添加javaBean  添加单条数据
    //server.addBean(user);
    server.addBeans(list);
    //提交
    server.commit();
}
```

```java
import org.apache.solr.client.solrj.beans.Field;

@Field("id")
private String id;
@Field("name")
private String name;
@Field("title")
private List<String> title;
@Field("price")
private Float price;
```





## 3. 删除索引库

```java
/**
     * 根据id的集合同时删除多条数据
     * @throws SolrServerException
     * @throws IOException
     */
@Test
public void deleteById() throws SolrServerException, IOException{
    String url = "http://localhost:8080/solr/MyCore1";
    HttpSolrServer server = new HttpSolrServer(url);
    //根据id删除对应的数据
    server.deleteById("7");
    server.commit();
}

/**
     * 根据id的集合同时删除多条数据
     * @throws SolrServerException
     * @throws IOException
     */
@Test
public void deleteByIds() throws SolrServerException, IOException{
    String url = "http://localhost:8080/solr/MyCore1";
    HttpSolrServer server = new HttpSolrServer(url);
    //根据id删除对应的数据
    List<String> ids = new ArrayList<String>();
    ids.add("1");
    ids.add("8");
    //根据ids删除指定的数据
    server.deleteById(ids);
    server.commit();
}

/**
     * 根据查询条件删除
     * @throws SolrServerException
     * @throws IOException
     */
@Test
public void deleteByQuery() throws SolrServerException, IOException{
    String url = "http://localhost:8080/solr/MyCore1";
    HttpSolrServer server = new HttpSolrServer(url);
    //删除了所有数据，能够成功执行    安全越界  慎用
    String query = "*:*";

    //根据指定的查询条件删除  删除所有匹配的结果
    //        String query = "title:聪明";
    server.deleteByQuery(query);
    server.commit();
}
```



## 4. 查询索引库

```java
@Test
public void searchSolrCore() throws IOException, SolrServerException {
    String url = "http://localhost:8080/solr/MyCore1";
    HttpSolrServer server = new HttpSolrServer(url);
    //字段名：参数值
    //SolrQuery query = new SolrQuery("title:互*");
    //布尔搜索
    //SolrQuery query = new SolrQuery("title:solr OR title:iphone");
    //子表达式查询
    //SolrQuery query = new SolrQuery("(title:solr OR title:聪明) AND (name:产品)");
    //相似度搜索，最大允许编辑次数为2次
    //		SolrQuery query = new SolrQuery("iphon~1");
    //数字范围搜索  包含起始值和结束值
    SolrQuery query = new SolrQuery("id:[5 TO 7]");
    //根据id字段倒叙排列
    query.setSort("id", SolrQuery.ORDER.desc);
    QueryResponse response = server.query(query);
    //解析搜索的结果集
    SolrDocumentList list = response.getResults();
    for(SolrDocument sd : list){
        System.out.println("id===="+sd.get("id"));
        System.out.println("name===="+sd.get("name"));
        System.out.println("title===="+sd.get("title"));
        System.out.println("===========next===========");
    }
    //解析搜索结果集的方式二  ：直接返回javaBean的方式
    List<ProductCopy> listCopy = response.getBeans(ProductCopy.class);
    for(ProductCopy productCopy : listCopy){
        System.out.println(productCopy);
    }
}
```





## 5. 高亮与分页

```java
/**
     * 搜索结果集的分页
     * 分页关键： 需要页码和每页显示条数
     *   获取结果集时，指定开始位置start和结束位置end
     * @throws SolrServerException
     */
@Test
public void searchSolrCorePageList() throws SolrServerException{
    String url = "http://localhost:8080/solr/MyCore1";
    HttpSolrServer server = new HttpSolrServer(url);
    //页码
    Integer page = 1;
    //每页显示条数
    Integer pageSize = 15;
    Integer start = (page -1 ) * pageSize;
    Integer end = page * pageSize;
    //字段名：参数值
    SolrQuery query = new SolrQuery("name:产品");
    //根据id字段倒叙排列
    query.setSort("id", SolrQuery.ORDER.desc);
    query.setStart(start);
    query.setRows(end);

    //开启高亮显示
    query.setHighlight(true);
    //高亮显示的标签前缀
    query.setHighlightSimplePre("<em color='red'>");
    //高亮显示的标签后缀
    query.setHighlightSimplePost("</em>");
    //添加需要高亮显示的字段
    query.addHighlightField("title");
    query.addHighlightField("name");
    QueryResponse response = server.query(query);
    //高亮显示的结果集
    Map<String, Map<String, List<String>>> highlighting = response.getHighlighting();
    //遍历的方式获取高亮的结果集
    for(String key : highlighting.keySet()){
        Map<String, List<String>> map = highlighting.get(key);
        for(String key2 : map.keySet()){
            List<String> list = map.get(key2);
            for(String str : list){
                System.out.println(str);
            }
        }
    }

    //解析搜索结果集的方式二  ：直接返回javaBean的方式
    List<ProductCopy> list = response.getBeans(ProductCopy.class);
    for(ProductCopy u : list){
        String highlightName = highlighting.get(u.getId()).get("name").get(0);
        //			System.out.println(highlightName);
        u.setName(highlightName);
        System.out.println(u);
    }

}
```





