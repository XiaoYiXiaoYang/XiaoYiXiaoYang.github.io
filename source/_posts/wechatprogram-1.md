---
title: 微信小程序基础
date: 2020-01-08 21:08:46
tags: [微信小程序]
categories: [学习笔记]
---

 学习微信小程序开发必须了解的基本知识

<!--more-->

每次学习一个新的领域的知识，开始就需要熬几天，因为接触新的东西还需要一个适应的过程吧，看了一个网课虽然快速入门了，知道是知道了但是写起代码来就生涩的很了，基础语法啥的还是再过一遍。

**基本**
注册---AppID（可以发布，可以在手机预览）

**框架**
JSON：JSON 是一种数据格式，并不是编程语言，在小程序中，JSON扮演的静态配置的角色。

```
{
  "pages": ["pages/index/index", "pages/logs/logs"],
  "window": {
    "backgroundTextStyle": "light",
    "navigationBarBackgroundColor": "#fff",
    "navigationBarTitleText": "WeChat",
    "navigationBarTextStyle": "black"
  }
}
```
pages字段 —— 用于描述当前小程序所有页面路径，这是为了让微信客户端知道当前你的小程序页面定义在哪个目录。
window字段 —— 定义小程序所有页面的顶部背景颜色，文字颜色定义等。


WXML:网页编程采用的是 HTML + CSS + JS 这样的组合，其中 HTML 是用来描述当前这个页面的结构，CSS 用来描述页面的样子，JS 通常是用来处理这个页面和用户的交互。

同样道理，在小程序中也有同样的角色，其中 WXML 充当的就是类似 HTML 的角色。

```
<view class="container">
  <view class="userinfo">
    <button wx:if="{{!hasUserInfo && canIUse}}">获取头像昵称</button>
    <block wx:else>
      <image src="{{userInfo.avatarUrl}}" background-size="cover"></image>
      <text class="userinfo-nickname">{{userInfo.nickName}}</text>
    </block>
  </view>
  <view class="usermotto">
    <text class="user-motto">{{motto}}</text>
  </view>
</view>
```

WXSS:WXSS 具有 CSS 大部分的特性

JS:一个服务仅仅只有界面展示是不够的，还需要和用户做交互：响应用户的点击、获取用户的位置等等。在小程序里边，我们就通过编写 JS 脚本文件来处理用户的操作。

```
<view>{{ msg }}</view>
<button bindtap="clickMe">点击我</button>
```

点击 button 按钮的时候，我们希望把界面上 msg 显示成 "Hello World"，于是我们在 button 上声明一个属性: bindtap ，在 JS 文件里边声明了 clickMe 方法来响应这次点击操作：

```
Page({
  clickMe() {
    this.setData({msg: 'Hello World'})
  }
})
```
响应用户的操作就是这么简单，

此外你还可以在 JS 中调用小程序提供的丰富的 API，利用这些 API 可以很方便的调起微信提供的能力，例如获取用户信息、本地存储、微信支付等。在前边的 QuickStart 例子中，在 pages/index/index.js 就调用了 wx.getUserInfo 获取微信用户的头像和昵称，最后通过 setData 把获取到的信息显示到界面上。更多 API 可以参考文档 小程序的API 。

**渲染层和逻辑层**
其中 WXML 模板和 WXSS 样式工作在渲染层，JS 脚本工作在逻辑层。

小程序的渲染层和逻辑层分别由2个线程管理：渲染层的界面使用了WebView 进行渲染；逻辑层采用JsCore线程运行JS脚本。一个小程序存在多个界面，所以渲染层存在多个WebView线程，这两个线程的通信会经由微信客户端（下文中也会采用Native来代指微信客户端）做中转，逻辑层发送网络请求也经由Native转发，


**注册程序及程序生命周期**
注册小程序。接受一个 Object 参数，其指定小程序的生命周期回调等。

App() 必须在 app.js 中调用，必须调用且只能调用一次。不然会出现无法预期的后果。
onLaunch(Object object)
小程序初始化完成时触发，全局只触发一次。

onShow(Object object)
小程序启动，或从后台进入前台显示时触发。
onHide()
小程序从前台进入后台时触发。

AppObject getApp(Object object)
获取到小程序全局唯一的 App 实例。
不要在定义于 App 内的函数中，或调用 App 前调用 getApp ，使用 this 就可以拿到 app 实例。
通过 getApp 获取实例之后，不要私自调用生命周期函数。

**页面注册及页面生命周期**
Page(Object object)   注册小程序中的一个页面。

data	Object			页面的初始数据
onLoad	function			生命周期回调—监听页面加载
onShow	function			生命周期回调—监听页面显示
onReady	function			生命周期回调—监听页面初次渲染完成
onHide	function			生命周期回调—监听页面隐藏
onUnload	function			生命周期回调—监听页面卸载
onPullDownRefresh	function			监听用户下拉动作
onReachBottom	function			页面上拉触底事件的处理函数
onShareAppMessage	function			用户点击右上角转发
onPageScroll	function			页面滚动触发事件的处理函数
onResize	function			页面尺寸改变时触发，详见 响应显示区域变化
onTabItemTap	function			当前是 tab 页时，点击 tab 时触发
其他	any			开发者可以添加任意的函数或数据到 Object 参数中，在页面的函数中用 this 可以访问

**模块化**
可以将一些公共的代码抽离成为一个单独的 js 文件，作为一个模块

在需要使用这些模块的文件中，使用 require 将公共代码引入

```
// common.js
function sayHello(name) {
  console.log(`Hello ${name} !`)
}
function sayGoodbye(name) {
  console.log(`Goodbye ${name} !`)
}

module.exports.sayHello = sayHello
exports.sayGoodbye = sayGoodbye
```

```
const common = require('common.js')
Page({
  helloMINA() {
    common.sayHello('MINA')
  },
  goodbyeMINA() {
    common.sayGoodbye('MINA')
  }
})
```

**数据绑定与条件渲染**
数据绑定
WXML 中的动态数据均来自对应 Page 的 data。

```
<view>{{ message }}</view>

Page({
  data: {
    message: 'Hello MINA!'
  }
})
```

条件渲染
wx:if
在框架中，使用 wx:if="{{condition}}" 来判断是否需要渲染该代码块：

```
<view wx:if="{{condition}}">True</view>
也可以用 wx:elif 和 wx:else 来添加一个 else 块：

<view wx:if="{{length > 5}}">1</view>
<view wx:elif="{{length > 2}}">2</view>
<view wx:else>3</view>
```

block wx:if
因为 wx:if 是一个控制属性，需要将它添加到一个标签上。如果要一次性判断多个组件标签，可以使用一个 <block/> 标签将多个组件包装起来，并在上边使用 wx:if 控制属性。

```
<block wx:if="{{true}}">
  <view>view1</view>
  <view>view2</view>
</block>
```

列表渲染
wx:for
在组件上使用 wx:for 控制属性绑定一个数组，即可使用数组中各项的数据重复渲染该组件。

默认数组的当前项的下标变量名默认为 index，数组当前项的变量名默认为 item

```
<view wx:for="{{array}}">
  {{index}}: {{item.message}}
</view>
Page({
  data: {
    array: [{
      message: 'foo',
    }, {
      message: 'bar'
    }]
  }
})
```

block wx:for
类似 block wx:if，也可以将 wx:for 用在<block/>标签上，以渲染一个包含多节点的结构块。

wx:key
如果列表中项目的位置会动态改变或者有新的项目添加到列表中，并且希望列表中的项目保持自己的特征和状态（如 <input> 中的输入内容，<switch> 的选中状态），需要使用 wx:key 来指定列表中项目的唯一的标识符。

wx:key 的值以两种形式提供

字符串，代表在 for 循环的 array 中 item 的某个 property，该 property 的值需要是列表中唯一的字符串或数字，且不能动态改变。
保留关键字 *this 代表在 for 循环中的 item 本身，这种表示需要 item 本身是一个唯一的字符串或者数字，如：
当数据改变触发渲染层重新渲染的时候，会校正带有 key 的组件，框架会确保他们被重新排序，而不是重新创建，以确保使组件保持自身的状态，并且提高列表渲染时的效率。

如不提供 wx:key，会报一个 warning， 如果明确知道该列表是静态

**模板**

使用 name 属性，作为模板的名字。然后在<template/>内定义代码片段，如：

```
<!--
  index: int
  msg: string
  time: string
-->
<template is="msgItem" data="{{...item}}" />

<template name="msgItem">
  <view>
    <text>{{index}}: {{msg}}</text>
    <text>Time: {{time}}</text>
  </view>
</template>
```

使用 is 属性，声明需要的使用的模板，然后将模板所需要的 data 传入，如：

```
Page({
  data: {
    item: {
      index: 0,
      msg: 'this is a template',
      time: '2016-09-15'
    }
  }
})
```

**事件**
事件是视图层到逻辑层的通讯方式。
事件可以将用户的行为反馈到逻辑层进行处理。
事件可以绑定在组件上，当达到触发事件，就会执行逻辑层中对应的事件处理函数。
事件对象可以携带额外信息，如 id, dataset, touches。

在组件中绑定一个事件处理函数。
如bindtap，当用户点击该组件的时候会在该页面对应的Page中找到相应的事件处理函数。

```
<view id="tapTest" data-hi="WeChat" bindtap="tapName">Click me!</view>
```
在相应的Page定义中写上相应的事件处理函数，参数是event。
```
Page({
  tapName(event) {
    console.log(event)
  }
})
```

事件绑定和冒泡
事件绑定的写法同组件的属性，以 key、value 的形式。

value 是一个字符串，需要在对应的 Page 中定义同名的函数。不然当触发事件的时候会报错。
bind事件绑定不会阻止冒泡事件向上冒泡，catch事件绑定可以阻止冒泡事件向上冒泡。

如在下边这个例子中，点击 inner view 会先后调用handleTap3和handleTap2(因为tap事件会冒泡到 middle view，而 middle view 阻止了 tap 事件冒泡，不再向父节点传递)，点击 middle view 会触发handleTap2，点击 outer view 会触发handleTap1。

```
<view id="outer" bindtap="handleTap1">
  outer view
  <view id="middle" catchtap="handleTap2">
    middle view
    <view id="inner" bindtap="handleTap3">
      inner view
    </view>
  </view>
</view>
```

**引用**
WXML 提供两种文件引用方式import和include。

import
import可以在该文件中使用目标文件定义的template，如：

在 item.wxml 中定义了一个叫item的template：

```
<!-- item.wxml -->
<template name="item">
  <text>{{text}}</text>
</template>
```
在 index.wxml 中引用了 item.wxml，就可以使用item模板：

```
<import src="item.wxml" />
<template is="item" data="{{text: 'forbar'}}" />
```

include 可以将目标文件除了 <template/> <wxs/> 外的整个代码引入，相当于是拷贝到 include 位置，如：

```
<!-- index.wxml -->
<include src="header.wxml" />
<view>body</view>
<include src="footer.wxml" />
```

```
<!-- header.wxml -->
<view>header</view>
```

```
<!-- footer.wxml -->
<view>footer</view>
```

------------



*2019-03-31 11:05:50 星期日
萧逸小杨*