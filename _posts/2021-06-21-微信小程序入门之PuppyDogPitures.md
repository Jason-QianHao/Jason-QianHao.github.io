---
title: 微信小程序入门之PuppyDogPitures
tags: 微信小程序
---

# 项目介绍

- 前情

  作为后端开发人员，每次写项目时，设计到前端页面的设计就很头疼，每次都要到网上找好看的模版，但哪能每次都能找得到呢（哭泣）。从html到html5，从css到js，各种文件跳转，让人头大。现在是2021年的6月份，马上快要进行今年的提前批的招聘，我准备的比较早，所以也很是快点迎接战斗，但是这不，只有几家开了提前批，bat还要等到7月初，一些机缘巧合下，就开始了前端学习（前端笔记[传送门]())。了解了一些前端知识后，觉得还是要实践，然而pc端服务器端的设计架构我基本已经掌握了，客户端移动端的设计却是只知道皮毛。之前了解过andriod、ios开发，app的开发比较笨重，学习起来会比较费时间（这不是还要大战offer了嘛)，所以就选择了微信小程序

- 正题

  这篇是微信小程序的入门项目，由豆瓣电影相关的小程序启发而来（因为现在豆瓣的api，我好像用不了了)，在github上查了很多api，最后选用了https://shibe.online，可以随机生成一些狗狗图片。




# 环境

- macOs
- 微信开发者工具
- 微信公众平台账号

# 项目地址

[PuppyDogPictures](https://github.com/Jason-QianHao/PuppyDogPictures)

# 系统设计

## 结构设计

 小程序有3个页面：

- 主页，随机推荐的10张狗狗图片

  ![image-20210621131354042](/../assets/%E5%BE%AE%E4%BF%A1%E5%B0%8F%E7%A8%8B%E5%BA%8F%E5%85%A5%E9%97%A8%E4%B9%8BPuppyDogPitures/image-20210621131354042.png)

- 搜索页，可以输入想要的狗狗图片数量，自动生成相应数量的狗狗图片

  ![image-20210621131438880](/../assets/%E5%BE%AE%E4%BF%A1%E5%B0%8F%E7%A8%8B%E5%BA%8F%E5%85%A5%E9%97%A8%E4%B9%8BPuppyDogPitures/image-20210621131438880.png)

- 个人页，简单的介绍

## 设计思路

1. 编写app.json文件，根据想达到的效果设置pages，window、tabBar等

   注意在编写过程中，对配置项有任何的不熟悉，都要及时查阅[微信官方小程序文档](https://developers.weixin.qq.com/miniprogram/dev/framework/)，文档写的很详细了，一来二去可以让我更熟悉文档了

2. 根据需要编写app.wxss和app.js文件

   app.wxss文件是设置小程序的全局样式，我这个新手小白选择设置一些容器的高度、颜色啥的。

   app.js文件在本项目中，没有使用可以删除官方模版中的一些代码，但是一定要保留`App({})`的定义！！！

3. 接下来就是编写每个页面了，以index页面为例

   我的习惯是，由需求到逻辑，就是先写wxml文件，先确定了页面的大致内容，然后再去js文件编写相应的事件。比如这里需要一组图片的地址，而图片的地址又是需要通过http/https请求而来，所以就很明确js的编写逻辑。这里调用wx.request编写请求的时候有些坑，我在文档末尾总结。

# 项目编译和预览

- 编译

  工具好像默认每次保存和启动的时候编译，mac环境可以在`设置-通用设置`进行自己的习惯性设置，我这里设置为了手动编译，点击编译按钮对修改的编译进行编译，可以在调试器中看见。

  ![image-20210621152516700](/../assets/%E5%BE%AE%E4%BF%A1%E5%B0%8F%E7%A8%8B%E5%BA%8F%E5%85%A5%E9%97%A8%E4%B9%8BPuppyDogPitures/image-20210621152516700.png)

- 预览

  这里可以在左边调试器中直接预览结果，也可以点击预览按钮，在手机上进行**开发版**的预览。

  <img src="/../assets/%E5%BE%AE%E4%BF%A1%E5%B0%8F%E7%A8%8B%E5%BA%8F%E5%85%A5%E9%97%A8%E4%B9%8BPuppyDogPitures/image-20210621153036868.png?height=" alt="image-20210621153036868" style="zoom:40%;" />

- **注意**：如果想让别人也可以体验小程序，需要进行相关的授权

1. 此项目需要appId，这个在[小程序官网](https://mp.weixin.qq.com/wxamp/devprofile/get_profile?token=1165682976&lang=zh_CN)注册账号即可拥有

2. 配置好后，点击上传即可，也会在官网`管理-版本管理-开发管理`中看到上传到项目

   <img src="/../assets/%E5%BE%AE%E4%BF%A1%E5%B0%8F%E7%A8%8B%E5%BA%8F%E5%85%A5%E9%97%A8%E4%B9%8BPuppyDogPitures/image-20210621153915199.png" alt="image-20210621153915199" style="zoom:50%;" />

   <img src="/../assets/%E5%BE%AE%E4%BF%A1%E5%B0%8F%E7%A8%8B%E5%BA%8F%E5%85%A5%E9%97%A8%E4%B9%8BPuppyDogPitures/image-20210621153835028.png" alt="image-20210621153835028" style="zoom:30%;" />

3. 一个账号只能由一个**体验/线上**的小程序。但是开发过程中，可以使用重复的appId，如果上传代码后，就会覆盖之前的项目和代码，慎重！！！

# 关于wx.request

​	这个api是用来发送网络请求的，但是使用过程中友谊的坑，这里记录2个：

1. url属性

   [官方文档](https://developers.weixin.qq.com/miniprogram/dev/framework/ability/network.html)说明了一些使用注意事项如下：

   <img src="/../assets/%E5%BE%AE%E4%BF%A1%E5%B0%8F%E7%A8%8B%E5%BA%8F%E5%85%A5%E9%97%A8%E4%B9%8BPuppyDogPitures/image-20210621154253150.png" alt="image-20210621154253150" style="zoom:50%;" />

   本项目中，就是使用了https协议，但是这个api没有经过ICP备案，所以只能选用第二个方案，在开发设置中关掉https等相关的检测。参考[这里](https://blog.csdn.net/qq_35132089/article/details/106372398)但是迎面而来的问题就是，手机端预览时，必须打开调试模式，真的无语～

2. 如果方法内部，需要调用setData方法修改页面的一些字段，不可以调用this，需要先赋值给其他变量

```javascript
onLoad: function () {
    var that = this; //这里需要赋值
    wx.showToast({ 
      title: '加载中/...',
      icon: 'loading',
      duration: 100000
    });
    wx.request({
      url: API_URL,
      header: {
        'content-type': 'application/json' // 默认值
      },
      success(res) {
        wx.hideToast();
        // console.log(res.data);
        that.setData({
          imageUrls: res.data
        })
      }
    });
}
```

# 总结

​	新手起步总是磕磕绊绊啊，以上如果有错误，还请批评指正～
