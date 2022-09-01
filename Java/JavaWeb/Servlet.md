# Servlet的生命周期

1. init只执行一次
2. service执行多次
3. detroy只执行一次

不建议程序员手动编写构造方法

​	因为手写有参构造可能会遗漏无参构造，导致Servlet无法实例化出错

WEB应用的流程

1. 在服务器启动后，首先查找配置文件，建立路径（写入HashMap）。
2. 在用户第一次请求时，根据路径通过反射创建对象调用方法
3. 创建对象 执行init方法，调用service方法
4. 在以后的请求中直接调用service方法
5. 服务器关闭时调用destroy方法

# Servlet的适配器设计模式

Servlet接口的方法仅service方法较为常用，init、destroy不常使用。而实现接口需要实现所有方法，对于代码来说是冗余的，这就类似于将220V的电压直接给手机充电，而没有经过电源适配器。

通过一个抽象类实现接口，再实现所有方法，将service方法改为abstract抽象方法。在实现service方法时，只需继承此抽象类，重写service方法即可。抽象类就是一个适配器。

在继承抽象类且不重写init方法时，Tomcat仍会调用，而此时由于子类没有init方法，故Tomcat会调用抽象类中的init方法。

init方法中有ServletConfig参数，为了取到此参数，在抽象类中创建成员变量，通过init方法对其赋值，子类即可通过get方法获取。

# Servlet的模板设计模式

为了解决Servlet的代码冗余问题，我们引入了适配器设计模式。为了取到init方法的参数，我们通过成员变量和get方法进行传递。

随之而来的问题是，由于通过init方法传递参数，我们必须使init不可被继承，即使用final修饰。而有时我们需要重写init方法，然而被final修饰的方法无法被继承，可我们又要取参数。

模板设计模式可轻松解决此问题，我们再写一个空参的不被final修饰的init方法，通过final修饰的init方法调用该方法即可。

幸运的是，Servlet的适配器设计模式及Servlet的模板设计模式都已由Web服务器设计者实现，JavaWeb程序员无需再担心这些问题。Web服务器提供的类为GenericServlet，在JavaWeb中，我们直接继承此类，重写抽象方法service即可。另外GenericServlet还提供了一些更有利于我们的方法，将在**ServletConfig**章节展现。

# ServletConfig

1. 什么是ServletConfig

   包名javax.servlet.ServletConfig

   显然，ServletConfig是Servlet规范的一员

   ServletConfig是一个接口

2. 谁去实现了这个接口：WEB服务器实现了

   ```java
   ServletConfig config = getServletConfig();
   writer.println(config);
   ```

   > org.apache.catalina.core.StandardWrapperFacade implements ServletConfig
   >
   > 结论：Tomcat服务器实现了ServletConfig接口

3. 一个Servlet对象中有一个ServletConfig对象

   Servlet对象和ServletConfig对象是一对一

4. 谁什么时候创建了ServletConfig

   > WEB服务器在调用init方法前先创建ServletConfig对象

   WEB服务器调用init(ServletConfig servletConfig)时需要创建ServletConfig

5. ServletConfig有什么用

   ServletConfig：Servlet对象配置信息

6. ServletConfig包装了什么信息

   ​	web.xml中的<servlet>标签内容

   ```xml
   <servlet>
       <servlet-name>servletConfig</servlet-name>
       <servlet-class>com.zyhwjl.javaweb.servlet.ConfigTestServlet</servlet-class>
   </servlet>
   ```

7. ServletConfig接口中有哪些方法

   ```xml
   <servlet>
       <servlet-name>servletConfig</servlet-name>
       <servlet-class>com.zyhwjl.javaweb.servlet.ConfigTestServlet</servlet-class>
       <init-param>
           <param-name>driver</param-name>
           <param-value>com.mysql.cj.jdbc.Driver</param-value>
       </init-param>
       <init-param>
           <param-name>url</param-name>
           <param-value>jdbc:mysql://localhost:1521/zzdb</param-value>
       </init-param>
   </servlet>
   ```

   以上<init-param>是初始化参数，这个参数信息会被Web服务器自动封装到ServletConfig对象中。

   通过ServletConfig对象的两个方法可以获取到web.xml中的初始化参数配置信息。

   ```java
   ServletConfig config = this.getServletConfig();
   Enumeration<String> initParameterNames = config.getInitParameterNames();
   while(initParameterNames.hasMoreElements()){
       String name = initParameterNames.nextElement();
       String value = config.getInitParameter(name);
   }
   ```

   而GenericServlet实现了ServletConfig，故可直接通过this调用方法

   ```java
   Enumeration<String> initParameterNames = this.getInitParameterNames();
   while(initParameterNames.hasMoreElements()){
       String name = initParameterNames.nextElement();
       String value = this.getInitParameter(name);
   }
   ```


# ServletContext

获取ServltContext

```java
ServletContext servletContext = config.getServletContext();
ServletContext servletContext1 = this.getServletContext();//继承了GenericServlet的方法
```

1. 什么是ServletContext

   ServletContext是一个接口，是Servlet规范的一员

2. ServletContext是谁实现的

   org.apache.catalina.core.ApplicationContextFacade implents ServletContext

3. ServletContext是谁在什么时候创建的

   Web服务器在启动时由Web服务器创建

   对于一个WebApp来说，ServletContext对象只有一个（一个Web服务器可以有多个WebApp），所有的ServletConfig都放在ServletContext中

   在服务器关闭时销毁

4. ServletContext怎么理解

   Context：Servlet的上下文对象/环境对象

   ServletContext对象对应的就是整个web.xml文件

   放在ServletContext对象中的数据，所有Servlet一定是共享的

5. 获取初始化ServletContext配置信息

   ```xml
   web.xml
   
   <!--    ServletContext配置信息-->
   <context-param>
       <param-name>pageSize</param-name>
       <param-value>10</param-value>
   </context-param>
   <context-param>
       <param-name>pageNumber</param-name>
       <param-value>1</param-value>
   </context-param>
   ```

   ```java
   Enumeration<String> initParameterNames = servletContext.getInitParameterNames();
   while(initParameterNames.hasMoreElements()){
       String initParameter = servletContext.getInitParameter(initParameterNames.nextElement());
       writer.println("<br>");
       writer.println(initParameter);
   }
   ```

6. 什么时候配置在<context-param>中，什么时候配置在<servlet><init-param>中

   全局配置（应用级配置），Servlet配置

   项目全局使用则全局配置，仅在Servlet中使用则写在<init-param>中

7. 一些其他方法

   ```java
   servletContext.getContextPath();//"/Servlet04"	获取应用路径
   servletContext.getRealPath("/index.html");//"C:\Users\zyhwj\IdeaProjects\JavaWeb\out\artifacts\Servlet04_Web_exploded\index.html"	获取文件的绝对路径 起点为web的根 '/'可以省略
   servletContext.log("你好，这是ServletContext记录的日志，日志路径为/CATALINA_HOME！");
   /*
   在IDEA中，由于可以配置多个Tomcat服务器，故IDEA采用拷贝多个Tomcat服务器的方式实现。具体为将Tomcat服务器复制至项目路径下
   在Tomcat启动时输出该日志信息：Using CATALINA_BASE:   "C:\Users\zyhwj\AppData\Local\JetBrains\IntelliJIdea2022.2\tomcat\0d4086f2-dcad-4797-957d-c240ce576993"
   指明当前Tomcat路径，路径下为Tomcat主要配置信息、日志信息、Web应用等:conf/log/work
   故日志会输出至 CATALINA_BASE/log 下
   日志包含：
   	catalina.2022-08-23.log	catalina控制台信息
   	localhost.2022-08-22.log	应用日志信息（ServletContext.log记录的日志信息）
   	localhost_access_log.2022-08-23.txt	访问日志信息
   servletContext.log记录到 localhost.2022-08-22.log 中
   */
   servletContext.log("未成年！", new RuntimeException("应用不适合未成年访问！"));
   ```

8. 应用域：ServletContext

   如果所有的**用户共享的、很少修改的、数据量很少**的数据，可以放在ServletContext这个应用域中

   为什么是用户共享的？不共享的数据没有意义，只有共享的数据才应该被放在应用域中

   为什么是很少修改的？因为所有用户共享数据，如果涉及修改操作必然会存在线程并发带来的安全问题

   为什么数据量要小？如果数据量较大，太占用堆内存，且这个对象的生命周期较长，这个对象只有在服务器关闭时才会被销毁，大数据量会影响服务器性能

   > **用户共享的、很少修改的、数据量很少**，这样的数据存放在ServletContext这个应用域中，会大大提高效率，因为应用域相当于一个缓存，放在缓存中的数据，下次使用时不需要从数据库再次获取，大大提升执行效率。

   - 存

     ```java
     public void setAttribute(String name,Object value);
     ```

   - 取

     ```java
     public Object getAttribute(String name);
     ```

   - 删

     ```java
     public void removeAttribute(String name);
     ```

9. 我们编写Servlet类时，实际上不会直接继承GenericServlet类

   因为我们是B/S结构，这种系统是基于HTTP超文本传输协议的，在Servlet规范中提供了一个类叫做HttpServlet，他是一个专门为HTTP协议准备的Servlet类。我们编写的Servlet类直接继承HttpServlet（HttpServlet是HTTP协议专用的），使用HttpServlet处理HTTP协议更加便捷。

   他们的继承关系：

   ```
   jakarta.servlet.Servlet
   jakarta.servlet.GenericServlet implements Servlet
   jakarta.servlet.http.HttpServlet extends GenericServlet
   
   我们编写的类应该继承HttpServlet
   ```
   

# Http协议

协议：多方遵循的规范、守则，已确保多方间沟通。在Http中主要作用为B、S间解耦合，两者不相互依赖。Http可应用于任何Web服务器和任何操作系统的任何浏览器间。

