---
layout: post
title: 细品JS-Object对象
categories : [JavaScript]
tags: [JS]
---
**The Object For JS**  

最近因某些人在学习JS，周末抽空整理了JS中一个最最基础，也最最重要玩样：**Object**  记住，它在JS中永远是主角。  
首先看下Object是什么，很简单，typeof Object -> "function" ,说明它是个函数。函数就分别拥有三个属性，分别是    \_\_proto\_\_、prototype、constructor。   
1.\_\_proto\_\_  
这是对象的内部原型（\_\_proto\_\_），所有构造器或函数的\_\_proto\_\_指向Function.prototype,它是一个空函数（Empty function）看下JS内置对象中的\_\_proto\_\_与Function.prototype的对比，或许你会发现什么....    
{% highlight javascript %}
Number.__proto__ === Function.prototype  // true
Boolean.__proto__ === Function.prototype // true
String.__proto__ === Function.prototype  // true
Object.__proto__ === Function.prototype  // true
Function.__proto__ === Function.prototype // true 
Array.__proto__ === Function.prototype   // true
RegExp.__proto__ === Function.prototype  // true
Error.__proto__ === Function.prototype   // true
Date.__proto__ === Function.prototype    // true
{% endhighlight %}

JS中有内置（build-in）构造器/对象共计12个（ES5中新加了JSON）,这里列举了可访问的9个构造器，剩下不能Global直接访问的，Arguments仅在函数调用时有JS引擎创建，Math、JSON是以对象形式存在的，无需new。它们的\_\_proto\_\_是Object.prototype。如下   
{% highlight javascript %}
Math.__proto__ === Object.prototype  // true 
JSON.__proto__ === Object.prototype  // true
{% endhighlight %}

上面说的“所有构造器/函数”当然包括自定义的，如下    
{% highlight javascript %}
//函数声明
function Base() {}
//函数表达式
var Foo = function() {}
Base.__proto__ === Function.prototype //true
Foo.__proto__ === Function.prototype  //true
{% endhighlight %}

这说明什么呢？所有的构造器都来自于Function.prototype，甚至包括构造器Object及Function自身。所有构造器都继承了Function.prototype的属性及方法。如length、call、apply、bind（ES5）。  

2.prototype   
在JS中所有的对象的属性和方法都继承Object.prototype，除非你显示的覆盖prototype属性，如Object.create(null);   
说到这个，在此就列举下Object.prototype下的属性和方法吧  
constructor、hasOwnProperty、isPrototypeOf、propertyIsEnumerable、toLocaleString、toString、valueOf
Function.prototype也是唯一一个typeof XXXX.prototype为"function"的prototype，其它的构造器都是一个对象。 
{% highlight javascript %}
console.log(typeof Function.prototype) // function
console.log(typeof Object.prototype)   // object
console.log(typeof Number.prototype)   // object
console.log(typeof Boolean.prototype)  // object
console.log(typeof String.prototype)   // object
console.log(typeof Array.prototype)    // object
console.log(typeof RegExp.prototype)   // object
console.log(typeof Error.prototype)    // object
console.log(typeof Date.prototype)     // object
console.log(typeof Object.prototype)   // object
{% endhighlight %}  

所有对象的\_\_proto\_\_都指向构造器的prototype，所有这些构造器的实例对象的\_\_proto\_\_指向谁？
先看看JS引擎内置构造器
{% highlight javascript %}
var obj = {name: 'jack'}
var arr = [1,2,3]
var reg = /hello/g
var date = new Date
var err = new Error('exception')
obj.__proto__ === Object.prototype // true
arr.__proto__ === Array.prototype // true
reg.__proto__ === RegExp.prototype // true
date.__proto__ === Date.prototype // true
err.__proto__ === Error.prototype // true
{% endhighlight %}  

再看看自定义的构造器，这里定义了一个Base   
{% highlight javascript %}
function Base(c) {
    this.c = c
}
var b = new Base('napster')
b.__proto__ === Base.prototype // true
{% endhighlight %}  

b是Base的实例对象，b的内部原型总是指向其构造器Base的prototype。每个对象都有一个constructor属性，可以获取它的构造器，因此以下打印结果也是恒等的。
{% highlight javascript %}  
function Base(c) {
    this.c = c
}
var b = new Base('napster')
b.__proto__ === b.constructor.prototype) // true
{% endhighlight %}   

3.constuctor  
很显然，每个JS对象都从原型继承了constructor属性  
{% highlight javascript %}  
var o = {};
o.constructor === Object; // true
var a = [];
a.constructor === Array; // true
var n = new Number(3);
n.constructor === Number; // true
{% endhighlight %}   

再举个列子展示constructor的输出。  
{% highlight javascript %}  
function Tree (name) {
   this.name = name;
}
var theTree = new Tree( "Redwood" );
console.log( "theTree.constructor is " + theTree.constructor );
//output
//theTree.constructor is function Tree (name) {
//    this.name = name;
//}
{% endhighlight %}   