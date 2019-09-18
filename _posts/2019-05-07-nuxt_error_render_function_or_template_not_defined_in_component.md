---
layout: post
title: 'Nuxt 报错：render function or template not defined in component: anonymous'
categories: 前端
tags: vue nuxt
excerpt: '记录了解决 Nuxt 报错 render function or template not defined in component: anonymous 问题的思路和办法'
---

## 问题
在 dev hot reload 模式下开发，本来是好的，添加了一段代码之后出现了这个错误。  
详细的错误信息如下：  
![](http://120.77.171.203/assets/img/posts/2019-05/1.jpg)

这提示看起来是系统级的错误，没什么头绪。于是google之，是找到挺多有提到这个错误的 issue，然而……都没看到有解决办法。

好吧，还是靠自己先看一下。

于是撤销大法好。。。  
我先注释掉添加的那部分代码，保存，果然没有再报错了。  
那就看看这部分添加的代码咯～

## 分析&解决

`browser.js`
```javascript
let browser = {};
const ua = navigator.userAgent;

browser = {
  weixin: ua.toLowerCase().indexOf('micromessenger') !== -1,
  ios: (/\(i[^;]+;( U;)? CPU.+Mac OS X/).test(ua),
  android: ua.indexOf('Android') > -1 || ua.indexOf('Adr') > -1
}

export default browser;
```

以上是我添加的 `browser.js` 文件，然后我在一个 `.vue` 文件中 `import` 了它。  
看这部分代码是没问题的，于是我再仔细看报错内容（PS：把那个 `show all frames` 勾选上）……  
![](http://120.77.171.203/assets/img/posts/2019-05/2.jpg)
这次发现了个关键点——是 `server render` 的错误。

嗯~~这就好理解了。  
`Nuxt` 的 `universal` 模式是支持同构的【https://zh.nuxtjs.org/api/configuration-mode/#mode-%E5%B1%9E%E6%80%A7】。  
PS：关于同构可以看看这篇文章（[聊一聊前端「同构」](https://juejin.im/entry/5b1631085188257d492adc9e)）

而对于 `Nuxt` 的同构，就是我们的项目代码在服务端（Node）和客户端（浏览器）都会运行，这是同构的好处，我们可以写一套代码，两端都能执行。但是也正是因为两端都会执行，我们就需要关注两种环境的不同。比如说服务端是 `node` 环境，所以它是没有 `window` 对象的。而看我添加的 `browser.js` 文件的代码，用到了 `navigator.userAgent`，其实它就是 `window` 的属性。所以找到问题原因了。  
修改的方式很简单，我们可以判断一下当前环境是什么环境，是服务端则不执行，而 `Nuxt` 也有提供 `process.client` 和 `process.server` 两个属性给我们用。所以修改代码如下： 

```javascript
let browser = {};

if (process.client) {
  const ua = navigator.userAgent;
  browser = {
    weixin: ua.toLowerCase().indexOf('micromessenger') !== -1,
    ios: (/\(i[^;]+;( U;)? CPU.+Mac OS X/).test(ua),
    android: ua.indexOf('Android') > -1 || ua.indexOf('Adr') > -1
  }
}

export default browser;
```

保存刷新页面，果然没再报错了~~

---

**【以下是废话】**

反思一下，自己对于框架这些还是停留于用的层面，有时候看到些底层一点的报错就毫无头绪，盲目google了。但是其实有时候仔细分析一下错误提示，可能会发现有用的信息……  
像这次的报错，我直接搜 `render function or template not defined in component: anonymous` 这个错误提示找到的 `github issue` 都是没有解决然而又 close 了的，因为这其实不是 `Nuxt` 的 bug，而是因为自己不知道要考虑同构代码的坑，导致服务端渲染失败。所以看问题还是要自己先分析思考，而且对于自己项目中用到的框架，还是需要去了解一下它的原理的，要知其然更要知其所以然！！
