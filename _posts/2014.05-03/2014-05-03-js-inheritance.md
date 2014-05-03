---
layout: post
title: JS继承5大方法
categories : [JavaScript]
tags: [继承,JS]
---
**1、构造继续法**  
{% highlight javascript %}
//定义一个Collection类型 
function Collection(size) 
{ 
  this.size = function(){return size}; //公有方法，可以被继承 
} 
  Collection.prototype.isEmpty = function(){ //静态方法，不能被继承 
  return this.size() == 0; 
} 
//定义一个ArrayList类型，它"继承"Collection类型 
function ArrayList() 
{ 
  var m_elements = []; //私有成员，不能被继承 
  m_elements = Array.apply(m_elements, arguments); 
  //ArrayList类型继承Collection 
  this.base = Collection; 
  this.base.call(this, m_elements.length); 
  this.add = function() 
  { 
    return m_elements.push.apply(m_elements, arguments); 
  } 
  this.toArray = function() 
  { 
    return m_elements; 
  } 
} 
ArrayList.prototype.toString = function() 
{ 
  return this.toArray().toString(); 
} 
//定义一个SortedList类型，它继承ArrayList类型 
function SortedList() 
{ 
  //SortedList类型继承ArrayList 
  this.base = ArrayList; 
  this.base.apply(this, arguments); 
  this.sort = function() 
  { 
    var arr = this.toArray(); 
    arr.sort.apply(arr, arguments); 
  } 
} 
//构造一个ArrayList 
var a = new ArrayList(1,2,3); 
console.log(a); 
console.log(a.size()); //a从Collection继承了size()方法 
console.log(a.isEmpty); //但是a没有继承到isEmpty()方法 
//构造一个SortedList 
var b = new SortedList(3,1,2); 
b.add(4,0); //b 从ArrayList继承了add()方法 
console.log(b.toArray()); //b 从ArrayList继承了toArray()方法 
b.sort(); //b 自己实现的sort()方法 
console.log(b.toArray()); 
console.log(b); 
console.log(b.size()); //b从Collection继承了size()方法 
{% endhighlight %}


**2、原型继承法**  
{% highlight javascript %}
//定义一个Point类型 
function Point(dimension) 
{ 
  this.dimension = dimension; 
} 
//定义一个Point2D类型，"继承"Point类型 
function Point2D(x, y) 
{ 
  this.x = x; 
  this.y = y; 
} 
Point2D.prototype.distance = function() 
{ 
  return Math.sqrt(this.x * this.x + this.y * this.y); 
} 
Point2D.prototype = new Point(2); //Point2D继承了Point 
//定义一个Point3D类型，也继承Point类型 
function Point3D(x, y, z) 
{ 
  this.x = x; 
  this.y = y; 
  this.z = z; 
} 
Point3D.prototype = new Point(3); //Point3D也继承了Point 
//构造一个Point2D对象 
var p1 = new Point2D(0,0); 
//构造一个Point3D对象 
var p2 = new Point3D(0,1,2); 
console.log(p1.dimension); 
console.log(p2.dimension); 
console.log(p1 instanceof Point2D); //p1 是一个 Point2D 
console.log(p1 instanceof Point); //p1 也是一个 Point 
console.log(p2 instanceof Point); //p2 是一个Point 
{% endhighlight %}  

以上两种方法是最常用的    
 
**3、实例继承法**  
在说此法例子之前，说说构造继承法的局限，如下： 
{% highlight javascript %}
function MyDate() 
{ 
  this.base = Date; 
  this.base.apply(this, arguments); 
} 
var date = new MyDate(); 
alert(date.toGMTString); //undefined，date并没有继承到Date类型，所以没有toGMTString方法 
{% endhighlight %}

核心对象的某些方法不能被构造继承，原因是核心对象并不像我们自定义的一般对象那样在构造函数里进行赋值或初始化操作换成原型继承法呢？，如下：   
{% highlight javascript %}
function MyDate(){} 
MyDate.prototype=new Date(); 
var date=new MyDate(); 
alert(date.toGMTString); //'[object]'不是日期对象，仍然没有继承到Date类型！ 
{% endhighlight %}  

现在，换成实例继承法：  
{% highlight javascript %} 
function MyDate() 
{ 
  var instance = new Date(); //instance是一个新创建的日期对象 
  instance.printDate = function(){ 
    document.write("<p> "+instance.toLocaleString()+"</p> "); 
  } //对instance扩展printDate()方法 
  return instance; //将instance作为构造函数的返回值返回 
} 
var myDate = new MyDate(); 
dwn(myDate.toGMTString()); //这回成功输出了正确的时间字符串，看来myDate已经是一个Date的实例 
{% endhighlight %}

继承成功了  
{% highlight javascript %} 
myDate.printDate(); //如果没有return instance，将不能以下标访问，因为是私有对象的方法 
{% endhighlight %}

**4、拷贝继承法**  
{% highlight javascript %}
Function.prototype.extends = function(obj) 
{ 
  for(var each in obj) 
  { 
    this.prototype[each] = obj[each]; 
    //对对象的属性进行一对一的复制，但是它又慢又容易引起问题 
    //所以这种“继承”方式一般不推荐使用 
  } 
} 
var Point2D = function(){ 
//…… 
} 
Point2D.extends(new Point()) 
{ 
//…… 
} 
{% endhighlight %}  

这种继承法似乎是用得很少的。   
**5、混合继承**  
{% highlight javascript %} 
function Point2D(x, y) 
{ 
  this.x = x; 
  this.y = y; 
} 
function ColorPoint2D(x, y, c) 
{ 
  Point2D.call(this, x, y); //这里是构造继承，调用了父类的构造函数 
  //从前面的例子看过来，这里等价于 
  //this.base=Point2D; 
  //this.base.call(this,x,y); 
  this.color = c; 
} 
ColorPoint2D.prototype = new Point2D(); //这里用了原型继承，让ColorPoint2D以Point2D对象为原型 
{% endhighlight %}