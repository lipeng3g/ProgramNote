---
layout: post
title:  "原型对象 Prototype"
date:   2017-09-08 02:51:29 +0800
categories: JavaScript
---
# 1. JavaScript 的原型对象 Prototype
<!-- TOC -->

- [1. JavaScript 的原型对象 Prototype](#1-javascript-%E7%9A%84%E5%8E%9F%E5%9E%8B%E5%AF%B9%E8%B1%A1-prototype)
    - [1.1. 什么是Prototype？](#11-%E4%BB%80%E4%B9%88%E6%98%AFprototype%EF%BC%9F)
    - [1.2. 原型链](#12-%E5%8E%9F%E5%9E%8B%E9%93%BE)
    - [1.3. 为什么使用Prototype更好？](#13-%E4%B8%BA%E4%BB%80%E4%B9%88%E4%BD%BF%E7%94%A8prototype%E6%9B%B4%E5%A5%BD%EF%BC%9F)
    - [1.4. 如何使用Prototype](#14-%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8prototype)
    - [1.5. Prototype 是一个动态的对象](#15-prototype-%E6%98%AF%E4%B8%80%E4%B8%AA%E5%8A%A8%E6%80%81%E7%9A%84%E5%AF%B9%E8%B1%A1)
    - [1.6. Prototype 的典型示例](#16-prototype-%E7%9A%84%E5%85%B8%E5%9E%8B%E7%A4%BA%E4%BE%8B)

<!-- /TOC -->
当您在JavaScript中定义函数时，它会附带一些预定义的属性; 其中之一是虚幻的原型。 在本文中，我将详细说明它是什么，以及为什么要在项目中使用它。
## 1.1. 什么是Prototype？
`原型对象 prototype`最初是一个空对象，可以添加成员 - 就​​像其他对象一样。

```JavaScript
var myObject = function( name ) {
	this.name = name;
	return this;
};
 
console.log(typeof myObject.prototype); // object
 
myObject.prototype.getName = function() {
	return this.name;
};
```

上面的代码创建了一个函数，然后赋值给 `myObject`。如果我调用 `myObject()`，它将返回 `window` 对象。因为它是在`全局作用域`内定义的，而且它还`没有被实例化`，所以 `this` 直接指向全局对象：
```JavaScript
console.log(myObject() === window); // true
```

## 1.2. 原型链
JavaScript 中定义或实例化任何一个对象的时候，它都会被附加一个名为 `__proto__` 的隐藏属性，原型链正是依靠这个属性才得以形成。但是千万别直接访问 `__proto__` 属性，因为`有些浏览器并不支持直接访问它`。
另外` __proto__` 和 对象的 `prototype` 属性也不是一回事，它们各自有各自的用途。而且，他们是携手工作的！

怎么理解呢？其实，当我们创建 `myObject` 函数时，实际上是创建了一个 `Function` 类型的对象：
```JavaScript
console.log(typeof myObject); // function
```
这里要说明一下，`Function 是 JavaScript 中预定义的一个对象`，所以它也有自己预定义的属性（如 length 和 arguments）和方法（如 call 和 apply），当然也有 `__proto__`，以此实现原型链。也就是说，JavaScript 引擎内可能有类似如下的代码片段：
```JavaScript
Function.prototype = {
	arguments: null,
	length: 0,
	call: function() {
		// secret code
	},
	apply: function(){
		// secret code
	},
    ...
};
```
事实上，JavaScript 引擎代码不可能这样简单，这里只是描述一下原型链是如何工作的。

我们定义了一个函数 `myObject`，它还有一个参数 `name`，但是并没有给它任何其它属性，例如 length 或者其它方法，如 call。那么下面这段代码为啥能正常执行呢？
```JavaScript
console.log(myObject.length); // 结果：1，是参数的个数
```
这是因为我们定义 `myObject` 时，同时也给它定义了一个 `__proto__` 属性，并赋值为 Function.prototype（参考前面的代码片段），所以我们能够像访问其它属性一样访问 myObject.length，即使我们并没有定义这个属性，因为它会顺着`__proto__` 原型链往上去找 length，最终在 Function 里面找到了。

那为什么找到的 `length` 属性的值是 `1`，而不是 `0` 呢，是什么时候给它赋值的呢？由于 myObject 是 Function 的一个实例：
```JavaScript
console.log(myObject instanceof Function); // true
console.log(myObject === Function); // false
```
当实例化一个对象的时候，对象的 `__proto__` 属性会被赋值为其构造者的原型对象，在本示例中就是 Function，此时构造器回去计算参数的个数，改变 length 的值。
```JavaScript
console.log(myObject.__proto__ === Function.prototype); // true
```
而当我们用 `new` 关键字创建一个新的实例时，新对象的 `__proto__` 将会被赋值为 `myObject.prototype`，因为现在的构造函数为 `myObject`，而非 `Function`。
```JavaScript
var myInstance = new myObject('foo');
console.log(myInstance.__proto__ === myObject.prototype); // true
```
新对象除了能访问 `Function.prototype` 中继承下来的 `call` 和 `apply` 外，还能访问从 `myObject` 中继承下来的 `getName` 方法：
```JavaScript
console.log(myInstance.getName()); // foo
 
var mySecondInstance = new myObject('bar');
 
console.log(mySecondInstance.getName()); // bar
console.log(myInstance.getName()); // foo
```
其实这相当于把原型对象当做一个蓝本，然后可以根据这个蓝本创建 N 个新的对象。

## 1.3. 为什么使用Prototype更好？
比方说，我们正在开发一个`canvas`游戏，同时在屏幕上需要几个（可能数百个）对象。每个对象都需要自己的属性，如`x`和`y`坐标，`宽度`，`高度`等等。

**我们可能需要这么做**
```JavaScript
var GameObject1 = {
    x: Math.floor((Math.random() * myCanvasWidth) + 1),
    y: Math.floor((Math.random() * myCanvasHeight) + 1),
    width: 10,
    height: 10,
    draw: function(){
        myCanvasContext.fillRect(this.x, this.y, this.width, this.height);
    }
   ...
};
 
var GameObject2 = {
    x: Math.floor((Math.random() * myCanvasWidth) + 1),
    y: Math.floor((Math.random() * myCanvasHeight) + 1),
    width: 10,
    height: 10,
    draw: function(){
        myCanvasContext.fillRect(this.x, this.y, this.width, this.height);
    }
    //... do this 98 more times ...
```
这将创建内存中的所有这些对象 - 所有这些对象都使用单独的绘制和任何其他可能需要的方法定义。这当然是不理想的，因为JavaScript会消耗浏览器内存，并使其运行非常缓慢，甚至停止响应。
虽然有时候可能不会有100个对象，但是仍然很致命的是，它将需要查找一百个不同的对象，而不仅仅是单个`Prototype`。

## 1.4. 如何使用Prototype
为了使应用程序运行得更快（并遵循最佳实践），我们可以（重新）定义`GameObject`的`Prototype`原型属性; `GameObject`的每个实例都将引用`GameObject.prototype`中的方法，就像它们是自己的方法一样。

```JavaScript
// define the GameObject constructor function
var GameObject = function(width, height) {
    this.x = Math.floor((Math.random() * myCanvasWidth) + 1);
    this.y = Math.floor((Math.random() * myCanvasHeight) + 1);
    this.width = width;
    this.height = height;
    return this;
};
 
// (re)define the GameObject prototype object
GameObject.prototype = {
    x: 0,
    y: 0,
    width: 5,
    width: 5,
    draw: function() {
        myCanvasContext.fillRect(this.x, this.y, this.width, this.height);
    }
};
```
然后我们可以把`GameObject`实例化100次
```JavaScript
var x = 100,
arrayOfGameObjects = [];
 
do {
    arrayOfGameObjects.push(new GameObject(10, 10));
} while(x--);
```
现在我们有一个100个`GameObjects`的数组，它们都共享了`draw`方法的相同Prototype和定义，它大大地节省了应用程序中的内存。
当我们调用`draw`方法的时候，它将会指向一个相同的`Function`
```JavaScript
var GameLoop = function() {
    for(gameObject in arrayOfGameObjects) {
        gameObject.draw();
    }
};
```
## 1.5. Prototype 是一个动态的对象
对象的原型是一个动态的对象，这意味着，如果在创建了所有的`GameObject`实例之后，我们决定，而不是绘制一个矩形，我们要绘制一个圆，我们可以相应地更新我们的`GameObject.prototype.draw`方法。
```JavaScript
GameObject.prototype.draw = function() {
    myCanvasContext.arc(this.x, this.y, this.width, 0, Math.PI*2, true);
}
```
而现在，所有以前的`GameObject`和任何未来的实例都会画一个圆。

## 1.6. Prototype 的典型示例

用过 `jQuery` 或者 `Prototype` 库的朋友可能知道，这些库中通常都会有 trim 这个方法。

**示例**
```JavaScript
String.prototype.trim = function() {
	return this.replace(/^\s+|\s+$/g, '');
};
```
**trim 用法**
```JavaScript
' foo bar   '.trim(); // 'foo bar'
```
但是这样做又有一个缺点，因为比较新版本的浏览器中的 `JavaScript` 引擎在 `String` 对象中本身就提供了 `trim` 方法， 那么我们自己定义的 trim 就会覆写它自带的 `trim`。其实，我们在定义 `trim` 方法之前，可以做个简单的检测，看是否需要自己添加这个方法：
```JavaScript
if(!String.prototype.trim) {
	String.prototype.trim = function() {
		return this.replace(/^\s+|\s+$/g, '');
	};
}
```
检查一下，如不存在 trim 这个方法，定义一个。

[原文地址](https://code.tutsplus.com/tutorials/prototypes-in-javascript--net-24949)