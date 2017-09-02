---
layout: post
title:  如何将自己编写的HTTP模块编译进Nginx
date:   2017-09-02 23:30:00 +0800
categories: Nginx
tag: Nginx模块开发和架构解析
---

* content 
{:toc}



为了让HTTP模块正常工作，首先需要把它编译进nginx。

nginx提供的一种简单的编译方式是：首先把源代码文件全部放到一个目录下，同时在该目录中编写一个文件用于通知nginx如何编译本模块，这个文件名必须为config。



{% highlight sh %}

# 如果只想开发一个HTTP模块，那么config文件中需要定义以下三个变量：
# (1)ngx_addon_name:仅在configure执行时使用，一般设置为模块名称;
# (2)HTTP_MODULES:保存所有的HTTP模块名称,每个模块间由空格相连;
# (3)NGX_ADDON_SRCS:用于指定新模块的源代码，多个待编译的源代码之间可以用空格相连。注意，在设置这个变量时可以使用$ngx_addon_dir变量，它等价于configure执行时--add-module=PATH的PATH参数。

# 对于我们的mytest模块，可以这样编写config文件：

ngx_addon_name=ngx_http_mytest_module
HTTP_MODULES="$HTTP_MODULES ngx_http_mytest_module"
NGX_ADDON_SRCS="$NGX_ADDON_SRCS $ngx_addon_dir/ngx_http_mytest_module.c"

{% endhighlight sh %}



这样，只要在configure脚本执行时加入参数–add-module=PATH(PATH就是我们的源代码和config文件的保存目录)，就可以在执行正常编译安装流程时完成nginx编译工作。

{% highlight sh %}
# 编译过程
./configure --prefix=/usr/local/nginx --add-module=/data/study/nginx-1.0.15/mymodule/test

make

make install

nginx -c "/usr/local/nginx/conf/nginx.conf"
{% endhighlight sh %}
[ngx_http_mytest_module 源码点这里](https://github.com/quyancong/nginx/tree/master/mymodule/test)

上述方法是最方便的，但有时我们需要更灵活的方式，比如重新决定各个模块的顺序或者在编译源代码时加入一些独特的编译选项，我们可以在执行configure后，对生成的objs/ngx_modules.c和objs/Makefile文件直接进行修改。(此种方法需慎用，不正确的修改可能导致nginx运作不正常，此处不详述)    


  
参考资料：
[http://blog.csdn.net/Leeds1993/article/details/52517935](http://blog.csdn.net/Leeds1993/article/details/52517935)