##spring boot
	启动原理
		EnableAutoConfiguration
		spring.factories
		@Conditional
		找到自动配置文件：自动扫描 | @EnableXXX
		默认大于配置
	注解
		@Enable注解：开启指定功能
		@ConfigurationProperties@value：属性映射
		@Conditional：条件注解
		@ComponentScan：默认扫描启动类所在包
		@configuration@bean
		@Import@ImportResource
		@SpringBootApplication
		@Profile:环境隔离
		@EnableConfigureProperty
	mvc
		监听器
		拦截器
		全局异常
		异步任务
		mvc配置
			继承WebMvcConfigurerAdapter -- 配置拦截器、类型转换器、视图解析
			继承WebApplicationInitializer -- 代替web.xml 配置servlet，过滤器，监听器
	组件
		数据访问
		缓存
		消息
		调度
	其他
		测试
		监控
		热部署


项目初始化
http://start.spring.io/

@SpringBootApplication
@configuration @EnabeAutoConfiguration @ComponentScan
＠RestController
@Import @Bean

＠RestController是spring4.0中的注解：RestController表明该类的每个方法返回对象而不是视图，以字符串的形式渲染结果，并直接返回给调用者，
它实际就是@Controller和@ResponseBody混合使用的简写方法。
@EnableAutoConfiguration注解是类级别的注解，它的意思就是开启自动配置，该注解会告知Boot要采用一种特定的方式来对应用进行配置。
这种方法会假设你开发的是一个web应用程序并进行自动配置,例如根据spring-boot-starter-web依赖，tomcat与spring MVC都自动依赖进来了
@EnableAutoConfiguration(exclude={DataSourceAutoConfiguration.class})  排除不想要的自动配置类

@Configuration注解该类，等价 与XML中配置beans；用@Bean标注方法等价于XML中配置bean
@Configuration 
相当于传统的xml配置文件，如果有些第三方库需要用到xml文件，建议仍然通过@Configuration类作为项目的配置主类——可以使用@ImportResource注解加载xml配置文件。
@Import 
用来导入其他配置类。
@ComponentScan 注解自动收集所有的Spring组件，包括 @Configuration 类。@service @Repository 

@SpringBootApplication 注解等价于以默认属性使用 @Configuration ， @EnableAutoConfiguration 和 @ComponentScan 。


@Profiles提供了一种隔离应用程序配置的方式，并让这些配置只能在特定的环境下生效。任何
Spring Profiles提供了一种隔离应用程序配置的方式，并让这些配置只能在特定的环境下生效。任何@Component或@Configuration都能被@Profile标记，从而限制加载它的时机

@ResponseBody
表示该方法的返回结果直接写入HTTP response body中
一般在异步获取数据时使用，在使用@RequestMapping后，返回值通常解析为跳转路径，加上@responsebody后返回结果不会被解析为跳转路径，而是直接写入HTTP response body中。比如
异步获取json数据，加上@responsebody后，会直接返回json数据。

@Component：
泛指组件，当组件不好归类的时候，我们可以使用这个注解进行标注。一般公共的方法我会用上这个注解

@AutoWired
byType方式。把配置好的Bean拿来用，完成属性、方法的组装，它可以对类成员变量、方法及构
造函数进行标注，完成自动装配的工作。
当加上（required=false）时，就算找不到bean也不报错。

@RequestParam  String a   ==》String a =request.getParameter("a")
@PathVariable  @RequestMapping("/list/{applyId}") ==》  public String list(@PathVariable Long applyId)

全局处理异常的：
@ControllerAdvice：
包含@Component。可以被扫描到。
统一处理异常。

@ExceptionHandler（Exception.class）：
用在方法上面表示遇到这个异常就执行以下方法。

@value注解来读取application.properties里面的配置

@propertySource 引入外部属性文件
@configurationProperties(prefix="")  组合注入属性文件，等同于多个@values("{}")
===============================================================
启动
#mvn spring-boot:run

#jar包
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
<properties>
    <java.version>1.8</java.version>
</properties>
 
mvn package
java -jar target/hello_world-0.0.1-SNAPSHOT.jar
查看jar包内容jar tvf target/hello_world-0.0.1-SNAPSHOT.jar

#war包

=============================================================
#热部署
devtools：是boot的一个热部署工具，当我们修改了classpath下的文件（包括类文件、属性文件、页面等）时，会重新启动应用（由于其采用的双类加载器机制，这个启动会非常快）
双类加载器机制：boot使用了两个类加载器来实现重启（restart）机制：base类加载器（简称bc）+restart类加载器（简称rc）。
bc：用于加载不会改变的jar（eg.第三方依赖的jar）
rc：用于加载我们正在开发的jar（eg.整个项目里我们自己编写的类）。
当应用重启后，原先的rc被丢掉、重新new一个rc来加载这些修改过的东西，而bc却不需要动一下。这就是devtools重启速度快的原因。

thymeleaf：boot推荐的模板引擎,页面热部署

pom.xml
<!-- thymeleaf -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
<!-- 
    devtools可以实现页面热部署（即页面修改后会立即生效，这个可以直接在application.properties文件中配置spring.thymeleaf.cache=false来实现），
    实现类文件热部署（类文件修改后不会立即生效），实现对属性文件的热部署。
    即devtools会监听classpath下的文件变动，并且会立即重启应用（发生在保存时机），注意：因为其采用的虚拟机机制，该项重启是很快的
 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional><!-- optional=true,依赖不会传递，该项目依赖devtools；之后依赖myboot项目的项目如果想要使用devtools，需要重新引入 -->
</dependency>


devtools
在我们的eclipse中还不起作用，这时候还需要对之前添加的spring-boot-maven-plugin做一些修改，如下：
<!-- 用于将应用打成可直接运行的jar（该jar就是用于生产环境中的jar） 值得注意的是，如果没有引用spring-boot-starter-parent做parent，且采用了上述的第二种方式，这里也要做出相应的改动 -->
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <fork>true</fork><!-- 如果没有该项配置，肯呢个devtools不会起作用，即应用不会restart -->
    </configuration>
</plugin>

thymeleaf 
application.rpoeprties
spring.thymeleaf.cache=false

默认情况下，/META-INF/maven，/META-INF/resources，/resources，/static，/templates，/public这些文件夹下的文件修改不会使应用重启，但是会重新加载（devtools内嵌了一个LiveReload server，当资源发生改变时，浏览器刷新）。
如果想改变默认的设置，可以自己设置不重启的目录：spring.devtools.restart.exclude=static/**,public/**，这样的话，就只有这两个目录下的文件修改不会导致restart操作了。
如果要在保留默认设置的基础上还要添加其他的排除目录：spring.devtools.restart.additional-exclude
如果想要使得当非classpath下的文件发生变化时应用得以重启，使用：spring.devtools.restart.additional-paths，这样devtools就会将该目录列入了监听范围。
=============================================================
文件目录
src/main/resources/templates：页面存放目录
src/main/resources/static：方式静态文件（css、js等）
src/main/resources/ 配置文件

src/main/java：类文件
src/test/java 测试类文件

target/classes : .classpath 配置编译输出 默认为src/main/resources、src/main/java
target/test-classes : .classpath 配置编译输出 默认为src/test/java
=============================================================
######属性配置
1.配置文件
src/main/resource/
默认配置文件application.properties application.yml
自定义配置文件 test.properties 通过@PropertiesSource(classpath:test.properties) 引入
2.命令行
java -jar xxx.jar --server.port=8888

#######自定义属性
com.didispace.blog.name=程序猿DD  
com.didispace.blog.title=Spring Boot教程  
com.didispace.blog.desc=${com.didispace.blog.name}正在努力写《${com.didispace.blog.title}》   参数间引用

######覆盖系统默认属性
https://my.oschina.net/liuyuantao/blog/806710

######多环境配置
选择配置属性环境
多环境配置文件名需要满足`application-{profile}.properties`的格式，其中`{profile}`对应你的环境标识
- application-dev.properties：开发环境
- application-prod.properties：生产环境
application.properties
spring.profiles.active=dev


选择类环境
会选择注解在@profile上的类
@Profile("dev")
public Class test1 implement test
@Profile("prod")
public Class test2 implement test
application.properties
spring.profiles.active=dev

除了spring.profiles.active来激活一个或者多个profile之外，还可以用spring.profiles.include来叠加profile
spring.profiles.include=test1,test2

#######获取属性
@Value("${com.didispace.blog.name}") 字段获取

configurationProperties(prefix="com.didispace.blog") 类获取
public Class A{
	private String name;
	private String title;
}


#######属性注入顺序（逐层覆盖）
1.根目录下的开发工具全局设置属性(当开发工具激活时为`~/.spring-boot-devtools.properties`)。

2.测试中的@TestPropertySource注解。

3.测试中的@SpringBootTest#properties注解特性。

4.命令行参数

5.SPRING_APPLICATION_JSON中的属性(环境变量或系统属性中的内联JSON嵌入)。

6.ServletConfig初始化参数。

7.ServletContext初始化参数。

8.java:comp/env里的JNDI属性

9.JVM系统属性

10.操作系统环境变量

11.随机生成的带random.* 前缀的属性（在设置其他属性时，可以应用他们，比如${random.long}）

12.应用程序以外的application.properties或者appliaction.yml文件

13.打包在应用程序内的application.properties或者appliaction.yml文件

14.通过@PropertySource标注的属性源

15.默认属性(通过`SpringApplication.setDefaultProperties`指定).


==============
===============================================
http://spring4all.com/

spring-cloud-starter-parent 具备spring-boot-starter-parent同样功能并附加Spring Cloud的依赖  
spring-cloud-starter-config 配置中心客户端依赖
spring-cloud-config-server 配置中心服务端依赖
spring-cloud-starter-eureka-server 注册中心的Eureka Server依赖  
spring-cloud-starter-eureka 注册中心的Eureka客户端依赖  
spring-cloud-starter-hystrix／zuul／feign／ribbon 断路器（Hystrix），智能路有（Zuul），客户端负载均衡（Ribbon）的依赖  

spring-boot-starter-parent 
spring-boot-starter  //Spring Boot starter的核心，包括自动配置的支持, logging 和 yml配置
spring-boot-starter-actuator //为应用添加了管理特性
spring-boot-starter-aop  //面向切面编程的支持，包括spring-aop和AspectJ
spring-boot-starter-jdbc  //jdbc数据库的支持
spring-boot-starter-mail  //对javax.mail的支持
spring-boot-starter-redis //对redis的支持，包括spring-redis
spring-boot-starter-security   //对spring-security的支持
spring-boot-starter-test  //常见的测试依赖，包括JUnit, Hamcrest， Mockito 和 spring-test 模块
spring-boot-starter-thymeleaf   //对渲染模板引擎的支持
spring-boot-starter-web  web依赖
spring-boot-devtools 热部署
=====================================================
spring 全局异常	 
spring ioc
spring aop  
spring 拦截器
spring boot
spring data
spring cloud
spring 测试

========================================
spring boot 注解
@ConfigurationProperties和@value都是将外部属性注入到对象，Spring Boot将尝试校验外部的配置，默认使用JSR-303（如果在classpath路径中）。
@EnableConfigurationProperties 如果未开启，则使用@ConfigurationProperties注解的class上面还需要添加@Component，否则配置类无法注入
@ImportResource 注解加载XML配置文件
@import(导入类到上下文环境) 在包扫描没有扫到的情况下使用

@bean 标示方法，将方法的返回类注册到上下文
@Configuration 注册类到上下文，用于标示配置类，等价 与XML中配置beans
@EnableAutoConfiguration 这个注解告诉Spring Boot根据添加的jar依赖猜测你想如何配置，配置哪些Spring组件到spring环境
@ComponentScan 默认扫描启动类所在的包里的对象，扫描作用：搜集配置了注解的类，初始化到spring环境中

@SpringBootApplication
等价于以默认属性使用 @Configuration ， @EnableAutoConfiguration 和 @ComponentScan

@Profiles
限制bean的加载时机，隔离应用

@resetController
相当于@controller和@responseBody
========================================
spring 注解
@autowired
按类型匹配
@resource
默认按名字匹配，可指定按类型匹配

@PostConstruct注释的方法将在类实例化后调用
而标注了 @PreDestroy 的方法将在类销毁之前调用。

@component 标示一个bean到上下文
@Repository、@Service、@Controller 比@component更加细化的，功能相同，对应dao，service，controller

@scope
bean的作用域
singleton 单例模式（默认）
prototype 每次获取新建一个实例
request 每一次HTTP请求都会产生一个新的bean 该bean仅在当前HTTP request内有效
session 每一次HTTP请求都会产生一个新的bean 该bean仅在当前HTTP session内有效
global session
作用域选择：有状态的bean设为prototype,无状态的设为singleton


@ControllerAdvice：
全局处理异常

@ExceptionHandler（Exception.class）：
用在方法上面表示遇到这个异常就执行以下方法


==================================================================
maven 依赖




==================================================================
请求参数的处理
	@RequestMapping
	@GetMappiing @PostMapping

	@PathVariable
	处理url模板中的参数
	@RequestMapping("/pets/{petId}")  
	public void findPet(@PathVariable String ownerId, @PathVariable String petId, Model model) {      
	 
	}

	@RequestHeader, @CookieValue
	处理request header部分：
	@RequestMapping("/displayHeaderInfo.do")  
	public void displayHeaderInfo(@RequestHeader("Accept-Encoding") String encoding,  
	                              @RequestHeader("Keep-Alive") long keepAlive)  {  
	   
	}
	@RequestMapping("/displayHeaderInfo.do")  
	public void displayHeaderInfo(@CookieValue("JSESSIONID") String cookie)  {  
	  
	}   

	@RequestParam
	处理application/x-www-form-urlencoded编码的表单提交（post，get）
	处理url中拼接的参数
	相当于 request.getParameter()

	@RequestBody
	1.取Request请求的body部分数据，使用系统默认配置的HttpMessageConverter进行解析，然后把相应的数据绑定到要返回的对象上；
	2.再把HttpMessageConverter返回的对象数据绑定到 controller中方法的参数上
	application/x-www-form-urlencoded， 可选（即非必须，因为这种情况的数据@RequestParam, @ModelAttribute也可以处理，当然@RequestBody也能处理）；
	multipart/form-data, 不能处理（即使用@RequestBody不能处理这种格式的数据）；
	其他格式， 必须（其他格式包括application/json, application/xml等。这些格式的数据，必须使用@RequestBody来处理）

	@ResponseBody
	该注解用于将Controller的方法返回的对象，通过适当的HttpMessageConverter转换为指定格式后，写入到Response对象的body数据区（json xml）


	不做处理
	1.形参：接受表单get和url中拼接的参数
	2.对象：接受表单get和post

	request
		header
			Host                    localhost:8080  
			Accept                  text/html,application/xhtml+xml,application/xml;q=0.9  
			Accept-Language         fr,en-gb;q=0.7,en;q=0.3  
			Accept-Encoding         gzip,deflate  
			Accept-Charset          ISO-8859-1,utf-8;q=0.7,*;q=0.7  
			Keep-Alive              300  
		body


	Content-Type：指定body编码
		application/x-www-form-urlencoded:默认的form提交编码方式
		multipart/form-data：在有文件上传时form提交编码
		application/json：提交json格式的数据，字符串
		text/xml：提交xml格式的数据，字符串

=======================================================================
返回信息处理
spring mvc返回方式：ModelAndView, Model, ModelMap, Map,View, String, void
	String
		指定返回的视图页面名称，结合设置的返回地址路径加上页面名称后缀即可访问到。
		注意：如果方法声明了注解@ResponseBody ，则会直接将返回值输出到页面。
	View
		可以返回pdf excel等
	ModelAndView
		通过ModelAndView构造方法可以指定返回的页面名称，也可以通过setViewName()方法跳转到指定的页面
	void 
		如果返回值为空，则响应的视图页面对应为访问地址，输出json或xml数据

	1.使用 String 作为请求处理方法的返回值类型是比较通用的方法，这样返回的逻辑视图名不会和请求 URL 绑定，具有很大的灵活性，而模型数据又可以通过 ModelMap 控制。
	2.使用void,map,Model 时，不跳转页面，返回数据，ajax请求
	3.使用String,ModelAndView 跳转页面，指定页面绑定数据

	返回类型
	jsp，使用string，modelView返回
	html，直接使用writer输出或者forward/redirect到HTML页面
	json，PrintWriter是直接输出json字符串或使用@responseBody返回类
	其他格式Velocity, FreeMarker, XML, PDF, Excel

======================================================================
spring mvc流程
1. 用户向服务器发送请求，请求被Spring 前端控制Servelt DispatcherServlet捕获；
2. DispatcherServlet对请求URL进行解析，得到请求资源标识符（URI）。然后根据该URI，调用HandlerMapping获得该Handler配置的所有相关的对象（包括Handler对象以及Handler对象对应的拦截器），最后以HandlerExecutionChain对象的形式返回；
3. DispatcherServlet 根据获得的Handler，选择一个合适的HandlerAdapter。（附注：如果成功获得HandlerAdapter后，此时将开始执行拦截器的preHandler(...)方法）
4.  提取Request中的模型数据，填充Handler入参，开始执行Handler（Controller)。 在填充Handler的入参过程中，根据你的配置，Spring将帮你做一些额外的工作：
HttpMessageConveter： 将请求消息（如Json、xml等数据）转换成一个对象，将对象转换为指定的响应信息
数据转换：对请求消息进行数据转换。如String转换成Integer、Double等
数据根式化：对请求消息进行数据格式化。 如将字符串转换成格式化数字或格式化日期等
数据验证： 验证数据的有效性（长度、格式等），验证结果存储到BindingResult或Error中
5.  Handler执行完成后，向DispatcherServlet 返回一个ModelAndView对象；
6.  根据返回的ModelAndView，选择一个适合的ViewResolver（必须是已经注册到Spring容器中的ViewResolver)返回给DispatcherServlet ；
7. ViewResolver 结合Model和View，来渲染视图
8. 将渲染结果返回给客户端。



spring 零配置
	@Configuration
	@ComponentScan("com.csonezp")
	@EnableWebMvc
	public class Config {
	    /**
	     * jsp视图解析器的bean
	     * @return
	     */
	    @Bean
	    public UrlBasedViewResolver setupViewResolver() {
	        UrlBasedViewResolver resolver = new UrlBasedViewResolver();
	        resolver.setPrefix("/WEB-INF/");
	        resolver.setSuffix(".jsp");
	        resolver.setViewClass(JstlView.class);
	        return resolver;
	    }

	    /**
	     * 配置数据源
	     * @return
	     */
	    @Bean(name = "dataSource")
	    public ComboPooledDataSource getDataSource() {
	        try {

	            ComboPooledDataSource dataSource = new ComboPooledDataSource();
	            dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/mfdb");
	            dataSource.setDriverClass("com.mysql.jdbc.Driver");
	            dataSource.setUser("root");
	            dataSource.setPassword("zp1228");
	            dataSource.setMaxPoolSize(75);
	            return dataSource;
	        } catch (Exception e) {
	            return null;
	        }
	    }
	}
	@Configuration注解就是告诉Spring这个是一个配置文件类，这里配置的Bean要交给Spring去管理。这个就是用来取代Beans.xml这种文件的。
	@ComponentScan("com.csonezp")这个注解就是配置包扫描用的，不必多说了
	@EnableWebMvc ，启用Spring MVC支持


=====================================================================spring 测试
spring 整合junit
	@RunWith(SpringJUnit4ClassRunner.class)
	@ContextConfiguration({"/config/application*.xml"})
	public class BaseJunit {
	    
	}

	public class UserServiceTest extends BaseJunit {
	    @Autowired
	    private UserService userService;
	    
	    @Test
	    public void c1() {
	        List<User> userList = userService.query(new User());
	        System.out.println(userList);
	    }
	}

	@RunWith
	指定测试类运行的环境，SpringJUnit4ClassRunner.class
	@ContextConfiguration
	加载spring配置文件
	@SpringApplicationConfiguration
	spring boot中使用
	@WebAppConfiguration 
	测试环境使用，加载spring上下文，在web中上下文是通过一个监听器加载的；value指定web应用的根
	@ContextHierarchy：指定容器层次
	@ContextHierarchy({  
	        @ContextConfiguration(name = "parent", locations = "classpath:spring-config.xml"),  
	        @ContextConfiguration(name = "child", locations = "classpath:spring-mvc.xml")  
	}) 


测试事务
	@Test
	@Transactional
	@Rollback(false)
	public void testTransaction(){
	accountService.account("yx","wx");
	}
	单元测试中默认事务只会Rollback而不会Commit,但设置@Rollback(false)则会Commit 


mock测试
	mock对象
	• 允许你测试依赖于其它对象的方法,但那个对象非常难实例化或是 太慢如资源类的等。 
	• 要测试控制器中的请求,但又不需要通过浏览器来完成,则可以通过Mock。 
	private MockMvc mockMvc;
	@Before
	public void getMockForWeb(){
	    this.mockMvc = MockMvcBuilders.webAppContextSetup(this.wac).build();
	}
	@Test
	public void testCustomer() throws Exception{
	    MvcResult result = this.mockMvc.perform(MockMvcRequestBuilders.get("/customer").
	            param("param_key","param_value").param("param_key","param_value")).
	            andDo(MockMvcResultHandlers.print()).
	            andReturn();
	   Assert.assertNotNull(result.getModelAndView());
	}
	解析: 
	mockMvc.perform 执行一个请求 
	MockMVCRequestBuilders.get(“URL”) 构造一个请求get或者post自己指定 
	ResultActions.andDo添加一个结果处理器，表示对结果做点什么事情，MockMvcResultHandlers.print()表示输出整个响应信息。 
	ResultActions.andReturn表示执行完后返回相应的结果。 
	另外，ResultActions.andException添加执行完成后的断言。


spring boot
	异常处理
	编码处理
	拦截器
	过滤器
	监听器
	依赖注入注解，扫描注解
controller
	请求格式
		@requestBody :json/post 
		@requestParam: form-urlEncode/post   url拼接/get
		@pathVariable：get
	返回格式
		json:统一的类转换成json{code,message,data}
		void：输出流
		String:回调响应
	参数为空验证：捕捉异常
	调用service：尽量调用一个service，多个service若有数据变更注意分布式事务
	swagger文档生成
service
	调用dao拿数据
	保证本地事务原子性
	不返回逻辑错误结果，直接抛异常

总体
	基础数据类型尽量写包装类型
	sql涉及的数据量较大的表尽量不要太多关联查询
	异常分门别类，由日志输出到不同文件，业务中不要处理异常，aop统一处理
	实体类完全对应数据库表数据，避免创建过多的类，尽量使用map代替，除非使用次数较多
	实体类，工具类，共用类放到common


spring boot包扫描
	自动扫描添加了注解的类，配置到环境中
	默认扫描启动类所在包下的所有类，也可自动指定扫描包
spring boot自动配置
	EnableAutoConfiguration
		Import({EnableAutoConfigurationImportSelector.class})
	EnableAutoConfigurationImportSelector
		加载spring.factories下key为EnableAutoConfiguration全限定名对应值
			DispatcherServletAutoConfiguration 注册DispatcherServlet 
			EmbeddedServletContainerAutoConfiguration.EmbeddedTomcat 注册Tomcat容器 
			ErrorMvcAutoConfiguration 注册异常处理器 
			HttpEncodingAutoConfiguration 注册编码过滤器CharacterEncodingFilter 
			HttpMessageConvertersAutoConfiguration 注册json或者xml处理器 
			JacksonAutoConfiguration 注册json对象解析器 
			MultipartAutoConfiguration 注册文件传输处理器 
			TransactionAutoConfiguration 注册事物管理处理器 
			ValidationAutoConfiguration 注册数据校验处理器 
			WebMvcAutoConfiguration 注册SpringMvc相关处理器
	配置组件到上下文
		自动配置类会根据项目中是否存在指定的类自动配置相应内容到spring上下文
	指定自动配置类
		去掉EnableAutoConfiguration，加入Import{class,class,class}
spring boot
	监听器，过滤器,servlet
		两种方式配置（以前是在web.xml配置）
		1、基于RegistrationBean的配置
			spring boot提供了 ServletRegistrationBean，FilterRegistrationBean，ServletListenerRegistrationBean来配置Servlet、Filter、Listener
		2.基于注解的配置
			@WebFilter @WebServlet @WebListener
	拦截器
		实现WebMvcConfigurerAdapter的addIntecptor方法
	数据类型转换 
		1.实现converter的convert方法
		2.@bean 配置
	事务
		失败时回滚，用于联合操作，时候其中一个后回滚
		1.开启事务 @EnableTransactionManagement
		2.配置事务管理器 
		@Bean
	    public PlatformTransactionManager txManager(DataSource dataSource) {
	        return new DataSourceTransactionManager(dataSource);
	    }
	全局异常处理
		@ControllerAdvice
		@ExceptionHandler(Throwable.class)
	MappingJackson2HttpMessageConverter
	WebMvcConfigurerAdapter	
		addFormatters
		addInterceptors
		addResourceHandlers
		addViewControllers
		configureViewResolvers
	返回html
		1.配置
			spring.thymeleaf.prefix: /templates/    
			spring.thymeleaf.suffix: .html
		2.资源
			/src/java/resources/templates/**.html
		3.编码
			return "**"
	请求&返回参数处理
		StringHttpMessageConverter	
			从请求和响应读取/编写字符串。默认情况下，它支持媒体类型 text/* 并使用文本/无格式内容类型编写。
		FormHttpMessageConverter	
			从请求和响应读取/编写表单数据。默认情况下，它读取媒体类型 application/x-www-form-urlencoded 
			并将数据写入 MultiValueMap<String,String>。
		MarshallingHttpMessageConverter	
			使用 Spring 的 marshaller/un-marshaller 读取/编写 XML 数据。它转换媒体类型为 application/xml 的数据。
		MappingJacksonHttpMessageConverter	
			使用 Jackson 的 ObjectMapper 读取/编写 JSON 数据。它转换媒体类型为 application/json 的数据。
		AtomFeedHttpMessageConverter	
			使用 ROME 的 Feed API 读取/编写 ATOM 源。它转换媒体类型为 application/atom+xml 的数据。
		RssChannelHttpMessageConverter	
			使用 ROME 的 feed API 读取/编写 RSS 源。它转换媒体类型为 application/rss+xml 的数据。
		json基本数据类型：数值（整型，浮点），字符串，布尔，null
	返回数据类型转换
		http://www.cnblogs.com/woshimrf/p/5189435.html