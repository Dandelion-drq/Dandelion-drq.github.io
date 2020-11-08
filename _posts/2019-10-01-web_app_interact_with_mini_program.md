---
layout: post
title: '网页如何与小程序交互通信'
categories: 前端
tags: 小程序 web
excerpt: '记录了网页与小程序交互通信的实现方式和一些坑。'
---


## 概述

网页与小程序交互和通信需要在小程序里使用 `web-view` 组件打开网页，而且被访问的网页需要引入微信的 `js-sdk`，通过 `js-sdk` 提供的接口来给小程序发信息。

具体可以参考小程序官方文档：[https://developers.weixin.qq.com/miniprogram/dev/component/web-view.html](https://developers.weixin.qq.com/miniprogram/dev/component/web-view.html)



## 小程序后台配置业务域名

这是小程序开发的限制，在 `web-view` 里要打开的网页的域名要先在微信公众平台小程序后台 `开发` --> `开发设置` --> `业务域名` 里添加一项，否则 `web-view` 无法打开这个网页。

PS：不想配置这个的话也可以在 `微信开发者工具` 里面开启 `不校验合法域名`。



## 网页引入`微信js-sdk`

`微信JS-SDK` 是微信提供的基于微信内的网页开发工具包。做过微信公众号网页开发的同学应该就知道了~~（那是出了名的难用……）~~。

官方文档链接：[https://developers.weixin.qq.com/doc/offiaccount/OA_Web_Apps/JS-SDK.html](https://developers.weixin.qq.com/doc/offiaccount/OA_Web_Apps/JS-SDK.html)



怎么引入我就不写了，同学们按上面的文档提示操作就好。

想用npm方式引用的话也可以找第三方，我用的是这个 [weixin-js-sdk](https://github.com/yanxi-me/weixin-js-sdk)。



## 网页向小程序发送消息

我做了个很简单的demo页，演示了导航到小程序页面和给小程序发消息的功能。

页面如下：

![](/assets/img/posts/2019-10/1.png)



页面是用 `vue` 写的，很简单，直接上代码了：

```html
<template>
    <section>
        <section>
            <h2>点击图片事件</h2>
            <div @click="onClick">
                <img src="https://via.placeholder.com/300x150" alt="">
            </div>
        </section>

        <section>
            <h2>与小程序交互</h2>
            <button @click="switchTab">回到小程序首页</button>
            <button @click="postMessage">给小程序发消息</button>
        </section>

    </section>
</template>

<script>
	const wx = require('weixin-js-sdk');
                    
    export default {
          data () {
            return {
                isMiniprogram: false, // 是否是小程序环境
            };
        },

        mounted () {
            this.wx = wx;
            this.wx.miniProgram.getEnv((res) => {
                console.log('getEnv', res, res.miniprogram);
                this.isMiniprogram = true;
            });
        },

        methods: {
            onClick () {
                console.log('点击图片', this.wx);
            },
            switchTab () {
                if (this.isMiniprogram) {
                    this.wx.miniProgram.switchTab({
                        url: '/pages/index/index',
                        success: function (res) {
                            console.log('success', res);
                        },
                        fail: function (err) {
                            console.log('fail', err);
                        },
                        complete: function (res) {
                            console.log('complete', res);
                        },
                    });
                }
            },
            postMessage () {
                if (this.isMiniprogram) {
                    this.wx.miniProgram.postMessage({
                        data: { foo: 'bar' },
                    });
                }
            },
        },
    };
</script>

<style lang="scss" scoped>
    button {
        padding: 10px;
        background: #333;
        color: #fff;
        border: none;
        margin-right: 10px;
        border-radius: 5px;
    }

    h2 {
        padding: 30px 0 20px 0;
    }

    section {
        text-align: center;
    }
</style>
```



这里有几个要注意的点：

- 导航到小程序页面的时候跟小程序开发的路由方法是一样的，即如果要导航到 `tabbar` 页面，不能用 `navigateTo` 方法，要用 `switchTab`，否则的话在小程序调试时不会发生任何跳转，而且居然没有任何错误提示。~~（我就是在这被坑了……）~~

- `wx.miniProgram.postMessage` 方法的参数格式是对象，而且要有 `data` 属性，即 `{data:...}` 这样，不然在小程序里收不到。
- 微信`js-sdk`官方文档说使用所有接口前都需要先配置权限，然而！！`miniProgram` 这一系列方法是可以不用 `config` 的……关于这个社区有人问过：[小程序跳转网页有bug吗](https://developers.weixin.qq.com/community/develop/doc/c540cf6d6c2840cda59c0010f6fd9b97)。因为我这个网页不想拿微信用户信息，所以这一点对我来说还是比较重要的。



## 小程序接收消息

小程序 `web-view` 的 `bindmessage` 不会在网页发送完消息后立刻收到，而是会在小程序后退、组件销毁、分享时触发并收到消息。而且如果网页发送了多次消息，在小程序接收的时候会把 `data` 合并。

> 网页向小程序 postMessage 时，会在特定时机（小程序后退、组件销毁、分享）触发并收到消息。e.detail = { data }，data是多次 postMessage 的参数组成的数组



直接看一下代码和输出

代码：

```html
<!-- wxml -->

<view class='web-view'>
  <web-view src="{{url}}" bindmessage="getMsgFromWeb" bindload="onWebLoad" binderror="onWebError"></web-view>
</view>
```

```js
// js

  getMsgFromWeb(e) {
    console.log('getMsgFromWeb', e.detail.data)
  },

  onWebLoad(e) {
    console.log('onWebLoad', e)
  },

  onWebError(e) {
    console.error('onWebError', e)
  }
```

输出：

![](/assets/img/posts/2019-10/2.png)

上面的截图是我点击了10次发送消息按钮，然后 `getMsgFromWeb` 的输出是在我点了回到小程序首页按钮后输出的。



## End

网页和小程序交互通信的基本实现方式就是这样啦，因为自己开发的时候被坑了一下，在此做个记录，希望对还在爬坑的同学有用~~