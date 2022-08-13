# JavaEE

JavaEE三大技术核心：

1. Servlet : 有Java编写的，可以运行在服务器中的动态页面
2. Filter : 过滤器，在请求到达Servlet前进行过滤操作
3. Listener : 监听器，监听Request对象、Session对象、Application对象的属性的变化和生命周期



## Servlet

Servlet是Server Applet的简称，翻译过来就是服务程序。简单的讲，Servlet实际上就是由Java编写的一块代码，它是一个Servlet接口的实现类，是运行在Web容器中的动态页面

Servlet在MVC模型中，承担的作用就是Controller（控制器）。

浏览器可以通过url访问到servlet

### 生命周期

所谓生命周期就是从创建到销毁的过程。

Serlvet的生命周期就是 init()、service() 、destroy()

Servlet运行在Servlet容器（web服务器）中，其生命周期由容器来管理，分为4个阶段：

1. 加载和实例化：默认情况下，当Servlet第一次被访问时，由容器创建Servlet对象

   ```java
   //默认是第一次访问的时候被创建，可以使用@WebServlet(urlPatterns = “/demo2”,loadOnStartup = 1)的loadOnStartup 修改成在服务器启动的时候创建。
   @WebServlet(urlPatterns = "/demo1",loadOnStartup = 1)
   //loadOnstartup的取值有两类情况
   //	（1）负整数:第一次访问时创建Servlet对象
   //	（2）0或正整数:服务器启动时创建Servlet对象，数字越小优先级越高（相当于把创建Servlet对象提前到服服务器启动时，让访问赶快）
   ```

   

2. 初始化：在Servlet实例化之后，容器讲调用Servlet的init()方法初始化这个对象，该方法只执行一次

3. 请求处理：每次请求Servlet时，Servlet容器都会调用Servlet的service()方法对请求进行处理

4. 服务终止：当需要释放内存或者容器关闭时，容器就会调用Servlet实例的destroy()方法完成资源的释放。在destroy()方法调用之后，容器会释放这个Servlet实例，该实例随后会被Java的垃圾收集器所回收

### Servlet接口、GenericServlet类、HttpServlet类

![image-20220812203440219](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220812203440219.png)

#### Servlet接口

Servlet接口中总共提供5个抽象方法

1. **init(ServletConfig)**：对Servlet进行初始化操作，当Servlet对象创建完成后，Web容器调用该方法。在Servlet生命周期中，该方法只会执行一次。在第一次访问Servlet时执行。

2. **service(ServletRequest, ServletResponse)**：用于对用户的请求进行处理和响应，每一次的客户端请求，都由该方法进行处理。

3. **destroy()：**对Servlet进行销毁，释放Servlet中占用的资源。服务器正常关闭时。

4. getServletInfo() ：获取Servlet的字符串信息，一般情况下不需要实现

5. getServletConfig() ：返回当前Servlet的配置信息的对象，也就是一个ServletConfig对象。

#### GenericServlet

GenericServlet类是一个抽象类，实现了Servlet、ServletConfig、Serializable接口。

GenericServlet类实现了Servlet接口中的init、destroy、getServletInfo、getServletConfig四个方法。没有实现Service方法。

#### HttpServlet

HttpServlet类也是一个抽象类，继承自GenericServlet。

HttpServlet类实现了Servlet接口的Service方法。(在Service方法内对参数进行类型的强转，又调用了一个重载的service方法)

实质是在重载的service方法中，根据请求类型的不同，将请求分流到不同的方法中。

![image-20220812210303509](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220812210303509.png)

**注意：一般我们在写Servlet的时候，继承HttpServlet**

### 配置Servlet

Servlet有两种配置方式   两种方式是等效的

1. 在web.xml中配置

   ```xml
   <!-- 配置Servlet的路径 -->
   <!-- 1. 为Servlet起一个名字 -->
   <servlet>
       <!-- 为Servlet起的一个别名 -->
       <servlet-name>myServlet</servlet-name>
       <!-- Servle类的全名 -->
       <servlet-class>com.situ.servlet.MyServlet</servlet-class>
   </servlet>
   
   <!-- 2. 为Servlet指定访问地址 -->
   <servlet-mapping>
       <!-- 前面已经起的Servlet的名字 -->
       <servlet-name>myServlet</servlet-name>
       <!-- 设置url的格式 -->
       <url-pattern>/myServlet</url-pattern>
   </servlet-mapping>
   ```

2. 使用注解

   在Servlet类的前面添加注解

   ```java
   @WebServlet("/url")
   ```

注：一个Servlet可与配置多个urlPattern

```java
@WebServlet("urlPatterns={"/demo01","/demo02"})
//在浏览器上输入http://localhost:8080/web-demo/demo1,http://localhost:8080/web-demo/demo2这两个地址都能访问到ServletDemo7的doGet方法。
```



### Request对象和Response对象

#### Request对象（HttpServletRequest）

Reqeust对象表示HTTP请求，包含了请求url，参数，客户端内容等属性

Request是Servlet.service()方法的一个参数，**类型为javax.servlet.http.HttpServletRequest**。在客户端发出每个请求时，服务器都会创建一个request对象，并把请求数据封装到request中，然后在调用Servlet.service()方法时传递给service()方法，这说明在service()方法中可以通过request对象来获取请求数据。

**request对象的功能**

（1）封装了请求头数据；

（2）封装了请求正文数据，如果是GET请求，那么就没有正文；

（3）request是一个域对象，可以把它当成Map来添加获取数据；

（4）request提供了请求转发和请求包含功能。

##### **request的域方法**

request是域对象！在JavaWeb中一共四个域对象

一个请求会创建一个request对象，如果在一个请求中经历了多个Servlet，那么多个Servlet就可以使用request来共享数据。现在我们还不知道如何在一个请求中经历几个Servlet。

```java
void setAttribute(String name, Object value)  //用来存储一个对象，也可以称之为存储一个域属性
Object getAttribute(String name)		//用来获取request中的数据
void removeAttribute(String name)		//用来移除request中的域属性
Enumeration getAttributeNames()			//获取所有域属性的名称；
```

##### **常用API**

```java
String getCharacterEncoding()	//获取请求编码，如果没有setCharacterEncoding()，那么返回null，表示使用ISO-8859-1编码；
void setCharacterEncoding(String code)	//设置请求编码，只对请求体有效！注意，对于GET而言，没有请求体！
String getContextPath()		//返回上下文路径，例如：/hello
String getRequestURI()		//返回请求URI路径，例如：/hello/oneServlet
StringBuffer getRequestURL()	//返回请求URL路径，例如：http://localhost/hello/oneServlet，即返回除了参数以外的路径信息
String getServletPath()		//返回Servlet路径，例如：/oneServlet
String getRemoteAddr()		//返回当前客户端的IP地址；
String getRemoteHost()		//返回当前客户端的主机名，但这个方法的实现还是获取IP地址；
String getScheme()			//返回请求协议，例如：http；
String getServerName()		//返回主机名，例如：localhost
int getServerPort()			//返回服务器端口号，例如：8080
    
//常用的获取base路径的语句
<%
    // http://localhost:8080/staffmgr/
    String basePath = request.getScheme() + "://" + request.getServerName() + ":" + request.getServerPort() +
            request.getContextPath() + "/";
%>
    
// 获取请求参数的方法
String getParameter(String name)  //通过指定名称获取参数值；
String[] getParameterValues(String name) //当多个参数名称相同时，可以使用方法来获取；
Enumeration getParameterNames()  //获取所有参数的名字；
Map getParameterMap()  //获取所有参数封装到Map中，其中key为参数名，value为参数值，因为一个参数名称可能有多个值，所以value的数据类型是String[]，而不是String。
```

##### 请求转发和请求包含

请求转发和请求包含，都表示由**多个Servlet共同来处理一个请求**
`RequestDispatcher rd = request.getRequestDispatcher("/MyServlet");  `使用request获取RequestDispatcher对象，方法的参数是被转发或包含的Servlet的Servlet路径

- **请求转发**：rd.forward(request,response);

  由下一个servlet来完成响应体，当前servlet还可以设置请求头（留头不留体）

  ![image-20220812221217455](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220812221217455.png)

- **请求包含**：rd.include(request,response);

  由两个Servlet共同完成未完成的响应体

  ![image-20220812221227200](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220812221227200.png)

**注：无论是请求转发还是请求包含，都在一个请求范围内！使用同一个request和response！**

#### Response对象（HttpServletResponse）

response是响应对象，向客户端输出响应正文（响应体）

response是Servlet.service方法的一个参数，类型为javax.servlet.http.**HttpServletResponse**。在客户端发出每个请求时，服务器都会创建一个response对象，并传入给Servlet.service()方法。response对象是用来对客户端进行响应的，这说明**在service()方法中使用response对象可以完成对客户端的响应工作。**

**response对象的作用**

（1）设置响应头信息

（2）发送状态码

（3）设置响应正文

**（4）重定向**

##### response响应正文（响应体）

response提供了两个响应流对象

1. response.getWriter()：获取字符流；

2. response.getOutputStream()：获取字节流；

当然，如果响应正文内容为字符，那么使用response.getWriter()，如果响应内容是字节，例如下载时，那么可以使用response.getOutputStream()。

**注意，在一个请求中，不能同时使用这两个流！也就是说，要么你使用repsonse.getWriter()，要么使用response.getOutputStream()，但不能同时使用这两个流。不然会抛出illegalStateException异常。**

注：

- 字符编码

  在使用response.getWriter()时需要注意默认字符编码为ISO-8859-1，如果希望设置字符流的字符编码为utf-8，可以使用response.setCharaceterEncoding(“utf-8”)来设置。这样可以保证输出给客户端的字符都是使用UTF-8编码的！

  但客户端浏览器并不知道响应数据是什么编码的！如果希望通知客户端使用UTF-8来解读响应数据，那么还是**使用response.setContentType("text/html;charset=utf-8")方法比较好，因为这个方法不只会调用response.setCharaceterEncoding(“utf-8”)，还会设置content-type响应头，客户端浏览器会使用content-type头来解读响应数据。**

- 缓冲区

  response.getWriter()是PrintWriter类型，所以它有缓冲区，缓冲区的默认大小为8KB。也就是说，在响应数据没有输出8KB之前，数据都是存放在缓冲区中，而不会立刻发送到客户端。当Servlet执行结束后，服务器才会去刷新流，使缓冲区中的数据发送到客户端。

##### **常用API**

```java
// 设置响应头信息
response.setHeader(“content-type”, “text/html;charset=utf-8”); //设置content-type响应头，该头的作用是告诉浏览器响应内容为html类型，编码为utf-8。而且同时会设置response的字符流编码为utf-8，即response.setCharaceterEncoding(“utf-8”)；
response.setHeader("Refresh","5; URL=http://www.baidu.com"); //5秒后自动跳转到百度主页。

// 设置状态码及其他方法
response.setContentType("text/html;charset=utf-8"); //等同与调用response.setHeader(“content-type”, “text/html;charset=utf-8”)；
response.setCharacterEncoding(“utf-8”); //设置字符响应流的字符编码为utf-8；
response.setStatus(200); 	//设置状态码；
response.sendError(404, “您要查找的资源不存在”); //当发送错误状态码时，Tomcat会跳转到固定的错误页面去，但可以显示错误信息

```

##### 重定向

什么是重定向？ 重定向就是浏览器发送一个请求，服务端响应告诉浏览器，让他再去请求一个指定路径

例如当你访问http://www.sun.com时，你会发现浏览器地址栏中的URL会变成http://www.[Oracle](http://lib.csdn.net/base/oracle).com/us/sun/index.htm，这就是重定向了。重定向是服务器通知浏览器去访问另一个地址，即再发出另一个请求。

![image-20220812223037939](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220812223037939.png)

实现重定向的方式：

1. 首先使用response对象向浏览器发送302的状态码，之后再设置一个Location，即给出一个可用的URL，由浏览器去访问新的URL，实现重定向。

   ```java
   public class AServlet extends HttpServlet {  
       public void doGet(HttpServletRequest request, HttpServletResponse response)  
               throws ServletException, IOException {  
           response.setStatus(302);   
           response.setHeader("Location", "http://www.baidu.com");   
       }  
   }
   //当访问AServlet后，会通知浏览器重定向到百度主页。客户端浏览器解析到响应码为302后，就知道服务器让它重定向，所以它会马上获取响应头Location，然发出第二个请求。
   ```

2. **使用response.sendRedirect()方法**

### Servlet响应页面和响应数据

**响应页面**

1、转发 — 也就是携带客户端发送的请求转发跳转到下一个页面，使用的是request请求对象。

```java
request.getRequestDispatcher("跳转页面路径").forward(request,response);
```

2、重定向 — 服务器接收到前端请求，通过响应将要跳转的页面地址响应给客户端，客户端再重新发送请求跳转到目标页面。

```java
response.sendRedirect("响应的页面路径")
```

**响应数据**

响应数据也是通过流的方式响应给前端，这里主要用到的是一条打印流（也就是之前**JSP中的内置对象out**）.响应数据一般都是响应的json格式的数据，前端获取到json格式的数据就相当于拿到了一个对象，**通过对象的方式去获取数据。**

### Servlet的线程安全问题

**Servlet是单实例多线程的**

同一个Servlet在服务器中只存在一个实例，不论是多少个访问，都调用的是同一个实例。（一个请求一个线程，但是都是同一个servlet实例）

所以在Servlet中要尽量避免使用全局变量（如果出现了成员变量，就要加同步锁来避免出现并发问题）

## JSP

Java Server Page：用Java编写的运行在服务器上的网页，这就是一个动态页面。

jsp是一种即可以使用HTML、CSS、JS，也可使用JSP中的动态元素，混合成的一种页面。

- jsp页面本质是一个Servlet程序
- 当我们第一次访问jsp页面的时间，tomcat服务器会帮我们把jsp页面翻译成为一个java源文件，并且对它进行编译成为.class字节码程序。

### JSP语法

JSP通常由前端代码，再加上JSP的指令标签、JSP的动作标签、JSP的内嵌Java代码组成。

#### 内嵌Java代码

- Java代码片段

  可以像在Java的方法中一样，在Java代码片段中编写Java代码。

  注：一个JSP中的所有代码片段可以理解为在同一个方法内部，**代码片段之间是拼接的**

  ```jsp
  <%
     // java代码
  %>
  ```

- 输出表达式

  将Java表达式的值输出到JSP页面中。

  ```jsp
  <%=Java表达式%>
  ```

- 声明代码块

  就像在Java的类中一样，可以声明变量和方法。

  这时声明的变量和方法可以在当前页面的任何位置调用。

  ```jsp
  <%!
      // 声明变量
      // 声明方法
  %>
  ```

#### JSP中的注释

1. html注释

   ```jsp
   <!-- html注释-->
   ```

2. java注释

   ```jsp
   <%  //java注释  %>
   ```

3. jsp注释

   ```jsp
   <%-- jsp注释 -->
   ```

   

### JSP标签

#### 指令标签

```jsp
<%@ 标签名 %>
```

- **page指令**：用来设置当前的JSP页面的一个配置信息。

  ```jsp
  <%@page import="java.util.Date"%>
  ```

  常见属性：

  ​	language：使用的语言

  ​	contentType：内容类型，作用相当于response.setContentType();

  ​	pageEncoding：设置当前页面保存时使用的编码格式。

  ​	import：引用其它的类。相当于java中的improt关键字。

  ​	isErrorPage：设置当前页面作为一个错误处理页面，默认值是false。

  ​	errorPage：设置当前页面抛出异常时处理页面。

- **include指令**：静态地引入其它页面。

  在翻译阶段将其它页面的代码，直接拷贝到当前页面。

  ```jsp
  <%@ include file="inner.jsp" %>
  ```

- **taglib指令**：引入自定义标签库。使用JSTL时会用到。

  <%@ taglib prefix="前缀" uri="自定义标签库的路径" %>

#### 动作标签

```jsp
<jsp:标签名></jsp:标签名>
```

- forward：用于页面的请求转发。由一个页面跳转到另一个页面。

- include：动态地引入一个外部页面。

  在页面执行时，执行被引入的页面，再将其执行结果，拼接到当前页面中来。

- param：在forward和include标签时，传递参数的。

### JSP运行原理

JSP文件在运行时会被翻译成xxx_jsp.java，这里面是一个类，继承自HttpJspBase抽象类，HttpJspBase继承HttpServlet类。因此jsp就是一个Servlet。

![image-20220813093222817](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220813093222817.png)

1. 客户端发送请求，请求访问jsp文件。

2. jsp容器先将jsp文件转化成一个java源文件（Java Servlet源程序），在转换过程中，如果发现jsp文件中存在任何语法错误，则中断转换过程，并向服务端和客户端返回出错信息。

3. 如果转换成功，则jsp容器将生成的java源文件编译成相应的字节码文件*.class。该class文件就是一个Servlet，Servlet容器会像处理其他Servlet一样来处理它。

4. 由Servlet容器加载转换后的Servlet类（.class文件）创建一个该Servlet（jsp页面的转换结果）实例，并执行Servlet的jspInit()方法。jspInit()方法在Servlet的整个生命周期中只会执行一次。

5. 执行jspService()方法来处理客户端的请求。对于每个请求，jsp容器都会创建一个新的线程来处理它。如果多个客户端同时请求该jsp文件，则jsp容器也会创建多个线程，使得每一个客户端请求都对应一个线程。jsp运行过程中采用这种多线程的执行方式可以极大地降低对系统资源的需求，提高系统的并发量并缩短相应时间。需要注意的是，由于第4步生成的Servlet是常驻内存的，所以响应的速度非常快。

6. 如果jsp文件被修改了，则服务器将根据设置来决定是否对该文件重新编译。如果需要重新编译，则使用重新编译后的结果取代内存中常驻的Servlet，并继续上述处理过程。

7. 虽然jsp效率很高，但在第一次调用的时候往往由于需要转换和编译，所以会产生一些轻微的延迟。此外，由于系统资源不足等原因，jsp容器可能会以某种不确定的方式将Servlet从内存中移除，发生这种情况时，首先会调用jspDestroy()方法，然后Servlet实例会被加入“垃圾收集”处理。

8. 当请求处理完成后，Web服务器以静态HTML网页的形式将HTTP response返回到您的浏览器中
   



### 九大内置对象

jsp中的内置对象，是指tomcat在翻译jsp页面成为Servlet源代码后，内置提供的九大对象，叫内置对象（可以在JSP页面中直接使用的对象）

1. request：请求对象，携带来自客户端的数据。

2. response：响应对象，向客户端返回数据。

3. session：会话对象，客户端与服务器的一次会话。

4. application：应用对象，当前的项目。ServletContext类型的，可以实现同一应用下的不同会话之间的数据共享

5. page：当前JSP页面。

6. pageContext：当前页面的上下文，获取其它的8个内置对象。

7. config：当前页面的配置。

8. out：输出对象。

9. exception：只存在于异常处理页面，当前抛出的异常对象。

### 四大作用域

|             |       类对象       |         范围          |
| :---------: | :----------------: | :-------------------: |
| pageContext | PageContextImpl类  |  当前jsp页面范围有效  |
|   request   | HttpServletRequest |    一次请求内有效     |
|   session   |    HttpSession     |  一个会话范围内有效   |
| application |   ServletContext   | 整个Web工程范围内有效 |

域对象是可以像map一样存取数据的对象，四个域对象功能一样，不同的是它们对数据的存取范围

优先顺序分别为  pageContext ===> request ===> session ===> application

![image-20220813094042183](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220813094042183.png)

### EL表达式

EL 全名为Expression Language。

语法：`${标识符}`

**作用**：

1. 获取数据

   EL表达式主要用于替换JSP页面中的脚本表达式，以从各种类型的web域 中检索java对象、获取数据。(某个web域 中的对象，访问[javabean](https://so.csdn.net/so/search?q=javabean&spm=1001.2101.3001.7020)的属性、访问list集合、访问map集合、访问数组)

2. 执行运算

   利用EL表达式可以在JSP页面中执行一些基本的关系运算、逻辑运算和算术运算，以在JSP页面中完成一些简单的逻辑运算。${user==null}

3. 调用Java方法

   EL表达式允许用户开发自定义EL函数，以在JSP页面中通过EL表达式调用Java类的方法。

注：默认情况下，EL表达式，可以访问request、session、application三个对象中的属性。优先级**request>session>application。**

#### 11个内置对象（隐式对象）

EL表达式语言中定义了11个隐含对象，使用这些隐含对象可以很方便地获取web开发中的一些常见对象，并读取这些对象的数据。
　　语法：**${隐式对象名称}**：获得对象的引用

| 序号 | 隐含对象名称     | 描述                                                         |
| ---- | ---------------- | ------------------------------------------------------------ |
| 1    | pageContext      | 对应于JSP页面中的pageContext对象（注意：取的是pageContext对象。） |
| 2    | pageScope        | 代表page域中用于保存属性的Map对象                            |
| 3    | requestScope     | 代表request域中用于保存属性的Map对象                         |
| 4    | sessionScope     | 代表session域中用于保存属性的Map对象                         |
| 5    | applicationScope | 代表application域中用于保存属性的Map对象                     |
| 6    | param            | 表示一个保存了所有请求参数的Map对象                          |
| 7    | paramValues      | 表示一个保存了所有请求参数的Map对象，它对于某个请求参数，返回的是一个string[] |
| 8    | header           | 表示一个保存了所有http请求头字段的Map对象，注意：如果头里面有“-” ，例Accept-Encoding，则要header[“Accept-Encoding”] |
| 9    | headerValues     | 表示一个保存了所有http请求头字段的Map对象，它对于某个请求参数，返回的是一个string[]数组。注意：如果头里面有“-” ，例Accept-Encoding，则要headerValues[“Accept-Encoding”] |
| 10   | cookie           | 表示一个保存了所有cookie的Map对象                            |
| 11   | initParam        | 表示一个保存了所有web应用初始化参数的map对象                 |

### JSTL

JSTL（Java server pages Standarded Tag Library) JSP的标准标签库。

以第三方自定义标签的方式使用。需要引入。（1.引入jar包；2.利用taglib指令标签引入）

JSTL的五个部分：

1） core：提供逻辑处理。

2） format：格式化。

3） functions：提供一些通用的函数。

4） xml：处理xml格式。

5） sql：处理数据库访问。

常用：

```jsp
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ taglib prefix="fmt" uri="http://java.sun.com/jsp/jstl/fmt" %>

// prefix 是自定义的前缀

<c:if test="条件"></c:if>
<c:choose>
	<c:when test="${age > 80 }">老年人</c:when>
	<c:when test="${age >= 18 }">成年人</c:when>
	<c:otherwise>儿童</c:otherwise>
</c:choose>
<c:forEach items="${arr }" var="num" varStatus="numStatus" 
	begin="1" end="5" step="2">
	<li>${num } - ${numStatus.index }</li>
</c:forEach>

<fmt:formatDate value="${now }" pattern="yyyy年MM月dd日 HH:mm:ss"/>
```

## Filter

过滤器，顾名思义就是对事物进行过滤的，在Web中的过滤器，当然就是对请求进行过滤，我们使用过滤器，就可以对请求进行拦截，然后做相应的处理，实现许多特殊功能。如登录控制，权限管理，过滤敏感词汇等.

简单来讲：过滤器就是对请求先进行过滤处理，然后请求才能继续发送

### 过滤器原理

当我们使用过滤器对请求进行过滤时，过滤器可以动态的分为3个部分（1.执行放行之前的代码；2.放行；3.执行放行后的代码）

- 第一部分代码会对游览器请求进行第一次过滤，然后继续执行
- 第二部分代码就是将游览器请求放行，如果还有过滤器，那么就继续交给下一个过滤器
- 第三部分代码就是对返回的Web资源再次进行过滤处理

![image-20220813100613284](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220813100613284.png)

![image-20220813102404884](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220813102404884.png)

**也就是说，不止请求会经过过滤器，我们的响应也会经过过滤器。**

### Filter接口

1） init：初始化。

2） doFilter：进请求进行过滤。

3） destory：销毁，用于释放资源。

### 使用Filter过滤器

Filter就在servlet-api.jar中

我们创建Filter时，只需要继承Filter接口就行

```java
public class EncodingFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        // 配置request和response的编码格式
        servletRequest.setCharacterEncoding("utf-8");
        servletResponse.setContentType("text/html; charset=utf-8");
        //放行
        filterChain.doFilter(servletRequest, servletResponse);
    }

    @Override
    public void destroy() {
    }
}
```

接下来就是要配置Filter了

Filter的配置方式和Servlet就基本一致

1. **web.xml配置**

   ```xml
   <filter>
      <filter-name>名字</filter-name>
      <filter-class>类的全限定名称</filter-class>
   </filter>
   <filter-mapping>
       <filter-name>名字</filter-name>
       <url-pattern>路径</url-pattern>
   </filter-mapping>
   
   <!--示例-->
   <filter>
       <filter-name>encodingFilter</filter-name>
       <filter-class>com.situ.staffmgr.filter.EncodingFilter</filter-class>
   </filter>
   <filter-mapping>
       <filter-name>encodingFilter</filter-name>
       <url-pattern>/*</url-pattern>
   </filter-mapping>
   ```

2. **注解配置**

   ```java
   //在Filter类上方加上该注解 参数为拦截的路径 可以为多个
   @WebFilter("/*");
   ```

   

### Filter的生命周期

 Filter有3个阶段，分别是初始化，拦截和过滤，销毁。

1. 初始化阶段：当服务器启动时，我们的服务器(Tomcat)就会读取配置文件，扫描注解，然后来创建我们的Filter。
2. 拦截和过滤阶段：只要请求资源的路径和拦截的路径相同，那么过滤器就会对请求进行过滤，这个阶段在服务器运行过程中会一直循环。
3. 销毁阶段：当服务器(Tomcat)关闭时，服务器创建的Filter也会随之销毁。
   

### 执行顺序

1） 先执行Web.xml中配置的Filter，再执行注解配置的。

2） 在Web.xml中先配置filter-mapping的先执行。

3） 都是注解，按照Filter类名的顺序执行。



## Listener

监听器用于**监听web应用中某些对象、信息的创建、销毁、增加，修改，删除等动作的发生**，然后作出相应的响应处理。当范围对象的状态发生变化的时候，服务器自动调用监听器对象中的方法。常用于统计在线人数和在线用户，系统加载时进行信息初始化，统计网站的访问量等等。

**主要是用来监听Request对象、Session对象、Application对象。（监听对象属性的变化、对象的生命周期）**

### 分类

1） ServletContextListener接口： 监听Application对象的生命周期。

2） ServletcontextAttributeListener接口：监听Application对象的属性变化。

3） HttpSessionListener接口：监听Session对象的生命周期。

4） HttpSessionAttributeListener接口：监听Session对象的属性变化。

5） ServletRequestListener接口：监听Request对象的生命周期。

6） ServletRequestAttributeListener接口：监听Request对象的属性变化。

### 配置方式

1. web.xml

   ```xml
   <listener>
       <listener-class>类名</listener-class>
   </listener>
   ```

2. 注解

   ```java
   @WebListener
   ```

   

## 会话

### 概念

会话用来识别不同的客户端，客户端和服务器之间发生的一系列连续的请求和响应的过程，打开浏览器进行操作到关闭浏览器的过程，就是一次会话。
会话状态是服务器和浏览器在会话过程中产生的状态信息，借助于会话状态，服务器能够把属于同一会话的一系列请求和响应过程关联起来。

会话技术大体上分为Cookie和Session两种

### Cookie

Cookie通过在客户端记录信息确定用户身份

#### Cookie的类型

Cookie是由用户客户端进行保存的（一般是浏览器），按其存储位置可分为：内存式Cookie和硬盘式Cookie。

- 内存式Cookie存储在内存中，浏览器关闭后就会消失，由于其存储时间较短，因此也被称为非持久Cookie或会话Cookie。


- 硬盘式Cookie保存在硬盘中，其不会随浏览器的关闭而消失，除非用户手工清理或到了过期时间。由于硬盘式Cookie存储时间是长期的，因此也被称为持久Cookie。



#### Cookie的工作原理

Cookie定义了一些HTTP请求头和HTTP响应头，通过这些HTTP头信息使服务器可以与客户进行状态交互。

客户端请求服务器后，如果服务器需要记录用户状态，服务器会在响应信息中包含一个Set-Cookie的响应头，客户端会根据这个响应头存储Cookie信息。再次请求服务器时，客户端会在请求信息中包含一个Cookie请求头，而服务器会根据这个请求头进行用户身份、状态等较验。

![image-20220813104705015](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220813104705015.png)

####  属性

![image-20220813120043817](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220813120043817.png)

![image-20220813120055149](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220813120055149.png)

Java中把Cookie封装成了javax.servlet.http.Cookie类。每个Cookie都是该Cookie类的对象。服务器通过操作Cookie类对象对客户端Cookie进行操作。

Cookie对象使用key-value属性对的形式保存用户状态，一个Cookie对象保存一个属性对，一个request或者response同时使用多个Cookie。因为Cookie类位于包javax.servlet.http.*下面，所以JSP中不需要import该类。

**注意：**

- Cookie并不提供修改、删除操作。如果要修改某个Cookie，只需要新建一个同名的Cookie，添加到response中覆盖原来的Cookie。

  **如果要删除某个Cookie，只需要新建一个同名的Cookie，并将maxAge设置为0，并添加到response中覆盖原来的Cookie。**注意是0而不是负数。负数代表其他的意义。
  修改、删除Cookie时，新建的Cookie除value、maxAge之外的所有属性，例如name、path、domain等，都要与原Cookie完全一样。否则，浏览器将视为两个不同的Cookie不予覆盖，导致修改、删除失败。

- 如果不希望Cookie在HTTP等非安全协议中传输，可以**设置Cookie的secure属性为true。浏览器只会在HTTPS和SSL等安全协议中传输此类Cookie**。ecure属性并不能对Cookie内容加密



### Session

Session通过在服务器端记录信息确定用户身份

Session是另一种记录客户状态的机制，不同的是Cookie保存在客户端浏览器中，而Session保存在服务器上。客户端浏览器访问服务器的时候，服务器把客户端信息以某种形式记录在服务器上。这就是Session。客户端浏览器再次访问时只需要从该Session中查找该客户的状态就可以了。（因为浏览器会带着sessionID,从而去找到服务器中对应的session）

#### Session的工作原理

当程序需要为某个客户端的请求创建一个session的时候，服务器首先检查这个客户端的请求里是否已包含了一个session标识，称为session id，如果已包含一个session id则说明以前已经为此客户端创建过session，服务器就按照session id把这个session检索出来使用（如果检索不到，可能会新建一个），如果客户端请求不包含session id，则为此客户端创建一个session并且生成一个与此session相关联的session id，session id的值应该是一个既不会重复，又不容易被找到规律以仿造的字符串，这个session id将被在本次响应中返回给客户端保存。

![image-20220813140455447](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220813140455447.png)

#### 保存sessionID的方式

1. **使用Cookie进行保存**

   保存session id的方式可以采用cookie，这样在交互过程中浏览器可以自动的按照规则把这个标识发挥给服务器。一般这个cookie的名字都是类似于SEEESIONID。然而，比如weblogic对于web应用程序生成的cookie，JSESSIONID=ByOK3vjFD75aPnrF7C2HmdnV6QZcEbzWoWiBYEnLerjQ99zWpBng!-145788764，它的名字就是JSESSIONID。

2. **URL重写（用url传递给服务器）**

   由于cookie可以被人为的禁止，必须有其他机制以便在cookie被禁止时仍然能够把session id传递回服务器。经常被使用的一种技术叫做URL重写，就是把session id直接附加在URL路径的后面。

   附加方式也有两种，一种是作为URL路径的附加信息，表现形式为：

   - `http://...../xxx;jsessionid=ByOK3vjFD75aPnrF7C2HmdnV6QZcEbzWoWiBYEnLerjQ99zWpBng!-145788764`

   另一种是作为参数附加在URL后面，表现形式为：

   - `http://...../xxx?jsessionid=ByOK3vjFD75aPnrF7C2HmdnV6QZcEbzWoWiBYEnLerjQ99zWpBng!-145788764 `

   **这两种方式对于用户来说是没有区别的，只是服务器在解析的时候处理的方式不同**，采用第一种方式也有利于把session id的信息和正常程序参数区分开来。为了在整个交互过程中始终保持状态，就必须在每个客户端可能请求的路径后面都包含这个session id。

3. 表单隐藏字段（不常用）

   ```html
   <form name="testform" action="/xxx"> 
       <input type="hidden" name="jsessionid"             
           value="ByOK3vjFD75aPnrF7C2HmdnV6QZcEbzWoWiBYEnLerjQ99zWpBng!-145788764"> 
       <input type="text"> 
   </form>
   ```

#### HttpSession的常用方法

![image-20220813141637133](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220813141637133.png)

![image-20220813141648881](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220813141648881.png)

#### Session的生命周期

Session在用户第一次访问服务器的时候自动创建。需要注意只有访问JSP、Servlet等程序时才会创建Session，只访问HTML、IMAGE等静态资源并不会创建Session。如果尚未生成Session，也可以使用request.getSession(true)强制生成Session。

Session生成后，只要用户继续访问，服务器就会更新Session的最后访问时间，并维护该Session。用户每访问服务器一次，无论是否读写Session，服务器都认为该用户的Session“活跃（active）”了一次。

由于会有越来越多的用户访问服务器，因此Session也会越来越多。为防止内存溢出，服务器会把长时间内没有活跃的Session从内存删除。这个时间就是Session的超时时间。如果超过了超时时间没访问过服务器，Session就自动失效了。

Session的超时时间为maxInactiveInterval属性，可以通过对应的getMaxInactiveInterval()获取，通过setMaxInactiveInterval(long interval)修改。另外，通过调用Session的invalidate()方法可以使Session失效。Tomcat中Session的默认超时时间为20分钟。通过setMaxInactiveInterval(int seconds)修改超时时间。Session的超时时间也可以在web.xml中修改。例如修改为60分钟：

```xml
<session-config>
   <session-timeout>60</session-timeout>      <!-- 单位：分钟 -->
</session-config>
```

**注意**：

- <session-timeout>参数的单位为分钟，而setMaxInactiveInterval(int s)单位为秒。
- 在Tomcat中，tomcat被正常关闭时，session里面的数据以及sessionID全部被序列化到本地硬盘中，在磁盘中生成sessions.ser文件。在再次启动tomcat时，sessions.ser里的数据sessionID会全部注入到一个新的session对象中，并且sessions.ser文件。因此，重启应用后，发现之前session里的数据还是能访问到的。（不要在IDEA中重启tomcat来验证，）