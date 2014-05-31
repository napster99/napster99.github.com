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
