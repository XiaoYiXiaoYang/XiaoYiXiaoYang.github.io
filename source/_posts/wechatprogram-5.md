---
title: 微信小程序（详情，收藏）
date: 2020-01-08 21:14:39
tags: [微信小程序]
categories: [学习笔记]
---

 继续学习项目新闻列表绑定

<!--more-->

**列表页面数据请求与绑定**

首先是请求的数据一页一页显示，请求到的新数据和旧数据进行拼接，
只请求一页内容

前台绑定 for循环，wx:for=feeds wx:key=article_id,则将feeds看做item，进行绑定输出
里面套循环wx:for=item.Tags  wx:key=item.tagName
驻足里面套数组

json  onReachBottomDistance :10px;滑动到距离底部10px，触发加载更多

```
data: {
    feeds: [],
    page: 1,
    weather: []
  },
```

```
if (that.data.page > 1) {
          var feedTemp = that.data.feeds;
          that.setData({
            feeds: feedTemp.concat(res), //拼接
            page: page + 1
          })
        } else {
          console.log(res);
          that.setData({
            feeds: res,
            page: page + 1
          })
        }
```

**带参数跳转详情页**

```
<view class="list" data-para="{{item}}" bindtap="tapItem">
```

参数item 触发函数tapItem
 var article = event.currentTarget.dataset.para;页面传到后端

```
//单项的点击事件
  tapItem: function(event) {
    var that = this; 
    var article = event.currentTarget.dataset.para;
    wx.navigateTo({
      url: "/pages/details/details?id=" + article.ArticleId
    })
  }
```

**详情页面开发**

```
<rich-text nodes="{{myrich}}"></rich-text>
```

```
// pages/details/details.js.js
var appInstance = getApp();
Page({

  /**
   * 页面的初始数据
   */
  data: {
   id : 0,
   article : '',
   myrich : ''
  },

  /**
   * 生命周期函数--监听页面加载
   */
  onLoad: function (options) {
    console.log(appInstance.globalData.openid)
    var that = this;
    that.setData({
      id: options.id
    });
    this.getArticle(that.data.id);
  },
  //通过 API 获取文章的详情
  getArticle : function (id) {
    wx.showLoading({
      title: '加载中',
    });
    var that = this; // this 指向性的问题

    wx.request({
      url: 'https://api.techfoco.com/feed/getarticle?articleid=' + id,
      method: 'get',
      header: {
        'content-type': 'application/json'
      },
      success: function (res) { //请求成功
        var res = JSON.parse(res.data);
        that.setData({
          article: res,
          myrich : res.Content
        })
      },
      fail: function () {
        wx.showToast({
          title: '服务器异常',
          duration: 1500
        })
      },
      complete: function () { //请求完成的时候，不管请求成功还是失败
        wx.hideLoading();
      }
    })

  }
})
```

豆瓣API 电影...

**收藏功能**

收藏按钮
功能：传参openid articleid，传送到数据库
收藏页：读取opend 的文章

获取openid：app里已经获取到了GlobalData中

```
var ins = getApp()
ins.GlobalData.openid
```

**优化原则及常见方案**

优化工具 Trace
性能面板
优化建议：setData
图片https://developers.weixin.qq.com/miniprogram/dev/framework/performance/


**上传、审核**

上传按钮
版本号

登录微信公众平台-》开发管理-》开发版本-》提交审核/体验版

**资源**
wxopen.club



------------

*
2019-04-07 21:49:51 星期日
萧逸小杨
*