---
title: 微信小程序（项目新闻列表）
date: 2020-01-08 21:13:37
tags: [微信小程序]
categories: [学习笔记]
---

 进一步学习api接口查询数据库信息返回到小程序

<!--more-->

**新闻页面**

**前台页面搭建**
源码

```
<view class="doc-container">
  <view class="doc-title">今日精选</view>
  <block wx:for="{{weather}}">
    <view>{{item.Code}}</view>
    <view>{{item.WeatherDate}}</view>
    <view>{{item.WeatherInfo}}</view>
    <view>{{item.WeatherWindLevel}}</view>
    
  </block>
  
  <block wx:for="{{feeds}}" wx:key="{{item.ArticleId}}">
    <view class="list" data-para="{{item}}" bindtap="tapItem">
      <view class="view_preinfo">
        <text class="list_preinfo">{{item.CreateDateTime}} / {{item.ArticleAuthor}}</text>
      </view>
      <text class="list_title">{{item.ArticleTitle}}</text>
      <view>
        <block wx:for="{{item.Tags}}" wx:key="{{item.TagName}}">
          <text class="list_tag" style="border: solid 1px {{item.BackgroundColor}};">{{item.TagName}}</text>
        </block>
      </view>
    </view>
  </block>
</view>
```

```
.list {
  margin: 10px;
  border: 1px solid #d8d8d8;
  border-radius: 5px;
  background-color: #fff;
  padding: 10px;
  box-shadow:0px 2px 5px #efefef;
}

.list_preinfo {
  color: #cecece;
  font-size: 11px;
}

.list_title {
  color: #545454;
  padding-top: 3px;
  padding-bottom: 3px;
  display: block;
  font-size: 15px;
}

.list_tag {
  display: inline;
  padding: 0.1em 0.3em 0.1em 0.3em;
  margin-right: 3px;
  font-size: 11px;
  line-height: 1;
  text-align: center;
  white-space: nowrap;
  vertical-align: baseline;
  border-radius: 0.25em;
  color: #626262;
  font-weight: normal;
}
```

**页面请求api及小程序网络请求详解**

这才是我关注的重点重点

用Chrome进行网络请求抓包调试

API地址要求
备案
在微信小程序后台允许
HTTPS


```
Page({
  data: {
    feeds: [],
    page: 1,
    weather: []
  },
  onLoad: function() {
    var that = this;
    this.getFeeds(that.data.page);
  },
  onReachBottom: function() {
    wx.showLoading({
      title: '加载更多文章',
    })
    var that = this;
    this.getFeeds(that.data.page);
  },
  getFeeds: function(page) {
    if (page == 1) {
      wx.showLoading({
        title: '加载中',
      })
    }
    var that = this
    //测试天气预报接口
    wx.request({
      url: 'https://api.gugudata.com/weather/weatherinfo?appkey=请去www.gugudata.com申请key&code=101020500',
      method: 'get',
      header: {
        'content-type': 'application/json'
      },
      success: function (res) {
        var res = res.data;
        that.setData({
          weather: res.Data
        })
      },

      fail: function () {
        wx.showToast({
          title: '服务器异常',
          duration: 1500
        })
      },
      complete: function () {
        
      }
    });
    wx.request({
      url: 'https://api.gugudata.com/news/techblogs?appkey=请去www.gugudata.com申请key&pageindex=' + page + '&pagesize=10',
      method: 'get',
      header: {
        'content-type': 'application/json'
      },
      success: function(res) {
        var res = res.data;

        if (that.data.page > 1) {
          var feedTemp = that.data.feeds;
          that.setData({
            feeds: feedTemp.concat(res),
            page: page + 1
          })
        } else {
          console.log(res);
          that.setData({
            feeds: res,
            page: page + 1
          })
        }
      },
      
      fail: function() {
        wx.showToast({
          title: '服务器异常',
          duration: 1500
        })
      },
      complete: function() {
        if (page >= 1) {
          wx.hideLoading()
        } else {
          //wx.stopPullDownRefresh()
        }
      }
    })
  },
  //单项的点击事件
  tapItem: function(event) {
    var that = this; 
    var article = event.currentTarget.dataset.para;
    wx.navigateTo({
      url: "/pages/details/details?id=" + article.ArticleId
    })
  }
})
```

------------

*2019-04-03 12:03:00 星期三
萧逸小杨*