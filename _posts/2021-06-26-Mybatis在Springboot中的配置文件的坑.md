---
title: Mybatis在Springboot中的配置文件的坑
tags: ORM
---

# Mybatis在Springboot中的配置文件的坑

- Mybatis的Maven依赖，如果引用了，但是没有配置mysql会在启动报错

  这个地方主要在maven项目中含有父模块、子模块的时候可能会碰到，注意一下可以将Mybatis的依赖放到需要使用mysql的子模块中，就不要放到父模块中统一配置了

- 连接MySql报错Unable to load authentication plugin 'caching_sha2_password'

  错误的原因是由于MySQL8.0之后的加密规则为caching_sha2_password，而在此之前的加密规则为mysql_native_password。解决方法：

```mysql
mysql -u root -p
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '密码'; // USER是表名
```

