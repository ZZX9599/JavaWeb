# Javaweb重点

# 1:JDBC

## 1.1:JDBC架构

<img src=" https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/image-20221214160932958.png" alt="image-20221214160932958" style="zoom:200%;" />

我们开发的同一套Java代码是无法操作不同的关系型数据库，因为每一个关系型数据库的底层实现细节都不一样。如果这样，问题就很大了，在公司中可以在开发阶段使用的是MySQL数据库，而上线时公司最终选用oracle数据库，我们就需要对代码进行大批量修改，这显然并不是我们想看到的。我们要做到的是同一套Java代码操作不同的关系型数据库，而此时sun公司就指定了一套标准接口（JDBC），JDBC中定义了所有操作关系型数据库的规则。众所周知接口是无法直接使用的，我们需要使用接口的实现类，而这套实现类（称之为：驱动）就由各自的数据库厂商给出

## 1.2:JDBC本质

* 官方（sun公司）定义的一套操作所有关系型数据库的规则，即接口
* 各个数据库厂商去实现这套接口，提供数据库驱动jar包
* 我们可以使用这套接口（JDBC）编程，真正执行的代码是驱动jar包中的实现类

## 1.3:JDBC好处

1：各数据库厂商使用相同的接口，Java代码不需要对不同数据库分别开发

2：可随时替换底层数据库，访问数据库的Java代码基本不变

以后编写操作数据库的代码只需要面向JDBC（接口），操作哪儿个关系型数据库就需要导入该数据库的驱动包，如需要操作MySQL数据库，就需要再项目中导入MySQL数据库的驱动包。如下图就是MySQL驱动包

## 1.4:标准步骤

1：导入jar包

2：注册驱动

```sql
Class.forName("com.mysql.cj.jdbc.Driver");
```

3：获取连接

```sql
Connection conn = DriverManager.getConnection(url, username, password);
```

Java代码需要发送SQL给MySQL服务端，就需要先建立连接

4：定义SQL语句

```sql
String sql =  “update…” ;
```

5：获取执行SQL对象

执行SQL语句需要SQL执行对象，而这个执行对象就是Statement对象

```sql
Statement stmt = conn.createStatement();
```

6：执行SQL

```sql
stmt.executeUpdate(sql);  
```

7：处理返回结果

8：释放资源

## 1.5:JDBC的API

### 1:DriverManager

DriverManager（驱动管理类）作用：注册对应的驱动【也就是数据库厂商实现的接口】

面向接口编程，实际上就是注册数据库厂商的那些实现类

![image-20221214162015454]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/image-20221214162015454.png)

这个类在java.sql下面，内部有一个注册驱动的方法

![image-20221214162339975]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/image-20221214162339975.png)

但是我们一般不这样写，而是  `Class.forName("com.mysql.cj.jdbc.Driver")`

来看看Mysql提供的Driver的源码

### 2:Driver

我们查询MySQL提供的Driver类，看它是如何实现的，源码如下：

<img src=" https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/image-20210725171635432.png" alt="image-20210725171635432" style="zoom:70%;" />

在该类中的静态代码块中已经执行了 `DriverManager` 对象的 `registerDriver()` 方法进行驱动的注册了，那么我们只需要加载 `Driver` 类，该静态代码块就会执行，所以一般我们都是采用Class.forName的形式

Class.forName("com.mysql.cj.jdbc.Driver")---->

导致Driver加载---->

导致DriverManager的registerDriver执行

### 3:Connection

叫做数据库连接对象，也就是连接数据库的对象，管理事务的操作就跟它有关系

使用 Connection conn = DriverManager.getConnection(url, username, password) 来拿到

参数说明：

* url ： 连接路径

  > 语法：jdbc:mysql://ip地址(域名):端口号/数据库名称?参数键值对1&参数键值对2…
  >
  > 示例：jdbc:mysql://127.0.0.1:3306/db1
  >
  > ==细节：==
  >
  > * 如果连接的是本机mysql服务器，并且mysql服务默认端口是3306，则url可以简写为：jdbc:mysql:///数据库名称?参数键值对
  >
  > * 配置 useSSL=false 参数，禁用安全连接方式，解决警告提示

* user ：用户名

* poassword ：密码



Connection（数据库连接对象）作用：

* 获取执行 SQL 的对象
* 管理事务



#### 1:获取执行SQL对象

1：普通执行SQL对象

```sql
Statement createStatement()
```

入门案例中就是通过该方法获取的执行对象



2：预编译SQL的执行SQL对象：防止SQL注入

```sql
PreparedStatement  prepareStatement(sql)
```

通过这种方式获取的 `PreparedStatement` SQL语句执行对象可以防止SQL注入

#### 2:事务管理

先回顾一下MySQL事务管理的操作：

* 开启事务 ： BEGIN   或者 START TRANSACTION
* 提交事务 ： COMMIT
* 回滚事务 ： ROLLBACK

> MySQL默认是自动提交事务

接下来学习JDBC事务管理的方法

Connection接口中定义了3个对应的方法：

1：setAutoCommit

2：commit

3：rollback



### 4:Statement

Statement的作用就是用来执行SQL语句。而针对不同类型的SQL语句使用的方法也不一样

执行DDL DML等操作类型的SQL语句，我们使用的是executeUpdate()

执行DQL这种查询的SQL语句，我们使用的是executeQuery()



注意：开发很少使用java代码操作DDL语句



### 5:ResultSet

这个对象封装了SQL查询语句的结果

而执行了DQL语句后就会返回该对象，对应执行DQL语句的方法如下：

```sql
ResultSet  executeQuery(sql)：执行DQL 语句，返回 ResultSet 对象
```

那么我们就需要从 `ResultSet` 对象中获取我们想要的数据

恰好`ResultSet` 对象提供了操作查询结果数据的方法，如下：

> boolean  next()
>
> * 将光标从当前位置向前移动一行 
> * 判断当前行是否为有效行
>
> 方法返回值说明：
>
> * true  ： 有效航，当前行有数据
> * false ： 无效行，当前行没有数据

> xxx  getXxx(参数)：获取数据
>
> * xxx : 数据类型；如： int getInt(参数) ；String getString(参数)
> * 参数
>   * int类型的参数：列的编号，从1开始
>   * String类型的参数： 列的名称 

如下图为执行SQL语句后的结果

<img src=" https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/image-20210725181320813.png" alt="image-20210725181320813" style="zoom:80%;" />

一开始光标指定于第一行前，如图所示红色箭头指向于表头行。当我们调用了 `next()` 方法后，光标就下移到第一行数据，并且方法返回true，此时就可以通过 `getInt("id")` 获取当前行id字段的值，也可以通过 `getString("name")` 获取当前行name字段的值。如果想获取下一行的数据，继续调用 `next()`  方法，以此类推

### 6:PreparedStatement

SQL注入是通过操作输入来修改事先定义好的SQL语句，用以达到执行代码对服务器进行攻

击的方法



例如：

用户名和密码输入正确就登陆成功，跳转到首页。用户名和密码输入错误则给出错误提示

但是我可以通过输入一些特殊的字符登陆到首页。

用户名随意写，密码写成 `' or '1' ='1`

这就是SQL注入漏洞，也是很危险的。当然现在市面上的系统都不会存在这种问题了

所以大家也不要尝试用这种方式去试其他的系统

这里就可以将SQL执行对象 `Statement` 换成 `PreparedStatement` 对象



1：获取 PreparedStatement 对象

```java
// SQL语句中的参数值，使用？占位符替代
String sql = "select * from user where username = ? and password = ?";

// 通过Connection对象获取，并传入对应的sql语句
PreparedStatement pstmt = conn.prepareStatement(sql);
```

2：设置参数值

上面的sql语句中参数使用 ? 进行占位，在之前之前肯定要设置这些 ?  的值

> PreparedStatement对象：setXxx(参数1，参数2)：给 ? 赋值
>
> * Xxx：数据类型 ； 如 setInt (参数1，参数2)
>
> * 参数：
>
>   * 参数1： ？的位置编号，从1 开始
>
>   * 参数2： ？的值

3：执行SQL语句

> executeUpdate();  执行DDL语句和DML语句
>
> executeQuery();  执行DQL语句
>
> ==注意：==
>
> * 调用这两个方法时不需要传递SQL语句，因为获取SQL语句执行对象时已经对SQL语句进行预编译了



执行上面语句就可以发现不会出现SQL注入漏洞问题了。

那么PreparedStatement又是如何解决的呢？

实际上它是将特殊字符进行了转义，把我们之前的那个` or '1' ='1`转为了真正的字符串

而不是特殊字符，这样就不会被处理为 or 关键字了



好处：

1：预编译SQL，性能更高

2：防止SQL注入问题



Java代码操作数据库流程如图所示：

<img src=" https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/image-20210725195756848.png" alt="image-20210725195756848" style="zoom:80%;" />

* 将sql语句发送到MySQL服务器端

* MySQL服务端会对sql语句进行如下操作

  * 检查SQL语句

    检查SQL语句的语法是否正确

  * 编译SQL语句。将SQL语句编译成可执行的函数

    检查SQL和编译SQL花费的时间比执行SQL的时间还要长。如果我们只是重新设置参数，那么检查SQL语句和编译SQL语句将不需要重复执行。这样就提高了性能

  * 执行SQL语句



## 1.6:数据库连接池

### 1:简介

> * 数据库连接池是个容器，负责分配、管理数据库连接(Connection)
>
> * 它允许应用程序重复使用一个现有的数据库连接，而不是再重新建立一个
>
> * 释放空闲时间超过最大空闲时间的数据库连接来避免因为没有释放数据库连接而引起的数据库连接遗漏
> * 好处
>   * 资源重用
>   * 提升系统响应速度
>   * 避免数据库连接遗漏

之前我们代码中使用连接是没有使用都创建一个Connection对象，使用完毕就会将其销毁

恰好最浪费时间的过程就是创建Connection对象的时间

这样重复创建销毁的过程是特别耗费计算机的性能的及消耗时间的

而数据库使用了数据库连接池后，就能达到Connection对象的复用，如下图

<img src=" https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/image-20210725210432985.png" alt="image-20210725210432985" style="zoom:80%;" />

连接池是在一开始就创建好了一些连接（Connection）对象存储起来。用户需要连接数据库时，不需要自己创建连接，而只需要从连接池中获取一个连接进行使用，使用完毕后再将连接对象归还给连接池；这样就可以起到资源重用，也节省了频繁创建连接销毁连接所花费的时间，从而提升了系统响应的速度

### 2:数据库连接池实现

* 标准接口：javax.sql.DataSource

  官方(SUN) 提供的数据库连接池标准接口，由第三方组织实现此接口。该接口提供了获取连接的功能：

  ```java
  Connection getConnection()
  ```

  那么以后就不需要通过 `DriverManager` 对象获取 `Connection` 对象

  而是通过连接池（DataSource）获取 `Connection` 对象

* 常见的数据库连接池

  * DBCP
  * C3P0
  * Druid

  我们现在使用更多的是Druid，它的性能比其他两个会好一些

* Druid（德鲁伊）

  * Druid连接池是阿里巴巴开源的数据库连接池项目 

  * 功能强大，性能优秀，是Java语言最好的数据库连接池之一

### 3:使用德鲁伊

> * 导入jar包 druid-1.1.12.jar
> * 定义配置文件
> * 加载配置文件
> * 获取数据库连接池对象
> * 获取连接

获取数据库连接池对象：使用  `DruidDataSourceFactory.createDataSource(配置文件)`

编写配置文件如下：

```properties
driverClassName=com.mysql.jdbc.cj.Driver
url=jdbc:mysql:///db1?useSSL=false&useServerPrepStmts=true
username=root
password=JXLZZX79
# 初始化连接数量
initialSize=5
# 最大连接数
maxActive=10
# 最大等待时间
maxWait=3000
```

使用druid的代码如下：

```java
/**
 * Druid数据库连接池演示
 */
public class DruidDemo {

    public static void main(String[] args) throws Exception {
        // 加载配置文件
        Properties prop = new Properties();
        prop.load(new FileInputStream("jdbc-demo/src/druid.properties"));
        
        // 获取连接池对象
        DataSource dataSource = DruidDataSourceFactory.createDataSource(prop);

        // 获取数据库连接 Connection
        Connection connection = dataSource.getConnection();
        
        // 获取到了连接后就可以继续做其他操作了
    }
}
```

# 2:HTTP协议

## 2.1:介绍

HTTP【HyperText Transfer Protocol】

超文本传输协议，规定了浏览器和服务器之间==数据传输的规则



1：数据传输的规则指的是请求数据和响应数据需要按照指定的格式进行传输

2：如果想知道具体的格式，可以打开浏览器，点击`F12`打开开发者工具查看



学习HTTP主要就是学习请求和响应数据的具体格式内容



## 2.2:特点

HTTP协议有它自己的一些特点，分别是:

1：基于`TCP`协议:：面向连接，安全

TCP是一种面向连接的(建立连接之前是需要经过三次握手)、可靠的、基于字节流的传输层

通信协议，在数据传输方面更安全



2：基于请求-响应模型的：一次请求对应一次响应

请求和响应是一一对应关系



3：HTTP协议是无状态协议：对于事物处理没有记忆能力。每次请求-响应都是独立的

无状态指的是客户端发送HTTP请求给服务端之后，服务端根据请求响应数据，响应完毕之

后是不会记录任何信息的，这种特性有优点也有缺点



4：缺点：多次请求间不能共享数据

请求之间无法共享数据会引发的问题，如:

* 京东购物，`加入购物车`和`去购物车结算`是两次请求
* HTTP协议的无状态特性，加入购物车请求响应结束后，并未记录加入购物车是何商品
* 发起去购物车结算的请求后，因为无法获取哪些商品加入了购物车，会导致此次请求无法正确展示数据
* 解决方式使用会话技术`Cookie、Session`



5：优点：速度快

## 2.3:请求数据格式

请求数据总共分为三部分内容，分别是==请求行==、==请求头==、==请求体==

![1627050004221]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/1627050004221.png)

1：请求行: HTTP请求中的第一行数据，包含一些最基本的信息，请求方式，协议版本等

请求方式有七种，最常用的是GET和POST



2：请求头: 第二行开始，格式为key: value形式

请求头中会包含若干个属性，常见的HTTP请求头有:

```xml
Host: 表示请求的主机名

User-Agent: 浏览器版本
例如Chrome浏览器的标识类似Mozilla/5.0 ...Chrome/79
IE浏览器的标识类似Mozilla/5.0 (Windows NT ...)like Gecko

Accept：表示浏览器能接收的资源类型，如text/*，image/*或者*/*表示所有

Accept-Language：表示浏览器偏好的语言，服务器可以据此返回不同语言的网页

Accept-Encoding：表示浏览器可以支持的压缩类型，例如gzip, deflate等
```



3：请求体: POST请求的最后一部分，存储请求参数

![1627050930378]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/1627050930378.png)

如上图红线框的内容就是请求体的内容，请求体和请求头之间是有一个空行隔开

此时浏览器发送的是POST请求，为什么不能使用GET呢?

这时就需要回顾GET和POST两个请求之间的区别了:

* GET请求请求参数在请求行中，没有请求体，POST请求请求参数在请求体中
* GET请求请求参数大小有限制，POST没有

## 2.4:响应数据格式

响应数据总共分为三部分内容，分别是==响应行==、==响应头==、==响应体==

![1627053710214]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/1627053710214.png)

1：响应行：响应数据的第一行，包含响应的基本信息



2：响应头：第二行开始，格式为key：value形式

响应头中会包含若干个属性，常见的HTTP响应头有:

```
Content-Type：表示该响应内容的类型，例如text/html，image/jpeg；

Content-Length：表示该响应内容的长度（字节数）；

Content-Encoding：表示该响应压缩算法，例如gzip；

Cache-Control：指示客户端应如何缓存，例如max-age=300表示可以最多缓存300秒
```

3：响应体： 最后一部分。存放响应数据

上图中<html>...</html>这部分内容就是响应体，它和响应头之间有一个空行隔开

## 2.5:状态码

### 1:整体分类

| 状态码分类 | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| 1xx        | **响应中**——临时状态码，表示请求已经接受，告诉客户端应该继续请求或者如果它已经完成则忽略它 |
| 2xx        | **成功**——表示请求已经被成功接收，处理已完成                 |
| 3xx        | **重定向**——重定向到其它地方：它让客户端再发起一个请求以完成整个处理。 |
| 4xx        | **客户端错误**——处理发生错误，责任在客户端，如：客户端的请求一个不存在的资源，客户端未被授权，禁止访问等 |
| 5xx        | **服务器端错误**——处理发生错误，责任在服务端，如：服务端抛出异常，路由出错，HTTP版本不支持等 |

状态码大全：https://cloud.tencent.com/developer/chapter/13553 



### 2:常见的响应状态码

| 状态码 |              英文描述               | 解释                                                         |
| ------ | :---------------------------------: | ------------------------------------------------------------ |
| 200    |               **OK**                | 客户端请求成功，即**处理成功**，这是我们最想看到的状态码     |
| 302    |              **Found**              | 指示所请求的资源已移动到由`Location`响应头给定的 URL，浏览器会自动重新访问到这个页面 |
| 304    |          **Not Modified**           | 告诉客户端，你请求的资源至上次取得后，服务端并未更改，你直接用你本地缓存吧。隐式重定向 |
| 400    |           **Bad Request**           | 客户端请求有**语法错误**，不能被服务器所理解                 |
| 403    |            **Forbidden**            | 服务器收到请求，但是**拒绝提供服务**，比如：没有权限访问相关资源 |
| 404    |            **Not Found**            | **请求资源不存在**，一般是URL输入有误，或者网站资源被删除了  |
| 428    |      **Precondition Required**      | **服务器要求有条件的请求**，告诉客户端要想访问该资源，必须携带特定的请求头 |
| 429    |        **Too Many Requests**        | **太多请求**，可以限制客户端请求某个资源的数量，配合 Retry-After(多长时间后可以请求)响应头一起使用 |
| 431    | **Request Header Fields Too Large** | **请求头太大**，服务器不愿意处理请求，因为它的头部字段太大。请求可以在减少请求头域的大小后重新提交。 |
| 405    |       **Method Not Allowed**        | 请求方式有误，比如应该用GET请求方式的资源，用了POST          |
| 500    |      **Internal Server Error**      | **服务器发生不可预期的错误**。服务器出异常了，赶紧看日志去吧 |
| 503    |       **Service Unavailable**       | **服务器尚未准备好处理请求**，服务器刚刚启动，还未初始化好   |
| 511    | **Network Authentication Required** | **客户端需要进行身份验证才能获得网络访问权限**               |

# 3:web服务器

## 1:介绍

什么是Web服务器

Web服务器是一个应用程序（==软件==）`对HTTP协议的操作进行封装`，使得程序员不必直接对协议进行操作，让Web开发更加便捷。主要功能是"提供网上信息浏览服务"

![1627058356051]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/1627058356051.png)

Web服务器是安装在服务器端电脑的一款软件，将来我们把自己写的Web项目部署到Web Tomcat服务器软件中，当Web服务器软件启动后，部署在Web服务器软件中的页面就可以直接通过浏览器来访问了

**Web服务器软件使用步骤**

* 准备静态资源
* 下载安装Web服务器软件
* 将静态资源部署到Web服务器上
* 启动Web服务器使用浏览器访问对应的资源



而对于Web服务器来说，Tomcat只是其中的一种，而除了Tomcat以外，还有很多优秀的Web

服务器，比如:

1：Jetty

2：Webogic

## 2:JavaEE规范

先来看Tomcat的相关概念:

* Tomcat是Apache软件基金会一个核心项目，是一个开源免费的轻量级Web服务器，支持

  了少量的JavaEE规范【Servlet/JSP】实现了部分规范

  

* 概念中提到了JavaEE规范，那什么又是JavaEE规范呢?

  JavaEE: Java Enterprise Edition，Java企业版。指Java企业级开发的技术规范总和。包含

  13项技术规范:JDBC、JNDI、EJB、RMI、JSP、Servlet、XML、JMS、Java IDL、JTS、

  JTA、JavaMail、JAF

  

* 因为Tomcat支持了Servlet和JSP规范，所以Tomcat也被称为Web容器、Servlet容器，所

  以我们的Servlet可以放进Tomcat内运行

  

* Tomcat的官网: https://tomcat.apache.org/ 从官网上可以下载对应的版本进行使用



总结Web服务器的作用

> 封装HTTP协议操作，简化开发
>
> 可以将Web项目部署到服务器中，对外提供网上浏览服务

# 4:Servlet

## 4.1:介绍

1：Servlet是JavaWeb最为核心的内容，它是Java提供的一门动态的web资源开发技术

2：使用Servlet就可以实现，根据不同的登录用户在页面上动态显示不同内容

3：Servlet是JavaEE规范之一，其实就是一个接口，将来我们需要定义Servlet类实现Servlet

接口，并由web服务器运行Servlet【因为web服务器实现了Servlet规范】



## 4.2:依赖

```xml
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>servlet-api</artifactId>
    <version>2.5</version>
    <scope>provided</scope>
</dependency>

<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>4.0.1</version>
    <scope>provided</scope>
</dependency>
```

这两个都是Servlet的依赖

但第一个是**servlet 3.0**以下的引入，第二个是**servlet 3.0**版本以上的引入

**servlet 3.0**之后引入了注解，如果想使用**servlet注解**需要引入第二种依赖

简单来说第二种依赖就是第一种依赖的升级版，包含了第一种依赖的全部功能



如果不配置scope，会把jar包发布，会跟容器里的jar包可能产生冲突

scope要用provided，则由容器（Tomcat）提供，不会发布

可以使用5个值：

provided，类似compile，期望JDK、容器或使用者会提供这个依赖。如servlet.jar

compile，默认值，适用于所有阶段，会随着项目一起发布

runtime，只在运行时使用，如JDBC驱动，适用运行和测试阶段

test，只在测试时使用，用于编译和运行测试代码。不会随项目发布

system，类似provided，需要显式提供包含依赖的jar，Maven不会在Repository中查找它





tomcat版本引发的问题

问题描述：

在使用10.0以上版本的tomcat后，会出现虚拟路径无法映射问题，而且会报500错

经查证发现，问题出现在依赖的版本

Tomcat 10将包命名空间从 javax.Servlet 更改为了 jakarta.Servlet

也就是说继续使用下面的依赖，他是无法识别的：

```xml
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>4.0.1</version>
    <scope>provided</scope>
</dependency>
```


如何解决：

具体有下面两种方法

1：更换tomcat版本到10.0以下

2：更换maven依赖为：

```xml
<dependency>
    <groupId>org.apache.tomcat</groupId>
    <artifactId>tomcat-servlet-api</artifactId>
    <version>10.0.8</version>
</dependency>
```


同时所有导包的路径也需要更改：

更改前：

```java
import javax.servlet.http.HttpServlet;
```


更改后：

```java
import jakarta.servlet.http.HttpServlet;
```

## 4.3:快速入门

我这里使用Tomcat9的版本

```xml
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>4.0.1</version>
    <!--
      此处为什么需要添加该标签?
      provided指的是在编译和测试过程中有效,最后生成的war包时不会加入
      因为Tomcat的lib目录中已经有servlet-api这个jar包
	  如果在生成war包的时候生效就会和Tomcat中的jar包冲突，导致报错
    -->
    <scope>provided</scope>
</dependency>
```

![image-20221214211512328]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/image-20221214211512328.png)

```java
@WebServlet("/user")
public class ServletDemo1 implements Servlet {

    public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
        System.out.println("servlet hello world~");
    }
    public void init(ServletConfig servletConfig) throws ServletException {

    }

    public ServletConfig getServletConfig() {
        return null;
    }

    public String getServletInfo() {
        return null;
    }

    public void destroy() {

    }
}
```

访问：http://localhost:8080/servlet01/user



## 4.4:执行流程

```java
public class ServletDemo1 implements Servlet {

    public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
        System.out.println("servlet hello world~");
    }
    public void init(ServletConfig servletConfig) throws ServletException {

    }

    public ServletConfig getServletConfig() {
        return null;
    }

    public String getServletInfo() {
        return null;
    }

    public void destroy() {

    }
}
```

Servlet程序已经能正常运行，但是我们需要思考个问题: 

我们并没有创建ServletDemo1类的对象，也没有调用对象中的service方法

为什么在控制台就打印了`servlet hello world~`这句话呢?



要想回答上述问题，我们就需要对Servlet的执行流程进行一个学习

![1627236923139]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/1627236923139.png)

* 浏览器发出`http://localhost:8080/web-demo/demo1`请求，从请求中可以解析出三部分内容，分别是`localhost:8080`、`web-demo`、`demo1`

  * 根据`localhost:8080`可以找到要访问的Tomcat Web服务器
  * 根据`web-demo`可以找到部署在Tomcat服务器上的web-demo项目
  * 根据`demo1`可以找到要访问的是项目中的哪个Servlet类，根据@WebServlet后面的值进行匹配

  

* 找到ServletDemo1这个类后，Tomcat Web服务器就会为ServletDemo1这个类创建一个对

  象，然后调用对象中的service方法【是Tomcat完成对象的创建和方法的调用】

  

* ServletDemo1实现了Servlet接口，所以类中必然会重写service方法供Tomcat Web服务器

  进行调用

  

* service方法中有ServletRequest和ServletResponse两个参数，ServletRequest封装的是请

  求数据，ServletResponse封装的是响应数据，后期我们可以通过这两个参数实现前后端

  的数据交互

  

1. Servlet由谁创建?Servlet方法由谁调用?

> Servlet由web服务器创建，Servlet方法由web服务器调用

2. 服务器怎么知道Servlet中一定有service方法?

> 因为我们自定义的Servlet，必须实现Servlet接口并重写其方法，而Servlet接口中有
>
> service方法



## 4.5:生命周期

介绍完Servlet的执行流程后，我们知道Servlet对象是由Tomcat Web服务器帮我们创建的

接下来咱们再来思考一个问题：Tomcat什么时候创建的Servlet对象?

要想回答上述问题，我们就需要对Servlet的生命周期进行一个学习

生命周期: 对象的生命周期指一个对象从被创建到被销毁的整个过程

Servlet运行在Servlet容器(web服务器)中，其生命周期由容器来管理，分为4个阶段：



1：加载和实例化【默认情况下，当Servlet第一次被访问时，才会由容器创建Servlet对象】

```xml
默认情况：Servlet会在第一次访问被容器创建，但是如果创建Servlet比较耗时的话，那么第一个访问的人等待的时间就比较长，用户的体验就比较差，那么我们能不能把Servlet的创建放到服务器启动的时候来创建，具体如何来配置?

@WebServlet(urlPatterns = "/demo1",loadOnStartup = 1)
loadOnstartup的取值有两类情况
1：负整数:第一次访问时创建Servlet对象。默认是-1
2：0或正整数:服务器启动时创建Servlet对象，数字越小优先级越高
```



2：初始化【在Servlet实例化之后，容器将调用Servlet的 init() 方法初始化这个对象，完成一些如加载配置文件、创建连接等初始化的工作。该方法只调用一次】



3：请求处理【每次请求Servlet时，Servlet容器都会调用Servlet的service()方法对请求进行处理】



4：服务终止【当需要释放内存或者容器关闭时，容器就会调用Servlet实例的destroy()方法完成资源的释放。在destroy()方法调用之后，容器会释放这个Servlet实例，该实例随后会被Java的垃圾收集器所回收】



```java
/**
* Servlet生命周期方法
*/
@WebServlet(urlPatterns = "/demo2",loadOnStartup = 1)
public class ServletDemo2 implements Servlet {

    /**
     *  初始化方法
     *  1.调用时机：默认情况下，Servlet被第一次访问时，调用
     *  loadOnStartup: 默认为-1，修改为0或者正整数，则会在服务器启动的时候，调用
     *  2.调用次数: 1次
     *  @param config
     *  @throws ServletException
     */
    public void init(ServletConfig config) throws ServletException {
        System.out.println("init...");
    }

    /**
     * 提供服务
     * 1.调用时机:每一次Servlet被访问时，调用
     * 2.调用次数: 多次
     * @param req
     * @param res
     * @throws ServletException
     * @throws IOException
     */
    public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException {
        System.out.println("servlet hello world~");
    }

    /**
     * 销毁方法
     * 1.调用时机：内存释放或者服务器关闭的时候，Servlet对象会被销毁，调用
     * 2.调用次数: 1次
     */
    public void destroy() {
        System.out.println("destroy...");
    }
    public ServletConfig getServletConfig() {
        return null;
    }

    public String getServletInfo() {
        return null;
    }
}
```

1：Servlet对象在什么时候被创建的?

> 默认是第一次访问的时候被创建，可以使用
>
> @WebServlet(urlPatterns = "/demo2",loadOnStartup = 1)的loadOnStartup 
>
> 修改成在服务器启动的时候创建

2：Servlet生命周期中涉及到三个方法，这三个方法是什么？什么时候被调用？调用几次？

>涉及到三个方法，分别是 init()、service()、destroy()
>
>init方法在Servlet对象被创建的时候执行，只执行1次
>
>service方法在Servlet被访问的时候调用，每访问1次就调用1次
>
>destroy方法在Servlet对象被销毁的时候调用，只执行1次

## 4.6:方法介绍

Servlet中总共有5个方法，我们已经介绍过其中的三个，剩下的两个方法作用分别是什么？

我们先来回顾下前面讲的三个方法，分别是:

1：初始化方法，在Servlet被创建时执行，只执行一次

```java
void init(ServletConfig config) 
```

2：提供服务方法， 每次Servlet被访问，都会调用该方法

```java
void service(ServletRequest req, ServletResponse res)
```

3：销毁方法，当Servlet被销毁时，调用该方法。在内存释放或服务器关闭时销毁Servlet

```java
void destroy() 
```



剩下的两个方法是：



4：获取Servlet信息

```java
String getServletInfo() 
// 该方法用来返回Servlet的相关信息，没有什么太大的用处，一般我们返回一个空字符串即可
public String getServletInfo() {
    return "";
}
```

5：获取ServletConfig对象

```java
ServletConfig getServletConfig()
```

ServletConfig对象，在init方法的参数中有，而Tomcat Web服务器在创建Servlet对象的时候会调用init方法，必定会传入一个ServletConfig对象，我们只需要将服务器传过来的ServletConfig进行返回即可。只需要定义一个变量接收init里面的参数即可

```java
import javax.servlet.*;
import javax.servlet.annotation.WebServlet;
import java.io.IOException;

/**
 * Servlet方法介绍
 */
@WebServlet(urlPatterns = "/demo3",loadOnStartup = 1)
public class ServletDemo3 implements Servlet {

    private ServletConfig servletConfig;
    /**
     *  初始化方法
     *  1.调用时机：默认情况下，Servlet被第一次访问时，调用
     *  loadOnStartup: 默认为-1，修改为0或者正整数，则会在服务器启动的时候，调用
     *  2.调用次数: 1次
     *  @param config
     *  @throws ServletException
     */
    public void init(ServletConfig config) throws ServletException {
        this.servletConfig = config;
        System.out.println("init...");
    }
    
    public ServletConfig getServletConfig() {
        return servletConfig;
    }
    
    /**
     * 提供服务
     * 1.调用时机:每一次Servlet被访问时，调用
     * 2.调用次数: 多次
     * @param req
     * @param res
     * @throws ServletException
     * @throws IOException
     */
    public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException {
        System.out.println("servlet hello world~");
    }

    /**
     * 销毁方法
     * 1.调用时机：内存释放或者服务器关闭的时候，Servlet对象会被销毁，调用
     * 2.调用次数: 1次
     */
    public void destroy() {
        System.out.println("destroy...");
    }
    
    public String getServletInfo() {
        return "";
    }
}
```

getServletInfo()和getServletConfig()这两个方法使用的不是很多

## 4.7:HttpServlet

通过上面的学习，我们知道要想编写一个Servlet就必须要实现Servlet接口，重写接口中的5个方法，虽然已经能完成要求，但是编写起来还是比较麻烦的，因为我们更关注的其实只有service方法，那有没有更简单方式来创建Servlet呢?

要想解决上面的问题，我们需要先对Servlet的体系结构进行下了解:

![1627240593506]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/1627240593506.png)

因为我们将来开发B/S架构的web项目，都是针对HTTP协议

所以我们自定义Servlet会通过继承HttpServlet来实现

```java
@WebServlet("/user")
public class MyServlet extends HttpServlet {
    
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.getWriter().println("Hello World");
    }
    
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("post...");
    }
}
```

1：要想发送一个GET请求，请求该Servlet

只需要通过浏览器发送`http://localhost:8080/web-demo/user`

就能看到doGet方法被执行了



2：要想发送一个POST请求，请求该Servlet，单单通过浏览器是无法实现的

这个时候就需要编写一个form表单来发送请求



Servlet的简化编写就介绍完了，接着需要思考两个问题:

1. HttpServlet中为什么要根据请求方式的不同，调用不同的方法?
2. 如何调用?



针对问题一：我们需要回顾之前的知识点

前端发送GET和POST请求的时候，参数的位置不一致，GET请求参数在请求行中，POST请求

参数在请求体中，为了能处理不同的请求方式，我们得在service方法中进行判断，然后写不

同的业务处理，这样能实现，但是每个Servlet类中都将有相似的代码，类似这样

```java
public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException {
    //获取请求方式，根据不同的请求方式进行不同的业务处理
    HttpServletRequest request = (HttpServletRequest)req;
    //1. 获取请求方式
    String method = request.getMethod();
    //2. 判断
    if("GET".equals(method)){
        // get方式的处理逻辑
    }else if("POST".equals(method)){
        // post方式的处理逻辑
    }
}
```

针对这个问题，有什么可以优化的策略么?

要解决上述问题，我们可以对Servlet接口进行继承封装

动态添加一个doGet方法和doPost方法，用来简化代码开发

```java
public class MyHttpServlet implements Servlet {
    public void init(ServletConfig config) throws ServletException {

    }

    public ServletConfig getServletConfig() {
        return null;
    }

    public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException {
        HttpServletRequest request = (HttpServletRequest)req;
        //1. 获取请求方式
        String method = request.getMethod();
        //2. 判断
        if("GET".equals(method)){
            // get方式的处理逻辑
            doGet(req,res);
        }else if("POST".equals(method)){
            // post方式的处理逻辑
            doPost(req,res);
        }
    }

    protected void doPost(ServletRequest req, ServletResponse res) {
    }

    protected void doGet(ServletRequest req, ServletResponse res) {
    }

    public String getServletInfo() {
        return null;
    }

    public void destroy() {

    }
}
```

有了MyHttpServlet这个类，以后我们再编写Servlet类的时候，只需要继承MyHttpServlet，重写父类中的doGet和doPost方法，就可以用来处理GET和POST请求的业务逻辑



将来页面发送的是GET请求，则会进入到doGet方法中进行执行，如果是POST请求，则进入到doPost方法。这样代码在编写的时候就相对来说更加简单快捷



类似MyHttpServlet这样的类Servlet中已经为我们提供好了，就是HttpServlet，HttpServlet做的事更多，不仅可以处理GET和POST，还可以处理其他五种请求方式



1：HttpServlet的使用步骤

> 继承HttpServlet
>
> 重写doGet和doPost方法

2：HttpServlet原理

> 获取请求方式，并根据不同的请求方式，调用不同的doXxx方法



## 4.8:urlPattern配置

Servlet类编写好后，要想被访问到，就需要配置其访问路径（urlPattern）

一个Servlet,可以配置多个urlPattern

```java
@WebServlet(urlPatterns = {"/users","/goods"})
```

urlPattern配置规则

### 1：精确匹配

```java
@WebServlet(urlPatterns = "/user/select")
```

访问路径`http://localhost:8080/web-demo/user/select`



### 2：目录匹配

```java
@WebServlet(urlPatterns = "/user/*")
```

访问路径`http://localhost:8080/web-demo/user/任意`

思考:

1. 访问路径`http://localhost:8080/web-demo/user`是否能访问到这个的doGet方法?
2. 访问路径`http://localhost:8080/web-demo/user/a/b`是否能访问到这个的doGet方法?
3. 访问路径`http://localhost:8080/web-demo/user/select`是否能访问到这个

答案是：能、能、不能

进而我们可以得到的结论是`/user/*`中的`/*`代表的是零或多个层级访问目录

精确匹配优先级要高于目录匹配



### 3：扩展名匹配

```java
@WebServlet(urlPatterns = "*.do")
```

访问路径`http://localhost:8080/web-demo/任意.do`



注意：

1：如果路径配置的不是扩展名，那么在路径的前面就必须要加`/`，否则会报错

2：如果路径配置的是`*.do`这种类型，那么在*.do的前面不能加`/`，否则会报错



### 4：任意匹配

```java
@WebServlet(urlPatterns = "/")
```

访问路径`http://localhost:8080/demo-web/任意`



注意：/   `和`  /*  的区别

配置了 /* 之后的路径：http://localhost:8080/demo-web/任意



1. 当我们的项目中的Servlet配置了 "/"，会覆盖掉tomcat中的DefaultServlet

   当其他的url-pattern都匹配不上时都会走这个Servlet

   

2. 当我们的项目中配置了"/*"，意味着匹配任意访问路径，但是不会覆盖默认的

   

3. DefaultServlet是用来处理静态资源，如果配置了"/"会把默认的覆盖掉，就会引发请求静

   态资源的时候没有走默认的而是走了自定义的Servlet类，最终导致静态资源不能被访问

## 4.9: XML配置

前面对应Servlet的配置，我们都使用的是@WebServlet，这个是Servlet从3.0版本后开始支持注解配置，3.0版本前只支持XML配置文件的配置方法

对于XML的配置步骤有两步:

* 编写Servlet类

```java
public class MyServlet extends MyHttpServlet {

    @Override
    protected void doGet(ServletRequest req, ServletResponse res) {

        System.out.println("get");
    }
    
    @Override
    protected void doPost(ServletRequest req, ServletResponse res) {
    }
}
```

* 在web.xml中配置该Servlet

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <!-- 
        Servlet 全类名
    -->
    <servlet>
        <!-- servlet的名称，名字任意-->
        <servlet-name>myServlet</servlet-name>
        <!--servlet的类全名-->
        <servlet-class>com.zzx.web.MyServlet</servlet-class>
    </servlet>

    <!-- 
        Servlet 访问路径
    -->
    <servlet-mapping>
        <!-- servlet的名称，要和上面的名称一致-->
        <servlet-name>myServlet</servlet-name>
        <!-- servlet的访问路径-->
        <url-pattern>/myServlet3</url-pattern>
    </servlet-mapping>
</web-app>
```

这种配置方式和注解比起来，确认麻烦很多，所以建议大家使用注解来开发

但是大家要认识上面这种配置方式，因为并不是所有的项目都是基于注解开发的



# 5:Request

## 5.1:简介

Request对象是被Web服务器封装的HTTP请求中有关请求的信息的对象

Request：主要用于获取请求数据

我们一般使用的是 HttpServletRequest 

注意：这是一个接口

* 浏览器会发送HTTP请求到后台服务器[Tomcat]
* HTTP的请求中会包含很多请求数据【请求行+请求头+请求体】
* 后台服务器[Tomcat]会对HTTP请求中的数据进行解析并把解析结果存入到一个对象中
* 所存入的对象即为request对象，所以我们可以从request对象中获取请求的相关参数
* 获取到数据后就可以继续后续的业务



```java
@WebServlet("/request")
public class RequestServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String name = req.getParameter("name");
        System.out.println(name);
    }
}
```



## 5.2:继承体系

在学习这节内容之前，我们先思考一个问题，比较细心的同学可能已经发现：

1：当我们的Servlet类实现的是Servlet接口的时候，service方法中的参数是ServletRequest和

ServletResponse

2：当我们的Servlet类继承的是HttpServlet类的时候，doGet和doPost方法中的参数就变成

HttpServletRequest和HttpServletReponse



ServletRequest和HttpServletRequest的关系是什么?

request对象是由谁来创建的?

request提供了哪些API，这些API从哪里查?

首先，我们先来看下Request的继承体系:

![1628740441008]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/1628740441008.png)

从上图中可以看出，ServletRequest和HttpServletRequest都是Java提供的

所以我们可以打开JavaEE提供的API文档

打开后可以看到：

![1628741839475]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/1628741839475.png)

所以ServletRequest和HttpServletRequest是继承关系，并且两个都是接口，接口是无法创建

对象，这个时候就引发了下面这个问题:

![1628742224589]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/1628742224589.png)

这个时候，我们就需要用到Request继承体系中的`RequestFacade`:

* 该类实现了HttpServletRequest接口，也就间接实现了ServletRequest接口
* Servlet类中的service方法、doGet方法或者是doPost方法最终都是由Web服务器[Tomcat]来调用的，所以Tomcat提供了方法参数接口的具体实现类，并完成了对象的创建
* 要想了解RequestFacade中都提供了哪些方法，我们可以直接查看JavaEE的API文档中关于ServletRequest和HttpServletRequest的接口文档，因为RequestFacade 实现了其接口就需要重写接口中的方法



对于上述结论，要想验证，可以编写一个Servlet，在方法中把request对象打印下

就能看到最终的对象是不是RequestFacade

我们输出request对象：org.apache.catalina.connector.RequestFacade@5cc235ef

发现实际上就是Tomcat的这个实现类



**小结**

* Request的继承体系为
  - ServletRequest【接口】-->HttpServletRequest【接口】-->RequestFacade【实现类】
* ServletRequest和HttpServletRequest的关系为继承关系
* Tomcat需要解析请求数据，封装为request对象，并且创建request对象传递到service方法
* 使用request对象，可以查阅JavaEE API文档的HttpServletRequest接口中方法说明

## 5.3:获取请求数据

HTTP请求数据总共分为三部分内容，分别是

请求行

请求头

请求体

对于这三部分内容的数据，分别该如何获取，首先我们先来学习请求行数据如何获取?

### 1:获取请求行数据

请求行包含三块内容，分别是`请求方式`、`请求资源路径`、`HTTP协议及版本`

![1628748240075]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/1628748240075.png)

对于这三部分内容，request对象都提供了对应的API方法来获取，具体如下:

* 获取请求方式: `GET`

```
String getMethod()
```

* 获取虚拟目录(项目访问路径): `/request-demo`

```
String getContextPath()
```

- 获取URI(统一资源标识符):`/request-demo/req1`

```
String getRequestURI()
```

* 获取URL(统一资源定位符): `http://localhost:8080/request-demo/req1`

```
StringBuffer getRequestURL()
```

* 获取请求参数(GET方式): `username=zhangsan&password=123`

```
String getQueryString()
```

### 2:获取请求头数据

对于请求头的数据，格式为`key: value`如下:

![1628768652535]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/1628768652535.png)

所以根据请求头名称获取对应值的方法为:

```
String getHeader(String name)
```

接下来，在代码中如果想要获取客户端浏览器的版本信息，则可以使用

### 3:获取请求体数据

浏览器在发送GET请求的时候是没有请求体的，所以需要把请求方式变更为POST

请求体中的数据格式如下:

![1628768665185]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/1628768665185.png)

对于请求体中的数据，Request对象提供了如下两种方式来获取其中的数据，分别是:



1：获取字节输入流，如果前端发送的是字节数据，比如传递的是文件数据，则使用该方法

```
ServletInputStream getInputStream()
该方法可以获取字节
```



2：获取字符输入流，如果前端发送的是纯文本数据，则使用该方法

```
BufferedReader getReader()
```

BufferedReader流是通过request对象来获取的，当请求完成后request对象就会被销毁

request对象被销毁后，BufferedReader流就会自动关闭，所以此处就不需要手动关闭流了

一般情况下，什么东西是我们创建的，什么东西就由我们自己销毁



接下来，大家需要思考，要想获取到请求体的内容该如何实现？

### 4:小结

HTTP请求数据中包含了`请求行`、`请求头`和`请求体`，针对这三部分内容，Request对象都提供了对应的API方法来获取对应的值:

* 请求行
  * getMethod()获取请求方式
  * getContextPath()获取项目访问路径
  * getRequestURL()获取请求URL
  * getRequestURI()获取请求URI
  * getQueryString()获取`GET`请求方式的请求参数
* 请求头
  * getHeader(String name)根据请求头名称获取其对应的值
* 请求体
  * 注意：浏览器发送的POST请求才有请求体
  * 如果是纯文本字符数据：getReader()
  * 如果是字节数据如文件数据：getInputStream()

## 5.4:获取参数通用方式

在学习下面内容之前，我们先提出两个问题：

* 什么是请求参数?
* 请求参数和请求数据的关系是什么?



1.什么是请求参数?

为了能更好的回答上述两个问题，我们拿用户登录的例子来说明

1.1 想要登录网址，需要进入登录页面

1.2 在登录页面输入用户名和密码

1.3 将用户名和密码提交到后台

1.4 后台校验用户名和密码是否正确

1.5 如果正确，则正常登录，如果不正确，则提示用户名或密码错误

上述例子中，用户名和密码其实就是我们所说的请求参数



2.什么是请求数据?

请求数据则是包含请求行、请求头和请求体的所有数据



3.请求参数和请求数据的关系是什么?

3.1 请求参数是请求数据中的部分内容

3.2 如果是GET请求，请求参数在请求行中

3.3 如果是POST请求，请求参数一般在请求体中



对于请求参数的获取,常用的有以下两种:

1：GET方式:

```
String getQueryString()
```

2：POST方式:

- 如果请求体全是字符类型的数据，就用这个

```
BufferedReader getReader();
```

- 如果请求体是字节数据，例如文件类型，就用这个

```
ServletInputStream getInputStream()
```

有了上述的知识储备，我们来实现一个案例需求:

（1）发送一个GET请求并携带用户名，后台接收后打印到控制台

（2）发送一个POST请求并携带用户名，后台接收后打印到控制台

此处大家需要注意的是GET请求和POST请求接收参数的方式不一样，具体实现的代码如下:

```java
@WebServlet("/req")
public class RequestDemo1 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String result = req.getQueryString();
        System.out.println(result);
    }
    
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        BufferedReader br = req.getReader();
        String result = br.readLine();
        System.out.println(result);
    }
}
```

相同的业务，代码冗余了，怎么改进？



request对象已经将上述获取请求参数的方法进行了封装，并且request提供的方法实现的功能更强大，以后只需要调用request提供的方法即可，在request的方法中都实现了哪些操作?



(1)根据不同的请求方式获取请求参数，获取的内容如下:

![1628778931277]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/1628778931277.png)

(2)把获取到的内容进行分割，内容如下:

![1628779067793]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/1628779067793.png)

(3)把分割后端数据，存入到一个Map集合中:

![1628779368501]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/1628779368501.png)

**注意**:因为参数的值可能是一个，也可能有多个，所以Map的值的类型为String数组

基于上述理论，request对象为我们提供了如下方法:

* 获取所有参数Map集合

```
Map<String,String[]> getParameterMap()
```

* 根据名称获取参数值（数组）

```
String[] getParameterValues(String name)
```

* 根据名称获取参数值(单个值)

```
String getParameter(String name)
```



**小结**

也就是request不管你是什么类型的请求方式，他把参数全部解析封装到了一个map里面

以后我们再写代码的时候，就只需要按照如下格式来编写:

```java
public class RequestDemo extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
       // 采用request提供的获取请求参数的通用方式来获取请求参数
       // 编写其他的业务代码...
    }
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        this.doGet(req,resp);
    }
}
```

## 5.5:请求参数中文乱码问题

问题展示：

(1)将req.html页面的请求方式修改为get

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<form action="/request-demo/req4" method="get">
    <input type="text" name="username"><br>
    <input type="password" name="password"><br>
    <input type="checkbox" name="hobby" value="1"> 游泳
    <input type="checkbox" name="hobby" value="2"> 爬山 <br>
    <input type="submit">

</form>
</body>
</html>
```

(2)在Servlet方法中获取参数，并打印

```java
@WebServlet("/req4")
public class RequestDemo4 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
       String username = request.getParameter("username");
       System.out.println(username);
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```

（3）启动服务器，页面上输入中文参数

![1628784323297]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/1628784323297.png)

（4）查看控制台打印内容

![1628784356157]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/1628784356157.png)

（5）把req.html页面的请求方式改成post,再次发送请求和中文参数

![1628784425182]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/1628784425182.png)

（6）查看控制台打印内容，依然为乱码

通过上面的案例，会发现，不管是GET还是POST请求，在发送的请求参数中如果有中文，在后台接收的时候，都会出现中文乱码的问题。具体该如何解决呢？



### 1:POST请求解决方案

* 分析出现中文乱码的原因：

  - 我们使用的request.getParameter()，内部是用的request的getReader()

  * POST的请求参数是通过request的getReader()来获取流中的数据
  * TOMCAT在获取流的时候采用的编码是ISO-8859-1
  * ISO-8859-1编码是不支持中文的，所以会出现乱码

* 解决方案：

  * 页面设置的编码格式为UTF-8
  * 把TOMCAT在获取流数据之前的编码设置为UTF-8
  * 通过request.setCharacterEncoding("UTF-8")设置编码，UTF-8也可以写成小写

修改后的代码为:

```java
@WebServlet("/req4")
public class RequestDemo4 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //1. 解决乱码: POST
        //设置字符输入流的编码，设置的字符集要和HTML页面保持一致
        request.setCharacterEncoding("UTF-8");
        //2. 获取username
        String username = request.getParameter("username");
        System.out.println(username);
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```

重新发送POST请求，就会在控制台看到正常展示的中文结果。



至此POST请求中文乱码的问题就已经解决，但是这种方案不适用于GET请求，这个原因是什么呢，咱们下面再分析

### 2:GET请求解决方案

刚才提到一个问题是`POST请求的中文乱码解决方案为什么不适用GET请求？`

* GET请求获取请求参数的方式是`request.getQueryString()`
* POST请求获取请求参数的方式是`request.getReader()`
* request.setCharacterEncoding("utf-8")是设置request处理流的编码
* getQueryString方法并没有通过流的方式获取数据

所以GET请求不能用设置编码的方式来解决中文乱码问题，那问题又来了，如何解决GET请

求的中文乱码呢? 





1. 首先我们需要先分析下GET请求出现乱码的原因:

 ![1628829610823]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/1628829610823.png)

(1)浏览器通过HTTP协议发送请求和数据给后台服务器（Tomcat）

(2)浏览器在发送HTTP的过程中会对`中文数据进行URL编码`

(3)在进行URL编码的时候会采用页面`<meta>`标签指定的UTF-8的方式进行编码，`张三`编码后的结果为`%E5%BC%A0%E4%B8%89`

(4)后台服务器(Tomcat)接收到`%E5%BC%A0%E4%B8%89`后会默认按照`ISO-8859-1`进行URL解码

(5)由于前后编码与解码采用的格式不一样，就会导致后台获取到的数据为乱码



思考: 如果把`req.html`页面的`<meta>`标签的charset属性改成`ISO-8859-1`，后台不做操

作，能解决中文乱码问题么?

答案是否定的，因为`ISO-8859-1`本身是不支持中文展示的，所以改了<meta>标签的

charset属性后，会导致页面上的中文内容都无法正常展示



到这，我们就可以分析出GET请求中文参数出现乱码的原因了

* 浏览器把中文参数按照HTML页面的<meta>写的`UTF-8`进行URL编码
* Tomcat对获取到的内容进行了`ISO-8859-1`的URL解码
* 在控制台就会出现乱码



解决方案：比较繁琐，而且老版本Tomcat读取get请求参数才是使用ISO-8859-1

Tomcat8.0之后，已将GET请求乱码问题解决，设置默认的解码方式为UTF-8



### 3:URL编码

Java中已经为我们提供了编码和解码的API工具类可以让我们更快速的进行编码和解码:

编码:

```java
java.net.URLEncoder.encode("需要被编码的内容","字符集(UTF-8)")
```

解码:

```java
java.net.URLDecoder.decode("需要被解码的内容","字符集(UTF-8)")
```

接下来咱们对`张三`来进行编码和解码

```java
public class URLDemo {

  public static void main(String[] args) throws UnsupportedEncodingException {
        String username = "张三";
        //1. URL编码
        String encode = URLEncoder.encode(username, "utf-8");
        System.out.println(encode); //打印:%E5%BC%A0%E4%B8%89

       //2. URL解码
       //String decode = URLDecoder.decode(encode, "utf-8");//打印:张三
       String decode = URLDecoder.decode(encode, "ISO-8859-1");//打印:`å¼ ä¸ `
       System.out.println(decode);
    }
}

```

到这，我们就可以分析出GET请求中文参数出现乱码的原因了，

* 浏览器把中文参数按照`UTF-8`进行了URL编码
* Tomcat对获取到的内容进行了`ISO-8859-1`的URL解码，格式不同则乱码了
* 在控制台就会出现类上`å¼ ä¸`的乱码，最后一位是个空格

![1628846824194]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/1628846824194.png)

怎么解决呢？

从上图可以看住，

在进行编码和解码的时候，不管使用的是哪个字符集，他们对应的`%E5%BC%A0%E4%B8%89`是一致的

那他们对应的二进制值也是一样的，为:

```
1110 0101 1011 1100 1010 0000 1110 0100 1011 1000 1000 1001
```

所以我们可以考虑把`å¼ ä¸`转换成字节，在把字节转换成`张三`，在转换的过程中是它们的编码一致，就可以解决中文乱码问题

```java
public class URLDemo {

  public static void main(String[] args) throws UnsupportedEncodingException {
        String username = "张三";
        //1. URL编码
        String encode = URLEncoder.encode(username, "utf-8");
        System.out.println(encode);
      
        //2. URL解码
        String decode = URLDecoder.decode(encode, "ISO-8859-1");

        System.out.println(decode); //此处打印的是对应的乱码数据

        //3. 转换为字节数据,编码
        byte[] bytes = decode.getBytes("ISO-8859-1");
        for (byte b : bytes) {
            System.out.print(b + " ");
        }
		//此处打印的是:-27 -68 -96 -28 -72 -119
        //4. 将字节数组转为字符串，解码
        String s = new String(bytes, "utf-8");
        System.out.println(s); //此处打印的是张三
    }
}
```



### 4:小结

中文乱码解决方案

* POST请求和GET请求的参数中如果有中文，后台接收数据就会出现中文乱码问题

  GET请求在Tomcat8.0以后的版本就不会出现了

* POST请求解决方案是：设置输入流的编码



## 5.6:Request请求转发

### 1:介绍

请求转发(forward)：一种在`服务器内部`的资源跳转方式

![1628851404283]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/1628851404283.png)

(1)浏览器发送请求给服务器，服务器中对应的资源A接收到请求

(2)资源A处理完请求发现还需要资源B处理才能完全处理好这个请求，将请求发给资源B

(3)资源B处理完后将结果响应给浏览器

(4)请求从资源A到资源B的过程就叫请求转发



### 2:基本使用

请求转发的实现方式

```java
req.getRequestDispatcher("资源B路径").forward(req,resp);
```

具体如何来使用，我们先来看下需求:

![1628854783523]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/1628854783523.png)

针对上述需求，具体的实现步骤为:

>1.创建一个RequestDemo5类，接收/req5的请求，在doGet方法中打印`demo5`
>
>2.创建一个RequestDemo6类，接收/req6的请求，在doGet方法中打印`demo6`
>
>3.在RequestDemo5的方法中使用
>
>req.getRequestDispatcher("/req6").forward(req,resp)进行请求转发
>
>4.启动测试

(1)创建RequestDemo5类

```java
@WebServlet("/req5")
public class RequestDemo5 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("demo5...");
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```

(2)创建RequestDemo6类

```java
/**
 * 请求转发
 */
@WebServlet("/req6")
public class RequestDemo6 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("demo6...");
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```

(3)在RequestDemo5的doGet方法中进行请求转发

```java
/**
 * 请求转发
 */
@WebServlet("/req5")
public class RequestDemo5 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("demo5...");
        //请求转发
        request.getRequestDispatcher("/req6").forward(request,response);
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```

(4)启动测试

访问`http://localhost:8080/request-demo/req5`



### 3:数据共享

请求转发资源间共享数据：使用Request对象，因为是同一个Request对象

此处主要解决的问题是把请求从`/req5`转发到`/req6`的时候，如何传递数据给`/req6`

需要使用request对象提供的三个方法:

* 存储数据到request域[范围,数据是存储在request对象]中

```
void setAttribute(String name,Object o);
```

* 根据key获取值

```
Object getAttribute(String name);
```

* 根据key删除该键值对

```
void removeAttribute(String name);
```

接着上个需求来:

![1628856995417]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/1628856995417.png)

> 1.在RequestDemo5的doGet方法中转发请求之前，将数据存入request域对象中
>
> 2.在RequestDemo6的doGet方法从request域对象中获取数据，并将数据打印到控制台
>
> 3.启动访问测试

(1)修改RequestDemo5中的方法

```java
@WebServlet("/req5")
public class RequestDemo5 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("demo5...");
        //存储数据
        request.setAttribute("msg","hello");
        //请求转发
        request.getRequestDispatcher("/req6").forward(request,response);

    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```

(2)修改RequestDemo6中的方法

```java
/**
 * 请求转发
 */
@WebServlet("/req6")
public class RequestDemo6 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("demo6...");
        //获取数据
        Object msg = request.getAttribute("msg");
        System.out.println(msg);

    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```

(3)启动测试

访问`http://localhost:8080/request-demo/req5`

此时就可以实现在转发多个资源之间共享数据



### 4:请求转发的特点

1：浏览器地址栏路径不发生变化

虽然后台从`/req5`转发到`/req6`，但是浏览器的地址一直是`/req5`，未发生变化



2：只能转发到当前服务器的内部资源，路径就跟我们的@WebServlet的定义一模一样



3：不能从一个服务器通过转发访问另一台服务器



4：一次请求，可以在转发资源间使用request共享数据



5：虽然后台从`/req5`转发到`/req6`，但是这个只有一次请求



# 6:Response

## 6.1:共性

前面讲解完Request对象，接下来我们回到刚开始的那张图:

![1628857632899]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/1628857632899.png)

* Request:使用request对象来获取请求数据
* Response:使用response对象来设置响应数据

Reponse的继承体系和Request的继承体系也非常相似:

![1628857761317]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/1628857761317.png)

 介绍完Response的相关体系结构后，接下来对于Response我们需要学习如下内容:

* Response设置响应数据的功能介绍
* Response完成重定向
* Response响应字符数据
* Response响应字节数据

## 6.2:设置响应数据

HTTP响应数据总共分为三部分内容，分别是响应行、响应头、响应体，对于这三部分内容

的数据，respone对象都提供了哪些方法来进行设置？

1：响应行

![1628858926498]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/1628858926498.png)

对于响应行，比较常用的就是设置响应状态码:

```
void setStatus(int code);
```



2：响应头

![1628859051368]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/1628859051368.png)

设置响应头键值对：

```
void setHeader(String name,String value);
```



3：响应体

![1628859268095]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/1628859268095.png)

对于响应体，是通过字符、字节输出流的方式往浏览器写

获取字符输出流:

```
PrintWriter getWriter();
```

获取字节输出流

```
ServletOutputStream getOutputStream();
```

这些都差不多，不再赘述了

## 6.3:Respones请求重定向

### 1:介绍

Response重定向(redirect):一种资源跳转方式

![1628859860279]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/1628859860279.png)

(1)浏览器发送请求给服务器，服务器中对应的资源A接收到请求

(2)资源A现在无法处理该请求，就会给浏览器响应一个302的状态码，并且会返回一个新的

地址location给浏览器

(3)浏览器接收到响应行的状态码为302，就会去找响应头为location的地址，然后重新发送请

求到location对应的访问地址去访问资源B

(4)资源B接收到请求后进行处理并最终给浏览器响应结果，这整个过程就叫重定向



### 2:使用

重定向的实现方式:

```
resp.setStatus(302);
resp.setHeader("location","资源B的访问路径");
```

具体如何来使用，我们先来看下需求:

![1628861030429]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/1628861030429.png)

针对上述需求，具体的实现步骤为:

> 1.创建一个ResponseDemo1类，接收/resp1的请求，在doGet方法中打印`resp1....`
>
> 2.创建一个ResponseDemo2类，接收/resp2的请求，在doGet方法中打印`resp2....`
>
> 3.在ResponseDemo1的方法中使用
>
> ​	response.setStatus(302);
>
> ​	response.setHeader("Location","/request-demo/resp2") 来给前端响应结果数据
>
> 4.启动测试

(1)创建ResponseDemo1类

```java
@WebServlet("/resp1")
public class ResponseDemo1 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("resp1....");
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```

(2)创建ResponseDemo2类

```java
@WebServlet("/resp2")
public class ResponseDemo2 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("resp2....");
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```

(3)在ResponseDemo1的doGet方法中给前端响应数据

```java
@WebServlet("/resp1")
public class ResponseDemo1 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("resp1....");
        //重定向
        //1.设置响应状态码 302
        response.setStatus(302);
        //2. 设置响应头 Location
        response.setHeader("Location","/request-demo/resp2");
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```

(4)启动测试

访问`http://localhost:8080/request-demo/resp1`



虽然功能已经实现，但是从设置重定向的两行代码来看，会发现除了重定向的地址不一样

其他的内容都是一模一样，所以request对象给我们提供了简化的编写方式，会自动添加302

状态码到响应行，Location到响应头里面，方式如下：

```
resposne.sendRedirect("/request-demo/resp2")
```

所以第3步中的代码就可以简化为：

```java
@WebServlet("/resp1")
public class ResponseDemo1 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("resp1....");
        //重定向
        resposne.sendRedirect("/request-demo/resp2")；
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```



### 3:重定向的特点

* 浏览器地址栏路径发送变化

  当进行重定向访问的时候，由于是由浏览器发送的两次请求，所以地址会发生变化

* 可以重定向到任何位置的资源(服务内容、外部均可)

  因为第一次响应结果中包含了浏览器下次要跳转的路径，所以这个路径是可以任意位置资源

* 两次请求，不能在多个资源使用request共享数据

  因为浏览器发送了两次请求，是两个不同的request对象，就无法通过request对象进行共享数据



### 4:对比

介绍完请求重定向和请求转发以后，接下来需要把这两个放在一块对比下：

![1628862170296]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/1628862170296.png)

## 6.4:路径问题

### 1:问题一

转发的时候路径上没有加`项目名`，而重定向加了

那么到底什么时候需要加，什么时候不需要加呢?

![1628862652700]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/1628862652700.png)

其实判断的依据很简单，只需要记住下面的规则即可:

* 浏览器使用：需要加虚拟目录(项目访问路径)
* 服务端使用：不需要加虚拟目录



对于转发来说，因为是在服务端进行的，所以不需要加虚拟目录

对于重定向来说，路径最终是由浏览器来发送请求，就需要添加虚拟目录

掌握了这个规则，接下来就通过一些练习来强化下知识的学习:

* `<a href='路径'>`
* `<form action='路径'>`
* req.getRequestDispatcher("路径")
* resp.sendRedirect("路径")



```
1.超链接，从浏览器发送，需要加
2.表单，从浏览器发送，需要加
3.转发，是从服务器内部跳转，不需要加
4.重定向，是由浏览器进行跳转，需要加
```

### 2:问题二

在重定向的代码中，`/request-demo`是固定编码的，如果后期放进Tomcat打包了，换了项目名，或者通过Tomcat插件配置了项目的访问路径，那么所有需要重定向的地方都需要重新修改，该如何优化?

![1628863270545]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/1628863270545.png)

我们可以在代码中动态去获取项目访问的虚拟目录，具体如何获取，我们可以借助前面咱们所学习的request对象中的getContextPath()方法获取项目名，修改后的代码如下:

```java
@WebServlet("/resp1")
public class ResponseDemo1 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("resp1....");

        //简化方式完成重定向
        //动态获取虚拟目录
        String contextPath = request.getContextPath();
        response.sendRedirect(contextPath+"/resp2");
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```

重新启动访问测试，功能依然能够实现，此时就可以动态获取项目访问的虚拟路径，从而降低代码的耦合度

## 6.5:响应字符数据

要想将字符数据写回到浏览器，我们需要两个步骤:

* 通过Response对象获取字符输出流： PrintWriter writer = resp.getWriter()

* 通过字符输出流写数据: writer.write("aaa")

接下来，我们实现通过些案例把响应字符数据给实际应用下:



1：返回一个简单的字符串`aaa`

```java
/**
 * 响应字符数据：设置字符数据的响应体
 */
@WebServlet("/resp3")
public class ResponseDemo3 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
         //1. 获取字符输出流
         PrintWriter writer = response.getWriter();
	     writer.write("aaa");
    }
    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```



2：返回一串html字符串，并且能被浏览器解析

```
PrintWriter writer = response.getWriter();
//content-type，告诉浏览器返回的数据类型是HTML类型数据，这样浏览器才会解析HTML标签
response.setHeader("content-type","text/html");
writer.write("<h1>aaa</h1>");
```



3：返回一个中文的字符串`你好`，需要注意设置响应数据的编码为`utf-8`

```java
//设置响应的数据格式及数据的编码
response.setContentType("text/html;charset=utf-8");
writer.write("你好");
```



## 6.6:响应字节数据

要想将字节数据写回到浏览器，我们需要两个步骤:

- 通过Response对象获取字节输出流
  - ServletOutputStream outputStream = resp.getOutputStream()
- 通过字节输出流写数据: outputStream.write(字节数据)



接下来，我们实现通过些案例把响应字符数据给实际应用下

1：返回一个图片文件到浏览器

```java
/**
 * 响应字节数据：设置字节数据的响应体
 */
@WebServlet("/resp4")
public class ResponseDemo4 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //1. 读取文件
        FileInputStream fis = new FileInputStream("d://a.jpg");
        //2. 获取response字节输出流
        ServletOutputStream os = response.getOutputStream();
        //3. 完成流的copy
        byte[] buff = new byte[1024];
        int len = 0;
        while ((len = fis.read(buff))!= -1){
            os.write(buff,0,len);
        }
        fis.close();
    }
    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```

上述代码中，对于流的copy的代码还是比较复杂的，所以我们可以使用别人提供好的方法来简化代码的开发，例如commons-io，或者hutool工具包等，具体的步骤是:



(1)pom.xml添加依赖

```xml
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.6</version>
</dependency>
```

(2)调用工具类方法

```
IOUtils.copy(fis,os);
```

优化后的代码：

```java
/**
 * 响应字节数据：设置字节数据的响应体
 */
@WebServlet("/resp4")
public class ResponseDemo4 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //1. 读取文件
        FileInputStream fis = new FileInputStream("d://a.jpg");
        //2. 获取response字节输出流
        ServletOutputStream os = response.getOutputStream();
        //3. 完成流的copy
      	IOUtils.copy(fis,os);
        fis.close();
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```

# 7:JSP

## 7.1:概述

JSP（全称：Java Server Pages）：Java 服务端页面

是一种`动态`的网页技术，其中既可以定义 HTML、JS、CSS等静态内容

还可以定义 Java代码的动态内容，也就是 `JSP = HTML + Java`

正因为是动态的技术，所以可以展现数据库之类的数据，而HTML不借助AJAX就不可以

如下就是 JSP 代码：

```jsp
<html>
    <head>
        <title>Title</title>
    </head>
    <body>
        <h1>JSP，Hello World</h1>
        <%
        	System.out.println("hello,jsp~");
        %>
    </body>
</html>
```

上面代码 `h1` 标签内容是展示在页面上，而 Java 的输出语句是输出在 idea 的控制台

那么，JSP 能做什么呢？

现在我们只用 `servlet` 实现功能，登陆后要显示很多数据库查询的信息，又需要好看的页

面，就需要使用大量的输出流去写html代码。加上输出的是一堆字符串，这个就很难排错

JSP 作用：简化开发，避免了在Servlet中直接输出HTML标签



简单的使用：

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    <h1>hello jsp</h1>

    <%
        System.out.println("hello,jsp~");
    %>
</body>
</html>
```

## 7.2:原理

JSP 本质上就是一个 Servlet，接下来我们来看看访问jsp时的流程

<img src=" https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/image-20210818111039350.png" alt="image-20210818111039350" style="zoom:70%;" />

1. 浏览器第一次访问 `hello.jsp` 页面
2. `tomcat` 会将 `hello.jsp` 转换为名为 `hello_jsp.java` 的一个 `Servlet`
3. `tomcat` 再将转换的 `servlet` 编译成字节码文件 `hello_jsp.class`
4. `tomcat` 会执行该字节码文件，向外提供服务



打开 `hello_jsp.java` 文件，来查看里面的代码

<img src=" https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/image-20210818112724462.png" alt="image-20210818112724462" style="zoom:80%;" />

由上面的类的继承关系可以看到继承了名为 `HttpJspBase` 这个类

![image-20221219160330206]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/image-20221219160330206.png)

<img src=" https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/image-20210818113118802.png" alt="image-20210818113118802" style="zoom:80%;" />

可以看到HttpJspBase类继承了 `HttpServlet` 

那么 `hello_jsp` 这个类就间接的继承了 `HttpServlet` 

也就说明 `hello_jsp` 是一个 `servlet`

继续阅读 `hello_jsp` 类的代码，可以看到有一个名为 `_jspService()` 的方法，该方法就

是每次访问 `jsp` 时自动执行的方法，和 `servlet` 中的 `service` 方法一样 

而在 `_jspService()` 方法中可以看到往浏览器写标签的代码：

<img src=" https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/image-20210818114008998.png" alt="image-20210818114008998" style="zoom:80%;" />

以前我们自己写 `servlet` 时，这部分代码是由我们自己来写，现在有了 `jsp` 后

由tomcat完成这部分功能



原理：

1：JSP页面会被Tomcat转化为Servlet

2：这个Servlet继承了HttpJspBase

3：HttpJspBase继承了HttpServlet

4：JSP页面的Servlet的service方法对应的是 `_jspService()`

## 7.3:脚本

JSP脚本用于在 JSP页面内定义 Java代码

JSP 脚本有如下三个分类：

* <%...%>：内容会直接放到  _jspService()  方法之中
* <%=…%>：内容会放到out.print()中，作为out.print()的参数
* <%!…%>：内容会放到_jspService()方法之外，被类直接包含



**代码演示：**

在 `hello.jsp` 中书写

```jsp
<%
    System.out.println("hello,jsp~");
    int i = 3;
%>
```

通过浏览器访问 `hello.jsp` 后，查看转换的 `hello_jsp.java` 文件

发现 i 变量定义在了 `_jspService()` 方法中

![image-20221219161820694]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/image-20221219161820694.png)

在 `hello.jsp` 中书写

```jsp
<%="hello"%>
<%=i%>
```

通过浏览器访问 `hello.jsp` 后，查看转换的 `hello_jsp.java` 文件，该脚本的内容被放在了 `out.print()` 中，作为参数

<img src=" https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/image-20210818123820571-16714373409652.png" alt="image-20210818123820571" style="zoom:80%;" />

在 `hello.jsp` 中书写

```jsp
<%!
    void show(){}
	String name = "zhangsan";
%>
```

通过浏览器访问 `hello.jsp` 后，查看转换的 `hello_jsp.java` 文件，该脚本的内容被放在了成员位置

<img src=" https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/image-20210818123946272-16714373409641.png" alt="image-20210818123946272" style="zoom:80%;" />

## 7.4:缺点

由于 JSP页面内，既可以定义 HTML 标签，又可以定义 Java代码，造成了以下问题：

* 书写麻烦：特别是复杂的页面

  既要写 HTML 标签，还要写 Java 代码

* 阅读麻烦

  上面案例的代码，相信你后期再看这段代码时还需要花费很长的时间去梳理

* 复杂度高：运行需要依赖于各种环境，JRE，JSP容器，JavaEE…

* 占内存和磁盘：JSP会自动生成.java和.class文件占磁盘，运行的是.class文件占内存

* 调试困难：出错后，需要找到自动生成的.java文件进行调试

* 不利于团队协作：前端人员不会 Java，后端人员不精 HTML

  如果页面布局发生变化，前端工程师对静态页面进行修改，然后再交给后端工程师

  由后端工程师再将该页面改为 JSP 页面

由于上述的问题， JSP 已逐渐退出历史舞台，以后开发更多的是使用 HTML +  Ajax 来替代前

端工程师负责前端页面开发，而后端工程师只负责前端代码开发

下来对技术的发展进行简单的说明

<img src=" https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/image-20210818150346332.png" alt="image-20210818150346332" style="zoom:80%;" />

1. 第一阶段：使用 `servlet` 即实现逻辑代码编写，也对页面进行拼接。这种模式我们之前也接触过

2. 第二阶段：随着技术的发展，出现了 `JSP` ，人们发现 `JSP` 使用起来比 `Servlet` 方便很多，但是还是要在 `JSP` 中嵌套 `Java` 代码，也不利于后期的维护

3. 第三阶段：使用 `Servlet` 进行逻辑代码开发，而使用 `JSP` 进行数据展示

   <img src=" https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/image-20210818151232955.png" alt="image-20210818151232955" style="zoom:67%;" />

4. 第四阶段：使用 `servlet` 进行后端逻辑代码开发，而使用 `HTML` 进行数据展示

   而这里面就存在问题，`HTML` 是静态页面，就需要借助 `ajax` 了



那既然 JSP 已经逐渐的退出历史舞台，那我们为什么还要学习 `JSP` 呢？原因有两点：

* 一些公司可能有些老项目还在用 `JSP` ，所以要求我们必须动 `JSP`
* 我们如果不经历这些复杂的过程，就不能体现后面阶段开发的简单

接下来我们来学习第三阶段，使用 `EL表达式` 和 `JSTL` 标签库替换 `JSP` 中的 `Java` 代码



## 7.5:EL表达式

### 1:概述

EL（全称Expression Language ）表达式语言，用于简化 JSP 页面内的 Java 代码

EL 表达式的主要作用：获取数据

其实就是从域对象中获取数据，然后将数据展示在页面上

而 EL 表达式的语法也比较简单，${expression} 

例如：${brands} 就是获取域中存储的 key 为 brands 的数据



### 2:简单使用

在 servlet 中封装一些数据并存储到 request 域对象中并转发到 `el-demo.jsp` 页面

```java
@WebServlet("/demo1")
public class ServletDemo1 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        
        //1. 准备数据
        List<Brand> brands = new ArrayList<Brand>();
        brands.add(new Brand(1,"三只松鼠","三只松鼠",100,"三只松鼠，好吃",1));
        brands.add(new Brand(2,"优衣库","优衣库",200,"优衣库，服适人生",0));
        brands.add(new Brand(3,"小米","小米科技有限公司",1000,"为发烧而生",1));

        //2. 存储到request域中
        request.setAttribute("brands",brands);

        //3. 转发到 el-demo.jsp
        request.getRequestDispatcher("/el-demo.jsp").forward(request,response);
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```

在 `el-demo.jsp` 中通过 EL表达式 获取数据

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    ${brands}
</body>
</html>
```

在浏览器的地址栏输入 `http://localhost:8080/jsp-demo/demo1`查看效果

### 3:域对象

JavaWeb中有四大域对象，分别是：

* page：当前页面有效
* request：当前请求有效
* session：当前会话有效
* application：当前应用有效

el 表达式获取数据，会依次从这4个域中寻找，直到找到为止。而这四个域对象的作用范围如下图所示

<img src=" https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/image-20210818152857407.png" alt="image-20210818152857407" style="zoom:60%;" />

例如： ${brands}，el 表达式获取数据

1：会先从page域对象中获取数据

2：如果没有再到 requet 域对象中获取数据

3：如果再没有再到 session 域对象中获取

4：如果还没有才会到 application 中获取数据



## 7.6:JSTL

### 1:概述

JSP标准标签库(Jsp Standarded Tag Library) ，使用标签取代JSP页面上的Java代码

如下代码就是JSTL标签

```jsp
<c:if test="${flag == 1}">
    男
</c:if>
<c:if test="${flag == 2}">
    女
</c:if>
```

上面代码看起来是不是比 JSP 中嵌套 Java 代码看起来舒服好了

而且前端工程师对标签是特别敏感的，他们看到这段代码是能看懂的

JSTL 提供了很多标签，常用的是`<c:forEach>` 标签和 `<c:if>` 标签



JSTL 使用也是比较简单的，分为如下步骤：

1：导入坐标

```xml
<dependency>
    <groupId>jstl</groupId>
    <artifactId>jstl</artifactId>
    <version>1.2</version>
</dependency>
<dependency>
    <groupId>taglibs</groupId>
    <artifactId>standard</artifactId>
    <version>1.1.2</version>
</dependency>
```

2：在JSP页面上引入JSTL标签库

```jsp
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %> 
```

3：使用标签



### 2:if标签

`<c:if>`：相当于 if 判断

属性：test，用于定义条件表达式



**代码演示：**

1：定义一个 `servlet` ，在该 `servlet` 中向 request 域对象中添加键是 `status` ，值为 `1` 的数据

```java
@WebServlet("/demo2")
public class ServletDemo2 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //1. 存储数据到request域中
        request.setAttribute("status",1);

        //2. 转发到 jstl-if.jsp
        request.getRequestDispatcher("/jstl-if.jsp").forward(request,response);
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```

定义 `jstl-if.jsp` 页面，在该页面使用 `<c:if>` 标签

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    <%--
        c:if：来完成逻辑判断，替换java  if else
    --%>
    <c:if test="${status ==1}">
        启用
    </c:if>

    <c:if test="${status ==0}">
        禁用
    </c:if>
</body>
</html>
```

### 3:forEach 标签

这里的`<c:forEach>`：相当于 for 循环

java中有增强for循环和普通for循环

JSTL 中的 `<c:forEach>` 也有两种用法



用法一：

类似于 Java 中的增强for循环。涉及到的 `<c:forEach>` 中的属性如下

* items：被遍历的容器

* var：遍历产生的临时变量

* varStatus：遍历状态对象

如下代码，是从域对象中获取名为 brands 数据，该数据是一个集合

遍历遍历，并给该集合中的每一个元素起名为 `brand`，是 Brand对象

在循环里面使用 EL表达式获取每一个Brand对象的属性值

```jsp
<c:forEach items="${brands}" var="brand">
    <tr align="center">
        <td>${brand.id}</td>
        <td>${brand.brandName}</td>
        <td>${brand.companyName}</td>
        <td>${brand.description}</td>
    </tr>
</c:forEach>
```

**代码演示：**

```jsp
<c:forEach items="${brands}" var="brand" varStatus="status">
      <tr align="center">
          <%--<td>${brand.id}</td>--%>
          <td>${status.count}</td>
          <td>${brand.brandName}</td>
          <td>${brand.companyName}</td>
          <td>${brand.ordered}</td>
          <td>${brand.description}</td>
          <c:if test="${brand.status == 1}">
              <td>启用</td>
          </c:if>
          <c:if test="${brand.status != 1}">
              <td>禁用</td>
          </c:if>
          <td><a href="#">修改</a> <a href="#">删除</a></td>
      </tr>
</c:forEach>
```



用法二：

类似于 Java 中的普通for循环。涉及到的 `<c:forEach>` 中的属性如下

* begin：开始数

* end：结束数

* step：步长

实例代码：

从0循环到10，变量名是 `i` ，每次自增1

```jsp
<c:forEach begin="0" end="10" step="1" var="i">
    ${i}
</c:forEach>
```

# 8:MVC模式和三层架构

MVC 模式和三层架构是一些理论的知识，将来我们使用了它们进行代码开发会让我们代码维护性和扩展性更好。

## 8.1:MVC模式

MVC 是一种分层开发的模式，其中：

* M：Model，业务模型，处理业务

* V：View，视图，界面展示

* C：Controller，控制器，处理请求，调用模型和视图

<img src=" https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/image-20210818163348642.png" alt="image-20210818163348642" style="zoom:70%;" />

控制器（serlvlet）用来接收浏览器发送过来的请求，控制器调用模型（JavaBean）来获取

数据，比如从数据库查询数据；控制器获取到数据后再交由视图（JSP）进行数据展示

**MVC 好处：**

* 职责单一，互不影响。每个角色做它自己的事，各司其职

* 有利于分工协作

* 有利于组件重用

## 8.2:三层架构

三层架构是将我们的项目分成了三个层面，分别是 `表现层`、`业务逻辑层`、`数据访问层`

<img src=" https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/image-20210818164301154.png" alt="image-20210818164301154" style="zoom:60%;" />

* 数据访问层：对数据库的CRUD基本操作
* 业务逻辑层：对业务逻辑进行封装，组合数据访问层层中基本功能，形成复杂的业务逻辑功能。例如 `注册业务功能` ，我们会先调用 `数据访问层` 的 `selectByName()` 方法判断该用户名是否存在，如果不存在再调用 `数据访问层` 的 `insert()` 方法进行数据的添加操作
* 表现层：接收请求，封装数据，调用业务逻辑层，响应数据

而整个流程是，浏览器发送请求，表现层的Servlet接收请求并调用业务逻辑层的方法进行业务逻辑处理，而业务逻辑层方法调用数据访问层方法进行数据的操作，依次返回到serlvet，然后servlet将数据交由 JSP 进行展示。

三层架构的每一层都有特有的包名称：

* 表现层： `com.zzx.controller` 或者 `com.zzx.web`
* 业务逻辑层：`com.zzx.service`
* 数据访问层：`com.zzx.dao` 或者 `com.zzx.mapper`

后期我们还会学习一些框架，不同的框架是对不同层进行封装的

<img src=" https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/image-20210818165439826.png" alt="image-20210818165439826" style="zoom:60%;" />

## 8.3:MVC和三层架构

通过 MVC 和 三层架构 的学习，有些人肯定混淆了。那他们有什么区别和联系？

<img src=" https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/image-20210818165808589.png" alt="image-20210818165808589" style="zoom:60%;" />

如上图上半部分是 MVC 模式，上图下半部分是三层架构

1： `MVC模式` 中的 C（控制器）和 V（视图）就是 `三层架构` 中的表现层

2： `MVC 模式` 中的 M（模型）就是 `三层架构` 中的 业务逻辑层 和 数据访问层



可以将 `MVC模式` 理解成是一个大的概念，而`三层架构` 是对 `MVC模式` 实现架构的思想

那么我们以后按照要求将不同层的代码写在不同的包下，每一层里功能职责做到单一

将来如果将表现层的技术换掉，而业务逻辑层和数据访问层的代码不需要发生变化

# 9:会话技术

## 9.1:会话跟踪技术概述

对于`会话跟踪`这四个词，我们需要拆开来进行解释，首先要理解什么是`会话`

然后再去理解什么是`会话跟踪`

* 会话：用户打开浏览器，访问web服务器的资源，会话建立，直到有一方断开连接，会话结束。在一次会话中可以包含多次请求和响应，注意分清楚会话和请求的区别

  * 从浏览器发出请求到服务端响应数据给前端之后，一次会话(在浏览器和服务器之间)就被建立了
  * 会话被建立后，如果浏览器或服务端都没有被关闭，则会话就会持续建立着
  * 浏览器和服务器就可以继续使用该会话进行请求发送和响应，上述的整个过程就被称之为会话

  用实际场景来理解下会话，比如在我们访问京东的时候，当打开浏览器进入京东首页后，浏览器和京东的服务器之间就建立了一次会话，后面的搜索商品,查看商品的详情,加入购物车等都是在这一次会话中完成。

  思考:下图中总共建立了几个会话?

  ![1629382713180]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/1629382713180.png)

  每个浏览器都会与服务端建立了一个会话，加起来总共是3个会话

* 会话跟踪：一种维护浏览器状态的方法，服务器需要识别多次请求是否来自于同一浏览器，以便在同一次会话的多次请求间共享数据

  * 服务器会收到多个请求，这多个请求可能来自多个浏览器，如上图中的6个请求来自3个浏览器
  * 服务器需要用来识别请求是否来自同一个浏览器
  * 服务器用来识别浏览器的过程，这个过程就是会话跟踪
  * 服务器识别浏览器后就可以在同一个会话中多次请求之间来共享数据

  那么我们又有一个问题需要思考，一个会话中的多次请求为什么要共享数据呢?有了这个数据共享功能后能实现哪些功能呢?

  * 购物车: `加入购物车`和`去购物车结算`是两次请求，但是后面这次请求要想展示前一次请求所添加的商品，就需要用到数据共享。

    ![1629383655260]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/1629383655260.png)

  * 页面展示用户登录信息:很多网站，登录后访问多个功能发送多次请求后，浏览器上都会有当前登录用户的信息[用户名]，比如百度、京东、码云等。

    ![1629383767654]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/1629383767654.png)

  * 网站登录页面的`记住我`功能:当用户登录成功后，勾选`记住我`按钮后下次再登录的时候，网站就会自动填充用户名和密码，简化用户的登录操作，多次登录就会有多次请求，他们之间也涉及到共享数据

    ![1629383921990]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/1629383921990.png)

  * 登录页面的验证码功能:生成验证码和输入验证码点击注册这也是两次请求，这两次请求的数据之间要进行对比，相同则允许注册，不同则拒绝注册，该功能的实现也需要在同一次会话中共享数据

    ![1629384004179]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/1629384004179.png)

通过这几个例子的讲解，相信大家对`会话追踪`技术已经有了一定的理解，该技术在实际开发中也非常重要。那么接下来我们就需要去学习下`会话跟踪`技术，在学习这些技术之前，我们需要思考：为什么现在浏览器和服务器不支持数据共享呢?

* 浏览器和服务器之间使用的是HTTP请求来进行数据传输
* HTTP协议是无状态的，每次浏览器向服务器请求时，服务器都会将该请求视为新的请求
* HTTP协议设计成无状态的目的是让每次请求之间相互独立，互不影响
* 请求与请求之间独立后，就无法实现多次请求之间的数据共享

分析完具体的原因后，那么该如何实现会话跟踪技术呢? 具体的实现方式有:

(1)客户端会话跟踪技术：Cookie

(2)服务端会话跟踪技术：Session

这两个技术都可以实现会话跟踪，它们之间最大的区别是：

Cookie是存储在浏览器端

Session是存储在服务器端

它们存储都是key-value结构

## 9.2:Cookie

### 1:目标

学习Cookie，我们主要解决下面几个问题：

* 什么是Cookie?
* Cookie如何来使用?
* Cookie是如何实现的?
* Cookie的使用注意事项有哪些?



### **2:概念**

Cookie即客户端会话技术，将数据保存到客户端，以后每次请求浏览器都会自动的携带浏览器保存的Cookie数据到后台服务器进行访问



### **3:工作流程**

![1629386230207]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/1629386230207.png)

* 服务端提供了两个Servlet，分别是ServletA和ServletB
* 浏览器发送HTTP请求1给服务端，服务端ServletA接收请求并进行业务处理
* 服务端ServletA在处理的过程中可以创建一个Cookie对象并将`name=zs`的数据存入Cookie
* 服务端ServletA在响应数据的时候，会把Cookie对象响应给浏览器
* 浏览器接收到响应数据，会把Cookie对象中的数据存储在浏览器`内存`中，此时浏览器和服务端就建立了一次会话
* 在同一次会话中浏览器再次发送HTTP请求2给服务端ServletB，浏览器会自动携带Cookie对象中的所有数据
* ServletB接收到请求和数据后，就可以获取到存储在Cookie对象中的数据，这样同一个会话中的多次请求之间就实现了数据共享

### **4:基本使用**

对于Cookie的使用，我们更关注的应该是后台代码如何操作Cookie，对于Cookie的操作主要分两大类，本别是发送Cookie和获取Cookie，对于上面这两块内容，分别该如何实现呢?



#### 1:发送Cookie

* 创建Cookie对象，并设置数据

```java
Cookie cookie = new Cookie("key","value");
```

* 发送Cookie到客户端：使用response对象

```java
response.addCookie(cookie);
```





#### 2:获取Cookie

- 获取客户端携带的所有Cookie，使用request对象

```java
Cookie[] cookies = request.getCookies();
```

- 遍历数组，获取每一个Cookie对象：for
- 使用Cookie对象方法获取数据

```java
cookie.getName();
cookie.getValue();
```



那么Cookie的底层到底是如何实现一次会话两次请求之间的数据共享呢?



### 5:原理

对于Cookie的实现原理是基于HTTP协议的,其中设计到HTTP协议中的两个请求头信息：

* 响应头:set-cookie
* 请求头: cookie

![1629393289338]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/1629393289338.png)

* 假设AServlet给前端发送Cookie，BServlet从request中获取Cookie
* 对于AServlet响应数据的时候，Tomcat服务器都是基于HTTP协议来响应数据
* 当Tomcat发现后端要返回的是一个Cookie对象之后，Tomcat就会自动在响应头中添加一行数据`Set-Cookie:username=zs`
* 浏览器获取到响应结果后，就可以从响应头获取到`Set-Cookie`的值`username=zs`，并将数据存储在浏览器的内存中
* 浏览器再次发送请求给BServlet的时候，浏览器会自动把Cookie里面的所有信息自动的发送给服务端，就会在请求头中添加`Cookie: username=zs`发送给服务端BServlet
* Request对象会把请求头中cookie对应的值封装成一个个Cookie对象，最终形成一个数组
* BServlet通过Request对象获取到Cookie[]后，就可以从中获取自己需要的数据



### 6:存活时间![1629423321737]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/1629423321737.png)

(1)浏览器发送请求给AServlet，AServlet会响应一个存有`usernanme=zs`的Cookie给浏览器

(2)浏览器接收到响应数据将cookie存入到浏览器内存中

(3)当浏览器再次发送请求给BServlet，BServlet就可以使用Request对象获取到Cookie数据

(4)在发送请求到BServlet之前，如果把浏览器关闭再打开进行访问，BServlet能否获取到

Cookie数据？



注意：浏览器关闭再打开不是指打开一个新的选显卡



针对上面这个问题，通过演示，会发现，BServlet中无法再获取到Cookie数据，而且可能会

报错，这是为什么呢?



默认情况下，Cookie存储在浏览器`内存`中，当浏览器关闭，内存释放，则Cookie被销毁

这个结论就印证了上面的演示效果，但是如果使用这种默认情况下的Cookie，有些需求就无

法实现，比如:

网站的登录页面上有一个`记住我`的功能，这个功能大家都比较熟悉

* 第一次输入用户名和密码并勾选`记住我`然后进行登录
* 下次再登陆的时候，用户名和密码就会被自动填充，不需要再重新输入登录
* 比如`记住我`这个功能需要记住用户名和密码一个星期，那么使用默认情况下的Cookie就会出现问题
* 因为默认情况，浏览器一关，Cookie就会从浏览器内存中删除，对于`记住我`功能就无法实现

所以我们现在就遇到一个难题是如何将Cookie持久化存储?

Cookie其实已经为我们提供好了对应的API来完成这件事，这个API就是setMaxAge

* 设置Cookie存活时间

```java
setMaxAge(int seconds)
```

参数值为:

1.正数：将Cookie写入浏览器所在电脑的`硬盘`，持久化存储。到时间自动删除

2.负数：默认值，Cookie在当前浏览器内存中，当浏览器关闭，则Cookie被销毁

3.零：直接删除对应Cookie

```java
//发送Cookie
//1. 创建Cookie对象
Cookie cookie = new Cookie("username","zs");

//设置存活时间   ，1周 7天
cookie.setMaxAge(60*60*24*7); //易阅读，需程序计算
//cookie.setMaxAge(604800); //不易阅读(可以使用注解弥补)，程序少进行一次计算

//2. 发送Cookie，response
response.addCookie(cookie);
```

### 7:存储中文

首先，先来演示一个效果，将之前`username=zs`的值改成`username=张三`，把汉字`张三`存入到Cookie中，看是什么效果:

```java
@WebServlet("/aServlet")
public class AServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
 		//发送Cookie
        String value = "张三";
        Cookie cookie = new Cookie("username",value);
        //设置存活时间   ，1周 7天
        cookie.setMaxAge(60*60*24*7);
        //2. 发送Cookie，response
        response.addCookie(cookie);
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```

报错了！表明Cookie不能直接存储中文

但是如果有这方面的需求，这个时候该如何解决呢?

这个时候，我们可以使用之前学过的一个知识点叫`URL编码`

所以如果需要存储中文，就需要进行转码，具体的实现思路为:

> 1.在AServlet中对中文进行URL编码，将编码后的值存入  Cookie  中
>
> 2.在BServlet中获取Cookie中的值，获取的值为URL编码后的值
>
> 3.将获取的值在进行URL解码，采用URLDecoder.decode()，就可以获取到对应的中文值

```java
@WebServlet("/aServlet")
public class AServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //发送Cookie
        String value = "张三";
        
        //对中文进行URL编码
        value = URLEncoder.encode(value, "UTF-8");
        System.out.println("存储数据："+value);
        
        //将编码后的值存入Cookie中
        Cookie cookie = new Cookie("username",value);
        
        //设置存活时间   ，1周 7天
        cookie.setMaxAge(60*60*24*7);
        //2. 发送Cookie，response
        response.addCookie(cookie);
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```



```java
@WebServlet("/bServlet")
public class BServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //获取Cookie
        //1. 获取Cookie数组
        Cookie[] cookies = request.getCookies();
        //2. 遍历数组
        for (Cookie cookie : cookies) {
            //3. 获取数据
            String name = cookie.getName();
            if("username".equals(name)){
                //获取的是URL编码后的值 %E5%BC%A0%E4%B8%89
                String value = cookie.getValue();
                //URL解码
                value = URLDecoder.decode(value,"UTF-8");
                System.out.println(name+":"+value);//value解码后为 张三
                break;
            }
        }

    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```

注意：高版本的Tomcat已经可以直接存入中文了，测试Tomcat9没问题

## 9.3:Session

### 1:目标

Cookie已经能完成一次会话多次请求之间的数据共享，之前我们还提到过Session也可以实现，那么:

- 什么是Session?
- Session如何来使用?
- Session是如何实现的?
- Session的使用注意事项有哪些?



### **2:概念**

Session：服务端会话跟踪技术：将数据保存到服务端

* Session是存储在服务端而Cookie是存储在客户端
* 存储在客户端的数据容易被窃取和截获，存在很多不安全的因素
* 存储在服务端的数据相比于客户端来说就更安全

### **3:工作流程**

 ![1629427173389]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/1629427173389.png)

* 在服务端的AServlet获取一个Session对象，把数据存入其中
* 在服务端的BServlet获取到相同的Session对象，从中取出数据
* 就可以实现一次会话中多次请求之间的数据共享了
* 现在最大的问题是如何保证AServlet和BServlet使用的是同一个Session对象



### **4:基本使用**

在JavaEE中提供了HttpSession接口，来实现一次会话的多次请求之间数据共享功能

具体的使用步骤为:



获取Session对象,使用的是request对象

```
HttpSession session = request.getSession();
```



Session对象提供的功能:

存储数据到 session 域中

```
void setAttribute(String name, Object o)
```

根据 key，获取值

```
Object getAttribute(String name)
```

根据 key，删除该键值对

```
void removeAttribute(String name)
```

**注意：**Session中可以存储的是一个Object类型的数据



### 5:原理分析

Session是基于Cookie实现的

一步步来分析下Session的具体实现原理：

(1)前提条件

![1629429063101]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/1629429063101.png)

Session要想实现一次会话多次请求之间的数据共享

就必须要保证多次请求获取Session的对象是同一个

```java
HttpSession session = request.getSession();
System.out.println(session);
```

只要我的浏览器不关闭，我一直访问，新增窗口访问，也一直是同一个对象

但是我们把浏览器关闭或者换浏览器来访问，结果就不一样了



所以Session实现的也是一次会话中的多次请求之间的数据共享

那么最主要的问题就来了，Session是如何保证在一次会话中获取的Session对象是同一个呢

![1629430754825]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/1629430754825.png)



生成Session对象的时候，Session对象会有一个唯一的标识叫做JSESSIONID

Tomcat响应数据回去的时候，会把这个JSESSIONID写入Cookie里面

添加`Set-Cookie:JESSIONID=3228E5F520B668DBAFA00811095A0A70`类似的

浏览器接收到响应结果后，会把响应头中的coookie数据存储到浏览器的内存中

浏览器在同一会话中访问其他资源的时候

会把`cookie: JESSIONID=3228E5F520B668DBAFA00811095A0A70`的格式添加到请求头中

并发送给服务器Tomcat

获取到请求后，从请求头中就读取cookie中的JSESSIONID值，然后就会到服务器内存中去寻找`id:3228E5F520B668DBAFA00811095A0A70`的session对象，如果找到了，就直接返回该对象，如果没有则新创建一个session对象

关闭打开浏览器后，因为浏览器的cookie已被销毁，所以就没有JESSIONID的数据，服务端获取到的session就是一个全新的session对象



至此，`Session是基于Cookie来实现的`这就话，我们就解释完了

### 6:钝化与活化

服务器重启后，Session中的数据是否还在？

要想回答这个问题，我们可以先看下下面这幅图，

![1629438984314]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/1629438984314.png) 

(1)服务器端AServlet和BServlet共用的session对象应该是存储在服务器的内存中

(2)服务器重新启动后，内存中的数据应该是已经被释放，对象也应该都销毁了

所以session数据应该也已经不存在了。但是如果session不存在会引发什么问题呢?



举个例子说明下

(1)用户把需要购买的商品添加到购物车，因为要实现同一个会话多次请求数据共享，所以假设把数据存入Session对象中

(2)用户正要付钱的时候接到一个电话，付钱的动作就搁浅了

(3)正在用户打电话的时候，购物网站因为某些原因需要重启

(4)重启后session数据被销毁，购物车中的商品信息也就会随之而消失

(5)用户想再次发起支付，就会出为问题



所以说对于session的数据，我们应该做到就算服务器重启了

也应该能把数据保存下来才对

分析了这么多，那么Tomcat服务器在重启的时候

session数据到底会不会保存以及是如何保存的，我们可以c测试一下



注意：这里所说的关闭和启动应该要确保是正常的关闭和启动，不是出故障的关闭



经过测试，会发现只要服务器是正常关闭和启动，session中的数据是可以被保存下来的

那么Tomcat服务器到底是如何做到的呢?

具体的原因就是:Session的钝化和活化

* 钝化：在服务器正常关闭后，Tomcat会自动将Session数据写入硬盘的文件中
* 活化：再次启动服务器后，从文件中加载数据到Session中



### 7:销毁

session的销毁会有两种方式：

1：默认情况下，无操作，30分钟自动销毁

* 对于这个失效时间，是可以通过配置进行修改的

  * 在项目的web.xml中配置

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
             version="3.1">
    
        <session-config>
            <session-timeout>100</session-timeout>
        </session-config>
    </web-app>
    ```

  * 如果没有配置，默认是30分钟，默认值是在Tomcat的web.xml配置文件中写死的

  ![image-20221220105721043]( https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/image-20221220105721043.png)



2：调用Session对象的invalidate()进行销毁



## 9.4:小结

Cookie 和 Session 都是来完成`一次会话内多次请求`间数据共享的

所需两个对象放在一块，就需要思考:

Cookie和Session的区别是什么?

Cookie和Session的应用场景分别是什么?

* 区别:
  * 存储位置：Cookie 是将数据存储在客户端，Session 将数据存储在服务端
  * 安全性：Cookie不安全，Session安全
  * 数据大小：Cookie最大3KB，Session无大小限制
  * 存储时间：Cookie可以通过setMaxAge()长期存储，Session默认30分钟
  * 服务器性能：Cookie不占服务器资源，Session占用服务器资源
* 应用场景:
  * 购物车：使用Cookie来存储
  * 以登录用户的名称展示：使用Session来存储
  * 记住我功能：使用Cookie来存储
  * 验证码：使用session来存储
* 结论
  * Cookie是用来保证用户在未登录情况下的身份识别
  * Session是用来保存用户登录后的数据

具体用哪个还是需要根据具体的业务进行具体分析

# 10:Filter

## 10.1:概述

Filter 表示过滤器，是 JavaWeb 三大组件(Servlet、Filter、Listener)之一

过滤器可以把对资源的请求拦截下来，从而实现一些特殊的功能。

如下图所示，浏览器可以访问服务器上的所有的资源（servlet、jsp、html等）

<img src=" https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/image-20210823184519509.png" alt="image-20210823184519509" style="zoom:50%;" />

而在访问到这些资源之前可以使用过滤器把请求拦截下来

也就是说在访问资源之前会先经过 Filter，如下图

<img src=" https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/image-20210823184657328.png" alt="image-20210823184657328" style="zoom:57%;" />

拦截器拦截到后可以做什么功能呢？

过滤器一般完成一些通用的操作

比如每个资源都要写一些代码完成某个功能，我们总不能在每个资源中写这样的代码吧

而此时我们可以将这些代码写在过滤器中，因为请求每一个资源都要经过过滤器

或者系统没登陆之前，我们只要直到其他资源的路径，就可以直接跳转。不安全

这显然和我们的要求不符。我们希望实现的效果是用户如果登陆过了才拥有这些权限

如果没有登陆就跳转到登陆页面让用户进行登陆，要实现这个效果需要在每一个资源中都写

上这段逻辑来判断是否登陆过，而像这种通用的操作，我们就可以放在过滤器中进行实现

这个就是`权限控制`，以后我们还会进行细粒度权限控制

过滤器还可以做 `统一编码处理`、 `敏感字符处理` 等等…



## 10.2:步骤

进行 `Filter` 开发分成以下三步实现

1：定义类，实现 Filter接口，并重写其所有方法

<img src=" https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/image-20210823191006878.png" alt="image-20210823191006878" style="zoom:60%;" />

2：配置Filter拦截资源的路径

在类上定义 `@WebFilter` 注解。而注解的 `value` 属性值 `/*` 表示拦截所有的资源

<img src=" https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/image-20210823191037163.png" alt="image-20210823191037163" style="zoom:67%;" />

3：在doFilter方法中进行操作，判断是否放行

<img src=" https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/image-20210823191200201.png" alt="image-20210823191200201" style="zoom:60%;" />

上述代码中的 `chain.doFilter(request,response)` 

就是放行，也就是让其访问本该访问的资源



```java
@WebFilter(filterName = "MyFilter",value = "/*")
public class MyFilter implements Filter {
    @Override
    public void init(FilterConfig config) throws ServletException {
        System.out.println("过滤器初始化");
    }

    @Override
    public void destroy() {
        System.out.println("过滤器销毁");
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws ServletException, IOException {
        System.out.println("过滤器生效，执行中...");
        chain.doFilter(request, response);
    }
}

```

我们访问资源的时候，就会打印`过滤器生效，执行中...`

## 10.3:执行流程

<img src=" https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/image-20210823194830074.png" alt="image-20210823194830074" style="zoom:70%;" />

如上图是使用过滤器的流程，我们通过以下问题来研究过滤器的执行流程：

* 放行后访问对应资源，资源访问完成后，还会回到Filter中吗？

  从上图就可以看出肯定 会 回到Filter中

* 如果回到Filter中，是重头执行还是执行放行后的逻辑呢？

  如果是重头执行的话，就意味着 `放行前逻辑` 会被执行两次

  肯定不会这样设计了，所以访问完资源后，会回到 `放行后逻辑`，执行该部分代码

通过上述的说明，我们就可以总结Filter的执行流程如下：

<img src=" https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/image-20210823195434581.png" alt="image-20210823195434581" style="zoom:70%;" />

接下来我们通过代码验证一下，在 `doFilter()` 方法前后都加上输出语句，如下

```java
System.out.println("放行前");
chain.doFilter(request, response);
System.out.println("放行后");
```

在访问的资源中，会打印Hello

最后的打印结果是：

```
放行前

Hello

放行后
```



## 10.4:拦截路径配置

拦截路径表示 Filter 会对请求的哪些资源进行拦截，使用 `@WebFilter` 注解进行配置

如：`@WebFilter("拦截路径")` 

拦截路径有如下四种配置方式：

* 拦截具体的资源：/index.jsp：只有访问index.jsp时才会被拦截
* 目录拦截：/user/*：访问/user下的所有资源，都会被拦截
* 后缀名拦截：*.jsp：访问后缀名为jsp的资源，都会被拦截
* 拦截所有：/*：访问所有资源，都会被拦截

拦截路径的配置方式和 `Servlet` 的请求资源路径配置方式一样，但是表示的含义不同



## 10.5:过滤器链

过滤器链是指在一个Web应用，可以配置多个过滤器，这多个过滤器称为过滤器链

如下图就是一个过滤器链，我们学习过滤器链主要是学习过滤器链执行的流程

<img src=" https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/image-20210823215835812.png" alt="image-20210823215835812" style="zoom:70%;" />

上图中的过滤器链执行是按照以下流程执行：

1. 执行 `Filter1` 的放行前逻辑代码
2. 执行 `Filter1` 的放行代码
3. 执行 `Filter2` 的放行前逻辑代码
4. 执行 `Filter2` 的放行代码
5. 访问到资源
6. 执行 `Filter2` 的放行后逻辑代码
7. 执行 `Filter1` 的放行后逻辑代码

以上流程串起来就像一条链子，故称之为过滤器链



那么过滤器谁的优先级更高呢？

我们使用的是注解配置Filter，而这种配置方式的优先级是按照过滤器类名(字符串)的自然排

序，比如有如下两个名称的过滤器 ： `BFilterDemo` 和 `AFilterDemo` 

那一定是 `AFilterDemo` 过滤器先执行



# 11:Listener

## 11.1:概述

Listener 表示监听器，是 JavaWeb 三大组件(Servlet、Filter、Listener)之一

监听器可以监听就是在 `application`，`session`，`request` 三个对象创建、销毁或者

往其中添加修改删除属性时自动执行代码的功能组件

request 和 session 我们学习过

而 `application` 是 `ServletContext` 类型的对象。

注意：`ServletContext` 代表整个web应用，在服务器启动的时候，tomcat会自动创建该

对象，在服务器关闭时会自动销毁该对象

## 11.2:分类

JavaWeb 提供了8个监听器：

<img src=" https://zzx-note.oss-cn-beijing.aliyuncs.com/javaweb/image-20210823230820586.png" alt="image-20210823230820586" style="zoom:80%;" />

只有 `ServletContextListener` 这个监听器后期我们会接触到，`ServletContextListener` 

是用来监听 `ServletContext` 对象的创建和销毁。

注意：`ServletContextListener` 接口中有以下两个方法

* `void contextInitialized(ServletContextEvent sce)`：`ServletContext` 对象被创建了会自动执行的方法
* `void contextDestroyed(ServletContextEvent sce)`：`ServletContext` 对象被销毁时会自动执行的方法

## 11.3:演示

演示一下 `ServletContextListener` 监听器

* 定义一个类，实现`ServletContextListener` 接口
* 重写所有的抽象方法
* 使用 `@WebListener` 进行配置

代码如下：

```java
@WebListener
public class ContextListener implements ServletContextListener{

    @Override
    public void contextInitialized(ServletContextEvent sce) {
        System.out.println("web容器创建成功");
    }

    @Override
    public void contextDestroyed(ServletContextEvent sce) {
        ServletContextListener.super.contextDestroyed(sce);
    }
}
```

启动服务器，就可以在启动的日志信息中看到 `contextInitialized()` 方法输出的内容

同时也说明了 `ServletContext` 对象在服务器启动的时候被创建了



常用的还有：HttpSessionListener监听器和HttpSessionAttributeListener监听器