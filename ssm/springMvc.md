

# 一.三层架构





![image-20200807143222669](springMvc/image-20200807143222669.png)





![image-20200908203749771](springMvc/image-20200908203749771.png)







# 二.清晰的角色划分：

前端控制器（DispatcherServlet）
请求到处理器映射（HandlerMapping）
处理器适配器（HandlerAdapter）
视图解析器（ViewResolver）
处理器或页面控制器（Controller）
验证器（ Validator）
命令对象（Command 请求参数绑定到的对象就叫命令对象）
表单对象（Form Object 提供给表单展示和提交到的对象就叫表单对象）。



<img src="springMvc/image-20200911194312635.png" alt="image-20200911194312635" style="zoom:130%;" />



在 SpringMVC 的各个组件中，==处理器映射器==、==处理器适配器==、==视图解析器==称为 SpringMVC 的三大组件。







# 三.文件上传



## 步骤一：导入依赖：

~~~java
<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.4</version>
</dependency>

~~~



## 步骤二：配置CommonsMultipartResolver解析器，

文件最大为100K，解析器的id必须为multipartResolver,否则会报400错误。

~~~JAVA
<!-- 配置上传解析器   -->
<bean id="multipartResolver"
		class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
      <!--  允许上传的最大限制   100K   -->
    <property name="maxUploadSize" value="102400"/>
</bean>

~~~



## 步骤三：配置form的==enctype="multipart/form-data"==

~~~java
<form action="${pageContext.request.contextPath}/file/upload"
      method="post" enctype="multipart/form-data">
    <input type="text" name="name" id="name"/><br>
    <input type="file" name="file" id="file" onchange="show(this)"><br>
    <input type="submit" value="上传"><br>

</form>
~~~



## 步骤四：使用js接收选定的文件名


    <script>
        function show(source) {
    	
        alert( $(source).val());
        
        //将图片的路径分割成数组
        var arrs = $(source).val().split("\\");
        
        //取出图片的名称，设置给name输入框
        var fileName = arrs[arrs.length - 1];
        alert(fileName);
    
        $("#name").val(fileName);
    }
    
    </script>


## 步骤五：编写控制器

==ServletContextAware==

```JAVA
@Controller
@RequestMapping("/file")
public class FileUpload implements ServletContextAware {


    private ServletContext servletContext;

    @GetMapping("/upload")
    public String toFileUpload(){
        System.out.println("跳转上传界面");
        return "fileUpload";
    }

    @PostMapping("/upload")
    public String doUpload(String name, @RequestParam("file") MultipartFile file, 									Model model){
        //name：图片的名称
        //判断file是否为null
        if(!file.isEmpty()){
            //不为空才执行上传
            try {
                //获取文件的字节数组
                byte[] bytes = file.getBytes();
                //创建file，文件上传之后的位置和名称
                File f = new File(servletContext.getRealPath("/upload/") + name);
//                File f = new File("C:\\Users\\admin\\Desktop\\新建文件夹\\ " + name);
                //写入
                FileUtils.writeByteArrayToFile(f,bytes);
                //上传成功
                model.addAttribute("msg", name + "上传成功!");

                System.out.println("上传成功");

            } catch (IOException e) {
                e.printStackTrace();
                //上传失败
                model.addAttribute("msg", name + "上传失败！");

                System.out.println("上传失败");

            }
        }
        //回到上传的页面
        return "fileUpload";
    }

    public void setServletContext(ServletContext servletContext) {
        this.servletContext = servletContext;
    }
}
```





# 四.异常处理





## 异常类

~~~java
@ControllerAdvice
public class GlobalException {

    @ExceptionHandler(Exception.class)
    public String OtherException(Exception e, Model model){
        model.addAttribute("Exception",e.getMessage());
        return "otherException";
    }

    @ExceptionHandler(TimeoutException.class)
    public String timeoutException(TimeoutException te,Model model){
        model.addAttribute("Timeout",te.getMessage());
        return "TimeOut";

    }

}
~~~





## 测试类

```
@Controller
public class ExceptionController {
    @GetMapping("/testException")
    public String testException() throws Exception {
        throw new Exception("亲，网络不好，请检查网络");
    }
}
```









# 五.拦截器



使用httpsession导入的依赖

~~~java
   <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>4.0.1</version>
    </dependency>
    <dependency>
            <groupId>javax.servlet.jsp</groupId>
            <artifactId>jsp-api</artifactId>
            <version>2.2.1-b03</version>
    </dependency>
~~~



## 时间拦截器

~~~java
public class TimeInterceptor extends HandlerInterceptorAdapter {
    private int openTime;
    private int closeTime;

    public TimeInterceptor() {
    }

    public TimeInterceptor(int openTime, int closeTime) {
        this.openTime = openTime;
        this.closeTime = closeTime;
    }

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        openTime = 6;
        closeTime = 23;

        System.out.println("时间拦截器开始生效....");
        Calendar calendar = Calendar.getInstance();
        int i = calendar.get(Calendar.HOUR_OF_DAY);
        if (openTime < i && closeTime > i) {
            return true;
        } else {
            request.getRequestDispatcher("/view/notInWorkTime.jsp").forward(request,response);
            return false;
        }

    }
}

~~~



## 登录拦截器

~~~java
//拦截器
public class loginInterceptor extends HandlerInterceptorAdapter {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse 		    							response, Object handler) throws Exception {
        
        System.out.println("登录拦截器开始生效。。。。。");
        HttpSession session = request.getSession();
        String user = (String) session.getAttribute("user");

        System.out.println("拦截器获取到user值为。。。"+user);

        if (user == null) {
            //还没登录，重定向到登录页面
            response.sendRedirect("http://localhost:8080/springmvc2_war_exploded/toLogin");
            return false;
        } else {
            //已经登录，放行
            return true;
        }

    }

}
~~~



## 配置拦截器



~~~java
  <!--    配置拦截器-->
    <!--    <bean id="loginInterceptor" class="com.zxq.interceptor.loginInterceptor"></bean>-->

    <mvc:interceptors>
        <mvc:interceptor>

            <!-- 拦截所有的请求，这个必须写在前面，也就是写在【不拦截】的上面 -->
            <mvc:mapping path="/**"/>

            <!-- 但是排除下面这些，也就是不拦截请求 -->
            <mvc:exclude-mapping path="/login*"/>
            <mvc:exclude-mapping path="/toLogin"/>
            <mvc:exclude-mapping path="upload"/>
            <mvc:exclude-mapping path="/testException"/>
            <bean class="com.zxq.interceptor.loginInterceptor"/>
        </mvc:interceptor>


        <mvc:interceptor>
            <mvc:mapping path="/**"/>
                
            <mvc:exclude-mapping path="/login*"/>
            <mvc:exclude-mapping path="/toLogin"/>
            <bean class="com.zxq.interceptor.TimeInterceptor"></bean>
        </mvc:interceptor>
    </mvc:interceptors>
~~~





# 六.常用注解



## @RequestParam

 //    @RequestParam注解的参数在URI中必须存在，否则报错
//    如果参数是可选的，需要@RequestParam(name="id",required=false)
//    在@RequestParam中还可以设置默认值
//    如果在地址栏输入http://localhost:8080/HelloSpringMVC/hello,
    // 没有输入username参数，就会为name赋值默认值xxxxx。

```java
@Controller
public class AnnoTestController {

    @RequestMapping("/requestParamTest")
    public String requestParamTest(@RequestParam(name = "username",
                                        defaultValue = "xxxx") String name) {
        System.out.println(name);
        return "loginSuccess";
    }
```



## @RequestMapping

中的value，path，method属性

==GET==请求通常用于==页面转向==，而==POST==请求用于==表单数据提交==。

如果在@RequestMapping中没有指定http的方法，即默认映射所有http方法。

```java
//params表示参数中必须有name
    @RequestMapping(value = "/hello" ,method = {RequestMethod.GET},params = "name")
    public String sayHello(String name, Model model,HttpServletRequest httpServletRequest) {


        System.out.println("name=" + name);

//        底层存储到request域中
        model.addAttribute("name", name);

        return "hello";
    }
```





## @PathVariable

从URL中提取变量

使用@PathVariable注解，可以创建RESTFul风格的web站点。

@PathVariable 解析使用URI模板

~~~JAVA
  //     restful风格
    @GetMapping("test/{name}/{password}")
    public String test(@PathVariable("name") String username,
                       @PathVariable("password") String password) {

        System.out.println(username + "...." + password);

        return "loginSuccess";

    }
~~~



## @GetMapping与@PostMapping



~~~java
Spring Framework 4.3之后，可以使用如下注解，简化@RequestMapping的写法：
@GetMapping
@PostMapping
@PutMapping
@DeleteMapping
@PatchMapping
~~~



~~~java
@RequestMapping("/user")
public class UserAction {

    //该login方法用于转发到登录页面,只用于get形式的提交
    @GetMapping("/login")
    public String login(){
        return "login";
    }
}
~~~



## @RequestBody

与消息转换器



~~~java
@RequestBody注解，指明方法参数与http的请求体数据绑定。
之前示例，@RequestBody用来接收前端传递给后端的json字符串，数据不是源于http的参数，而是源于http的请求体。
转换http请求体到方法参数，需要使用HttpMessageConverter。消息转换器还负责把方法参数转换为http回应体(response body)

~~~

~~~java
@PostMapping("/login")
@ResponseBody
public String login(String username,String password){
    System.out.println(username + "---" + password);
    //模拟登录
    User user = new User(username, password, "13852480369");
    //返回登录的结果
    Map<String,Object> map = new HashMap<String,Object>();
    map.put("username",user.getUsername());
    map.put("password",user.getPassword());
    map.put("phone",user.getPhone());
    JSONObject jsonObject = new JSONObject(map);
    return jsonObject.toJSONString();
}
}
~~~







## @RestController

如果你的控制器，只服务于JSON、XML、多媒体数据等，是REST风格，则通常使用@RestController注解。

@RestController相当于控制器中所有@RequestMapping方法，都使用了@ResponseBody注解。@RestController是一种简化写法。

==@RestController = @ResponseBody + @Controller==

在微服务开发中，所有的对外服务，一般都使用@RestController

~~~java
@Controller
public class HelloAction {

    @RequestMapping("/say")
    @ResponseBody
    public String say(){
        return "say hello to SpringMVC";
    }
}
~~~





## @ControllerAdvice

​	 定义全局异常解析器

~~~java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(Exception.class)
    public ModelAndView otherException(Exception e){
        ModelAndView mv = new ModelAndView();
        //设置数据存入model
        mv.addObject("msg",e.getMessage());
        //设置转发的视图的名称
        mv.setViewName("error");
        return mv;
    }

    //该方法能够捕获的异常的类型
    @ExceptionHandler(MaxUploadSizeExceededException.class)
    public ModelAndView maxUploadSizeExceededException(MaxUploadSizeExceededException e){
        ModelAndView mv = new ModelAndView();
        //设置数据存入model
        mv.addObject("msg","上传的文件大小超出限制");
        //设置转发的视图的名称
        mv.setViewName("error");
        return mv;
    }
}

~~~





























































































# 七.过滤器，防止乱码



## 1.客户端到服务端

web.xml

~~~java
<!--防乱码过滤器  客户端向服务端-->

    <filter>
        <filter-name>CharacterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>

        <!-- 设置过滤器中的属性值 -->
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>

        <!-- 启动过滤器 -->
        <init-param>
            <param-name>forceEncoding</param-name>
            <param-value>true</param-value>
        </init-param>

    </filter>


    <filter-mapping>
        <filter-name>CharacterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
~~~





## 2.服务端到客户端



spring-mvc.xml

~~~java
 <!--    不接收静态资源-->
    <mvc:default-servlet-handler/>

    <!--    开启注解-->
    <mvc:annotation-driven conversion-service="conversionService">

        <!--  配置消息转换器，设置String的编码    服务器向客户端  -->
        <mvc:message-converters>
            <bean class="org.springframework.http.converter.StringHttpMessageConverter">
                <constructor-arg value="UTF-8"></constructor-arg>
                <property name="writeAcceptCharset" value="false"></property>
            </bean>
        </mvc:message-converters>

    </mvc:annotation-driven>
~~~





# 八.转发与重定向



## 转发

~~~java
//    转发
    @RequestMapping("/testForward")
    public String testForward() {
        System.out.println("AccountController 的 testForward 方法执行了。。。。");
        return "forward:/view/hello.jsp";
    }
~~~



```
//    需要注意的是，如果用了 formward：则路径必须写成实际视图 url，不能写逻辑视图。
//    它相当于“request.getRequestDispatcher("url").forward(request,response)”。使用请求
//    转发，既可以转发到 jsp，也可以转发到其他的控制器方法。
```





## 重定向



使用==redirect==方式的好处：

防止客户端提交的数据，在多个控制器间共享（内部forward，意味着POST数据，可以在其它控制间共享，有很大的安全隐患）

防止表单数据，重复提交

~~~java
    /**
     * 重定向
     * @return
     */
    @RequestMapping("/testRedirect")
    public String testRedirect() {
        System.out.println("AccountController 的 testRedirect 方法执行了。。。。");
        return "redirect:testReturnModelAndView";
    }
~~~



~~~java
  return "redirect:"+ basePath +"/user/main";
~~~



```
    //    contrller 方法提供了一个 String 类型返回值之后，它需要在返回值里使用:redirect:
//    它相当于“response.sendRedirect(url)”。需要注意的是，如果是重定向到 jsp 页面，则 jsp 页面不
//    能写在 WEB-INF 目录中，否则无法找到。
```







~~~java
@Controller
@RequestMapping("/user")
public class UserAction {

    @GetMapping("/login")
    public View login(HttpServletRequest request){
        //重定向到action的方法中，给的参数是action方法的路径
//        return new RedirectView("main.do");
//        return new RedirectView(request.getContextPath()+"/user/main.do");
        String basePath = request.getScheme()+"://" + request.getServerName() +":" +request.getServerPort()+ request.getContextPath();
        System.out.println(basePath);
        return new RedirectView(basePath + "/user/main.do");
    }

    @RequestMapping("/main")
    public String main(){
        System.out.println("-----main----");
        //返回视图的名称
        return "main";
    }
}

~~~





# 九.文件下载

## 1.基于ResponseEntity实现

~~~java
 @RequestMapping("/testHttpMessageDown1")
    public ResponseEntity<byte[]> download(HttpServletRequest request) throws IOException {

        File file = new File("C:\\Users\\admin\\Desktop\\新建文本文档.txt");
        byte[] body = null;
        InputStream is = new FileInputStream(file);
        int count=0;
        if (count==0){
            count=is.available();
        }
        body = new byte[count];
        is.read(body);
        
        HttpHeaders headers = new HttpHeaders();
        headers.add("Content-Disposition", "attchement;filename=" + file.getName());
        HttpStatus statusCode = HttpStatus.OK;
        ResponseEntity<byte[]> entity = new ResponseEntity<byte[]>(body, headers, statusCode);
        return entity;
    }
~~~





## 2.Java通用下载实现

~~~java
 @RequestMapping("/testHttpMessageDown2")
    public static void download(String fileName, String filePath,
                                HttpServletRequest request, HttpServletResponse response)
            throws Exception {
        //声明本次下载状态的记录对象
        DownloadRecord downloadRecord = new DownloadRecord(fileName, filePath, request);
        //设置响应头和客户端保存文件名
        response.setCharacterEncoding("utf-8");
        response.setContentType("multipart/form-data");
        response.setHeader("Content-Disposition", "attachment;fileName=" + fileName);
        //用于记录以完成的下载的数据量，单位是byte
        long downloadedLength = 0l;
        try {
            //打开本地文件流
            InputStream inputStream = new FileInputStream(filePath);
            //激活下载操作
            OutputStream os = response.getOutputStream();

            //循环写入输出流
            byte[] b = new byte[2048];
            int length;
            while ((length = inputStream.read(b)) > 0) {
                os.write(b, 0, length);
                downloadedLength += b.length;
            }

            // 这里主要关闭。
            os.close();
            inputStream.close();
        } catch (Exception e) {
            downloadRecord.setStatus(DownloadRecord.STATUS_ERROR);
            throw e;
        }
        downloadRecord.setStatus(DownloadRecord.STATUS_SUCCESS);
        downloadRecord.setEndTime(new Timestamp(System.currentTimeMillis()));
        downloadRecord.setLength(downloadedLength);
        //存储记录
    }
~~~

**说明**

要一次读取多个字节时，经常用到==InputStream.available()==方法，这个方法可以在读写操作前先得知数据流里有多少个字节可以读取。需要注意的是，如果这个方法用在从本
地文件读取数据时，一般不会遇到问题，但如果是用于网络操作，就经常会遇到一些麻烦。比如，Socket通讯时，对方明明发来了1000个字节，但是自己的程序调用available()方法却只得到900，或者100，甚至是0，感觉有点莫名其妙，怎么也找不到原因。其实，这是因为网络通讯往往是间断性的，一串字节往往分几批进行发送。本地程序调用available()方法有时得到0，这可能是对方还没有响应，也可能是对方已经响应了，但是数据还没有送达本地。对方发送了1000个字节给你，也许分成3批到达，这你就要调用3次available()方法才能将数据总数全部得到。
   如果这样写代码：

~~~java
 int count = in.available();
 byte[] b = new byte[count];
 in.read(b);
~~~

   在进行网络操作时往往出错，因为你调用available()方法时，对发发送的数据可能还没有到达，你得到的count是0。
     需要改成这样：

~~~java
 int count = 0;
 while (count == 0) {
  count = in.available();
 }
 byte[] b = new byte[count];
 in.read(b);
~~~







## 3.**下载记录信息实体**

~~~java
public class DownloadRecord {
    public static final int STATUS_ERROR = 0;
    public static final int STATUS_SUCCESS = 1;
    private String uid;
    private String ip;
    private int port;
    private String ua;
    private String fileName;
    private String filePath;
    private long length;
    private int status;
    private Timestamp startTime;
    private Timestamp endTime;

    public DownloadRecord() {
    }

    public DownloadRecord(String fileName, String filePath,
                          HttpServletRequest request) {
        this.uid = UUID.randomUUID().toString().replace("-","");
        this.fileName = fileName;
        this.filePath = filePath;
        this.ip = request.getRemoteAddr();
        this.port = request.getRemotePort();
        this.ua = this.ua = request.getHeader("user-agent");
        this.startTime = new Timestamp(System.currentTimeMillis());
    }
~~~



## 比较

1、 基于ResponseEntity的实现的局限性还是很大，从代码中可以看出这种下载方式是一种一次性读取的下载方式，在文件较大的时候会直接抛出内存溢出（我自己亲测一个1.8G的文件在执行下载操作的时候直接抛出了内存溢出）。还有就是这种方式在进行下载统计的时候也存在局限性，无法统计在下载失败的情况已完成下载量，因此限制了对下载的功能扩展。虽然这种实现方式有局限性，但是也有着优点——简洁。在很多时候我们并不需要那么复杂的下载功能时，这种实现就应该是首选了。
2、 然而下载java通用实现在功能上比第一种实现更加丰富，对下载的文件大小无限制（循环读取一定量的字节写入到输出流中，因此不会造成内存溢出，但是在下载人数过多的时候应该还是出现一些异常，不过下载量较大的文件一般都会使用ftp服务器来做吧），另外因为是这种实现方式是基于循环写入的方式进行下载，在每次将字节块写入到输出流中的时都会进行输出流的合法性检测，在因为用户取消或者网络原因造成socket断开的时候，系统会抛出SocketWriteException，系统可以捕捉这个过程中抛出的异常，当捕捉到异常的时候我们可以记录当前已经传输的数据量，这样就可以完成下载状态和对应状态下载量和速度之类的数据记录。另外这种方式实现方式还可以实现一种断点续载的功能。

