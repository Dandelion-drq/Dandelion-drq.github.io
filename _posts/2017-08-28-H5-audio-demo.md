---
layout: post
title: 'H5 audio 音频标签自定义样式修改以及添加播放控制事件'
categories: 前端
cover: '/assets/img/posts/2017-08/cover.jpg'
tags: html css javascript
excerpt: '本文介绍了如何对 Html5 <audio> 音频标签进行自定义样式修改以及添加播放控制事件'
---

## 说明： 
需求要求这个音频标签首先要是可适配移动端浏览器的，音频样式就是参考微信做的。

最终效果如下：
![](http://120.77.171.203:8080/images/H5-audio.gif)

---

## 具体实现

### 思路：
H5 的 `<audio>` 标签是由浏览器负责实现默认样式的。所以不同的浏览器样式不一样，有些还不太美观。所以我们一般会去掉默认样式，自己重新写。具体操作是定义 `<audio>` 的时候去掉 `controls` 属性，这样就可以隐藏原生的 `audio`， 然后就可以加上自己写的 html + css 代码了。最后用 `js` 捕获 `audio` 对象，为它添加各种播放控制事件。

---

### 1. 定义标签
这个很简单，就是用H5 `<audio>` 标签定义音频的方式。

html 代码：
```html
<div class="audio-wrapper">
    <audio>
        <source src="Files/Audio/2017-08/1.mp3" type="audio/mp3">
    </audio>
    <div class="audio-left"><img id="audioPlayer" src="image/play.png"></div>
    <div class="audio-right">
        <p style="max-width: 536px;">Beta-B_Kan R. Gao.mp3</p>
        <div class="progress-bar-bg" id="progressBarBg"><span id="progressDot"></span>
            <div class="progress-bar" id="progressBar"></div>
        </div>
        <div class="audio-time"><span class="audio-length-current" id="audioCurTime">00:00</span><span class="audio-length-total">01:06</span></div>
    </div>
</div>
```

css 代码：
```css
.audio-wrapper {
    background-color: #fcfcfc;
    margin: 10px auto;
    max-width: 670px;
    height: 70px;
    border: 1px solid #e0e0e0;
}

.audio-left {
    float: left;
    text-align: center;
    width: 18%;
    height: 100%;
}

.audio-left img {
    width: 40px;
    position: relative;
    top: 15px;
    margin: 0;
    display: initial;   /* 解除与app的样式冲突 */
    cursor: pointer;
}

.audio-right {
    margin-right: 2%;
    float: right;
    width: 80%;
    height: 100%;
}

.audio-right p {
    font-size: 15px;
    height: 35%;
    margin: 8px 0;

    /* 歌曲名称只显示在一行，超出部分显示为省略号 */
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
    max-width: 243px;   /* 要适配小屏幕手机，所以最大宽度先设小一点，后面js根据屏幕大小重新设置 */    
}

.progress-bar-bg {
    background-color: #d9d9d9;
    position: relative;
    height: 2px;
    cursor: pointer;
}

.progress-bar {
    background-color: #649fec;
    width: 0;
    height: 2px;
}

.progress-bar-bg span {
    content: " ";
    width: 10px;
    height: 10px;
    border-radius: 50%;
    -moz-border-radius: 50%;
    -webkit-border-radius: 50%;
    background-color: #3e87e8;
    position: absolute;
    left: 0;
    top: 50%;
    margin-top: -5px;
    margin-left: -5px;
    cursor: pointer;
}

.audio-time {
    overflow: hidden;
    margin-top: -1px;
}

.audio-length-total {
    float: right;
    font-size: 12px;
}

.audio-length-current {
    float: left;
    font-size: 12px;
}
```

### 2. 添加播放控制事件
- 获取音频对象

```javascript
var audio = document.getElementsByTagName('audio')[0];
```

- 播放/暂停控制

```javascript
    // 点击播放/暂停图片时，控制音乐的播放与暂停
    audioPlayer.addEventListener('click', function () {
        // 改变播放/暂停图片
        if (audio.paused) {
            audio.play();
            audioPlayer.src = './image/pause.png';
        } else {
            audio.pause();
            audioPlayer.src = './image/play.png';
        }
    }, false);
```

- 更新进度条与当前播放时间

```javascript
// 监听音频播放时间并更新进度条
audio.addEventListener('timeupdate', function () {
    updateProgress(audio);
}, false);

/**
 * 更新进度条与当前播放时间
 * @param {object} audio - audio对象
 */
function updateProgress(audio) {
    var value = audio.currentTime / audio.duration;
    document.getElementById('progressBar').style.width = value * 100 + '%';
    document.getElementById('progressDot').style.left = value * 100 + '%';
    document.getElementById('audioCurTime').innerText = transTime(audio.currentTime);
}

/**
 * 音频播放时间换算
 * @param {number} value - 音频当前播放时间，单位秒
 */
function transTime(value) {
    var time = "";
    var h = parseInt(value / 3600);
    value %= 3600;
    var m = parseInt(value / 60);
    var s = parseInt(value % 60);
    if (h > 0) {
        time = formatTime(h + ":" + m + ":" + s);
    } else {
        time = formatTime(m + ":" + s);
    }

    return time;
}

/**
 * 格式化时间显示，补零对齐
 * eg：2:4  -->  02:04
 * @param {string} value - 形如 h:m:s 的字符串 
 */
function formatTime(value) {
    var time = "";
    var s = value.split(':');
    var i = 0;
    for (; i < s.length - 1; i++) {
        time += s[i].length == 1 ? ("0" + s[i]) : s[i];
        time += ":";
    }
    time += s[i].length == 1 ? ("0" + s[i]) : s[i];

    return time;
}
```

- 播放完成时把进度调回开始的位置

```javascript
// 监听播放完成事件
audio.addEventListener('ended', function () {
    audioEnded();
}, false);

/**
 * 播放完成时把进度调回开始的位置
 */
function audioEnded() {
    document.getElementById('progressBar').style.width = 0;
    document.getElementById('progressDot').style.left = 0;
    document.getElementById('audioCurTime').innerText = transTime(0);
    document.getElementById('audioPlayer').src = './image/play.png';
}
```

### 3. 添加进度调节事件
- 点击进度条跳到指定位置播放

```javascript
    // 点击进度条跳到指定点播放
    // PS：此处不要用click，否则下面的拖动进度点事件有可能在此处触发，此时e.offsetX的值非常小，会导致进度条弹回开始处（简直不能忍！！）
    var progressBarBg = document.getElementById('progressBarBg');
    progressBarBg.addEventListener('mousedown', function (event) {
        // 只有音乐开始播放后才可以调节，已经播放过但暂停了的也可以
        if (!audio.paused || audio.currentTime != 0) {
            var pgsWidth = parseFloat(window.getComputedStyle(progressBarBg, null).width.replace('px', ''));
            var rate = event.offsetX / pgsWidth;
            audio.currentTime = audio.duration * rate;
            updateProgress(audio);
        }
    }, false);
```

- 拖动进度条到指定位置播放

```javascript
/**
 * 鼠标拖动进度点时可以调节进度
 * @param {*} audio
 */
function dragProgressDotEvent(audio) {
    var dot = document.getElementById('progressDot');

    var position = {
        oriOffestLeft: 0, // 移动开始时进度条的点距离进度条的偏移值
        oriX: 0, // 移动开始时的x坐标
        maxLeft: 0, // 向左最大可拖动距离
        maxRight: 0 // 向右最大可拖动距离
    };
    var flag = false; // 标记是否拖动开始

    // 鼠标按下时
    dot.addEventListener('mousedown', down, false);
    dot.addEventListener('touchstart', down, false);

    // 开始拖动
    document.addEventListener('mousemove', move, false);
    document.addEventListener('touchmove', move, false);

    // 拖动结束
    document.addEventListener('mouseup', end, false);
    document.addEventListener('touchend', end, false);

    function down(event) {
        if (!audio.paused || audio.currentTime != 0) { // 只有音乐开始播放后才可以调节，已经播放过但暂停了的也可以
            flag = true;

            position.oriOffestLeft = dot.offsetLeft;
            position.oriX = event.touches ? event.touches[0].clientX : event.clientX; // 要同时适配mousedown和touchstart事件
            position.maxLeft = position.oriOffestLeft; // 向左最大可拖动距离
            position.maxRight = document.getElementById('progressBarBg').offsetWidth - position.oriOffestLeft; // 向右最大可拖动距离

            // 禁止默认事件（避免鼠标拖拽进度点的时候选中文字）
            if (event && event.preventDefault) {
                event.preventDefault();
            } else {
                event.returnValue = false;
            }

            // 禁止事件冒泡
            if (event && event.stopPropagation) {
                event.stopPropagation();
            } else {
                window.event.cancelBubble = true;
            }
        }
    }

    function move(event) {
        if (flag) {
            var clientX = event.touches ? event.touches[0].clientX : event.clientX; // 要同时适配mousemove和touchmove事件
            var length = clientX - position.oriX;
            if (length > position.maxRight) {
                length = position.maxRight;
            } else if (length < -position.maxLeft) {
                length = -position.maxLeft;
            }
            var progressBarBg = document.getElementById('progressBarBg');
            var pgsWidth = parseFloat(window.getComputedStyle(progressBarBg, null).width.replace('px', ''));
            var rate = (position.oriOffestLeft + length) / pgsWidth;
            audio.currentTime = audio.duration * rate;
            updateProgress(audio);
        }
    }

    function end() {
        flag = false;
    }
}
```

---
最后总的 `js` 代码如下：
```javascript
document.addEventListener('DOMContentLoaded', function () {
    // 设置音频文件名显示宽度
    var element = document.querySelector('.audio-right');
    var maxWidth = window.getComputedStyle(element, null).width;
    document.querySelector('.audio-right p').style.maxWidth = maxWidth;

    // 初始化音频控制事件
    initAudioEvent();
}, false);

function initAudioEvent() {
    var audio = document.getElementsByTagName('audio')[0];
    var audioPlayer = document.getElementById('audioPlayer');

    // 点击播放/暂停图片时，控制音乐的播放与暂停
    audioPlayer.addEventListener('click', function () {
        // 监听音频播放时间并更新进度条
        audio.addEventListener('timeupdate', function () {
            updateProgress(audio);
        }, false);

        // 监听播放完成事件
        audio.addEventListener('ended', function () {
            audioEnded();
        }, false);

        // 改变播放/暂停图片
        if (audio.paused) {
            // 开始播放当前点击的音频
            audio.play();
            audioPlayer.src = './image/pause.png';
        } else {
            audio.pause();
            audioPlayer.src = './image/play.png';
        }
    }, false);

    // 点击进度条跳到指定点播放
    // PS：此处不要用click，否则下面的拖动进度点事件有可能在此处触发，此时e.offsetX的值非常小，会导致进度条弹回开始处（简直不能忍！！）
    var progressBarBg = document.getElementById('progressBarBg');
    progressBarBg.addEventListener('mousedown', function (event) {
        // 只有音乐开始播放后才可以调节，已经播放过但暂停了的也可以
        if (!audio.paused || audio.currentTime != 0) {
            var pgsWidth = parseFloat(window.getComputedStyle(progressBarBg, null).width.replace('px', ''));
            var rate = event.offsetX / pgsWidth;
            audio.currentTime = audio.duration * rate;
            updateProgress(audio);
        }
    }, false);

    // 拖动进度点调节进度
    dragProgressDotEvent(audio);
}

/**
 * 鼠标拖动进度点时可以调节进度
 * @param {*} audio
 */
function dragProgressDotEvent(audio) {
    var dot = document.getElementById('progressDot');

    var position = {
        oriOffestLeft: 0, // 移动开始时进度条的点距离进度条的偏移值
        oriX: 0, // 移动开始时的x坐标
        maxLeft: 0, // 向左最大可拖动距离
        maxRight: 0 // 向右最大可拖动距离
    };
    var flag = false; // 标记是否拖动开始

    // 鼠标按下时
    dot.addEventListener('mousedown', down, false);
    dot.addEventListener('touchstart', down, false);

    // 开始拖动
    document.addEventListener('mousemove', move, false);
    document.addEventListener('touchmove', move, false);

    // 拖动结束
    document.addEventListener('mouseup', end, false);
    document.addEventListener('touchend', end, false);

    function down(event) {
        if (!audio.paused || audio.currentTime != 0) { // 只有音乐开始播放后才可以调节，已经播放过但暂停了的也可以
            flag = true;

            position.oriOffestLeft = dot.offsetLeft;
            position.oriX = event.touches ? event.touches[0].clientX : event.clientX; // 要同时适配mousedown和touchstart事件
            position.maxLeft = position.oriOffestLeft; // 向左最大可拖动距离
            position.maxRight = document.getElementById('progressBarBg').offsetWidth - position.oriOffestLeft; // 向右最大可拖动距离

            // 禁止默认事件（避免鼠标拖拽进度点的时候选中文字）
            if (event && event.preventDefault) {
                event.preventDefault();
            } else {
                event.returnValue = false;
            }

            // 禁止事件冒泡
            if (event && event.stopPropagation) {
                event.stopPropagation();
            } else {
                window.event.cancelBubble = true;
            }
        }
    }

    function move(event) {
        if (flag) {
            var clientX = event.touches ? event.touches[0].clientX : event.clientX; // 要同时适配mousemove和touchmove事件
            var length = clientX - position.oriX;
            if (length > position.maxRight) {
                length = position.maxRight;
            } else if (length < -position.maxLeft) {
                length = -position.maxLeft;
            }
            var progressBarBg = document.getElementById('progressBarBg');
            var pgsWidth = parseFloat(window.getComputedStyle(progressBarBg, null).width.replace('px', ''));
            var rate = (position.oriOffestLeft + length) / pgsWidth;
            audio.currentTime = audio.duration * rate;
            updateProgress(audio);
        }
    }

    function end() {
        flag = false;
    }
}

/**
 * 更新进度条与当前播放时间
 * @param {object} audio - audio对象
 */
function updateProgress(audio) {
    var value = audio.currentTime / audio.duration;
    document.getElementById('progressBar').style.width = value * 100 + '%';
    document.getElementById('progressDot').style.left = value * 100 + '%';
    document.getElementById('audioCurTime').innerText = transTime(audio.currentTime);
}

/**
 * 播放完成时把进度调回开始的位置
 */
function audioEnded() {
    document.getElementById('progressBar').style.width = 0;
    document.getElementById('progressDot').style.left = 0;
    document.getElementById('audioCurTime').innerText = transTime(0);
    document.getElementById('audioPlayer').src = './image/play.png';
}

/**
 * 音频播放时间换算
 * @param {number} value - 音频当前播放时间，单位秒
 */
function transTime(value) {
    var time = "";
    var h = parseInt(value / 3600);
    value %= 3600;
    var m = parseInt(value / 60);
    var s = parseInt(value % 60);
    if (h > 0) {
        time = formatTime(h + ":" + m + ":" + s);
    } else {
        time = formatTime(m + ":" + s);
    }

    return time;
}

/**
 * 格式化时间显示，补零对齐
 * eg：2:4  -->  02:04
 * @param {string} value - 形如 h:m:s 的字符串 
 */
function formatTime(value) {
    var time = "";
    var s = value.split(':');
    var i = 0;
    for (; i < s.length - 1; i++) {
        time += s[i].length == 1 ? ("0" + s[i]) : s[i];
        time += ":";
    }
    time += s[i].length == 1 ? ("0" + s[i]) : s[i];

    return time;
}
```

---
### 参考：
[音频（audio）自定义样式以及控制操作面板](http://www.jianshu.com/p/653a860b8dcb)

[HTML5中 audio标签的样式修改](http://www.jzdlink.com/webarticle/css/20161124906.html)

---
### 完整代码
https://gitee.com/Dandelion_/html_demo/tree/master/H5-audio