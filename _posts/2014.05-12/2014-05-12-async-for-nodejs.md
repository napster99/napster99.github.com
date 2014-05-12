---
layout: post
title: 异步编程 For NodeJS
categories : [NodeJS]
tags: [async,NodeJS]
---
前言：有异步I/O,必有异步编程。  
上一章描述了Node如何通过事件循环实现异步，包括与各种I/O多路复用搭配实现的异步I/O以及与I/O无关的异步。  
Node是首个将异步大规模带到应用层的平台，它从内在运行机制到API的设计，无不透露出异步的气息来。异步的高性能为它带来了高度的赞誉，而异步编程也为其带来部分的诋毁。  

前述章节中亦描述异步I/O在应用层面不流行的原因,那便是异步编程在流程控制中，业务表达并不太适合自然语言的线性思维习惯。较少人能适应直接面对事件驱动进行编程，唯独对它熟悉的主要是GUI开发者，如前端工程师或GUI工程师。前端工程师以为常并能够娴熟地处理各种DOM事件和浏览器中的事件，RyanDahl偏好事件驱动，而JavaScript在浏览器中也正契合事件驱动的过程，这也使得前后端的JavaScript在执行原理和风格上都趋于一致。虽然语言执行在不同的环境，但除了宿主提供的API有所不同外，并不让人觉得是一门新语言。V8和异步I/O在性能上带来的提升，前后端javascript编程风格一致，是Node能够迅速成功流行起来的主要原因。

**函数式编程**  
在开始异步编程之前，先得知晓Javascript现今的回调函数和深层嵌套的来龙去脉。在javascript中，函数（function）作为一等公民，使用上非常自由，无论调用它，或者作为参数，或者作为返回值均可。函数的灵活性是javascript比较吸引人的地方之一，它与古老的Lisp语言颇具渊源。javascript在诞生之前，Brendan Eich借鉴了Scheme语言，吸引了函数式编程的精华，将函数作为一等公民便是典型案例。  
鉴于函数式编程在近年来重新火热，而前端类图书中较少述及这部分知识，这里稍作补充，因为它是javascript异步编程的基础。  
1、高阶函数  
在通常的语言中，函数的参数只接受基本的数据类型或是对象引用，返回值也只是基本数据类型和对象引用。  
下面的代码为常规的参数传递和返回：  
{% highlight javascript %}
function foo(xx){
    return x;
}
{% endhighlight %}

高阶函数则是可以把函数作为参数，或是将函数作为返回值的函数，如下面代码所示：  
{% highlight javascript %}
function foo(x) {
    return function() {
        return x;
    }
}
{% endhighlight %}  

高阶函数可以将函数作为输入或返回值的变化看起来虽细小，但是对于C/C++语言而言，通过指针也可以达到相同的效果。但对于程序编写，高阶函数则比普通的函数要灵活许多。除了通常意义的函数调用返回外，还形成了一种后续传递风格（Continuation Passing Style）的结果接收方式，而非单一的返回值形式。后续传递风格的程序编写将函数的业务重点从返回值转移到了回调函数中：  
{% highlight javascript %}
function foo(x, bar) {
    return bar(x);
}
{% endhighlight %}    

以上面的代码为例，对于相同的foo函数，传入的bar参数不同，则可以得到不同的结果。一个经典的例子便是数组的sort()方法，它是一个货真价实的高阶函数，可以接受一个方法作为参数与运算排序：  
{% highlight javascript %}
var points = [40,100,1,5,25,10];
points.sort(function(a, b) {
    return a - b;
});
//[1,5,10,25,40,100]
{% endhighlight %}  

通过改动sort()方法的参数，可以决定不同的排序方式，从这里可以看出高阶函数的灵活性来。结合Node提供的最基本的事件模块可以看到，事件的处理方式正是基于高阶函数的特性来完成的。在自定义事件实例中，通过为相同事件注册不同的回调函数，可以很灵活地处理业务逻辑。
示例代码如下：  
{% highlight javascript %}
var emitter = new event.EventEmitter();
emitter.on('event_foo',function() {
    //TODO
});
{% endhighlight %}  

高阶函数在javascript中比比皆是，其中ECMAScript5中提供的一些数组方法（forEach()、map()、reduce()、reduceRight()、filter()、every()、some()）十分典型。  

2、偏函数用法  
偏函数用法是指创建一个调用另外一个部分————参数或变量已经预置的函数————的函数的用法。这句话想对较为拗口，下面我们以实例来说明：  
{% highlight javascript %}
var toString = Object.prototype.toString;
var isString = function(obj){
    return toString.call(obj) === '[object String]';
};
var isFunction = function(obj) {
    return isFunction.call(obj) === '[object Function]';
};
{% endhighlight %}  

在javascript中进行类型判断时，我们通常会进行类似上述代码的定义。这段代码固然不复杂，只有两个函数的定义，但是里面存在的问题是我们需要重复去定义一些相似的函数，如果有更多的isXXX(),就会出现更多的冗余代码。为了解决重复定义问题，我们引入一个新函数，这个心函数可以如工厂一样批量创建一些类似的函数。在下面的代码中，我们通过isType()函数预先指定type的值，然后返回一个新的函数：
{% highlight javascript %}
var isType = function(type) {
    return function(obj) {
        return toString.call(obj) === '[object ' + type + ']';
    };
};
var isString  = isType('String');
var isFunction  = isType('Function');
{% endhighlight %}  

可以看出，引入isType()函数后，创建isString()、isFunction()函数就变得简单多了。这种通过指定部分参数产生一个新的定制函数的形式就是偏函数。  
偏函数应用在异步编程中也十分常见，著名类库Underscore提供的after()方法即是偏函数应用，其定义如下：  
{% highlight javascript %}
_.after = function(times, func) {
    if(times <= 0) return func();
    return function() {
        if(--times < 1) {
            return func.apply(this,arguments);
        };
    };
};
{% endhighlight %}  

这个函数可以根据传入的times参数和具体方法，生成一个需要调用多次才真正执行实际函数的函数。  

**异步编程的优势与难点**    
曾经的单线程模型在同步I/O的影响下，由于I/O调用缓慢，在应用层面导致CPU与I/O无法重叠进行。为了照顾编程人员的阅读思维习惯，同步I/O盛行了很多年。但在日新月异的技术大潮面前，性能问题摆在了编程人员的面前。提升性能的方式过去多用线程的方式解决，但是多线程的引入在业务逻辑方面制造的麻烦也不少。从操作系统调度多线程的上下文切换开销，到实际编程里的锁、同步等问题，让开发人员头疼的时候也并不少。另一个解决I/O性能的方案是通过C/C++调用操作系统底层接口，自己手工完成异步I/O，这能够达到很高的性能，但是调试和开发门槛均十分高，在帮助业务解决问题上，需要花费较大的精力。Node利用javascript及其内部异步库，将异步直接提升到业务层面，这是一种创新。  

1.优势  
Node带来的最大特性莫过于基于事件驱动的非阻塞I/O模型，这是它的灵魂所在。非阻塞I/O可以使CPU与I/O并不相互依赖等待，让资源得到更好的利用。对于网络应用而言，并行带来的想象空间更大，延展而开的是分布式和云。并行使得各个单点之间能够更有效地组织起来，这也是Node在云计算厂商中广受青睐的原因。如果采用传统的同步I/O模型，分布式计算中性能的折扣将会是明显的。利用事件循环的方式，javascript线程像一个分配任务和处理结果的大管家,I/O线程池里的各个I/O线程都是小二，负责兢兢业业地完成分配来的任务，小二与管家之间互不依赖，所以保持整体的高效率。这个利用事件循环的经典调度方式在很多地方都存在应用，最典型的是UI编程，如IOS应用开发等。  
这个模型的缺点则在于管家无法承担过多的细节性任务，如果承担太多，则会影响到任务的调度，管家忙个不停，小二却得不到活干，结局则是整体效率的降低。  
换言之，Node是为了解决编程模型中阻塞I/O的性能问题，采用了单线程模型，这导致Node更像一个处理I/O密集问题的能手，而CPU密集型则取决于管家的能耐如何。  
由于事件循环模型需要应对海量请求，同时作用在单线程上，就需要防止任何一个计算耗费过多的CPU时间片。至于是计算密集型，还是I/O密集型，只要计算不影响异步I/O的调度，那就不构成问题。建议对CPU的耗用不要超过10ms，或者将大量的计算分解成诸多的小量计算，通过setImmediate()进行调度。只要合理利用Node的异步模型与V8的高性能，就可以充分发挥CPU和I/O资源的优势。

2.难点  
Node令异步编程如此风行，这也是异步编程首次大规模出现在业务层面。它借助异步I/O模型及V8高性能引擎，突破单线程的性能瓶颈，让javascript在后端达到实用价值。另一方面，它也统一了前后端javascript的编程模型。对已异步编程带来的新鲜感与不适感，开发者们有着不同程度的感受。接下来，我们梳理下异步编程的难点，以更好地利用Node。  
###难点1：异常处理 
过去我们处理异常时，通常使用类Java的try/catch/final语句块进行异常捕获，示例代码如下：  
{% highlight javascript %}
try{
    JSON.parse(json);
}catch (e) {
    //TODO
}
{% endhighlight %}  

但是这对于异步编程而言并不一定适用。异步I/O的实现主要包含两个阶段：提交请求和处理结果。这个两个阶段中间有事件循环的调度，两者彼此不关联。异步方法则通常在第一个阶段提交请求后立即返回，因为异常不不一定发生在这个阶段，try/catch的功效在此处不会发挥任何作用。异步方法的定义如下所示：   
{% highlight javascript %}
var async = function(callback) {
    process.nextTick(callback);
}
{% endhighlight %}  

调用async()方法后，callback被存起来，直到下一个事件循环（Tick）才会取出来执行。尝试对异步方法进行try/catch操作只能捕获当次事件循环内的异常，对callback执行时抛出的异常将无能为力，示例代码如下：  
{% highlight javascript %}
try{
    async(callback);
}catch(e) {
    //TODO
}
{% endhighlight %}  

Node在处理异常上形成了一种约定，将异常作为回调函数的第一个实参传回，如果为空值，则表明异步调用没有异常抛出：  
{% highlight javascript %}
async(function(err, status) {
    //TODO
})
{% endhighlight %}  

在我们自行编写的异步方法上，也需要去遵循这样一些原则：  
原则一：必须执行调用者传入的回调函数   
原则二：正确传递回异常供调用者判断  
示例代码如下：  
{% highlight javascript %}
var async = function(callback) {
    process.nextTick(function() {
        var results = something;
        if(error) {
            return callback(error);
        }
        callback(null, results);
    });
};
{% endhighlight %}  

在异步方法的编写中，另一个容易犯的错误是对用户传递的回调函数进行异常捕获，示例代码如下：  
{% highlight javascript %}
try{
    req.body = JSON.parse(buf, options.reviver);
    callback();
}catch(err) {
    err.body = buf;
    err.status = 400;
    callback(err);
}
{% endhighlight %}  

上述代码的意图是捕获JSON.parse()中可能出现的异常，但是却不小心包含了用户传递的回调函数。这意味着如果回调函数中有异常抛出，将会进入cahtch()代码块中执行，于是回调函数将会被执行两次。这显然不是预期的情况，可能导致业务混乱。正确的捕获应当为：   
{% highlight javascript %}
try{
    req.body = JSON.parse(buf, options.reviver);
}catch(err) {
    err.body = buf;
    err.status = 400;
    return callback(err);
}
callback();
{% endhighlight %}  

在编写异步方法时，只要将异常正确地传递给用户的回调方法即可，无须过多处理。  

###难点2：函数嵌套过深  
这或许是Node被人诟病最多的地方。在前端开发中，DOM事件相对而言不会存在相互依赖或需要多个事件一起协作的场景，较少存在异步多级依赖的情况。不过在Node中Mongodb查询多表数据的时候，会出现多级依赖的情形，使用genertor可以很好的解决，后续再议。下面的彼此独立的DOM事件绑定：  
{% highlight javascript %}
$(selector).click(function(event) {
    //TODO
});
$(selector).change(function(event) {
    //TODO
});
{% endhighlight %}  
但是对于Node而言，事务中存在多个异步调用的场景比比皆是。比如一个遍历目录的操作，其代码如下：  
{% highlight javascript %}
fs.readdir(path.join(__dirname,'..'),function(err, files) {
    files.forEach(function(filename,index) {
        fs.readFile(filename, 'utf-8', function(err, file) {
            //TODO
        });
    });
});
{% endhighlight %}  

对于上述场景，由于两次操作存在依赖关系，函数嵌套的行为也许情有可原。那么在网页渲染的过程中，通常需要数据、模板、资源文件，这三者互相之间并不依赖，但最终渲染结果中三者缺一不可。如果采用默认的异步方法调用，程序也许将会如下所示：  
{% highlight javascript %}
fs.readFile(template_path,'utf-8', function(err, template) {
    db.query(sql,function(err,data) {
        l10n.get(function(err, resources) {
        //TODO
        });
    });
});
{% endhighlight %} 

这在结果的保证上是没有问题的，问题在于这并没有利用好异步I/O带来的并行优势。这是异步编程的典型问题，为此有人曾说，因为嵌套的深度，未来最难看的代码必将从Node中诞生。但是实际情况没有想象得那么糟糕，且看后面如何解决该问题。  

###难点3：阻塞代码  
对于进入javascript世界不久的开发者，比较纳闷这门编程语言竟然没有sleep()这样的线程沉睡功能，唯独能用于延时操作的只有setInterval()和setTimeout()这两个函数。但是让人惊讶的是，这个两个函数并不能阻塞后续代码的持续执行。所有，有多半的开发者会写出下述这样的代码来实现sleep(1000)的效果：   
{% highlight javascript %}
//TODO
var start = new Date();
while(new Date() - start < 1000) {
    //TODO
}
//需要阻塞的代码
{% endhighlight %} 

但是事实是糟糕的，这段代码会持续占用CPU进行判断，与真正的线程沉睡想去甚远，完全破坏了事件循环的调度。由于Node单线程的原因，CPU资源全都会用于为这段代码服务，导致其余任何请求都会得不到响应。  
遇见这样的需求时，在统一规划业务逻辑之后，调用setTimeout()的效果会更好。  

###难点4：多线程编程   
我们在谈论javascript的时候，通常谈的是单一线程上执行的代码，这在浏览器中指的是javascript执行线程与UI渲染共用的一个线程，在Node中，只是没有UI渲染的部分，模型基本相同。对于服务器而言，如果服务器多核CPU，单个Node进程实质上是没有充分利用多核CPU的。随着现今业务的复杂化，对于多核CPU利用的要求也越来越高。浏览器提出了web workers，它通过将javascript执行与UI渲染分离，可以很好地利用多核CPU为大量计算服务。同时前端web workers也是一个利用消息机制合理使用多核CPU的理想模型。  
遗憾在于前端浏览器存在对标准的滞后性，web workers并没有广泛应用起来。另外web workers能解决利用CPU和减少阻塞UI渲染，但是不能解决UI渲染的效率问题。Node借鉴了这个模式，child_process是其基础API，cluster模块是更深层次的应用。借助web workers的模式，开发人员要更多地去面临跨线程的编程，这对于以往的javascript编程经验是较少考虑的  

###难点5：异步转同步   
习惯异步编程的同学，也许能够从容面对异步编程带来的副产品，比如嵌套回调、业务分散等问题。Node提供了绝大部分的异步API和少量的同步API，偶尔出现的同步需求将会因为没有同步API让开发者突然无所适从。目前，Node中试图同步式编程，但并不能得到原生支持，需要借助库或者编译等手段来实现。但对于异步调用，通过良好的流程控制，还是能够将逻辑梳理成顺序式的形式。
