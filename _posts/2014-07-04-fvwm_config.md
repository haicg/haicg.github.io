---
layout: post
title: Fvwm 初探（安装）
description: 在Ubuntu 14.04上安装fvwm作为窗口管理器。
keywords: 
---
本文的测试平台是Ubuntu 14.04
<h3>第一步：安装fvwm</h3>
<!--http://pygments.org/docs/lexers/-->
{% highlight bash %}
   sudo apt-get install fvwm 
{% endhighlight %}
这一步完成后就可以先logout，然后重新登陆，在进入登陆状态的时候，会发现在Sess
ion中会多一个fvwm的选项（登陆用户名的右边的那个Ubuntu的图标就是选项入口），
然后选择这一项进入。此时你就可以看到一个光秃秃的界面，表明你的fvwm已经安装成
功。
<b>注意：</b>如果你是用源码安装的则需要在～/目录下新建一个叫xsession的文件。
<br>
具体的内容如下：
{% highlight bash %}
    #!/bin/sh
    # Various other commands here, ensuring that they're backgrounded where necessary
    # i.e.:
    # Start FVWM
    exec fvwm
{% endhighlight %}

<h3>第二步:安装fvwm-themes</h3>
如果你是fvwm新手的话建议你先用人家的主题，然后根据自己的需求再进行改进。我的
原则是先运行起来，后改进。Fvwm-themes的下载地址是：
<a href=" http://fvwm-themes.sourceforge.net/">http://fvwm-themes.
sourceforge.net</a>,这个是其主页。你可以根据需要下载源码还是二进制包。
要下载fvwm-themes和fvwm-themes-extra这两个包文件。
然后你选用对应安装命令安装这两个包就可以了，如果是deb的包你就可以用 dpkg
这个工具安装，如果是源码的话你就要配置、编译、安装。fvwm-themes-extra 的安装
方法在下载主页上有介绍，也可以直接下载二进制包，用包管理软件进行安装。

<!--http://box-look.org/content/show.php/Lethe?content=91022  一个主题的网址-->
<h3>第三步：运行fvwm-themes</h3>
安装完成之后就是要对fvwm-themes进行一个简单的配置。
{% highlight bash %}
    fvwm-themes-start
    cp ~/.fvwm
    cp themes-rc .fvwm2rc
{% endhighlight %}
到此就完成了fvwm基本配置。你也可以根据自己的需要去定制主题。同时这个主题包也内置了很多主题供选择，选择的方法：左键->Theme-management->themes.

<h3>可能遇到的问题：</h3>
1.Theme-management 这个选项没有出现
解决方法：到~/.fvwm/ 目录下去查看一下.fvwm2rc文件是否存在，然后用themes-rc 来替代，如果曾经做过修改定制，则需要先备份：
{% highlight bash %}
    cp .fvwm2rc fvwm2rc.tmp
    cp themes-rc .fvwm2rc
{% endhighlight %}

如果你遇到什么问题，可以发邮件和我联系。
联系方式见联系我的页面。

