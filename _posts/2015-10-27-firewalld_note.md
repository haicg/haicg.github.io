---
layout: post
title: Firewalld 配置记录
description: 主要记录在配置firewall过程遇到的问题及学习到的知识。
keywords: Ubuntu
---
### 安装
测试环境使用的是Ubuntu14.04，直接使用<pre>sudo apt-get install firewalld</pre>

### 配置的途径
1. 直接使用界面配置，在命令行下输入<pre>firewall-config</pre>,然后就会出现一个配置的对话框，对话框的配置，可以参看"4.5. USING FIREWALLS"[1]。
2. 使用命令行配置 <pre>firewall-cmd</pre>,这个命令常用的一些参数都可以在文献[1]中找到，也有一些中文文献，如"FirewallD/zh-cn"[2]。下面有一个样例来介绍几个最基本的用法。
3. 直接修改相对应的配置文件，来实现相关的过滤规则。

### 样例
目标：指定的IP访问本机的SSH服务，其他主机不可以访问本机的任何资源。<br/>
第一步：将默认zone 设定成dmz,<pre>firewall-cmd --set-default-zone=dmz</pre> 这一步可有可无，但是决定了下面你要对哪一个zone做出修改。</br>
第二步:修改dmz下面的规则，这边通过修改/etc/firewall/zones/下面对应的文件来实现，如果没有则创建。此处需要创建一个dmz.xml,具体内容如下：
<br/>
```xml
<?xml version="1.0" encoding="utf-8"?>
<zone>
  <short>DMZ</short>
  <description>For computers in your demilitarized zone that are publicly-accessible with limited access to your internal network. Only selected incoming connections are accepted.</description>
  <!-- 下面是第一个规则就是让指定的IP可以访问本地的SSH服务  -->
  <rule family="ipv4">
    <service name="ssh"/>
    <source address="192.168.1.107"/>  <!-- 这边可以用掩码来指定一个局域网 -->
    <accept/>
  </rule>

<!-- 下面一个规则是对外关闭所有的服务，这边没有考虑IPV6 -->
  <rule family="ipv4">
    <drop/>
  </rule>
</zone>
```
第三步:将修改生效，firewall-cmd --complete-reload 。
###注意
1. firewall和ufw或者iptable不能同时运行，同时运行不能起到过滤的效果。<br/>
2. 防火墙的过滤规则是自上而下的，如果上面满足这个条件，则处理，没有找到对应的规则，继续向下比对规则。<br/>

[1]: https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Security_Guide/sec-Using_Firewalls.html  "Redhat 官方手册 4.5. USING FIREWALLS"
[2]: https://fedoraproject.org/wiki/FirewallD/zh-cn "FirewallD/zh-cn"
