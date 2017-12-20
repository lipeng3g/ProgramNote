---
layout: post
title:  "JavaScript事件学习"
date:   2017-09-10 21:50:29 +0800
categories: JavaScript
---
<!-- TOC -->

- [1. 鼠标事件](#1-鼠标事件)
- [2. 事件绑定](#2-事件绑定)
    - [2.1. DOM中直接绑定](#21-dom中直接绑定)
    - [2.2. JS事件绑定](#22-js事件绑定)
- [3. 事件注册](#3-事件注册)
    - [3.1. 事件注册方式](#31-事件注册方式)
        - [3.1.1. IE以外](#311-ie以外)
        - [3.1.2. IE](#312-ie)
    - [3.2. 获取事件对象和事件源](#32-获取事件对象和事件源)
    - [3.3. 取消事件默认行为](#33-取消事件默认行为)
    - [3.4. 阻止事件冒泡](#34-阻止事件冒泡)
    - [3.5. 通用解决方案](#35-通用解决方案)
- [4. 事件委托](#4-事件委托)
- [5. 事件流](#5-事件流)
    - [5.1. 事件类型](#51-事件类型)

<!-- /TOC -->

# 1. 鼠标事件 
| 顺序        | 条件           | 事件  |
| ------------- |:-------------| -----|
| 1      | 鼠标移动到目标元素上 | 首先触发mouseover  |
| 2      | 光标继续在元素上移动      |   不断触发mousemove  |
| 3 | 按下鼠标上的设备(左键,右键,滚轮……)      |    触发mousedown |
| 4 | 设备弹起的时候  | 触发mouseup   |
| 5 | 目标元素的滚动条发生移动时(滚动滚轮/拖动滚动条。。)  | 触发scroll    |
| 6 | 滚动滚轮  | 触发mousewheel   |
| 7 | 鼠标移出元素  | 触发mouseout    |

# 2. 事件绑定
## 2.1. DOM中直接绑定
```Javascript
<input type="button" value="click me" onclick="hello()">

<script>
function hello(){
 alert("hello world!");
}
</script>
```

## 2.2. JS事件绑定
```Javascript
<input type="button" value="click me" id="btn">

<script>
document.getElementById("btn").onclick = function(){
 alert("hello world!");
}
</script>
```

传统绑定的`优点`
- 非常简单和稳定，可以确保它在你使用的不同浏览器中运作一致
- 处理事件时，this关键字引用的是当前元素，这很有帮组

传统绑定的`缺点 `
- 传统方法只会在事件冒泡中运行，而非捕获和冒泡
- 一个元素一次只能绑定一个事件处理函数。新绑定的事件处理函数会覆盖旧的事件处理函数
- 事件对象参数(e)仅非IE浏览器可用

# 3. 事件注册 

平常我们绑定事件的时候用dom.onxxxx=function(){}的形式 
这种方式是给元素的onxxxx属性赋值，只能绑定有一个处理句柄。 
但很多时候我们需要绑定多个处理句柄到一个事件上，而且还可能要动态的增删某个处理句柄 

## 3.1. 事件注册方式
### 3.1.1. IE以外 
```JavaScript
target.addEventListener(type,listener,useCapture) 
target.removeEventListener(type,listener,useCapture); 
target ：文档节点、document、window 或 XMLHttpRequest。 
type ：字符串，事件名称，不含“on”，比如“click”、“mouseover”、“keydown”等。 
listener ：实现了 EventListener 接口或者是 JavaScript 中的函数。 
useCapture ：是否使用捕捉，一般用 false。
```
### 3.1.2. IE 
```JavaScript
target.attachEvent(type, listener); 
target.detachEvent(type, listener); 
target ：文档节点、document、window 或 XMLHttpRequest。 
type ：字符串，事件名称，含“on”，比如“onclick”、“onmouseover”、“onkeydown”等。 
listener ：实现了 EventListener 接口或者是 JavaScript 中的函数。
```

_两者使用的原理：可对执行的优先级不一样，实例讲解如下：_

`ele.attachEvent("onclick",method1); `

`ele.attachEvent("onclick",method2); `

`ele.attachEvent("onclick",method3); `

执行顺序为`method3`->`method2`->`method1`

`ele.addEventListener("click",method1,false); `

`ele.addEventListener("click",method2,false); `

`ele.addEventListener("click",method3,false); `

执行顺序为`method1`->`method2`->`method3`

兼容后的方法
```JavaScript
var func = function() {};
//例：addEvent(window,"load",func) 
function addEvent(elem, type, fn) {
    if (elem.attachEvent) {
        elem.attachEvent('on' + type, fn);
        return;
    }
    if (elem.addEventListener) {
        elem.addEventListener(type, fn, false);
    }
}
//例：removeEvent(window,"load",func) 
function removeEvent(elem, type, fn) {
    if (elem.detachEvent) {
        elem.detachEvent('on' + type, fn);
        return;
    }
    if (elem.removeEventListener) {
        elem.removeEventListener(type, fn, false);
    }
}
```
## 3.2. 获取事件对象和事件源
```JavaScript
function eventHandler(e){ 
    //获取事件对象 
    e = e || window.event;//IE和Chrome下是window.event FF下是e 
    //获取事件源 
    var target = e.target || e.srcElement;//IE和Chrome下是srcElement FF下是target 
} 
```

## 3.3. 取消事件默认行为
```JavaScript
function eventHandler(e) {
    e = e || window.event;
    // 防止默认行为 
    if (e.preventDefault) {
        e.preventDefault(); //IE以外 
    } else {
        e.returnValue = false; //IE 
        //注意：这个地方是无法用return false代替的 
        //return false只能取消元素 
    }
}
```

## 3.4. 阻止事件冒泡 
```JavaScript
function myParagraphEventHandler(e) {
    e = e || window.event;
    if (e.stopPropagation) {
        e.stopPropagation(); //IE以外 
    } else {
        e.cancelBubble = true; //IE 
    }
}
```

## 3.5. 通用解决方案
[传送门](http://dean.edwards.name/my/events.js)
```Javascript
function addEvent(element, type, handler) {
  // assign each event handler a unique ID
  if (!handler.$$guid) handler.$$guid = addEvent.guid++;
  // create a hash table of event types for the element
  if (!element.events) element.events = {};
  // create a hash table of event handlers for each element/event pair
  var handlers = element.events[type];
  if (!handlers) {
    handlers = element.events[type] = {};
    // store the existing event handler (if there is one)
    if (element["on" + type]) {
      handlers[0] = element["on" + type];
    }
  }
  // store the event handler in the hash table
  handlers[handler.$$guid] = handler;
  // assign a global event handler to do all the work
  element["on" + type] = handleEvent;
};
// a counter used to create unique IDs
addEvent.guid = 1;

function removeEvent(element, type, handler) {
  // delete the event handler from the hash table
  if (element.events && element.events[type]) {
    delete element.events[type][handler.$$guid];
  }
};

function handleEvent(event) {
  // grab the event object (IE uses a global event object)
  event = event || window.event;
  // get a reference to the hash table of event handlers
  var handlers = this.events[event.type];
  // execute each event handler
  for (var i in handlers) {
    this.$$handleEvent = handlers[i];
    this.$$handleEvent(event);
  }
};
```

# 4. 事件委托 
例如，你有一个很多行的大表格，在每个<tr>上绑定点击事件是个非常危险的想法，因为性能是个大问题。流行的做法是使用*事件委托*。 

事件委托描述的是将事件绑定在容器元素上，然后通过判断点击的target子元素的类型来触发相应的事件。 

事件委托依赖于事件冒泡，如果事件冒泡到table之前被禁用的话，那以下代码就无法工作了。
```JavaScript
myTable.onclick = function() {
    e = e || window.event;
    var targetNode = e.target || e.srcElement;
    // 测试如果点击的是TR就触发 
    if (targetNode.nodeName.toLowerCase() === 'tr') {
        alert('You clicked a table row!');
    }
}
```

当你有一个`List`，需要动态添加`li标签`，标签又需要绑定事件时，使用事件委托，可以避免每次添加`li`后的动态绑定

```Javascript
<ul id="list">
 <li id="item1" >item1</li>
 <li id="item2" >item2</li>
 <li id="item3" >item3</li>
</ul>

<script>
var list = document.getElementById("list");

document.addEventListener("click",function(event){
 var target = event.target;
 if(target.nodeName == "LI"){
 alert(target.innerHTML);
 }
})

var node=document.createElement("li");
var textnode=document.createTextNode("item4");
node.appendChild(textnode);
list.appendChild(node);

</script>

```

当点击item4时，item4有事件响应。说明`事件委托可以为新添加的DOM元素动态的添加事件`。


# 5. 事件流 

## 5.1. 事件类型

DOM同时支持两种事件模型：**捕获型事件**和**冒泡型事件** 
并且每当某一事件发生时，都会经过**捕获阶段**->**处理阶段**->**冒泡阶段**(有些浏览器不支持捕获) 

> 捕获阶段是由上层元素到下层元素的顺序依次。而冒泡阶段则正相反。

1.  Document
2.  Html
3.  Body
4.  Div

> 当事件触发时body会先得到有事件发生的信息，然后依次往下传递，直到到达最详细的元素。这就是事件捕获阶段。

> 还记得事件注册方法ele.addEventListener(type,handler,flag)吧，**Flag是一个Boolean值，true表示事件捕捉阶段执行，false表示事件冒泡阶段执行**。 

> 接着就是事件冒泡阶段。从下往上 依次执行事件处理函数(当然前提是当前元素为该事件注册了事件句柄)。 
在这个过程中，`可以阻止事件的冒泡，即停止向上的传递`。 
`阻止冒泡有时是很有必要的`，例如 

代码如下:
```JavaScript
<div onclick=funcA()> 
    <button onclick=funcB()>Click</button> 
</div> 
```

`本意是如果点击div中按钮以外的位置时执行funcA,点击button时执行funcB。但是实际点击button时就会先后执行funcB,funcA。 
而如果在button的事件句柄中阻止冒泡的话，div就不会执行事件句柄了。`