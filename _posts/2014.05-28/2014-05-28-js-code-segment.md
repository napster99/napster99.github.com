---
layout: post
title: JS代码片段
categories : [JavaScript]
tags: [JS]
---

**1.高仿Function.prototype.bind方法**  
由于bind()方法在IE6、IE7下不支持，所以需要模拟实现一个。  
代码如下     
{% highlight javascript %}
(function () {
            var proxy = function (fn, target) {
                var proxy = function () {
                    if (2 < arguments.length) { 
                        var privateArgs = Array.prototype.slice.call(arguments, 2); 
                        return function () {
                            var args = Array.prototype.slice.call(arguments);
                            Array.prototype.unshift.apply(args,privateArgs);
                            return fn.apply(target, args);
                        }
                    }
                    return function () {
                        return fn.apply(target, arguments);
                    }
                }
                return proxy.apply(null, arguments);
            };
            /*支持原生的使用原生的*/
            Function.prototype.bind = Function.prototype.bind ||
            function (target) {
                if (1 < arguments.length) {
                    var args = Array.prototype.slice.call(arguments, 1); 
                    args.unshift(this, target);
                    return proxy.apply(null, args);
                }
                return proxy(this, target);
            };
})();
{% endhighlight %}

**2.创建新类**  
{% highlight javascript %}
/**
 * Creates a new class
 *
 * @param superClass
 * @param methods
 */
function clazz(SuperClass, methods) {
    var constructor = function () {};
    constructor.prototype = new SuperClass;
    constructor.prototype.constructor = constructor;
    constructor.prototype.parent = SuperClass.prototype;
    constructor.prototype = $.extend(constructor.prototype, methods);
    return constructor;
}
{% endhighlight %}



**3.绑定函数的this指针(scope)**  
{% highlight javascript %}
function bind(func, obj)
{
    return function(){
        func.apply(obj, arguments);
    };
}
var obj = {
    name : "WEB应用开发",
    url  : "http://www.zeroplace.cn"
};
var func = bind(function(param){
    alert(this.name + " " + this.url + " " + param);
}, obj);
func("很不错的网站");  
{% endhighlight %}  

**4. 获取和设置COOKIE**  
{% highlight javascript %}
function setCookie(key, value, path, domain, expires, secure)  
{
    var cookie = [];
    cookie.push(key + "=" + escape(value || ""));
    if (!value) {
        expires = new Date(Date.now() - 1).toGMTString();
    }
    if (path) {
        cookie.push("path=" + path);
    }
    if (domain) {
        cookie.push("domain=" + domain);
    }
    if (expires) {
        cookie.push("expires=" + expires);
    }
    if (secure) {
        cookie.push("expires");
    }
    console.log(document.cookie = cookie.join(";"));
}
function getCookie(name)
{
    var pattern = new RegExp(name + "=(.*?)(?:;|$)");  
    var arr = document.cookie.match(pattern);
    if (arr) {
        return arr[1];
    } else {
        return null;
    }
}
{% endhighlight %}  

**5.JS时间格式化**  
{% highlight javascript %}
// 对Date的扩展，将 Date 转化为指定格式的String  
// 月(M)、日(d)、小时(h)、分(m)、秒(s)、季度(q) 可以用 1-2 个占位符，   
// 年(y)可以用 1-4 个占位符，毫秒(S)只能用 1 个占位符(是 1-3 位的数字)   
// 例子：   
// (new Date()).Format("yyyy-MM-dd hh:mm:ss.S") ==> 2006-07-02 08:09:04.423   
// (new Date()).Format("yyyy-M-d h:m:s.S")      ==> 2006-7-2 8:9:4.18   
Date.prototype.Format = function(fmt) { //author: meizz
    var o = {
        "M+": this.getMonth() + 1, //月份
        "d+": this.getDate(), //日 
        "h+": this.getHours(), //小时
        "m+": this.getMinutes(), //分
        "s+": this.getSeconds(), //秒
        "q+": Math.floor((this.getMonth() + 3) / 3), //季度
        "S": this.getMilliseconds() //毫秒
    };
    if (/(y+)/.test(fmt)) fmt = fmt.replace(RegExp.$1, (this.getFullYear() + "").substr(4 - RegExp.$1.length));
    for (var k in o)
        if (new RegExp("(" + k + ")").test(fmt)) fmt = fmt.replace(RegExp.$1, (RegExp.$1.length == 1) ? (o[k]) : (("00" + o[k]).substr(("" + o[k]).length)));
    return fmt;
}
//调用：
var time1 = new Date().Format("yyyy-MM-dd");
var time2 = new Date().Format("yyyy-MM-dd HH:mm:ss");
{% endhighlight %}  


**6.JS克隆对象**  
{% highlight javascript %}
function clone(obj) {  
    var o;
    switch (typeof obj) {
        case "undefined':
            break;
        case 'string':
            o = obj + '';
            break;
        case 'number':
            o = obj - 0;
            break;
        case 'boolean':
            o = obj;
            break;
        case 'object':
            if (obj === null) {
                o = null;
            } else {
                if (obj instanceof Array) {
                    o = [];
                    for (var i = 0, len = obj.length; i < len; i++) {
                        o.push(clone(obj[i]));
                    }
                } else {
                    o = {};
                    for (var k in obj) {
                        o[k] = clone(obj[k]);
                    }
                }
            }
            break;
        default:
            o = obj;
            break;
    }
    return o;
}
{% endhighlight %}  


**7.手机端判断是否苹果浏览器**  
{% highlight javascript %}
var scale = 1.0, ratio = 1, assetsHost = 'g.tbcdn.cn', assetsVersion = '0.5.14', isTestEnv = false;
(function() {
    if (location.host.match(/(waptest|wapa)\.taobao\.com$/i)) {
        assetsHost = 'g.assets.daily.taobao.net';
        assetsVersion = '0.5.14';
        isTestEnv = true;
    }
    if (window.devicePixelRatio === 2 && window.navigator.appVersion.match(/iphone/gi)) {
        scale = 0.5;
        ratio = 2;
    }
    var text = '<meta name="viewport" content="initial-scale=' + scale + ', maximum-scale=' + scale +', minimum-scale=' + scale + ', user-scalable=no" />'
        + '<link' + ' type="text/css" rel="styleSheet" href="http://' + assetsHost + '/mtb/app-index/' + assetsVersion + '/index' + ratio + '.css"' + ' />';
    document.write(text);
})();
{% endhighlight %}  


**8.原生javascript获取元素样式属性值的方法**  
{% highlight javascript %}
function attrStyle(elem,attr){ 
    if(elem.attr){ 
        //若样式存在于html中,优先获取 
        return elem.style[attr]; 
    }else if(elem.currentStyle){ 
        //IE下获取CSS属性最终样式(同于CSS优先级) 
        return elem.currentStyle[attr]; 
    }else if(document.defaultView && document.defaultView.getComputedStyle){ 
        //W3C标准方法获取CSS属性最终样式(同于CSS优先级) 
        //注意,此法属性原格式(text-align)获取的,故要转换一下 
        attr=attr.replace(/([A-Z])/g,'-$1').toLowerCase(); 
        //获取样式对象并获取属性值 
        return document.defaultView.getComputedStyle(elem,null).getPropertyValue(attr); 
    }else{ 
        return null; 
    } 
}
{% endhighlight %} 


**9.解决mouseover与mouserout事件不停切换的问题（问题不是由冒泡产生的）**  
{% highlight javascript %}
function checkHover(e, target) {
  if (getEvent(e).type == "mouseover") {
    return !contains(target, getEvent(e).relatedTarget
            || getEvent(e).fromElement)
            && !((getEvent(e).relatedTarget || getEvent(e).fromElement) === target);  
  } else {
    return !contains(target, getEvent(e).relatedTarget
            || getEvent(e).toElement)
            && !((getEvent(e).relatedTarget || getEvent(e).toElement) === target);
  }
}
function contains(parentNode, childNode) {
  if (parentNode.contains) {
    return parentNode != childNode && parentNode.contains(childNode);
  } else {
    return !!(parentNode.compareDocumentPosition(childNode) & 16);
  }
}
//取得当前window对象的事件
function getEvent(e) {
    return e || window.event;
} 
{% endhighlight %} 

