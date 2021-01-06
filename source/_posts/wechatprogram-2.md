---
title: 微信小程序（散知识点）
date: 2020-01-08 21:11:21
tags: [微信小程序]
categories: [学习笔记]
---

 实战一个咕咕监控小项目

<!--more-->

**网络API请求与列表绑定**

wx.canIUse	判断小程序的API，回调，参数，组件等是否在当前版本可用
wx.request(Object object)    发起 HTTPS 网络请求。

url	string		是	开发者服务器接口地址	
data	string/object/ArrayBuffer		否	请求的参数	
header	Object		否	设置请求的 header，header 中不能设置 Referer。
content-type 默认为 application/json	
method	string	GET	否	HTTP 请求方法	
dataType	string	json	否	返回的数据格式	
responseType	string	text	否	响应的数据类型	1.7.0
success	function		否	接口调用成功的回调函数	
fail	function		否	接口调用失败的回调函数	
complete	function		否	接口调用结束的回调函数（调用成功、失败都会执行）

```
wx.request({
  url: 'test.php', // 仅为示例，并非真实的接口地址
  data: {
    x: '',
    y: ''
  },
  header: {
    'content-type': 'application/json' // 默认值
  },
  success(res) {
    console.log(res.data)
  }
})
```

最终发送给服务器的数据是 String 类型，如果传入的 data 不是 String 类型，会被转换成 String 。转换规则如下：

对于 GET 方法的数据，会将数据转换成 query string（encodeURIComponent(k)=encodeURIComponent(v)&encodeURIComponent(k)=encodeURIComponent(v)...）
对于 POST 方法且 header['content-type'] 为 application/json 的数据，会对数据进行 JSON 序列化
对于 POST 方法且 header['content-type'] 为 application/x-www-form-urlencoded 的数据，会将数据转换成 query string （encodeURIComponent(k)=encodeURIComponent(v)&encodeURIComponent(k)=encodeURIComponent(v)...）

**底部导航**

```
"tabBar": {
    "list": [
      {
        "pagePath": "pages/user/login",
        "text": "首页",
        "iconPath":"imgs/icon_speed.png",
        "selectedIconPath":"imgs/icon_speed_HL.png"
      },
      {
        "pagePath": "pages/index/index",
        "text": "个人中心",
        "iconPath":"imgs/icon_account.png",
        "selectedIconPath":"imgs/icon_account_HL.png"
      }
    ]
  }
```

**登录页面**

```
<!--logs.wxml-->
<view class="page">
    <view class="page__bd">
      <view class="section">
        <input type="text" bindinput="bindEmailInput" placeholder="邮箱" auto-focus />
      </view>
      <view class="section">
        <input type="password" bindinput="bindPasswordInput" placeholder="密码" />
      </view>
      <view class="section">
        <view class="btn-area">
           <button type="primary" ontap="login">登录</button>
        </view>
      </view>
</view>
```
wx.showToast函数，显示消息提示框 （登录）
wx.hideToast(Object object)  隐藏消息提示框
wx.showModal(Object object)  显示模态对话框（登录失败）

wx.reLaunch(Object object)  关闭所有页面，打开到应用内的某个页面
wx.switchTab(Object object)  跳转到 tabBar 页面，并关闭其他所有非 tabBar 页面 （路由跳转）
wx.navigateTo(Object object)  保留当前页面，跳转到应用内的某个页面。但是不能跳到 tabbar 页面。使用 wx.navigateBack 可以返回到原页面。
wx.navigateBack(Object object)  关闭当前页面，返回上一页面或多级页面。

获取表单值 bindEmailInput,bindPasswordInput

```
Page({
	data: {
		email: '',
		password: ''
	},
	bindEmailInput: function(e) {
		this.setData({email: e.detail.value})
	},
	bindPasswordInput: function(e) {
		this.setData({password: e.detail.value})
	},
	login: function(e) {
		wx.showToast({title: '登录请求中', icon: 'loading', duration: 10000});
		//网络请求开始
		wx.request({
			url: 'https://api.gugujiankong.com/account/Login?email=' + this.data.email + '&password=' + this.data.password,
			header: {
				'Content-Type': 'application/json'
			},
			success: function(res) {
				wx.hideToast();
				if (res.data.LoginStatus == 1) {
					//进行一些用户状态的存储
					//进行 tab 的切换
					wx.switchTab({
						url:'../../pages/index/index',
						success:function(){
							console.log("called switchtab.");
						}
					});
				} else {
					wx.showModal({title: '登录失败', content: '请检查您填写的用户信息！', showCancel: false, success: function(res) {
							//回调函数
						}});
				}
			}
		});
	}
})

```

**工具**
网址：ssl-checker.online-domain-tools.com
可以查看网站详细的ssl检测，后台接口一定要支持https检测

IPV6检测
网址：ipv6-test.com/validate.php

**地图的基本概念与组件的使用**

参考https://developers.weixin.qq.com/miniprogram/dev/component/map.html

```
<!-- map.wxml -->
<map
  id="map"
  longitude="113.324520"
  latitude="23.099994"
  scale="14"
  controls="{{controls}}"
  bindcontroltap="controltap"
  markers="{{markers}}"
  bindmarkertap="markertap"
  polyline="{{polyline}}"
  bindregionchange="regionchange"
  show-location
  style="width: 100%; height: 300px;"
></map>
```

对应的js文件

```
// map.js
Page({
  data: {
    markers: [{
      iconPath: '/resources/others.png',
      id: 0,
      latitude: 23.099994,
      longitude: 113.324520,
      width: 50,
      height: 50
    }],
    polyline: [{
      points: [{
        longitude: 113.3245211,
        latitude: 23.10229
      }, {
        longitude: 113.324520,
        latitude: 23.21229
      }],
      color: '#FF0000DD',
      width: 2,
      dottedLine: true
    }],
    controls: [{
      id: 1,
      iconPath: '/resources/location.png',
      position: {
        left: 0,
        top: 300 - 50,
        width: 50,
        height: 50
      },
      clickable: true
    }]
  },
  regionchange(e) {
    console.log(e.type)
  },
  markertap(e) {
    console.log(e.markerId)
  },
  controltap(e) {
    console.log(e.controlId)
  }
})
```

**小程序的flex布局**

参考：http://cssreference.parryqiu.com/
参考：https://developers.weixin.qq.com/miniprogram/dev/component/view.html


**用户登录及用户信息获取**

wx.getUserInfo(Object object)调用前需要 用户授权 scope.userInfo。
获取用户信息。

```
wx.getUserInfo({
  success(res) {
    const userInfo = res.userInfo
    const nickName = userInfo.nickName
    const avatarUrl = userInfo.avatarUrl
    const gender = userInfo.gender // 性别 0：未知、1：男、2：女
    const province = userInfo.province
    const city = userInfo.city
    const country = userInfo.country
  }
})
```

------------

*2019-03-31 23:09:29 星期日
萧逸小杨*