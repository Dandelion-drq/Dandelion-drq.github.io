---
layout: post
title: '小程序性能优化实践总结'
categories: 前端
tags: 小程序 性能优化
excerpt: '记一次微信小程序性能优化实践。'
---

## 项目简述 & 问题

先简单介绍一下项目，就是一个比较常规的点餐小程序。

界面如图：

![](/assets/img/posts/2020-10/1.png)

左边是分类菜单，右边是长列表，有多个分类的商品，单个分类滚动完后可以继续滚动切换到下一个分类，同时左边的分类菜单选中态会跟着切换到当前商品列表显示的分类。

考虑到更好的用户体验，以及参考了美团等点餐小程序，这个商品列表的数据是一次性返回的。**目前遇到的问题就是，当商品数量比较多时，首次渲染时间很长，而且页面会卡顿。**


## 优化方向

### 逻辑优化

小声bb：其实就是原来代码（由于历史原因）写得太烂了……OTL

先放个图👇

![](/assets/img/posts/2020-10/2.png)

小声bb：连小程序都看不下去了，要警告了o(╯□╰)o

微信开发者工具都有警告了，而且提示里面也有定位到具体代码的位置，所以关键就是这个 `setData` ！！！

我们可以先看看官方对于小程序性能以及 `setData` 优化的一些建议。（[https://developers.weixin.qq.com/miniprogram/dev/framework/performance/tips.html](https://developers.weixin.qq.com/miniprogram/dev/framework/performance/tips.html)）

具体实践：

#### 1. `setData` 不能一次性传太多数据，如果列表太长，可以分开渲染【比如转化为二维数组，每次循环渲染一个数组】。

v1：简单粗暴版

```js
// 每次渲染一个分类
// 假设goodsList是一个二维数组
goodsList.forEach((item, index) => {
    this.setData({
        [`goodsList[${index}]`]: item
    })
})
```

像上面这样写会有一个问题，页面首屏渲染是快了，但是点击页面操作（比如加购按钮等），页面会卡住，等一下才有反应，操作反馈延迟严重。  
其实这是因为，这个循环是把单次 `setData` 数量减少了，但是却变成了循环多次 `setData`，我们看着首屏显示好了，但是其实其他分类（其他数组）还在渲染，线程还是忙碌状态，JS 线程一直在编译执行渲染，点击事件不能及时传递到逻辑层，逻辑层亦无法及时将操作处理结果及时传递到视图层。

v2：定时器hack版

既然js线程忙着渲染，那我们可以强制让它先停下来。于是有了v2的定时器hack版。

```js
// 每次渲染一个分类
let len = data.goodsList ? data.goodsList.length : 0;
let idx = 0
let timer = setInterval(() => {
    if (idx < len) {
        that.setData({
            [`goodsList[${idx}]`]: data.goodsList[idx]
        });
        idx++
    } else {
        clearInterval(timer)
    }
}, 15);
```

现在首屏渲染速度问题解决了，点击按钮延迟响应问题也解决了。就是代码有点hack，逼死强迫症o(╯□╰)o

v3：大杀器——虚拟列表

虚拟列表简单说原理就是只渲染当前显示屏幕区域以及前n屏和后n屏的数据，用一个单独的字段保存当前需要显示的数组（就是当前一屏+前n屏+后n屏），每次列表滚动的时候重新计算需要显示的数据，更新这个字段，页面就会相应更新了。这样就能保证页面上的元素节点数量不会太多，就可以支持大量数据的长列表需求。

更详细的原理和实现各位同学们可以自己搜一下，此处不展开。

小程序官方也有开源的虚拟列表组件：[recycle-view](https://developers.weixin.qq.com/miniprogram/dev/extended/functional/recycle-view.html)


#### 2. `setData` 可以支持颗粒更新，指定到具体的属性。

比如加购等操作，需要更新商品右上角的小数字，可以这样写：

```js
this.setData({
    [`goodsList[${categoryIndex}][${goodsIndex}].num`]: goodsItem.num
})
```


#### 3. 跟页面无关的数据不要保存在 `data`，不要用 `setData` 更新，因为 `setData` 会触发页面渲染。

eg：
```js
Page({
    data: {
        ...
    },
    // 跟页面渲染无关的数据
    state: {
        hasLogin: false,
    },
    ...
})

// 更新的时候直接赋值就行
this.state.hasLogin = true
```
PS：或者甚至不需要挂载到 `page` 对象下，直接用普通变量保存。


#### 4. 图片大小优化

在长列表中图片大小如果不加限制，大量的大图会占用很多内存，有可能导致iOS客户端内存占用上升，从而触发系统回收小程序页面。除了内存问题外，大图片也会造成页面切换的卡顿。  
解决办法就是根据当前显示的图片区域大小，取尺寸刚好合适（2倍-3倍图）的图片。  
建议图片用CDN，一般CDN服务厂商提供图片服务的都会提供裁剪图片的接口，然后接口只返回原图链接，前端根据需要传参数裁剪图片。前端具体做法可以写公共的图片处理方法，或者自己封装图片组件。

附常用图片CDN服务商图片裁剪API文档：
- [阿里云OSS图片缩放](https://help.aliyun.com/document_detail/44688.html)
- [七牛云图片处理](https://developer.qiniu.com/dora/api/1279/basic-processing-images-imageview2)


#### 5. 减少不必要的数据请求

比如在该点餐页面进入时需要获取定位，然后根据定位获取最近的门店，前面两个接口都需要请求（具体可以根据业务需求），而最后如果获取到的距离最近的门店跟上次一样，则不需要重新获取店铺详情和商品数据。


### 体验优化

#### 1. 合并短时间内的多个loading提示

还是该点餐页面流程，像上文说过的，进入页面时需要获取定位接口，等定位接口返回结果了再拿定位取值去获取距离最近的店铺，最后才是请求店铺和商品数据。  
这三个接口是串行的。此时如果我们每个接口都弹出一个loading提示，就会出现loading显示一会儿，消失，又显示一会儿，又消失……这样的现象，这样的体验是不太好的。  
建议可以通过封装请求，并且在请求里统一处理loading，来合并短时间内多次发起请求的多个loading。

eg：
```js
let showLoadingTimer = null;
let showRequestLoading = false; // 标记是否正在显示loading

/**
 * 封装request
 * @param {*} {showLoading：是否需要显示loading, options：request参数，如url，data等} 
 */
function request({showLoading = true, ...options}) {
    // 显示request loading
    handleShowLoading(showLoading)

    wx.request({
        ...
        complete() {
            // 关闭request loading
            handleShowLoading(false)
        }
    })
}

/**
 * 封装request loading
 * 短时间内如果调用多次showLoading，会合并在一起显示，而不是每个都闪现一下
 * @param showLoading
 */
function handleShowLoading(showLoading) {
    if (showLoading) {
        // 显示loading
        clearTimeout(showLoadingTimer);
        if (!showRequestLoading) {
            showRequestLoading = true;
            wx.showNavigationBarLoading();
            wx.showLoading({ title: "加载中", mask: true })
        }
    } else {
        // 200ms后关闭loading
        showLoadingTimer = setTimeout(() => {
            showRequestLoading = false;
            wx.hideNavigationBarLoading();
            wx.hideLoading()
        }, 200)
    }
}
```

#### 2. 整个页面初次加载时可以使用页面loading动画或骨架屏，优化加载中体验。

#### 3. 静默获取、更新数据

比如这个点餐页每次 `onShow` 都会调用定位接口和获取最近门店接口，但是不显示loading，用户就没有感知，体验比较好。


### 接口优化

需要关注接口的粒度控制。
因为有时候合并接口，前端可以减少一次请求，体验更好；但有时候如果接口的数据太多，响应太慢，就可以考虑是否某部分数据可以后置获取，让主要的页面内容先渲染出来，根据这个设计来拆分接口。

比如项目中的点餐页面，原来购物车数据和商品规格弹窗显示的详情数据都是在获取店铺商品接口一次性返回的，而这个接口本来由于设计需要一次返回所有商品，就会造成数据量太大，而且后端需要查询的表也更多。于是把获取购物车，和商品详情接口都拆分为单独的接口，获取店铺商品接口的响应时间就减少了，页面也能更快显示出来。


## 总结

其实上面提到的逻辑优化和接口优化很多都是细节，并不是太高深的技术，我们平时迭代的时候就可以注意。而体验方面的优化则需要前端同学在前端技术以外更多关注用户体验和设计方面的知识啦，而且这也是一个有追求的前端应该具备的技能……←_←
所以嘛……技术路漫漫，大家共勉吧


## 参考文章

[https://developers.weixin.qq.com/miniprogram/dev/framework/performance/tips.html](https://developers.weixin.qq.com/miniprogram/dev/framework/performance/tips.html)