---
title: Mybatis对mapper层中，参数的绑定
tags: ORM
---

# Mybatis对mapper层中，参数的绑定

> 参考链接：
>
> - [https://www.cnblogs.com/lenve/p/11229590.html](https://www.cnblogs.com/lenve/p/11229590.html)

在写项目的过程中用到mybatis层中，有关于`@Param`的使用和参数的绑定一直是个坑，这里记录一下

## mapper层的格式

```java
public interface MessageInfoMapping {
	@Select("select a.id, a.message_name, a.message_url, a.message_pwd, a.message_author, a.created, a.updated, "
			+ "b.type_name from `message_info` as a inner join `message_type` as b "
			+ "on a.message_typeId = b.id")
	List<MessageInfoEntity> getMessageAll();
  
	@Insert("insert into `message_info` (`message_name`, `message_url`, `message_typeId`,`message_pwd`, `message_author`) "
			+ "values(#{messageName}, #{messageUrl}, #{messageTypeId}, #{messagePwd}, #{messageAuthor})")
	void addMessage(MessageInfoEntity message);
  
	@Select("select * from `message_info` where message_url = #{messageUrl}")
	MessageInfoEntity getMessageByUrl(String messageUrl);
  
	@Delete("delete from `message_info` where message_url = #{messageUrl}")
	void deleteMessageByUrl(String messageUrl);
  
	@Update("update `message_info` SET message_name = #{message.messageName}, message_url = #{message.messageUrl}, "+ "message_typeId=#{message.messageTypeId}, message_pwd=#{message.messagePwd}, message_author=#{message.messageAuthor} "+ "where message_url=#{oldUrl}")
	void updateMessage(@Param("message")MessageInfoEntity message, @Param("oldUrl") String oldUrl);
}
```

​	引入mybatis依赖后，先创建实体类entity，再写mapper层的接口。mapper层可以不用加任何springboot注解哦～，可以统一在启动类加上扫包`@MapperScan(...)`。

​	接口里面就是SQL常见的`crud`操作了。下面主要说明和记录一下方法传参数的时候坑。

## 由参数问题引发的bug

​	有时候可能会觉得莫名其妙，有的时候一个参数明明不用添加 @Param 注解，有的时候，却需要添加，不添加会报错。

## 常见需要使用`@Param`的场景

首先，如下几个需要添加 @Param 注解的场景，相信大家都已经有共识了：

- 第一种：方法有多个参数，需要 @Param 注解

例如下面这样：

```java
@Mapper
public interface UserMapper {
    Integer insert(@Param("username") String username, @Param("address") String address);
}
```

对应的 XML 文件如下：

```xml
<insert id="insert" parameterType="org.javaboy.helloboot.bean.User">
    insert into user (username,address) values (#{username},#{address});
</insert>
```

这是最常见的需要添加 @Param 注解的场景。

- 第二种：方法参数要取别名，需要 @Param 注解

当需要给参数取一个别名的时候，我们也需要 @Param 注解，例如方法定义如下：

```java
@Mapper
public interface UserMapper {
    User getUserByUsername(@Param("name") String username);
}
```

对应的 XML 定义如下：

```xml
<select id="getUserByUsername" parameterType="org.javaboy.helloboot.bean.User">
    select * from user where username=#{name};
</select>
```

老实说，这种需求不多，费事。

- 第三种：XML 中的 SQL 使用了 $ ，那么参数中也需要 @Param 注解

$ 会有注入漏洞的问题，但是有的时候你不得不使用 $ 符号，例如要传入列名或者表名的时候，这个时候必须要添加 @Param 注解，例如：

```java
@Mapper
public interface UserMapper {
    List<User> getAllUsers(@Param("order_by")String order_by);
}
```

对应的 XML 定义如下：

```xml
<select id="getAllUsers" resultType="org.javaboy.helloboot.bean.User">
    select * from user
    <if test="order_by!=null and order_by!=''">
        order by ${order_by} desc
    </if>
</select>
```

前面这三种，都很容易懂，相信很多小伙伴也都懂，除了这三种常见的场景之外，还有一个特殊的场景，经常被人忽略。

- 第四种，那就是动态 SQL ，如果在动态 SQL 中使用了参数作为变量，那么也需要 @Param 注解，即使你只有一个参数。

如果我们在动态 SQL 中用到了 参数作为判断条件，那么也是一定要加 @Param 注解的，例如如下方法：

```java
@Mapper
public interface UserMapper {
    List<User> getUserById(@Param("id")Integer id);
}
```

定义出来的 SQL 如下：

```xml
<select id="getUserById" resultType="org.javaboy.helloboot.bean.User">
    select * from user
    <if test="id!=null">
        where id=#{id}
    </if>
</select>
```

这种情况，即使只有一个参数，也需要添加 @Param 注解，而这种情况却经常被人忽略！

## mapper层的SQL语句传入List

- 首先，我遇到过的错误：

```
Parameter array not found. Available parameters are [collection, list]
```

  报错原因主要是，当我们要查询一些的信息时，可能会采用list集合或者数组作为参数传入方法中。

```xml
<select id="findSomeUsers" resultType="user3" parameterType="list">
  select * from user where id in
  <foreach collection="noList" index="index" item="no" open="(" separator="," close=")">
 	#{no}
  </foreach>
</select>
```
  这时报错是因为，传递一个 List 实例或者数组作为参数对象传给 MyBatis,MyBatis 会自动将它包装在一个 Map 中,用名称在作为键。List 实例将会以“list” 作为键,而数组实例将会以“array”作为键。解决这个异常的两种方式是：
  1.在方法参数前面加上你遍历的集合的名称，比如你在foreach的collection中写的是noList，那么你就在传入的list参数前面加上一个注解@Param(“noList”)。
  2.将foreach的collection中的值改成list即可。

- 在实际开发中，上述报错，下面代码为一项目中使用的正确代码：

```java
@Select({ "<script>", "select * from `product_category` where category_type in ",
"<foreach collection='categoryTpyeList' item='item' index='index' open='(' separator=',' close=')'>",
"#{item}", "</foreach>", "</script>" })
public List<ProductCategoryEntity> findByCategoryTpye(
  @Param("categoryTpyeList") List<Integer> categoryTpyeList
);
```
​	- collection： 指定要遍历的集合（三种情况 list，array，map） ！！！！在这种使用注解sql的情况下，这里请填写mapper方法中集合的名称

​	- item：将当前遍历出的元素赋值给指定的变量 （相当于for循环中的i）

​	- separator:每个元素之间的分隔符

​	- index:索引。遍历list的时候是index就是索引，item就是当前值

​	- \#{变量名}就能取出变量的值也就是当前遍历出的元素

