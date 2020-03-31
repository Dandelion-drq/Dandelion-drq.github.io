---
layout: post
title: 'javascript 正则表达式的使用'
categories: 前端
cover: 'http://120.77.171.203/assets/img/posts/2018-07/1.jpg'
tags: javascript 正则表达式
excerpt: 整理的关于js正则表达式的一些笔记。
---

## 1. 语法
有两种定义正则表达式的方式

- 字面量形式
```js
var expression = /pattern/flags
```
引用 MDN 的解释：
> **pattern**：正则表达式的文本。
> 
> **flags**：标志，可以是具有以下值的任意组合：
>  
> + g：全局匹配；找到所有匹配，而不是在第一个匹配后停止
> + i：忽略大小写
> + m：多行; 将开始和结束字符（^和$）视为在多行上工作（也就是，分别匹配每一行的开始和结束（由 \n 或 \r 分割），而不只是只匹配整个输入字符串的最开始和最末尾处。
> + u：Unicode; 将模式视为Unicode序列点的序列（ES6 新增）
> + y：粘性匹配; 仅匹配目标字符串中此正则表达式的lastIndex属性指示的索引(并且不尝试从任何后续的索引匹配)。（ES6 新增。这个不太好理解可以看看两个栗子[RegExp.prototype.sticky[MDN]](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/RegExp/sticky)，[What does regex' flag 'y' do?](https://stackoverflow.com/questions/4542304/what-does-regex-flag-y-do)）

eg：
```javascript
var pattern = /at/gi;
```

- 构造函数形式
```js
var expression = new RegExp(pattern, flags);
```

eg：
```js
var pattern2 = new RegExp("at", "gi");
```

这两种写法其实效果是一样的，**都是创建了一个新的 `RegExp` 实例**。

`RegExp` 的每个实例都有以下属性：
- `flags`：所有标志。（ES6 新增）
- `global`：Boolean，表示是否设置了 g 标志。
- `ignoreCase`：Boolean，表示是否设置了 i 标志。
- `multiline`：Boolean，表示是否设置了 m 标志。
- `lastIndex`：表示从哪个位置开始搜索下一个匹配项，从0算起。(https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/RegExp/lastIndex)
- `source`：当前正则表达式对象的模式文本的字符串，该字符串不会包含正则字面量两边的斜杠以及任何的标志字符。

---

## 2. 怎么用

简单的字符串匹配可以用 `String.match()` 方法。
eg：
```javascript
var str = "cat, bat, sat, fat";
var pattern = /.at/g;
str.match(pattern);
```
输出：  
![](https://images2018.cnblogs.com/blog/893839/201807/893839-20180727180548352-2042032721.jpg)


如果只需要知道是否有匹配而不关心内容，可以用 `RegExp.test()` 方法。该方法返回布尔值，表示是否匹配成功。
eg：
```javascript
var str = "cat, bat, sat, fat";
var pattern = /.at/g;
pattern.test(str);
```
输出：  
![](https://images2018.cnblogs.com/blog/893839/201807/893839-20180727180604650-1322729798.jpg)


如果需要用到捕获组的时候，用 `RegExp.exec()` 方法。该方法返回匹配结果，没有匹配成功时则返回 `null`。
eg：
```javascript
var pattern = /aaa and (bbb and (ccc))/g;
var text = "this is aaa and bbb and ccc";
pattern.exec(text);
```
输出：  
![](https://images2018.cnblogs.com/blog/893839/201807/893839-20180727180615390-2050391078.jpg)
其中数组中第一项是与整个模式匹配的字符串，其他项是与模式中的捕获组匹配的字符串。如果模式中没有捕获组，则该数组只包含一项。
