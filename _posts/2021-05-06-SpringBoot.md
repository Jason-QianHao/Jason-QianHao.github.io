---
title: Springboot和常见框架整合
tags: Spring
---
*由简书搬迁而来，bug和格式修改中，请看[**原文链接**](https://www.jianshu.com/p/d10d9cce534d)*
> 目录  
>  1 Springboot简介  
>  2 Springboot整合依赖汇总  
>  3 常见问题汇总  

# 1 Springboot简介
Spring Boot让我们的Spring应用变的更轻量化。比如：你可以仅仅依靠一个Java类来运行一个Spring引用。你也可以打包你的应用为jar并通过使用java -jar来运行你的Spring Web应用。

Spring Boot的主要优点：
- 为所有Spring开发者更快的入门
- 开箱即用，提供各种默认配置来简化项目配置
- 内嵌式容器简化Web项目
- 没有冗余代码生成和XML配置的要求

# 2 Springboot整合依赖汇总

## 2.1 Spring Web基础

### 2.1.1 Web基础依赖
```
<parent>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-parent</artifactId>
     <version>1.5.2.RELEASE</version>
     <relativePath /> <!-- lookup parent from repository -->
 </parent>
<dependencies>
   <!-- 集成web -->
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-web</artifactId>
   </dependency>
</dependencies>
 <build>
     <plugins>
         <plugin>
             <groupId>org.springframework.boot</groupId>
             <artifactId>spring-boot-maven-plugin</artifactId>
         </plugin>
         <plugin>
             <groupId>org.apache.maven.plugins</groupId>
             <artifactId>maven-surefire-plugin</artifactId>
             <version>2.4.2</version>
             <configuration>
                 <skipTests>true</skipTests>
             </configuration>
         </plugin>
         <plugin>
             <groupId>org.apache.maven.plugins</groupId>
             <artifactId>maven-clean-plugin</artifactId>
             <version>3.0.0</version>
             <configuration>
                 <skipTests>true</skipTests>
             </configuration>
         </plugin>
     </plugins> 
</build>
```
**spring-boot-starter-web会自动引入spring-webmvc，spring-boot-starter-validation，spring-boot-starter，spring-boot-starter-json，spring-boot-starter-tomcat 这5个基础依赖**

### 2.1.2 配置和启动

（1）spring-boot-starter-web添加了Tomcat和Spring MVC
（2）Springboot的几种启动方式：
- @EnableAutoConfiguration，auto-configuration将假定你正在开发一个web应用并相应地对Spring进行设置。
```
@RestController
@EnableAutoConfiguration
public class HelloController {
     @RequestMapping("/hello")
     public String index() {
          return "Hello World";
     }
     public static void main(String[] args) {
         SpringApplication.run(HelloController.class, args);
     }
 }
```
`@ComponentScan(basePackages = "com.itmayiedu.controller") 控制器扫包`
```
 @ComponentScan(basePackages = "com.itmayiedu.controller")
 @EnableAutoConfiguration
 public class App {
     public static void main(String[] args) {
          SpringApplication.run(App.class, args);
     }
 }
```
- **（推荐) @SpringBootApplication注解**
可以简化部分配置，但mybatis的mapper扫包和springcloud的eureka启动等不能少。
```
@SpringBootApplication
@EnableEurekaClient
@MapperScan(basePackages="com.qian.mapper")
public class MemberApp {
     public static void main(String[] args) {
        SpringApplication.run(MemberApp.class, args);
     }
 }
```
（3）yml配置文件
```
server:
   port: 8080
   context-path: /project
```
## 2.2 页面渲染

### 2.2.1 静态资源访问
在我们开发Web应用的时候，需要引用大量的js、css、图片等静态资源。
Spring Boot默认提供静态资源目录位置需置于classpath下，目录名需符合如下规则：
- /static
- /public
- /resources
- /META-INF/resources
>举例：我们可以在src/main/resources/目录下创建static文件夹，在该位置放置一个图片文件。启动程序后，尝试访问http://localhost:8080/D.jpg。如能显示图片，配置成功。

### 2.2.2 Freemarker模板引擎渲染web视图

#### 2.2.2.1 模板引擎
模板的诞生是为了将显示与数据分离，模板技术多种多样，但其本质是将模板文件和数据通过模板引擎生成最终的HTML代码。使用户界面与业务数据（内容）分离而产生的，它可以生成特定格式的文档，用于网站的模板引擎就会生成一个标准的HTML文档在原有的HTML页面中来填充数据。最终达到渲染页面的目的。

![image](https://upload-images.jianshu.io/upload_images/24777208-942b91f3d5455c2e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

模板技术并不是什么神秘技术，干的是拼接字符串的体力活。模板引擎就是利用正则表达式识别模板标识，并利用数据替换其中的标识符。比如：Hello, <%= name%>

![image](https://upload-images.jianshu.io/upload_images/24777208-81637e7a9aaa566a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

数据是{name: '木的树'}，那么通过模板引擎解析后，我们希望得到Hello, 木的树。模板的前半部分是普通字符串，后半部分是模板标识，我们需要将其中的标识符替换为表达式。

#### 2.2.2.2 Freemarker

（1）maven依赖

**使用jar工程即可！**
```
<dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-freemarker</artifactId>
</dependency>
```
（2）yml配置
```
spring:
   freemarker:
       allow-request-override : false
       cache : true
       check-template-location : true
       charset : UTF-8
       content-type : text/html
       expose-request-attributes : false
       expose-session-attributes : false
       expose-spring-macro-helpers : false
       suffix : .ftl
       template-loader-path : classpath:/templates/
```
**其实上述suffix可以该为.html即可支持html格式文件，测试有效！**

（3）启动
```
@RequestMapping("/index")
public String index(Map<String, Object> map) {
     map.put("name","美丽的天使...");
     return "index";
 }
```
Map或者modelMap和request转发是一样的

### 2.2.3 JSP

#### 2.2.3.1 maven依赖
```
<dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-tomcat</artifactId>
 </dependency>
 <dependency>
     <groupId>org.apache.tomcat.embed</groupId>
     <artifactId>tomcat-embed-jasper</artifactId>
 </dependency>
```
**maven中使用war类型工程,这点区别于freemarker！**

#### 2.2.3.2 yml配置
```
spring: 
     mvc: 
         view: 
             prefix: /WEB-INF/jsp/
             suffix: .jsp
```
#### 2.2.3.3 启动
```
 @Controller 
 public class IndexController {
     @RequestMapping("/index")
     public String index() {
         return "index";
     }
 }
```
## 2.3 数据库访问

### 2.3.1 JdbcTemplate

#### 2.3.1.1 maven依赖
```
<dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-jdbc</artifactId>
 </dependency>
 <dependency>
     <groupId>mysql</groupId>
     <artifactId>mysql-connector-java</artifactId>
     <version>5.1.21</version>
 </dependency>
```
#### 2.3.1.2 yml配置
```
spring:
   datasource :
       url : jdbc:mysql://localhost:3306/test
       username : root
       password : root
       driver-class-name : com.mysql.jdbc.Driver
```
#### 2.3.1.3 调用
```
@Service
public class UserServiceImpl implements UserService {
     @Autowired
     private JdbcTemplate jdbcTemplate;
     public void createUser(String name, Integer age) {
         System.*out*.println("ssss");
         jdbcTemplate.update("insert into users values(null,?,?);", name, age);
     }
 }
```
### 2.3.2 Mybatis
#### 2.3.2.1 maven依赖
```
<dependency>
     <groupId>org.mybatis.spring.boot</groupId>
     <artifactId>mybatis-spring-boot-starter</artifactId>
     <version>1.1.1</version>
 </dependency>
 <dependency>
     <groupId>mysql</groupId>
     <artifactId>mysql-connector-java</artifactId>
     <version>5.1.46</version>
 </dependency>
```
#### 2.3.2.2 yml配置
```
spring:
   datasource :
       url : jdbc:mysql://localhost:3306/test
       username : root
       password : root
       driver-class-name : com.mysql.jdbc.Driver
```
#### 2.3.2.3 调用
（1）定义mapper接口
```
public interface UserMapper {
     @Select("SELECT * FROM USERS WHERE NAME = #{name}")
     User findByName(@Param("name") String name);
     @Insert("INSERT INTO USERS(NAME, AGE) VALUES(#{name}, #{age})")
     int insert(@Param("name") String name, @Param("age") Integer age);
}
```
（2）@MapperScan(basePackages = "com.mapper")扫包
```
@ComponentScan(basePackages = "com.qian")
@MapperScan(basePackages = "com.qian.mapper")
@SpringBootApplication
public class App {
     public static void main(String[] args) {
     SpringApplication.*run*(App.class, args);
    }
 }
```
### 2.3.3 Springjpa
#### 2.3.3.1 maven依赖
```
<dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
     <groupId>mysql</groupId>
     <artifactId>mysql-connector-java</artifactId>
     <version>5.1.21</version>
</dependency>
```
#### 2.3.3.2 yml配置
```
spring:
   datasource :
       url : jdbc:mysql://localhost:3306/test
       username : root
       password : root
       driver-class-name : com.mysql.jdbc.Driver
```
#### 2.3.3.3 调用和启动

（1）创建实体类
```
@Entity(name = "users")
public class User {
     @Id
     @GeneratedValue
     private Integer id;
     @Column
     private String name;
     @Column
     private Integer age;
       // ..get/set方法
}
```
（2）创建Dao接口
```
public interface UserDao extends JpaRepository<User, Integer> {
}
```
（3）@EnableJpaRepositories(basePackages = "com.qian.dao")启动
```
@ComponentScan(basePackages = { "com.qian" })
@EnableJpaRepositories(basePackages = "com.qian.dao")
@EnableAutoConfiguration
@EntityScan(basePackages = "com.itmayiedu.entity")
public class App {
     public static void main(String[] args) {
         SpringApplication.*run*(App.class, args);
    }
}
```
## 2.4 @Configuration配置

### 2.4.1 @ConfigurationProperties
如果我们需要取 N 个配置项，通过 @Value（“${name}”） 的方式去配置项需要一个一个去取，这就显得有点 low 了。我们可以使用 @ConfigurationProperties 。标有 @ConfigurationProperties 的类的**所有属性和配置文件中相关的配置项进行绑定**。（默认从全局配置文件中获取配置值），绑定之后我们就可以通过这个类去访问全局配置文件中的属性值了。

（1） 在主配置文件中添加如下配置
```
person: 
     name: kundy
     age: 13
     sex: male
```
（2）创建配置类，由于篇幅问题这里省略了 setter、getter 方法，但是实际开发中这个是必须的，否则无法成功注入。另外，@Component 这个注解也还是需要添加的。
```
@Component
@ConfigurationProperties(prefix="person")
public class Person{
     private String name;
     private Integer age;
     private String sex;
}
```
这里 @ConfigurationProperties 有一个 prefix 参数，主要是用来指定该配置项在配置文件中的前缀。

### 2.4.2 @Bean

#### 2.4.2.1 使用说明

（1）@Bean注解相当于spring的xml配置文件<bean>标签，告诉容器注入一个bean。

（2）@Bean 注解作用在方法上

（3）@Bean 指示一个方法返回一个 Spring 容器管理的 Bean

（4）@Bean 方法名与返回类名一致，首字母小写

（5）@Bean 一般和 @Component 或者 @Configuration 一起使用

（6）@Bean 注解默认作用域为单例 singleton 作用域，可通过 @Scope(“prototype”) 设置为原型作用域

#### 2.4.2.2 Bean 名称

（1）默认情况下 Bean 名称就是方法名，比如下面 Bean 名称便是 myBean：
```
@Bean
public MyBean myBean() {
     return new MyBean();
}
```
（2）@Bean 注解支持设置别名。比如下面除了主名称 myBean 外，还有个别名 myBean1（两个都可以使用）
```
@Bean("myBean1")
public MyBean myBean() {
     return new MyBean();
}
```
（3）@Bean 注解可以接受一个 String 数组设置多个别名。比如下面除了主名称 myBean 外，还有别名 myBean1、myBean2（三个都可以使用）
```
@Bean({"myBean1", "myBean2"})
public MyBean myBean() {
     return new MyBean();
}
```
#### 2.4.2.3 @Bean 与其他注解一起使用

（1）@Bean 注解常常与 @Scope、@Lazy，@DependsOn 和 @link Primary 注解一起使用：

@Profile 注解：为在不同环境下使用不同的配置提供了支持，如开发环境和生产环境的数据库配置是不同的

@Scope 注解：将 Bean 的作用域从单例改变为指定的作用域

@Lazy 注解：只有在默认单例作用域的情况下才有实际效果

@DependsOn 注解：表示在当前 Bean 创建之前需要先创建特定的其他 Bean

（2）比如下面样例，Bean 的作用域默认是单例的，我们配合 @Scope 注解将其改成 prototype 原型模式（每次获取 Bean 的时候会有一个新的实例）
```
@Bean()
@Scope("prototype")
public MyBean myBean() {
     return new MyBean();
}
```
#### 2.4.2.4 Bean 初始化和销毁时调用相应的方法 

（1）实际开发中，经常会遇到在 Bean 使用之前或使用之后做些必要的操作，Spring 对 Bean 的生命周期的操作提供了支持：我们可以通过 @Bean 注解的 initMethod 和 destrodMethod 进行指定 Bean 在初始化和销毁时需要调用相应的方法。

（2）下面是一个简单的样例：
```
public class MyBean {
     public void init() {
        System.out.println("MyBean开始初始化...");
     }
     public void destroy() {
         System.out.println("MyBean销毁...");
     }
     public String get() {
         return "MyBean使用...";
     }
}
> 
> @Bean(initMethod="init", destroyMethod="destroy")
> 
> public MyBean myBean() {
> 
>     return new MyBean();
> 
> }
```
### 2.4.3 **@Configuration**

#### **2.4.3.1 使用说明**

（1）@Configuration注解底层是含有@Component ，所以@Configuration 具有和 @Component 的作用。

（2）@Configuration注解相当于spring的xml配置文件中<beans>标签，里面可以配置bean。

#### 2.4.3.2 Demo
```
import org.springframework.context.annotation.Bean;
 import org.springframework.context.annotation.Configuration;
@Configuration
public class MyConfigration {
     @Bean
     public String **hello()** {
         return "welcome to hangge.com";
     }
}
```
```
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
@RestController
public class HelloController {
    @Autowired
    String hello;

    @GetMapping("/test")
     public String test() {
         return hello;
     }
}
```
## 2.5 事务管理

### 2.5.1 @Transactional
 springboot默认集成事物,只主要在方法上加上@Transactional即可。

### 2.5.1 分布式事务管理
使用springboot+jta+atomikos 分布式事物管理。

**maven依赖：**
```
<dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-jta-atomikos</artifactId>
</dependency>
```
## 2.6 Lombok

### 2.6.1 maven依赖
```
<!--Lombok -->
<dependency>
     <groupId>org.projectlombok</groupId>
     <artifactId>lombok</artifactId>
     <version>1.18.6</version>
</dependency>
<dependency>
     <groupId>net.logstash.logback</groupId>
     <artifactId>logstash-logback-encoder</artifactId>
     <version>4.9</version>
</dependency>
```
### 2.6.2 **安装lomBok插件**

（1）下载lombok.jar包[https://projectlombok.org/download.html](https://projectlombok.org/download.html)

（2）运行Lombok.jar: java -jar D:\software\lombok.jar。D:\software\lombok.jar这是windows下lombok.jar所在的位置。数秒后将弹出一框，以确认eclipse的安装路径。

（3）确认完eclipse的安装路径后，点击install/update按钮，即可安装完成

（4）安装完成之后，请确认eclipse安装路径下是否多了一个lombok.jar包，并且其配置文件eclipse.ini中是否 添加了如下内容:
```
-javaagent:lombok.jar
-Xbootclasspath/a:lombok.jar
```
那么恭喜你已经安装成功，否则将缺少的部分添加到相应的位置即可，重启eclipse或myeclipse。
##2.7 其他
###2.7.1 使用@Scheduled创建定时任务
- 在Spring Boot的主类中加入@EnableScheduling注解，启用定时任务的配置。
- 设置时间方式：
1. 固定时间频率运行方法。@Scheduled(fixedRate=30000)
2. 延迟指定的时间运行方法。@Scheduled(fixedDelay=30000)
3. 按照 cron 表达式定义的时间方式运行方法。@Scheduled(cron="0 0 * * * *")
```
@Component
public class ScheduledTasks {
    private static final SimpleDateFormat dateFormat = new SimpleDateFormat("HH:mm:ss");
    @Scheduled(fixedDelay = 5000)
    public void reportCurrentTime() {
        System.out.println("现在时间：" + dateFormat.format(new Date()));
    }
}
```
###2.7.2 @Async实现异步调用
启动加上@EnableAsync，需要执行异步方法上加入 @Async

###2.7.3 发布打包
（1）使用mvn package 打包

（2）使用 “java –jar 包名” 运行应用

- 注意事项
如果报错没有主清单,在pom文件中新增
```
<build>
     <plugins>
         <plugin>
             <groupId>org.apache.maven.plugins</groupId>
             <artifactId>maven-compiler-plugin</artifactId>
             <configuration>
                 <source>1.8</source>
                 <target>1.8</target>
             </configuration>
         </plugin>
         <plugin>
                 <groupId>org.springframework.boot</groupId>
                 <artifactId>spring-boot-maven-plugin</artifactId>
                 <configuration>
                     <maimClass>com.qian.app.App</maimClass>
                 </configuration>
                 <executions>
                     <execution>
                         <goals>
                             <goal>repackage</goal>
                         </goals>
                     </execution>
                 </executions> 
         </plugin>
     </plugins>
</build>
```
###2.7.4 说明
关于ActiveMQ、Redis、Zookeeper、Dubbo、Spring Cloud 的相关依赖，在其各自专题介绍。

#3 常见问题汇总
##3.1 Spring中访问静态资源
注意这里不是Springboot环境，仅仅为Spring环境。在SpringMVC3.0之后推荐使用下列配置，该配置的作用是：DispatcherServlet不会拦截以/static开头的所有请求路径，并当作静态资源交由Servlet处理。
```
<mvc:annotation-driven />
<mvc:resources location="/img/" mapping="/img/**"/>
<mvc:resources location="/js/" mapping="/js/**"/>
<mvc:resources location="/css/" mapping="/css/**"/>
```
- 说明：
location元素表示webapp目录下的static包下的所有文件；
mapping元素表示以/static开头的所有请求路径，如/static/a 或者/static/a/b；

##3.2 Spring MVC Controller接收前端参数的几种方式
(1) 普通方式-请求参数名和Controller方法的参数一致
```
/** 
* 请求参数名和Controller方法的参数一致 
 * produces 设置返回参数的编码格式可以设置返回数据的类型以及编码，可以是json或者xml 
 * { 
 * @RequestMapping(value="/xxx",produces = {"application/json;charset=UTF-8"})
 * 或
 * @RequestMapping(value="/xxx",produces = {"application/xml;charset=UTF-8"})
 * 或
 * @RequestMapping(value="/xxx",produces = "{text/html;charset=utf-8}")
 * }
 * @param name 用户名
 * @param pwd 密码
 * @return
 *
*/
@RequestMapping(value = "/add", method = RequestMethod.GET, produces = {"application/json;charset=UTF-8"})
@ResponseBody
public String addUser(String name, String pwd){
    return"name:" + name + ",pwd:" + pwd;
 } 
```
`http://localhost:8080/sty/param/add.action?name=张三&pwd=123456`

(2) 对象方式-请求参数名和Controller方法中的对象的参数一致
```
@Controller
@RequestMapping("/param")publicclass TestParamController {
    privatestaticfinalLogger logger = LoggerFactory.getLogger(TestParamController.class);
    /**    * 请求参数名和Controller方法的参数一致
    * produces 设置返回参数的编码格式可以设置返回数据的类型以及编码，可以是json或者xml
    * }
    * @param user 用户信息
    * @return    *
    */    @RequestMapping(value = "/addByObject", method = RequestMethod.GET, produces = {"application/json;charset=UTF-8"})
    @ResponseBody
    public String addUserByObject(User user){
        logger.debug("name:" + user.getName() + ",pwd:" + user.getPwd());
        return"name:" + user.getName() + ",pwd:" + user.getPwd();
    }
} 
```
`http://localhost:8080/sty/param/addByObject.action?name=张三&pwd=123456 `

（3）自定义方法参数名-当请求参数名与方法参数名不一致时
可以在参数中增加@RequestParam注解。如果在方法中的参数增加了该注解，说明请求的url该带有该参数，否则不能执行该方法。如果在方法中的参数没有增加该注解，说明请求的url无需带有该参数，也能继续执行该方法。
```
　　@RequestParam(defaultValue="0")可设置默认值(仅当传入参数为空时)。

　　@RequestParam(value="id")可接受传入参数为id的值，覆盖该注解注释的字段。

　　@RequestParam(name="name",defaultValue = "李四") String u_name   如果传入字段”name”为空，默认u_name的值为”李四”。若传入”name”不为空，默认u_name值为传入值。
/** * 自定义方法参数名-当请求参数名与方法参数名不一致时

* @param u_name 用户名

* @param u_pwd 密码

* @return*/@RequestMapping(value = "/addByDifName", method = RequestMethod.GET, produces = {"application/json;charset=UTF-8"})

@ResponseBodypublicString addUserByDifName(@RequestParam("name") String u_name, @RequestParam("pwd")String u_pwd){

    logger.debug("name:" + u_name + ",pwd:" + u_pwd);

    return"name:" + u_name + ",pwd:" + u_pwd;

}
```
访问：` http://localhost:8080/sty/param/addUserByDifName.action?name=张三&pwd=123456`。

(4) HttpServletRequest方式
```
/** * 通过HttpServletRequest接收

* @param request

* @return*/@RequestMapping(value = "/addByHttpServletRequest", method = RequestMethod.GET, produces = {"application/json;charset=UTF-8"})

@ResponseBodypublic String addUserByHttpServletRequest(HttpServletRequest request){

    String name = request.getParameter("name");

    String pwd = request.getParameter("pwd");

    logger.debug("name:" + name + ",pwd:" + pwd);

    return"name:" + name + ",pwd:" + pwd;

} 
```
访问：`http://localhost:8080/sty/param/addByHttpServletRequest.action?name=张三&pwd=123456`

- 注意：其他方式的获取，基础都是request请求。自动封装也是从request中获取后进行的封装。

##3.3 Spring MVC的转发和重定向
###3.3.1 转发
使用request也行。    
```
@RequestMapping("/helloForward")
publicString helloForward(@RequestParam(value="name", required=false, defaultValue="World2017") String name, Model model) {
        model.addAttribute("name", name);
        return"hello";
    }
```
###3.3.2 重定向
（1）RedirectAttributes类
```
/** * 使用RedirectAttributes类

    * @param name

    * @param redirectAttributes

    * @return*/    @RequestMapping("/helloRedirect")

    publicString helloRedirect(@RequestParam(value="name", required=false ) String name, RedirectAttributes redirectAttributes) {

        redirectAttributes.addFlashAttribute("name", name);

        return"redirect:/hello";

    }
```
（2）借助Session传值
```
    /**    * 常规做法，重定向之前把参数放进Session中，在重定向之后的controller中把参数从Session中取出并放进ModelAndView

    * @param name

    * @param request

    * @return*/   

    @RequestMapping("/helloRedirect2")

    publicString helloRedirect2(@RequestParam(value="name", required=false ) String name, HttpServletRequest request) {

        request.getSession().setAttribute("name", name);

        return"redirect:/hello2";

    }

@RequestMapping("/hello2")

    public String hello2(Model model,HttpServletRequest request) {

        HttpSession session = request.getSession();

        model.addAttribute("name",session.getAttribute("name"));

        return"hello";     

    }
```
##3.4 Input标签上传多个图片到服务器
（1）前端页面
```
<form action="getData" style="font-size: 14px;" method="post"  ENCTYPE="multipart/form-data">
    <td><input type="file" name="file" multiple="multiple"></td>
</form>
```
（2）服务器端
```
public void getData(@RequestParam(value = "file", required = false) List<MultipartFile> file,
HttpServletRequest req) {
  try {
    for (MultipartFile f : file) {
        //....
    }
  }
}
```
##3.5 The field file exceeds its maximum permitted size of 1048576 bytes
Spring Boot做文件上传时出现了The field file exceeds its maximum permitted size of 1048576 bytes.错误，显示文件的大小超出了允许的范围。文档说明表示，每个文件的配置最大为1Mb，单次请求的文件的总数不能大于10Mb。要更改这个默认值需要在配置文件（如application.properties）中加入两个配置。

Spring Boot1.4版本后配置更改为:
```
spring.http.multipart.maxFileSize = 10Mb 
spring.http.multipart.maxRequestSize=100Mb 
```
Spring Boot2.0之后的版本配置修改为:
```
spring.servlet.multipart.max-file-size = 10MB
spring.servlet.multipart.max-request-size=100MB
```
##3.6 服务器发送可下载文件到浏览器
```
@RequestMapping(value = "downloadZip", method = RequestMethod.GET)
    public void downloadZip(HttpServletResponse response,String id) throws Exception {
        String fileName=workCardPhotoFileService.downloadZipFile(id);
        if (StringUtils.isNotEmpty(fileName)){
            response.setContentType("application/application/vnd.ms-excel");
            response.setHeader("Content-disposition",
                    "attachment;filename=" + fileName);
            download(response.getOutputStream(),fileName);
        }
    }

public void download(OutputStream os, String fileName) throws IOException {
        //获取服务器文件
        File file = new File("/Users/Desktop/download/workcardphoto/"+fileName);
        InputStream ins = new FileInputStream(file);
        byte[] b = new byte[1024];
        int len;
        while((len = ins.read(b)) > 0){
            os.write(b,0,len);
        }
        os.flush();
        os.close();
        ins.close();
    }
```
##3.7 @requestBody注解作用
通过@requestBody可以将请求体中的JSON字符串绑定到相应的bean上，当然，也可以将其分别绑定到对应的字符串上。
```
　$.ajax({
　　　　　　　　url:"/login",
　　　　　　　　type:"POST",
　　　　　　　　data:'{"userName":"admin","pwd","admin123"}',
　　　　　　　　content-type:"application/json charset=utf-8",
　　　　　　　　success:function(data){
　　　　　　　　　　alert("request success ! ");
　　　　　　　　}
　　　　});

@requestMapping("/login")
public void login(@requestBody User){
　　System.out.println(userName+" ："+pwd);
}
```
##3.8 Springboot访问HTML页面
此部分在测试过程中，一旦引入spring-boot-starter-thymeleaf，整个项目的pom文件就报错，具体原因排查中，了解的小伙伴也可以一起讨论。

在具体实践中，大都使用freemarker进行页面设计，且前述提到过，可以使用freemarker进行Html页面的访问。

###3.8.1 Springboot下maven依赖
```
<dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-thymeleaf</artifactId>
 </dependency>

<!--避坑包-->

      <dependency>

          <groupId>net.sourceforge.nekohtml</groupId>

          <artifactId>nekohtml</artifactId>

          <version>1.9.22</version>

      </dependency>
```
###3.8.2 yml配置
```
spring:
     thymeleaf:
         prefix: classpath:/templates/
```
在LEGACYHTML5下，application.properties配置文件如下。因为在默认配置下，thymeleaf对.html的内容要求很严格，比如<meta charset=”UTF-8″ />，如果少最后的标签封闭符号/，就会报错而转到错误页。也比如你在使用Vue.js这样的库，然后有<div v-cloak></div>这样的html代码，也会被thymeleaf认为不符合要求而抛出错误。因此，建议增加下面这段：
```
    spring.thymeleaf.mode = LEGACYHTML5
```
spring.thymeleaf.mode的默认值是HTML5，其实是一个很严格的检查，改为LEGACYHTML5可以得到一个可能更友好亲切的格式要求。
```
#<!-- 关闭thymeleaf缓存 开发时使用 否则没有实时画面-->
spring.thymeleaf.cache=false
## 检查模板是否存在，然后再呈现
spring.thymeleaf.check-template-location=true
#Content-Type值
spring.thymeleaf.content-type=text/html
#启用MVC Thymeleaf视图分辨率
spring.thymeleaf.enabled=true
## 应该从解决方案中排除的视图名称的逗号分隔列表
##spring.thymeleaf.excluded-view-names=
#模板编码
spring.thymeleaf.mode=LEGACYHTML5
# 在构建URL时预先查看名称的前缀
spring.thymeleaf.prefix=classpath:/templates/
# 构建URL时附加查看名称的后缀.
spring.thymeleaf.suffix=.html
# 链中模板解析器的顺序
#spring.thymeleaf.template-resolver-order= o
# 可以解析的视图名称的逗号分隔列表
#spring.thymeleaf.view-names=
#thymeleaf end
```