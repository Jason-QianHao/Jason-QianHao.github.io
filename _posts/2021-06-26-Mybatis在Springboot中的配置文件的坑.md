---
title: Mybatis在Springboot中的配置文件的坑
tags: ORM
---

## Mybatis的Maven依赖，如果引用了，但是没有配置mysql会在启动报错

这个地方主要在maven项目中含有父模块、子模块的时候可能会碰到，注意一下可以将Mybatis的依赖放到需要使用mysql的子模块中，就不要放到父模块中统一配置了

## 连接MySql报错Unable to load authentication plugin 'caching_sha2_password'

错误的原因是由于MySQL8.0之后的加密规则为caching_sha2_password，而在此之前的加密规则为mysql_native_password。解决方法：

```mysql
mysql -u root -p
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '密码'; // USER是表名
```

## mysql异常java.math.BigInteger cannot be cast to java.lang.Long解决

- 报错：

  ```
  org.apache.hadoop.hive.metastore.HiveMetaException: Failed to get schema version.
  Underlying cause: java.sql.SQLException : java.lang.ClassCastException: java.math.BigInteger cannot be cast to java.lang.Long
  ```

- 解决

  原来项目的jar包版本是5.1.27会报错，换成5.1.46就ok了：mysql-connector-java-5.1.46，版本问题！

## Unknown system variable 'query_cache_size' 

解决参考：[版本问题](https://blog.csdn.net/m0_46975599/article/details/105939399)

