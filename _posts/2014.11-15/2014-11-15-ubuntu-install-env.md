---
layout: post
title: Ubuntu下安装NodeJS，npm，express，mongodb
categories : [Linux, NodeJS,npm, express, mongodb]
tags: []
---
Ctrl + Alt + T，打开终端，输入以下命令安装：  
{% highlight javascript %}
sudo apt-get install nodejs
{% endhighlight %}  
安装完成后，终端输入nodejs，就能进入node命令啦；

但是正常下应该是输入node进入命令而不是nodejs；

在Ubuntu下node是属于其他应用的，不过继续输入下面的命令安装，完成后就可以使用node进入命令啦。  
{% highlight javascript %}
sudo apt-get install nodejs-legacy
{% endhighlight %}  
nodejs安装完后，继续安装npm；  
{% highlight javascript %}
sudo apt-get install npm  
sudo npm install -g express  
{% endhighlight %}  
//-g是执行全局安装
{% highlight javascript %}
sudo npm install -g express@3.4.8  
{% endhighlight %}  
//@后面是要安装的版本
 
注：在4.x以后，还需要安装 express-generator ，不然 express 用不了  
{% highlight javascript %}
sudo npm install -g express-generator
 {% endhighlight %}  
启动是 npm start  
在用 npm 安装 package 时，有时候会报 304 ，这个时候就需要清一下缓存啦：  
{% highlight javascript %}
npm cache clean
{% endhighlight %}  
接下来安装数据库， MongoDB  
{% highlight javascript %}
sudo apt-get install mongodb-clients
sudo apt-get install mongodb-server
{% endhighlight %}  
安装完MongoDB，终端下运行mongo启动服务！