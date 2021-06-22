---
title: Redis环境搭建（MacOS)
tags: Redis
---

# Redis环境搭建（MacOS)

# 环境

- MacOS

# Redis配置步骤

​	有两种方案可以配置Redis，【1】官网下载安装包方式【2】通过homebrew安装方式

## 官网下载安装包方式

1. 下载安装包

   去[Redis官网下载](https://redis.io/download)**Stable** 稳定版
   
   <img src="/../assets/Redis/image-20210622143154263.png" alt="image-20210622143154263" style="zoom:40%;" />

2. 安装

   这里通过命令行安装：

- 编译测试：`sudo make test`

- 编译安装：`sudo make install`

  安装完后可以看见目录如下：

<img src="/../assets/Redis/image-20210622144110428.png" alt="image-20210622144110428" style="zoom:50%;" />

3. Redis配置

   注意这里坑比较多！！！先找到`redis.conf`文件，这是redis的配置文件，我们一般需要配置的是服务器的登陆密码。可以打开文件后搜索`requirepass` 修改后面的的秘密即可，注意还要去掉前面的`#`。

<img src="/../assets/Redis/image-20210622144447552.png" alt="image-20210622144447552" style="zoom:40%;" />

4. 启动Redis服务

   这里是个大坑！！！注意到上一步我们设置了conf文件，如果想要配置生效，启动的时候需要加上conf文件。直接打开终端，将conf和`redis-server`文件拖入，后者先cd进入上面的redis解压目录。

   <img src="/../assets/Redis/image-20210622144847683.png" alt="image-20210622144847683" style="zoom:50%;" />

<img src="/../assets/Redis/image-20210622145120931.png" alt="image-20210622145120931" style="zoom:50%;" />

​	然后回车即可看见服务正常启动，看到下面的界面就可以了。

<img src="/../assets/Redis/image-20210622145342686.png" alt="image-20210622145342686" style="zoom:50%;" />

5. 注意

   上述带有conf文件的启动方式，即使你通过`control+c`关掉服务后，redis服务会依然在后台启动，若是修改配置后重启，可以在mac的`活动监视器`搜索`redis	`关掉服务，再通过步骤4启动服务。

<img src="/../assets/Redis/image-20210622145259965.png" alt="image-20210622145259965" style="zoom:40%;" />

## 通过homebrew安装方式

1. 需要先配置homebrew

   网上与很多相关的博客，我这里汇总了[mac下常见的服务和软件安装方式汇总](https://jason-qianhao.github.io/_posts/2021-06-22-Mac下常见的服务和软件安装方式汇总/)，找到`homebrew`的安装方式。

2. 通过homebrew配置redis

   这个网上也是有很多了，试试这个[传送门](https://blog.csdn.net/qq_41855420/article/details/103691030)

# Redis服务层代码

​	配置好Redis是开发的第一步，如何使用才是关键，这里列出我平时通过Springboot使用Redis时的部分代码。我一般都是把Redis放在service层，供其他业务service调用。

1. Redis的yaml配置

```yaml
spring:
#redis
   redis:
      host: localhost
      password: 123456
      port: 6379
      pool:
         max-idle: 100
         min-idle: 1
         max-active: 1000
         max-wait: -1
```

2. RedisSerice

   列出的是Redis五种数据类型的crud，方法的传入参数可以根据自己的业务逻辑相应修改。

```java
package com.qian.service;

import java.util.HashMap;
import java.util.HashSet;
import java.util.List;
import java.util.Map;
import java.util.Map.Entry;
import java.util.Set;
import java.util.concurrent.TimeUnit;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Component;

@Component
public class RedisService {

	@Autowired
	private RedisTemplate<Object, Object> redisTemplate;

	/*
	 * 字符串类型操作
	 */
	/**
	 * redisTemplate操作普通字符串(存值)
	 * 
	 * @param key
	 * @param value
	 */
	public void redisSetString(String key, String value) {
		redisTemplate.opsForValue().set(key, value);
	}

	/**
	 * redisTemplate操作普通字符串 （取值）
	 * 
	 * @param key
	 */
	public Object redisGetString(String key) {
		return redisTemplate.opsForValue().get(key);
	}

	
	/*
	 * 列表类型操作
	 */
	/**
     * 将一个list集合存放到redis当中
     * 
     * @param key
     */
    public void redisSetList(String key, List<Object> list) {
//        List<Integer> list = Arrays.asList(9, 2, 3, 4);
        for (Object obj : list) {
            // 从当前的数据 向右添加 
             redisTemplate.opsForList().rightPush(key, obj);
            // 从当前的数据 向左添加 
//            redisTemplate.opsForList().leftPush(key, obj);
        }
    }
    
    /**
     * 获取list(获取所有的数据)
     * 
     * @param key
     * @return
     */
    public Object getList(String key) {
        return redisTemplate.opsForList().range(key, 0, getListSize(key));
    }
    
    /**
     * 获取list指定key的长度
     * 
     * @param key
     * @return
     */
    public Long getListSize(String key) {
        return redisTemplate.opsForList().size(key);
    }
    
    
	/*
	 * 哈希类型操作
	 */
    /**
     * 将map存放到reids
     *
     * @param key
     */
    public void setHash(String key) {
        Map<String, String> hashMap = new HashMap();
        //使用RedisTemplate  有些情况会乱码
        hashMap.put("redis", "redis");
        hashMap.put("mysql", "mysql");
        for (Entry<String, String> keyValue : hashMap.entrySet()) {
            redisTemplate.opsForHash().put(key, keyValue.getKey(), keyValue.getValue());
        }
    }
    
    /**
     * 获取指定key1的值
     * 
     * @param key
     * @param key1
     * @return
     */
    public Object getHash(String key, String key1) {
        // 检测 是否 存在该键
        boolean isKey = redisTemplate.opsForHash().hasKey(key, key1);
        return redisTemplate.opsForHash().get(key, key1);
    }
    
    /**
     * 获取指定key的所有值
     * 
     * @param key
     * 
     * @return
     */
    public Object getHash(String key) {
        return redisTemplate.opsForHash().entries(key);
    }
    
    /**
     * 根据具体key移除具体的值
     * 
     * @param key
     * 
     * @return
     */
    public void removeKey(String key, String key1) {
        redisTemplate.opsForHash().delete(key, key1);
    }
    
    /**
     * 移除key值 则key里面的所有值都被移除
     * 
     * @param key
     * 
     * @return
     */
    public void removeStringKey(String key) {
        redisTemplate.delete(key);
    }
    
    
	/*
	 * 集合类型操作
	 */
    /**
     * set存入redis中
     * 
     * @param key
     */
    public void setSet(String key) {
        Set<Object> set = new HashSet();
        set.add("setKey");
        set.add("tesetKey");
        for (Object object : set) {
            redisTemplate.opsForSet().add(key, object);
        }
    }
    
    /**
     * 从redis中取出set
     * 
     * @param key
     * @return
     */
    public Object getSet(String key) {
        return redisTemplate.opsForSet().members(key);
    }
    
    
	/*
	 * 有序集合类型操作
	 */
    /**
     * sortset存入redis中
     * 
     * @param key
     */
    public void setZSet(String key) {
        Set<Object> set = new HashSet();
        set.add("setKey");
        set.add("tesetKey");
        int i = 0;
        for (Object object : set) {
            i++;
            redisTemplate.opsForZSet().add(key, object, i);
        }
    }
    
    /**
     * 从redis中取出sortset
     * 
     * @param key
     * @return
     */
    public Object getZSet(String key) {
        Long size = redisTemplate.opsForZSet().size(key);
        return redisTemplate.opsForZSet().rangeByScore(key, 0, size);
    }
    
    
    /*
     * RedisTemplate操作5种基本类型数据，有一些共同的API 
     */
    /**
     * 
     * 指定缓存失效时间
     * 
     * @param key
     *            键
     * 
     * @param time
     *            时间(秒)
     * 
     * @return
     * 
     */
    public boolean expire(String key, long time) {
        try {
            if (time > 0) {
                redisTemplate.expire(key, time, TimeUnit.SECONDS);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }
    
    /**
     * 
     * 判断key是否存在
     * 
     * @param key
     *            键
     * 
     * @return true 存在 false不存在
     * 
     */
    public boolean checkKey(String key) {
        try {
            return redisTemplate.hasKey(key);
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }
    
}

```
