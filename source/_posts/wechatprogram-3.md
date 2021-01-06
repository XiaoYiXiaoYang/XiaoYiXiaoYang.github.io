---
title: 微信小程序（项目个人中心）
date: 2020-01-08 21:12:29
tags: [微信小程序]
categories: [学习笔记]
---

 通过搭建实战项目快速学习上手

<!--more-->

**知识点梳理**
基础知识差不多就是前2节整理的，相似的内容可以到小程序开发文档中查看使用
小程序开发好发布后通过腾讯服务器返回页面，而我们还会使用api在后台借助于自己的服务器为小程序提供服务。

更好用的是腾讯云，一体化，一键...可是我没有

数据源：http://techfoco.com/

**个人中心页面**

新建文件夹usercenter，在文件夹下新建pages命名usercenter，则会自动新建出usercenter.wxml .wss .js .json几个文件，而且会自动在app.json下配置pages
```
 "pages": [
    "pages/index/index",
    "pages/logs/logs",
    "pages/usercenter/usercenter"
  ],
```

接着设置tabBar
```
 "tabBar": {
    "selectedColor": "#1AAD16",
    "list": [
      {
        "pagePath": "pages/index/index",
        "text": "头条",
        "iconPath": "images/news.png",
        "selectedIconPath": "images/news_s.png"
      },
      {
        "pagePath": "pages/fav/fav",
        "text": "收藏",
        "iconPath": "images/star.png",
        "selectedIconPath": "images/star_s.png"
      },
      {
        "pagePath": "pages/usercenter/usercenter",
        "text": "个人中心",
        "iconPath": "images/user.png",
        "selectedIconPath": "images/user_s.png"
      }
    ]
  }
```

设置小程序名字"navigationBarTitleText": "TechFoco 技术博文头条",

weui：https://github.com/Tencent/weui
weuicss：https://github.com/Tencent/weui-wxss

下载weui.css  重命名.wxss
放在同一级目录下
在app.wxss导入 @import 'weui.wxss'

在小程序项目中使用，说白了看中哪个github的组件，赋值粘贴，改成微信小程序的html

显示用户头像和昵称
```
 <!-- 如果只是展示用户头像昵称，可以使用 <open-data /> 组件 -->
   <view class="userinfo">  
      <view class="userinfo-avatar">
        <open-data type="userAvatarUrl"></open-data>
      </view>
       <open-data type="userNickName"></open-data>
    </view>
```

js代码

```
Page({
  data: {
    canIUse: wx.canIUse('button.open-type.getUserInfo')
  },
  
  onLoad: function () {
    // 查看是否授权
    wx.getSetting({
      success: function (res) {
        if (res.authSetting['scope.userInfo']) {
          // 已经授权，可以直接调用 getUserInfo 获取头像昵称
          wx.getUserInfo({
            success: function (res) {
              console(res.userInfo)
            }
          })
        }
      }
    })
  },
})
```

**登录逻辑**

同一个用户关注了不同的小程序，会对应不同的openid
用户下次进入小程序还可以根据用户的openid读取数据库信息

小程序请求code，然后发送到第三方服务器，第三方服务器将appid，appsecret，code发送到腾讯微信服务器，然后腾讯微信服务器会返回session_key，openid/unionid，第三方服务器再通过openid查询数据库，返回数据

用户关注了一个小程序，小程序就可以获取到唯一的openid对应于这个用户
用户又关注了另一个小程序，小程序又获取到一个openid唯一对应这个用户，两个openid不一样

用户关注了A,B,C三个小程序，有3个不同的openid，但是这三个小程序是同一家公司旗下的产品，这时三个小程序拥有唯一对应于这个用户的unionid

session_key  数据签名校验
https://developers.weixin.qq.com/miniprogram/dev/api/wx.checkSession.html

授权 wx.authorize(Object object)
提前向用户发起授权请求。调用后会立刻弹窗询问用户是否同意授权小程序使用某项功能或获取用户的某些数据，但不会实际调用对应接口。如果用户之前已经同意授权，则不会出现弹窗，直接返回成功。

*登录并获取openid*
注意后台api设置 接口id

```
//app.js
App({
  onLaunch: function () {
    var that = this
    wx.login({
      success: function (res) {
        if (res.code) {
          //发起网络请求
          wx.request({
            url: 'https://api.techfoco.com/account/wxlogin',
            data: {
              code: res.code
            },
            header: { //这里写你借口返回的数据是什么类型，这里就体现了微信小程序的强大，直接给你解析数据，再也不用去寻找各种方法去解析json，xml等数据了
              'Content-Type': 'application/json'
            },
            success: function (res) {
              var res = JSON.parse(res.data);
              that.globalData.openid = res.openid;
            }
          })
        } else {
          console.log('登录失败！' + res.errMsg)
        }
      }
    });
  },
  globalData: {
    openid: ''
  }
})
```

------------

*2019-04-01 21:55:50 星期一
萧逸小杨*