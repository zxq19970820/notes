# Jsp教程



### B/S和C/S区别:

浏览器 | 需要安装客户端

客户端维护 | 客户端需要单独维护升级

与操作系统品台的关系最小化 | 对客户端操作系统有限制

在响应速度和安全性上需要花费更多设计成本 | 能充分发挥客户端处理能力,客户端响应快

### B/S的工作原理:

B/S架构采用请求/响应模式进行交互

### URL:

uniform Resource Locator统一资源定位符;

Url组成:

 http://localhost:8080/news/index.html

 协议+主机IP地址:端口号+项目资源地址

 域名:端口号;

### Tomcat服务器:

 Apache Jakarta的开源项目

 JSP/Servlet容器 web容器

### 修改端口号:

打开tomcat的conf文件夹找到sever.xml文件,修改Connector port端口号;

### 语法:

~~~java
<%%>小脚本

<%@%>指令

<%!%>声明 也可以用 <jsp:declaration>代码片段</jsp:declaration>代替

<%=%>表达式 等价XML语句:<jsp:expression>代码片段</expression>

<%----%> 注释;
~~~



### JSP指令:

| 指令            | 描述                                                 |
| --------------- | ---------------------------------------------------- |
| <%@paga …%>     | 定义页面的依赖属性,比如脚本语言,error页面,缓存需求等 |
| <%@ include …%> | 包含其他文件                                         |
| <%@ taglib …%>  | 引入标签库的定义,也可以是自定义标签                  |

### JSP行为:

| **语法**        | **描述**                                                   |
| --------------- | ---------------------------------------------------------- |
| jsp:include     | 用于在当前页面中包含静态或动态资源                         |
| jsp:useBean     | 寻找和初始化一个JavaBean组件                               |
| jsp:setProperty | 设置 JavaBean组件的值                                      |
| jsp:getProperty | 将 JavaBean组件的值插入到 output中                         |
| jsp:forward     | 从一个JSP文件向另一个文件传递一个包含用户请求的request对象 |
| jsp:plugin      | 用于在生成的HTML页面中包含Applet和JavaBean对象             |
| jsp:element     | 动态创建一个XML元素                                        |
| jsp:attribute   | 定义动态创建的XML元素的属性                                |
| jsp:body        | 定义动态创建的XML元素的主体                                |
| jsp:text        | 用于封装模板数据                                           |

### JSP隐式对象:

| **对象**    | **描述**                                                     |
| ----------- | ------------------------------------------------------------ |
| request     | **HttpServletRequest**类的实例                               |
| response    | **HttpServletResponse**类的实例                              |
| out         | **PrintWriter**类的实例，用于把结果输出至网页上              |
| session     | **HttpSession**类的实例                                      |
| application | **ServletContext**类的实例，与应用上下文有关                 |
| config      | **ServletConfig**类的实例                                    |
| pageContext | **PageContext**类的实例，提供对JSP页面所有对象以及命名空间的访问 |
| page        | 类似于Java类中的this关键字                                   |
| Exception   | **Exception**类的对象，代表发生错误的JSP页面中对应的异常对象 |

### 执行原理:

1. #### 用户通过浏览器访问jsp时,tomcat负责把index.jsp转化(翻译)为index_jsp.java

   1. jsp声明–>java成员方法
   2. jsp小脚本–>Java文件中的_jspService()方法中的一段代码
   3. jsp表达式–>java文件中的_jspService()方法中的一段代码, outprint(表达式)
   4. 普通的html–>java文件中的_jspService()方法中的一段代码, outwrite(html代码)

2. #### 服务器再把java文件编译成class问价

3. #### 服务器执行class文件

   1. 对生成的java类实例化
   2. 调用实例化后对象的_jspService()方法 输出html给浏览器

4. #### 浏览器渲染返回后的html数据

注意:当index.jsp未修改时,再次调用,直接取之前编译好的class执行

 当index.jsp修改内容后,再次调用,服务器重新进行翻译–编译–执行

### 关于中文乱码问题:

1. #### JSP页面本身的乱码:

   ```java
   pageEncoding = "UTF-8"
   1
   ```

2. #### 浏览器渲染页面时采用的编码:

   一旦一种编码格式指定,另外一种编码格式不进行指定默认使用指定的编码格式

   ```java
   contentType = "text/html;charset=UTF-8"
   1
   ```

3. #### 服务器(web容器)保存数据时采用的编码(request)[默认编码为ISO-8859-1 不支持中文]

   ##### get方法

   ```java
   request.setCharacterEncoding("UTF-8")
   1
   ```

   ##### post-1方法

   ```java
   String name = request.getParamter("name")
   name = new String(name.getBytes("ISO-8859-1"),"UTF-8")
   12
   ```

   ##### post-2方法

   修改tomcat 中 server.xml中connector…最后添加 URIEncodign=“UTF-8”/>

### 转发和重定向:

1. #### 转发

   - 在服务器端发挥作用,将同一个请求在服务器之间进行传递

   - 地址栏不会转向跳转地址

   - request中的参数不会丢失,跳转地址也可以访问到

     request.getRequestDispatcher(“welcome.jsp”).forward(request,response);

2. #### 重定向

   - 在客户端重新发送新的请求,跳转页面

   - 地址栏显示跳转地址

   - request参数丢失,跳转地址无法访问

     response.sendRedirect(“welcome.jsp”);

### session会话:

#### 会话是个啥?

同一个浏览器在一段时间(从第一个请求到session对象失效)内与web服务器一连串相关的交互过程

过程:

#### session与窗口的关系:

1. 访问统一项目,不同浏览器的session对象不同,同一浏览器的不同窗口session对象相同
2. 通过超链接打开子窗体时,子窗体和父窗体的session相同

#### 常用的方法

| 方法名称                                   | 说明                                           |
| ------------------------------------------ | ---------------------------------------------- |
| String getId()                             | 获取session id                                 |
| void setMaxInactiveInterval(int interval)  | 设定session的非活动时间                        |
| int getMaxInactiveInterval                 | 获取session的有效非活动时间,以秒为单位         |
| void setAttribute(String key,Object value) | 以键值对(key/value)的形式将对象保存到session中 |
| void invalidate()                          | 设置session对象失效                            |
| Object getAttribute(String key)            | 通过key获取session中保存的对象                 |
| void removeAttribute(String key)           | 从session中删除指定的key对应的对象             |

#### 关于web.xml问题:

项目A-web.xml tomcat-web.xml

tomcat-web.xml + 项目A-web.xml ==>合并—>对项目A有效

1. 如果有重复的选项,遵守的是项目A的web.xml
2. tomcat web.xml是全局配置 项目A 的只针对自己
3. 各项目之间,web.xml互不影响

#### session对象的失效

1. 手动失效 --session立即失效

   ```java
   <%
       session.invalidate();
   %>
   123
   ```

2. 超时失效 以秒为单位:

   ```java
   <%
       session.setMaxInactiveInterval(10);
   %>
   123
   ```

3. 在tomcat中或者项目的web.xml中配置–以分钟为单位

   ```xml
   <session-config>
       <session-timeout>1</session-timeout>
   </session-config>
   123
   ```

### include指令

页面A 引入页面B 的代码:

原理:a.jsp + b.jsp ==> c.jsp(未形成文件) —>翻译成c_jsp.java —>编译成.class -->执行

(先合并后执行)

```java
<%@include file="loginControl.jsp"%>`//page page
1
```

pageContext的include方法的原理:

b.jsp–>b.java–>编译运行–>结果+a.jsp—>a.java—>c.java(先执行后合并)

```java
<%
    pageContext.include("loginControl.jsp");	//page null
%>
123
```

### application对象

作用:实现用户之间的数据共享

存储时间:从项目启动到项目关闭

作用域:对应整个项目上下文

常用方法:

| 方法名                       | 说明               |
| ---------------------------- | ------------------ |
| String getRealPath           | 获取项目的实际路径 |
| void setAttribute(key,value) | 以键值对保存对象值 |
| Object getAttribute(key)     | 根据Key获取对象值  |

### 四大作用域:

作用域规定的是变量的有效期限和范围

1. page:pageContext 当前界面有效
2. request: request,当前请求(转发)经过的界面有效
3. session: session,当前会话有效(从浏览器打开–>浏览器关闭)
4. application :application 当前应用有效(从应用打开到应用关闭)

### cookie和session对比

| session                | cookie               |
| ---------------------- | -------------------- |
| 在服务器端保存用户信息 | 在客户端保存用户信息 |
| 保存数据是Object       | 保存的数据是String   |
| 随着会话的结束而销毁   | 可以长期保存在客户端 |
| 可以保存重要信息       | 保存不重要的用户信息 |