---
layout: post
title: Grunt相关插件
categories : [NodeJS]
tags: [Grunt,NodeJS]
---
**1、grunt-contrib-htmlmin**  
用途不说了，从字面意思上看，连傻瓜也看得懂，言外之意：看懂了你就是傻瓜-- 
好吧，压缩html代码用的  
{% highlight javascript %}
htmlmin: {
    dist: {
        options: {
            removeComments: true, // 去注析
            collapseWhitespace: true //去换行
        },
        files: {
            'dist/html/index.html': ['src/html/index.html']
        }
    }
}
{% endhighlight %}

**2、grunt-contrib-cssmin**  
压缩css代码用的  
{% highlight javascript %}
cssmin: {
    with_banner: {
        options: {
            banner: '/* 注释，想写啥就写啥 */'
        },
        files: {
            'dist/css/combo.css': ['src/css/base.css','src/css/index.css']
        }
    }
}
{% endhighlight %}

**3、grunt-contrib-uglify**  
压缩js代码用的 ，还有合并
{% highlight javascript %}
uglify: { // jsmin
    options: {
        mangle: false
    },
    build: {
        files: {
            'dist/js/comm.js': ['src/js/comm.js']
        }
    }
}
{% endhighlight %}

**4、grunt-contrib-imagemin**  
压缩图片代码用的 
{% highlight javascript %}
imagemin: {
    dist: {
        options: {
            optimizationLevel: 3
        },
        files: {
            'dist/images/photo.png': 'src/images/photo.png',
            'dist/images/badge.jpg': 'src/images/badge.jpg'   //这个没成功，有时间再看看
        }
    }
}
{% endhighlight %}

**5、grunt-contrib-less**  
玩less的童鞋，你懂得，不解释 
{% highlight javascript %}
less: {
    dist: {
        files: {
            'assets/css/main-min.css': ['assets/css/base.less', 'assets/css/main.less']
        },
        options: {
            yuicompress: true   //无效，没压缩，有时间再看看 -- 
        }
    }
}
{% endhighlight %}

**6、grunt-contrib-watch**  
好东西，对文件实时监控，可见即可得的编译方式，很不错，有点像NodeJS里面的supervisor  
{% highlight javascript %}
watch: {
    scripts: {
        files: ['assets/css/*.less'],
        tasks: ['less']
    }
}
{% endhighlight %}

**7、grunt-contrib-connect**  
为文件建立站点，实现通过浏览器访问文件的功能
{% highlight javascript %}
  connect: {
      options: {
        port: 9000,
        hostname: '*', //默认就是这个值，可配置为本机某个 IP，localhost 或域名
        livereload: 35729 //声明给 watch 监听的端口
      },
      server: {
        options: {
          open: true,
          base: [
                ''  //主目录 这里到底从哪里开始的，试了很久网页没法显示，为此已吐一斤血...
          ]
        }
      }
    }
{% endhighlight %}

...

后续还有很多grunt好用的插件等待收集，不过让我一个个验明正身才好啊！