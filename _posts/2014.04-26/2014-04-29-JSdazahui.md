---
layout: post
title: JS大杂烩
categories : [JavaScript]
tags: [JavaScript]
---
**js call 方法**

调用一个对象的一个方法，以另一个对象替换当前对象。 

{% highlight javascript %}
call([thisObj[,arg1[, arg2[, [,.argN]]]]]) 
{% endhighlight %}
参数 
thisObj 
可选项。将被用作当前对象的对象。 
arg1, arg2, , argN 
可选项。将被传递方法参数序列。 
说明 
call 方法可以用来代替另一个对象调用一个方法。call 方法可将一个函数的对象上下文从初始的上下文改变为由 thisObj 指定的新对象。 
如果没有提供 thisObj 参数，那么 Global 对象被用作 thisObj。 

乍一看，很容易把人看迷胡，先做一些简单的说明 
{% highlight javascript %}
obj1.method1.call(obj2,argument1,argument2) 
{% endhighlight %}

如上，call的作用就是把obj1的方法放到obj2上使用，后面的argument1..这些做为参数传入． 
举一个具体的例子 
{% highlight javascript %}
function add(a,b) 
{ 
    alert(a+b); 
} 
function sub(a,b) 
{ 
    alert(a-b); 
}
add.call(sub,3,1); 
{% endhighlight %}

这个例子中的意思就是用 add 来替换 sub，add.call(sub,3,1) == add(3,1) ，所以运行结果为：alert(4); // 注意：js 中的函数其实是对象，函数名是对 Function 对象的引用。 


看一个稍微复杂一点的例子 
{% highlight javascript %}
function Class1() 
{ 
    this.name = "class1"; 
    this.showNam = function() 
    { 
        alert(this.name); 
    } 
} 
function Class2() 
{ 
    this.name = "class2"; 
} 
var c1 = new Class1(); 
var c2 = new Class2(); 
c1.showNam.call(c2); 
{% endhighlight %}

注意，call 的意思是把 c1 的方法放到c2上执行，原来c2是没有showNam() 方法，现在是把c1 的showNam()方法放到 c2 上来执行，所以this.name 应该是 class2，执行的结果就是 ：alert（"class2"）; 

怎么样，觉得有意思了吧，可以让a对象来执行b对象的方法，这是java程序员所不敢想的。还有更有趣的，可以用 call 来实现继承 
{% highlight javascript %}
function Class1() 
{ 
    this.showTxt = function(txt) 
    { 
        alert(txt); 
    } 
} 
function Class2() 
{ 
    Class1.call(this); 
} 
var c2 = new Class2(); 
c2.showTxt("cc"); 
{% endhighlight %}
这样 Class2 就继承Class1了，Class1.call(this) 的 意思就是使用 Class1 对象代替this对象，那么 Class2 中不就有Class1 的所有属性和方法了吗，c2 对象就能够直接调用Class1 的方法以及属性了，执行结果就是：alert（“cc”）; 

对的，就是这样，这就是 javaScript 如何来模拟面向对象中的继承的，还可以实现多重继承。 
{% highlight javascript %}
function Class10() 
{ 
    this.showSub = function(a,b) 
    { 
        alert(a-b); 
    } 
} 
function Class11() 
{ 
    this.showAdd = function(a,b) 
    { 
        alert(a+b); 
    } 
} 
function Class2() 
{ 
    Class10.call(this); 
    Class11.call(this); 
} 
{% endhighlight %}

很简单，使用两个 call 就实现多重继承了 

当然，js的继承还有其他方法，例如使用原型链，这个不属于本文的范畴，只是在此说明call 的用法
说了call ，当然还有 apply，这两个方法基本上是一个意思
区别在于 call 的第二个参数可以是任意类型，而apply的第二个参数必须是数组

**在IE中起效（被选中的文字）**
{% highlight javascript %}
function test()
{
  var o=document.getElementById("inp1")
  var r = document.selection.createRange();
  if(o.createTextRange().inRange(r))
    alert(r.text);
}
{% endhighlight %}

页面文本倒序查找IE中运行firefox 中报错：document.body.createTextRange is not a function
[在此错误处中断]   

{% highlight html %}
var rng = document.body.createTextRange();
<body>
abababababababa
<input value="倒序查找a" onclick=myfindtext("a") type="button">
</body>
<script type="text/javascript" src="jquery-1.6.js"></script>
<script type="text/javascript">
var rng = document.body.createTextRange();
function myfindtext(text)
  {
    rng.collapse(false);
    if(rng.findText(text,-1,1))
    {
      rng.select();
      rng.collapse(true);
    }else{
      alert("end");
    }
  }
  //myfindtext("a");
</script>
{% endhighlight %}
光标定位

{% highlight javascript %}
function setPos(num)
{
  var e =document.getElementById("text5");
  var r =e.createTextRange();
  r.moveStart('character',num);
  r.collapse(true);
  r.select();
}
{% endhighlight %}
得到光标的位置

{% highlight html%}
<body>
  <input type="text" id="txt1" value="1234567890">
  <input type="button" value="得到光标位置" onclick=getPos(txt1)>
</body>
<script type="text/javascript" src="jquery-1.6.js"></script>
<script type="text/javascript">
  function getPos(obj){
    obj.focus();
    var s=document.selection.createRange();
    s.setEndPoint("StartToStart",obj.createTextRange())
    alert(s.text.length);
  }
</script>
{% endhighlight %}

选中一段文字
{% highlight javascript %}
function sel(obj,num)
{
  var rng=obj.createTextRange()
  var sel = rng.duplicate();
  sel.moveStart("character", num);
  sel.setEndPoint("EndToStart", rng);
  sel.select();
}
{% endhighlight %}
*******************************************华丽丽的分割线*************************************************
=====

**事件处理**
{% highlight javascript %}
var el = document.getElementById('xx');
function a(){
  alert(this.id);
  this.id = 'nap';
  el.removeEventListener('click',a,true);  //移除该函数
}
function b(){
  alert(this.id);
}
el.addEventListener("click",a,true);
el.addEventListener("click",b,true);
{% endhighlight %}

最后一个参数如果是true，表示在捕获阶段调用事件处理程序，
      如果是false，表示在冒泡阶段调用事件处理程序。

注：通过addEventListener()添加的时间处理程序只能使用removeEventListener()来移除
移除时传入的参数与添加处理程序时使用的参数相同，这也意味着通过addEventListener()
添加的匿名函数将无法移除。

IE中特有的方法。
{% highlight javascript %}
var el = document.getElementById('xx');
function a(){
  alert(this == window);  //true  在使用attachEvent()方法的情况下，事件处理程序会在全局作用于中运行。
}
el.attachEvent('onclick',a); ---第一个参数是加上on的
//绑定多次的话先从最后一个绑定方法开始执行。
el.detachEvent("onclick",handler);
{% endhighlight %}

{% highlight javascript %}
var EventUtil = {
  addHandler : function(element,type,handler){
    if(element.addEventListener){
      element.addEventListener(type,handler,false);
    }else if(element.attachEvent){
      element.attachEvent('on'+type,handler);
    }else{
      element['on'+type] = handler;
    }
  },
  removeHandler : function(element,type,handler){
    if(element.removeEventListener){
      element.removeEventListener(type,handler,false); 
    }else if(element.detachEvent){
      element.detachEvent('on'+type,handler);
    }else{
      element['on'+type] = null;
    }
  },
  getEvent : function(event){
    return event?event:window.event;
  },
  getTarget : function(event){
    return event.target || event.srcElement;
  },
  preventDefault : function(event){
    if(event.preventDefault){
      event.preventDefault();
    }else{
      event.returnValue = false;
    }
  },
  stopPropagation : function(){
    if(event.stopPropagation){
      event.stopPropagation();
    }else{
      event.cancelBubble = true;
    }
  },
  getRelatedTarget : function(event){
    if(event.relatedTarget){
      return event.relatedTarget;
    }else if(event.toElement){
      return event.toElement;
    }else if(event.fromElement){
      return event.fromElement;
    }else{
      return null;
    }
  },
  getButton  :  function(event){
    if(document.implementation.hasFeature('MouseEvents','2.0')){
      return event.button;
    }else{
      switch(event.button){
        case 0:
        case 1:
        case 3:
        case 5:
        case 7:
          return 0;
        case 2:
        case 6:
          return 2;
        case 4:
          return 1;
      }
    }
  },
  getCharCode  : function(event){
    if(typeof event.charCode == 'number'){
      return event.charCode;
    }else{
      return event.keyCode;
    }
  }
};
{% endhighlight %}

event对象的属性与方法  
bubbles boolean 只读 表明事件是否冒泡  
cancelable boolean 只读 表明是否可以取消事件的默认行为  
currentTarget Element 只读 其事件处理程序当前正在处理事件的那个元素  
detail Integer 只读 与事件相关的细节信息  
eventPhase Integer 只读 调用事件处理程序的阶段：1 表示捕获阶段 2 表示处于目标 3 表示冒泡阶段  
preventDefault() Function 只读 取消事件的默认行为。如果是cancelable是true 则可以使用这个方法  
stopProgpagation() Function 只读 取消事件的进一步捕获或冒泡，如果bubbles为true，则可以使用这个方法  
target Elment 只读 事件的目标  
type String 只读 被触发的事件的类型  
view AbstractView 只读 与事件相关联的抽象视图。等同于发生事件的window对象。

{% highlight javascript %}
var handler = function(event){
  switch(event.type){   //  event.type firefox 里有效  IE无效（不确定） --->>> event = window.event;  
    case 'click':
      alert('click');
      break;
    case 'mouseover':
      event.target.style.backgroundColor = 'red';
      break;
    case 'mouseout':
      event.target.style.backgroundColor = '';
      break;
  }
}
//IE 下如果事件处理程序是使用attachEvent()添加的，那么就会有一个event对象作为参数被传入事件处理程序函数中，
btn.attachEvent('onclick',function(event){ 
  alert(event.type);
});

var obj = document.getElementById('offline');

obj.onclick = handler;
obj.onmouseover = handler;
obj.onmouseout = handler;

event.stopPropagation();   //针对firefox safari chrome opare有效 IE无效

window.event.returnValue = false;  //可以阻止默认行为  
{% endhighlight %}

IE不支持事件捕获，只能取消事件冒泡 但 stopPropagation()可以同时取消事件捕获和冒泡

鼠标事件
click dblclick mousedown mouseup mouseover mouseout mousemove
在同一个元素上相继触发mousedown和mouseup事件，才会触发click事件
mousedown-mouseup-click-dblclick  
1.客户区坐标位置
event.clientX和event.clientY  
2.屏幕坐标位置
event.screenX 和 event.screenY  
3.修改键
Shift Ctrl Alt Meta(Windows下为Windows键 ，Mac下为Cmd键)
event.shiftKey --- > boolean

{% highlight javascript %}
EventUtil.addHandler(obj,'click',function(event){
  event = EventUtil.getEvent(event);
  var keys = new Array();
  if(event.shiftKey && event.altKey ) {
    alert('shsift');
  }
event.shiftKey  event.ctrlKey event.altKey event.metaKey(IE不支持) 
})  
{% endhighlight %}

4.相关元素
event.relatedTarget event.toElement event.fromElement  
5.鼠标按钮
IE也提供了button属性，但这个属性的值与DOM的button属性有很大差异.IE下列表  
0:表示没有按下按钮  
1:表示按下了主鼠标按钮  
2:表示按下了次鼠标按钮  
3:表示同时按下了主、次按钮  
4：表示按下了中间的鼠标按钮  
5：表示同时按下了主鼠标按钮和中间的鼠标按钮  
6：表示同时按下了次鼠标按钮和中间的鼠标按钮  
7：表示同时按下了三个鼠标按钮  
{% highlight javascript %}
document.implementation.hasFeature('MouseEvents','2.0')  IE-->false  Firefox && Chrome-->true
{% endhighlight %}  

“DOM2级事件”规范在event对象中还提供了detail属性


*******************************************华丽丽的分割线*************************************************
=====

一、仅定义text-overflow:ellipsis; 不能实现省略号效果。     
二、定义text-overflow:ellipsis; white-space:nowrap; 同样不能实现省略号效果。   
三、同时应用： text-overflow:ellipsis; white-space:nowrap; overflow:hidden;  可实现所想要得到的溢出文本显示省略号效果： 
{% highlight css %}
  .grayHead{
    FILTER: gray(color=#ffedff);  //头像灰白效果（下线效果）
  }
{% endhighlight %} 

*******************************************华丽丽的分割线*************************************************
=====
**我的正则**  
\xxx  查找以八进制数 xxx 规定的字符。  
  var str="Visit W3School. Hello World!";  
  var patt1=/\127/g;-->W  
\xdd  查找以十六进制数 dd 规定的字符。  
  var str="Visit W3School. Hello World!";  
  var patt1=/\x57/g;-->W  
\uxxxx  查找以十六进制数 xxxx 规定的 Unicode 字符。  
  var str="Visit W3School. Hello World!";  
  var patt1=/\u0057/g;-->W  

n+  匹配任何包含至少一个 n 的字符串。   +表示一次或者多次，等同于{1,}  
n*  匹配任何包含零个或多个 n 的字符串。 *表示0次或者多次，等同于{0,}  
n?  匹配任何包含零个或一个 n 的字符串。 ?表示0次或者1次，等同于{0,1}  
var patt1=/10?/g;  

n{X}  匹配包含 X 个 n 的序列的字符串。  
n{X,Y}  匹配包含 X 或 Y 个 n 的序列的字符串。 
n{X,} 匹配包含至少 X 个 n 的序列的字符串。   
n$  匹配任何结尾为 n 的字符串。   
^n  匹配任何开头为 n 的字符串。  
?=n 匹配任何其后紧接指定字符串 n 的字符串。  
  var str="Is this all there is";  
  var patt1=/is(?= all)/g;  
?!n 匹配任何其后没有紧接指定字符串 n 的字符串。  

**RegExp 对象属性**  
1.global  RegExp 对象是否具有标志 g。   
2.ignoreCase  RegExp 对象是否具有标志 i。
{% highlight javascript %}
var patt1 = new RegExp("S","i");  //在此设置
if(patt1.ignoreCase) {
      document.write("ignoreCase property is set");
} else {
      document.write("ignoreCase property is NOT set.");
}
{% endhighlight %} 

3.lastIndex 一个整数，标示开始下一次匹配的字符位置。  
{% highlight javascript %}
  var str = "The rain in Spain stays mainly in the plain";
  var patt1 = new RegExp("ain", "g");
  for(i = 0; i < 4; i++) 
    {
    patt1.test(str)
    document.write("ain found. index now at: " + patt1.lastIndex);
    //ain found. index now at: 8
    //ain found. index now at: 17
    //ain found. index now at: 28
    //ain found. index now at: 43
    document.write("<br />");
    }
  </script>
  {% endhighlight %} 

4.multiline RegExp 对象是否具有标志 m。
{% highlight javascript %}  
var str = "Visit W3School.com.cn";
var patt1 = new RegExp("s","m");
if(patt1.multiline) {
  document.write("Multiline property is set");
} else {
  document.write("Multiline property is NOT set.");
}
{% endhighlight %} 

5.source  正则表达式的源文本。
{% highlight javascript %}  
var patt1 = new RegExp("W3S","g");
document.write("The regular expression is: " + patt1.source); //输出：W3S
{% endhighlight %} 

SyntaxError - 如果 pattern 不是合法的正则表达式，或 attributes 含有 "g"、"i" 和 "m" 之外的字符，抛出该异常。

**RegExp 对象方法**  
1.compile 编译正则表达式。   
  定义和用法  
  compile() 方法用于在脚本执行过程中编译正则表达式,也可用于改变和重新编译正则表达式。   
  {% highlight javascript %}  
  /*
   * 语法：RegExpObject.compile(regexp,modifier)  
   * regexp  正则表达式。  
   * modifier  规定匹配的类型。"g" 用于全局匹配，"i" 用于区分大小写，"gi" 用于全局区分大小写的匹配。
   */
  var str="Every man in the world! Every woman on earth!";
    patt=/man/g;
  var str2=str.replace(patt,"person");//Every person in the world! Every woman on earth!
    alert(patt instanceof RegExp);  //true
    patt=/(wo)?man/g;
    patt.compile(patt);
    str2=str.replace(patt,"person");//Every person in the world! Every person on earth!
{% endhighlight %}  

2.exec  检索字符串中指定的值。返回找到的值，并确定其位置。  
  定义和用法  
    exec() 方法用于检索字符串中的正则表达式的匹配。
  {% highlight javascript %}  
  /*
   * 语法：RegExpObject.exec(string) 
   * 返回值：
   * 返回一个数组，其中存放匹配的结果。如果未找到匹配，则返回值为 null。
   */
    var str = "Visit W3School, W3School is a place to study web technology."; 
    var patt = new RegExp("W3School","g");
    var result;
    while ((result = patt.exec(str)) != null)  {
      document.write(result); //W3School
      document.write("<br />");
      document.write(patt.lastIndex);
      document.write("<br /><br />");
     }
{% endhighlight %} 

3.test  检索字符串中指定的值。返回 true 或 false。  
  定义和用法  
    test() 方法用于检测一个字符串是否匹配某个模式.  
 {% highlight javascript %} 
 /*
  * 语法：RegExpObject.test(string)
  * 返回值
  * 如果字符串 string 中含有与 RegExpObject 匹配的文本，则返回 true，否则返回 false。
  * 说明调用 RegExp 对象 r 的 test() 方法，并为它传递字符串 s，与这个表da式是等价的：(r.exec(s) != null)。
  */
    var str = "Visit W3School";
    var patt1 = new RegExp("W3School");
    var result = patt1.test(str);   //true
    var text = 'zhangxiaojian';
    alert(text.substring(0,1)+'   '+text.substring(5));
{% endhighlight %} 

*******************************************华丽丽的分割线*************************************************
=====
**use strict**  
ECMAScript5将严格模式（strict mode）引入了Javascript中，目的是允许开发人员能够选择“更好”的Javascript版本，这个版本能用不同的方式处理那些普遍而又臭名昭著的错误。一开始的时候，我对该模式抱着怀疑的态度，因为当时在只有一款浏览器（Firefox）支持严格模式。时至今日，所有的主流浏览器的最新版本——包括IE10与Opera12——都支持严格模式。使用严格模式的时机已经成熟了。

它带来了什么？
严格模式给Javascript的运行方式带来了许多不同，我将它们分为了两类：明显的（obvious），以及微妙的（subtle）。那些微妙的改变是为了解决微妙的问题，我不打算在这里对其进行赘述。如果你对这些细节感兴趣，请参考Dmitry Soshnikov的精彩文章，《ECMA-262-5 in Detail. Chapter 2. Strict Mode》。我对介绍明显的变化更感兴趣：它们是你开始使用严格模式之前所必须了解的，也可能给你带来最多好处。

在开始介绍特殊特性之前，你需要记住，严格模式的目标之一是允许更快地调试错误。帮助开发者调试的最佳途径是当确定的问题发生时抛出相应的错误（throw errors when certain patterns occur），而不是悄无声息地失败或者表现出奇怪的行为（这正是如今不在严格模式下的Javascript做的）。严格模式下的代码抛出更多的错误信息，这是好事，因为它能帮助开发者很快注意到一些必须立即解决的问题。

去除with语句（Eliminates with）
首先，严格模式去除了with语句。当with语句出现在严格模式中时，它会被认为是非法的Javascript语句并抛出语法错误。所以，使用严格模式的第一步就是确保你没有在使用with。

// 在严格模式中将导致语法错误
with (location) {
alert(href);
}
防止意外的全局变量（Prevents accidental globals）
第二点是，变量在赋值之前必须声明。在非严格模式下，给一个未声明的变量赋值将自动生成一个该名字的全局变量。这是Javascript中最普遍的错误之一。严格模式中，这样做将抛出一个错误。

// 严格模式中抛出一个错误
{% highlight javascript %} 
(function() {
someUndeclaredVar = "foo";
}());
取消this值的强制转换（Eliminates this coercion）
另一个重要的变化是，当this值为null或undefined时，不会再将其强制转换为全局对象。也就是说，this保留了它的原始值，也因此可能会导致一些依赖于强制转换的代码发生错误。例如：
window.color = "red";
function sayColor() {
// 严格模式下，this不会指向window
alert(this.color);
}
// 以下两种情况，在严格模式下都抛出错误
sayColor();
sayColor.call(null);
根本而言，this值必须赋值，否则将保留undefined值。这意味着调用构造函数时若漏掉了new关键字也会导致错误：
function Person(name) {
this.name = name;
}
{% endhighlight %} 

// 严格模式下导致错误
var me = Person("Nicholas");
在这段代码里，调用Person构造函数时缺少了new关键字，此时this值为undefined。由于你不能给undefined添加属性，这段代码抛出了一个错误。在非严格模式下，this会强制转换为全局对象，因此name属性能够被正确赋值为全局变量。
拒绝重复（No duplicates）
当你做了大量的编码的时候，你很容易在对象中定义了重复的属性或者给函数定义了重复的参数名。严格模式下，这两种情况都会导致错误的发生：
严格模式下错误 - 重复参数
function doSomething(value1, value2, value1) {
//code
}
严格模式下错误 - 重复属性
var object = {
foo: "bar",
foo: "baz"
};
这两者都是语法错误，在代码执行之前将抛出错误。
更安全的eval()（Safer eval()）
eval()没有被移除，但它在严格模式下发生了一些变化。最大的改变是：在eval()语句中声明的变量以及函数不会在包含域中创建。例如：
{% highlight javascript %} 
(function() {
eval("var x = 10;");
// 非严格模式下，x为10
// 严格模式下，x没有声明，抛出一个错误
alert(x);
}());
任意由eval()创建的变量或函数仍呆在eval()里。然而，你可以通过从eval()中返回一个值的方式实现值的传递：
(function() {
var result = eval("var x = 10, y = 20; x + y");
// 严格模式与非严格模式下都能正常工作（得到30）
alert(result);
}());
{% endhighlight %} 

不可改变引发的错误（Errors for immutables）
ECMAScript 5 同时引入了修改属性特征的能力，例如设置一个属性为只读或者冻结整个对象的结构（freezing an entire object’s structure）。在非严格模式下，试图修改一个不可变的属性时将悄无声息地失败。你可能在使用一些原生APIs的时候已经遇到这类问题。严格模式将保证无论你在何时试图使用一种不被允许的方式修改一个对象或对象的属性时抛出错误。
{% highlight javascript %} 
var person = {};
Object.defineProperty(person, "name" {
writable: false,
value: "Nicholas"
});
{% endhighlight %} 

// 非严格模式下将悄无声息地失败，严格模式则抛出错误
person.name = "John";
这个例子中，name属性被设置为只读。在非严格模式下，对name的赋值将悄无声息地失败；而在严格模式下，一个错误将被抛出。

注：如果你在使用ECMAScript属性能力（the ECMAScript attribute capabilities），我强烈推荐你开启严格模式。如果你在改变对象的可变性（mutability of objects），你将遇到一堆错误，而它们在非严格模式下将被安静地带过。

该如何使用它？
在现代浏览器中很容易启用严格模式，只需添加下面一条语句：

"use strict";虽然这看起来只是一个没有赋值给变量的字符串，但它确实地指示了Javascript引擎切换为严格模式（那些不支持严格模式的浏览器只是简单地读取了这个字符串然后继续像平常一样运行）。你可以在全局或函数中使用它。话虽这么说，你永远不应该在全局中使用它。全局地使用这条指示，意味着同个文件下的所有代码都在严格模式下运行。
// 别这么做
"use strict";
function doSomething() {
// 这将在严格模式下运行
}
function doSomethingElse() {
// 这也是
}
这看起来似乎不是个大问题，但在我们这个不同脚本堆积在一起的世界里（our world of aggressive script concatenation）将导致大麻烦。只要有一份脚本全局地包含这条指令，其它串联的脚本也将在严格模式下运行（可能引发一些你从没预想到的错误）。
因此，最好只在函数内使用严格模式，例如：
function doSomething() {
"use strict";
// 严格模式下运行
}
function doSomethingElse() {
// 非严格模式下运行
}
如果你想讲严格模式应用于多个函数，可以使用如下模式（ immediately-invoked function expression (IIFE)）：
{% highlight javascript %} 
(function() {
"use strict";
function doSomething() {
  // this runs in strict mode
}
function doSomethingElse() {
  // so does this
}
}());
{% endhighlight %} 

**结论**  
我强烈建议每一个人都开始使用严格模式。现在已经有足够多的浏览器支持该模式，它将把你从藏身代码的错误中拯救出来。
你需要确保你没有全局地包含启用指令，但可以频繁地使用IIFEs给任意多的代码应用严格模式。一开始，你将碰到从没遇过的错误——这是很正常的。
切换到严格模式后，你需要做足够多的测试来保证你已hold住你的代码。
一定不能只是将“use strict”扔进你的代码然后就假设不会有错误发生。至少的至少，你该开始使用这个异常有用的语言特性来写更好的代码了。

