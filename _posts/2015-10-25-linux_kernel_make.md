---
layout: post
title: Linux kernel Make
description: 记录我在编译kernel 中遇到的问题及相关的解决方法。
keywords: kernel,compile
---

*   使用make gconf 时遇到编译问题，undefined reference to symbol 'dlsym@@GLIBC_2.2.5' <br>
根据参考链接[1]的提示，可能是Makefile中缺少ldl链接库。<br/>
用Vim打开对应的Makefile， vim scripts/kconfig/Makefile <br/>
<pre>

HOSTLOADLIBES_qconf	= -L$(QTLIBPATH) -Wl,-rpath,$(QTLIBPATH) -l$(QTLIB) -ldl -lm
HOSTCXXFLAGS_qconf.o	= -I$(QTDIR)/include -D LKC_DIRECT_LINK

HOSTLOADLIBES_gconf	= `pkg-config gtk+-2.0 gmodule-2.0 libglade-2.0 --libs`
HOSTCFLAGS_gconf.o	= `pkg-config gtk+-2.0 gmodule-2.0 libglade-2.0 --cflags` \
                          -D LKC_DIRECT_LINK

</pre> <br/>
因为使用的是gconf ，那么对应的依赖库是HOSTLOADLIBES_gconf	= `pkg-config gtk+-2.0 gmodule-2.0 libglade-2.0 --libs`,明显可以看出这个依赖库中没有添加-ldl,这个问题也就解决了。<br/>
下面就继续了解一下ldl这个库的作用，根据参考链接[2],[3],可知这个库主要用于显示的调用动态链接库，使用显式调用的好处就是不需要在编译阶段去指定动态链接库，可以直接在运行时确定。<br/>

[1]: http://askubuntu.com/questions/527665/undefined-reference-to-symbol-expglibc-2-2-5        "undefined reference to symbol 'exp@@GLIBC_2.2.5'"
[2]: http://linux.die.net/man/3/dlsym "dlsym(3) - Linux man page"
[3]: http://www.cnblogs.com/Xiao_bird/archive/2010/03/01/1675821.html "Linux下动态链接库的使用"

