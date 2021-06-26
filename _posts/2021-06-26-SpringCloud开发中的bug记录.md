---
title: SpringCloud开发中的bug记录
tags: Spring
---

# SpringCloud开发中的bug记录

​	对于SpringCloud，在使用过程中，很多参数的匹配和注解的使用都很严苛，建议有注解的地方，改写就写，不要省略！！！

## @RequestBody和@RequestParam的使用

@RequestBody主要用来接收前端传递给后端的json字符串中的数据的(请求体中的数据的)；GET方式无请求体，所以使用@RequestBody接收数据时，**前端不能使用GET方式提交数据，而是用POST方式进行提交**。在后端的同一个接收方法里，@RequestBody与@RequestParam()可以同时使用，@RequestBody最多只能有一个，而@RequestParam()可以有多个。

- 一个请求，只有一个RequestBody；一个请求，可以有多个RequestParam。

- 当同时使用@RequestParam（）和@RequestBody时，@RequestParam（）指定的参数可以是普通元素、数组、集合、对象等等(即:当，@RequestBody 与@RequestParam()可以同时使用时，原SpringMVC接收参数的机制不变，只不过RequestBody 接收的是请求体里面的数据；而RequestParam接收的是key-value里面的参数，所以它会被切面进行处理从而可以用普通元素、数组、集合、对象等接收)。即：如果参数时放在请求体中，传入后台的话，那么后台要用@RequestBody才能接收到；如果不是放在请求体中的话，那么后台接收前台传过来的参数时，要用@RequestParam来接收，或则形参前什么也不写也能接收。

- 如果参数前写了@RequestParam(xxx)，那么前端必须有对应的xxx名字才行(不管其是否有值，当然可以通过设置该注解的required属性来调节是否必须传)，如果没有xxx名的话，那么请求会出错，报400。

- 如果参数前不写@RequestParam(xxx)的话，那么就前端可以有可以没有对应的xxx名字才行，如果有xxx名的话，那么就会自动匹配；没有的话，请求也能正确发送。追注：这里与feign消费服务时不同；**feign消费服务时，如果参数前什么也不写，那么会被默认是@RequestBody的**。

可以参考博文：[https://www.cnblogs.com/jpfss/p/10966585.html](https://www.cnblogs.com/jpfss/p/10966585.html)

**tips**: 就像文章开头说的，SpringCloud开发过程中，设计到更多的网络调用和工程之间的引用、调用，对参数一些要求比纯Springboot项目更严格，所以建议，`@RequestBody和@RequestParam`注解，一定要写！！！

```java
public interface User {

	// 注意，这里后面通过feign被继承，需要带上工程的Context-path
	
	/*
	 * 注册
	 */
	@RequestMapping("/member/regist")
	public String regist(@RequestBody UserEntity userEntity);
	
	/*
	 * 登录
	 *   登录后需要将key-value放入redis
	 */
	@RequestMapping("/member/login")
	public String login(@RequestBody UserEntity userEntity, @RequestParam("name") String name);
	
	/*
	 * 通过token查询用户
	 */
	@RequestMapping("/member/getUserBytoken")
	public String getUserBytoken(@RequestParam("token") String token);
	
	/*
	 * 使用openId关联用户信息
	 */
	@RequestMapping("/member/loginByOpenId")
	public String loginByOpenId(@RequestParam("openId") String openId);
}
```



## feign的调用

在feignClient(服务提供者)端，要注意暴露接口的返回值为JSON，可以用`@Controller + @ResponsedBody`或者`@RestController`修饰。否则报错`feign.FeignException: status 404 readin`。

比如下面是我的项目中的一段，这里是服务端代码，注意应该是`@RestController`，就是返回值为字符串，但是很容易误写成`Controller`，就会报上面错。

```java
@RestController
@Slf4j
public class UserImplController implements User{
	
	@Autowired
	private UserService userService;

	/*
	 * 注册（非 Javadoc）
	 * @see com.qian.FeignApi.User#regist(com.qian.entity.UserEntity)
	 */
	@RequestMapping("/regist") // 这里需要覆盖User接口中的RequestMapping
	@Override
	public String regist(@RequestBody UserEntity userEntity) {
		// TODO 自动生成的方法存根
		String username = userEntity.getUsername();
		String password = userEntity.getPassword();
		String email = userEntity.getEmail();
		String phone = userEntity.getPhone();
		if(username.equals("") || username == null) {
			log.info("UserImplController/regist, 用户名为空");
			return ResultApi.error("用户名不能为空");
		}
		if(password.equals("") || password == null) {
			log.info("UserImplController/regist, 密码为空");
			return ResultApi.error("密码不能为空");
		}
		if(email.equals("") || email == null) {
			log.info("UserImplController/regist, 邮箱为空");
			return ResultApi.error("邮箱不能为空");
		}
		if(phone.equals("") || phone == null) {
			log.info("UserImplController/regist, 手机号为空");
			return ResultApi.error("手机号不能为空");
		}
		String registResult = userService.regist(userEntity);
		return registResult;
	}
	// ....
}
```

若不是restful风格，也要注意服务端和feign消费者接口返回值的统一。

## loadbalance error ...client: item，这类错误，可能需要重启一下工程即可

​	这个就很无语了～，也可能是eclipse或者IDE的问题，就是注意一下，遇到不可思议，unbelieveable的问题，重启优先，或者IDE清理项目/缓存。

