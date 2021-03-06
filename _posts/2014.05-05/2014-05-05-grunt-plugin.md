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
	release : {
	   files: {
			'jsall.min.js': ['../theone10/zepto-m.js', '../theone10/ratchet.js','../theone10/swipe.js','../theone10/URLConfig.js']
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

**8、grunt-contrib-watch + grunt-contrib-connect + grunt-livereload**  
优点：自动搭建静态文件服务器，不需在自己电脑上搭建Web服务器。  
不需要浏览器插件的支持（不现定于某个浏览器）。  
不需要给网页手动添加livereload.js。  
缺点：对于刚接触的人，配置略显复杂。  

1. 安装我们所需要的3个插件：grunt-contrib-watch、grunt-contrib-connect、connect-livereload  
执行命令：npm install --save-dev grunt-contrib-watch grunt-contrib-connect connect-livereload   
2. 修改Gruntfile.js文件：
{% highlight javascript %}
module.exports = function(grunt) {
    // LiveReload的默认端口号，你也可以改成你想要的端口号
    var lrPort = 35729;
    // 使用connect-livereload模块，生成一个与LiveReload脚本
    // <script src="http://127.0.0.1:35729/livereload.js?snipver=1" type="text/javascript"></script>
    var lrSnippet = require('connect-livereload')({ port: lrPort });
    // 使用 middleware(中间件)，就必须关闭 LiveReload 的浏览器插件
    var lrMiddleware = function(connect, options) {
        return [
            // 把脚本，注入到静态文件中
            lrSnippet,
            // 静态文件服务器的路径
            connect.static(options.base),
            // 启用目录浏览(相当于IIS中的目录浏览)
            connect.directory(options.base)
        ];
    };
    // 项目配置(任务配置)
    grunt.initConfig({
        // 读取我们的项目配置并存储到pkg属性中
        pkg: grunt.file.readJSON('package.json'),
        // 通过connect任务，创建一个静态服务器
        connect: {
            options: {
                // 服务器端口号
                port: 8000,
                // 服务器地址(可以使用主机名localhost，也能使用IP)
                hostname: 'localhost',
                // 物理路径(默认为. 即根目录)
                base: '.'
            },
            livereload: {
                options: {
                    // 通过LiveReload脚本，让页面重新加载。
                    middleware: lrMiddleware
                }
            }
        },
        // 通过watch任务，来监听文件是否有更改
        watch: {
            client: {
                // 我们不需要配置额外的任务，watch任务已经内建LiveReload浏览器刷新的代码片段。
                options: {
                    livereload: lrPort
                },
                // '**' 表示包含所有的子目录
                // '*' 表示包含所有的文件
                files: ['*.html', 'css/*', 'js/*', 'images/**/*']
            }
        }
    }); // grunt.initConfig配置完毕
    // 加载插件
    grunt.loadNpmTasks('grunt-contrib-connect');
    grunt.loadNpmTasks('grunt-contrib-watch');
    // 自定义任务
    grunt.registerTask('live', ['connect', 'watch']);
};
{% endhighlight %}

**9、grunt-contrib-concat**  
用于合并任意文件  
{% highlight javascript %}
concat : {
  options : {
    //文件内容的分隔符
    separator : ";",
    stripBanners : true,
    banner: '/*! <%= pkg.name %> - v<%= pkg.version %> - ' +
                    '<%= grunt.template.today("yyyy-mm-dd") %> */'
  },
  dist : {
    src : ['src/*.js'],
    dest : 'dest/build.built.js'
  }
}
{% endhighlight %}

**10、grunt-contrib-copy**  
上面代码将src子目录（只包含它下面的第一层文件和子目录），拷贝到dest子目录下面（即dest/src目录）。如果要更准确控制拷贝行为，比如只拷贝文件、不拷贝目录、不保持目录结构  
{% highlight javascript %}
copy: {
  main: {
    expand: true,
    cwd: 'src/',
    src: '**',
    dest: 'dest/',
    flatten: true,
    filter: 'isFile'
  }
}
{% endhighlight %}

**11、grunt-contrib-clean**  
删除目录及文件
{% highlight javascript %}
clean: {
  build: {
    src: ["dest/"]
  }
}
{% endhighlight %}

**12、grunt-contrib-compress**  
用于压缩文件和目录成为zip包
{% highlight javascript %}
compress : {
     main: {
        options: {
          archive: 'archive.zip'
        },
        files: [
          {src: ['src/*'], dest: 'internal_folder/',filter : 'isFile'}, //src下所有的js
          {src: ['src/**'], dest: 'internal_folder2/'}  // src下的所有目录和文件
        ]
      }
}
{% endhighlight %}

**13、grunt-contrib-jshint**  
jshint用于javascript代码检查（并会给出建议），发布js代码前执行jshint任务，可以避免出现一些低级语法问题。  
jshint拥有非常丰富的配置，可以自由控制检验的级别。
{% highlight javascript %}
jshint: {
    options: {
        //大括号包裹
        curly: true,
        //对于简单类型，使用===和!==，而不是==和!=
        eqeqeq: true,
        //对于首字母大写的函数（声明的类），强制使用new
        newcap: true,
        //禁用arguments.caller和arguments.callee
        noarg: true,
        //对于属性使用aaa.bbb而不是aaa['bbb']
        sub: true,
        //查找所有未定义变量
        undef: true,
        //查找类似与if(a = 0)这样的代码
        boss: true,
        //指定运行环境为node.js
        node: true
    },
    //具体任务配置
    files: {
        src: ['src/*.js']
    }
}
{% endhighlight %}

...

后续还有很多grunt好用的插件等待收集，不过让我一个个验明正身才好啊！