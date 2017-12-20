---
layout: post
title:  "关于scrollHeight和scrollTop取值为0的问题"
date:   2017-11-07 05:51:29 +0800
categories: JavaScript
---
# 关于scrollHeight和scrollTop取值为0的问题
<!-- TOC -->

- [关于scrollHeight和scrollTop取值为0的问题](#%E5%85%B3%E4%BA%8Escrollheight%E5%92%8Cscrolltop%E5%8F%96%E5%80%BC%E4%B8%BA0%E7%9A%84%E9%97%AE%E9%A2%98)
- [1. 基础](#1-%E5%9F%BA%E7%A1%80)
    - [1.1. offsetTop与style.top的区别](#11-offsettop%E4%B8%8Estyletop%E7%9A%84%E5%8C%BA%E5%88%AB)
- [2. 问题及解决](#2-%E9%97%AE%E9%A2%98%E5%8F%8A%E8%A7%A3%E5%86%B3)
- [3. 总结](#3-%E6%80%BB%E7%BB%93)

<!-- /TOC -->

[原文地址--简书](http://www.jianshu.com/p/46087c0ace05)

# 1. 基础

- `obj.offsetTop` 指 obj 距离上方或上层控件的位置，整型，单位像素。
- `obj.offsetLeft` 指 obj 距离左方或上层控件的位置，整型，单位像素。
- `obj.offsetWidth` 指 obj 控件自身的宽度，整型，单位像素。
- `obj.offsetHeight` 指 obj 控件自身的高度，整型，单位像素。（`clientHeight` + `滚动条` + `边框`）
- `obj.scrollHeight` 网页内容的高度，最小值是`clientHeight`。

## 1.1. offsetTop与style.top的区别

- `offsetTop` 返回的是`数字`，而 `style.top` 返回的是`字符串`，除了数字外还带有单位：px。
- `offsetTop` `只读`， `style.top` `可读写`。

    如果没有给 HTML 元素指定过 top 样式，则 style.top 返回的是undefined。（`也就是说内联样式与外联样式和内嵌样式都没有设置style.xx则js是无法设置该元素相应的的style.xx`）

# 2. 问题及解决

`document.body.scrollHeight`与`document.documentElement.scrollHeight`都是获取网页页面内容的高。

但是今天我发现使用他们时并没有达到我想要的效果，仔细查看发现firefox下document.body.scrollHeight的值始终是0

可以发现

`chrome`对`document.documentElement.scrollHeight`&`document.documentElement.scrollTop`是不能识别的

`firefox`和`IE11`不能识别`document.body.scrollHeight`&`document.body.scrollTop`

所以要考虑的网页的兼容性，建议两种获取方法都要写在代码里。

另外我也测试clientXxx，写在总结里吧。

# 3. 总结

在js中需要取scrollHeight值和scrollTop值时，我们需要使用：
```Javascript
var scrollHeight=document.body.scrollHeight==0?document.documentElement.scrollHeight:document.body.scrollHeight;
```
```Javascript
var scrollTop=document.body.scrollTop==0?document.documentElement.scrollTop:document.body.scrollTop;
```
关于scrollWidth(下面两种都可以)
```Javascript
var oWidth=document.body.scrollWidth;
var oWidth=document.documentElement.scrollWidth;
```

取clientXxx值只能
```Javascript
var wWidth=document.documentElement.clientXxx;
```