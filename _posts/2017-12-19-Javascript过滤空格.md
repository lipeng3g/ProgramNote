---
layout: post
title:  "Javascript过滤前后空格"
date:   2017-12-19 05:51:29 +0800
categories: JavaScript
---
# Javascript过滤前后空格
<!-- TOC -->

- [Javascript过滤前后空格](#javascript%E8%BF%87%E6%BB%A4%E5%89%8D%E5%90%8E%E7%A9%BA%E6%A0%BC)
    - [循环检查](#%E5%BE%AA%E7%8E%AF%E6%A3%80%E6%9F%A5)
    - [正则表达式替换](#%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F%E6%9B%BF%E6%8D%A2)
    - [JQuery](#jquery)

<!-- /TOC -->
## 循环检查
**推荐 ☆☆☆**
```Javascript
//供使用者调用 
function trim(s){ 
  return trimRight(trimLeft(s)); 
} 
//去掉左边的空白 
function trimLeft(s){ 
  if(s == null) { 
    return ""; 
  } 
  var whitespace = new String(" \t\n\r"); 
  var str = new String(s); 
  if (whitespace.indexOf(str.charAt(0)) != -1) { 
    var j=0, i = str.length; 
    while (j < i && whitespace.indexOf(str.charAt(j)) != -1){ 
      j++; 
    } 
    str = str.substring(j, i); 
  } 
  return str; 
} 

//去掉右边的空白
function trimRight(s){ 
  if(s == null) return ""; 
  var whitespace = new String(" \t\n\r"); 
  var str = new String(s); 
  if (whitespace.indexOf(str.charAt(str.length-1)) != -1){ 
    var i = str.length - 1; 
    while (i >= 0 && whitespace.indexOf(str.charAt(i)) != -1){ 
      i--; 
    } 
    str = str.substring(0, i+1); 
  } 
  return str; 
}    
```

## 正则表达式替换
**推荐 ☆☆☆☆**
```Javascript

String.prototype.Trim = function() //去左右空格;
{ 
    return this.replace(/(^\s*)|(\s*$)/g, ""); 
} 
String.prototype.LTrim = function() //去左空格;
{ 
    return this.replace(/(^\s*)/g, ""); 
} 
String.prototype.RTrim = function() //去右空格;
{ 
    return this.replace(/(\s*$)/g, ""); 
} 

//去左空格;
function ltrim(s){
    return s.replace(/(^\s*)/g, "");
}
//去右空格;
function rtrim(s){
    return s.replace(/(\s*$)/g, "");
}
//去左右空格;
function trim(s){
    return s.replace(/(^\s*)|(\s*$)/g, "");
}
```

## JQuery
**推荐 ☆☆☆☆☆**
```Javascript
// 使用方法
$.trim(str) 
// Jquery实现方法
function trim(str){  
    return str.replace(/^(\s|\u00A0)+/,'').replace(/(\s|\u00A0)+$/,'');  
}
```
