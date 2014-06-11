---
layout: post
title: usbmon 翻译 
description: usbmon.txt的翻译
keywords: GitHub,jekyll
---

usbmon.txt 的英文原文链接：<a href="https://www.kernel.org/doc/Documentation/usb/usbmon.txt">https://www.kernel.org/doc/Documentation/usb/usbmon.txt </a>
<div style="text-indent: 2em">
<pre style="word-wrap: break-word; white-space: pre-wrap;">
原文：
* Introduction

The name "usbmon" in lowercase refers to a facility in kernel which is
used to collect traces of I/O on the USB bus. This function is analogous
to a packet socket used by network monitoring tools such as tcpdump(1)
or Ethereal. Similarly, it is expected that a tool such as usbdump or
USBMon (with uppercase letters) is used to examine raw traces produced
by usbmon.

The usbmon reports requests made by peripheral-specific drivers to Host
Controller Drivers (HCD). So, if HCD is buggy, the traces reported by
usbmon may not correspond to bus transactions precisely. This is the same
situation as with tcpdump.

Two APIs are currently implemented: "text" and "binary". The binary API
is available through a character device in /dev namespace and is an ABI.
The text API is deprecated since 2.6.35, but available for convenience.

翻译：
*介绍
    usbmon是linux内核中用来抓取USB总线上的输入或输出数据的工具。 这个功能类似于网络监视工具tcpdump或Ethereal抓取socket的封包。我们期望能有一个工具usbdump或USBMon 来检验通过usbmon抓取到的原始数据。
    usbmon 的请求是通过外围设备驱动向主USB控制器发送的。所以，如果主USB控制器比较繁忙的话，则通过usbmon抓取到的数据可能和总线处理的数据不完全一致。这种情况在tcpdump中也会出现。
    现在提供两种类型的输出接口，文本类型和二进制类型。二进制的API在/dev 下字符设备上是可以使用的。这也是一个ABI接口。文本输出API在2.6.35版本以后就不建议使用了，但是依然可以为你提供方便。

原文：
* How to use usbmon to collect raw text traces

Unlike the packet socket, usbmon has an interface which provides traces
in a text format. This is used for two purposes. First, it serves as a
common trace exchange format for tools while more sophisticated formats
are finalized. Second, humans can read it in case tools are not available.

翻译：
*如何使用usbmon抓取USB的原始数据
和socket的封包不同，usbmod提供了一个将原始数据以文本格式输出的接口。这样做有两个目的。首先它可以作为一个转换工具，将复杂的格式转换成简单的文本格式。
其次可以不借助任何工具就可以阅读。
</pre>
<pre>
原文：
To collect a raw text trace, execute following steps.
译文：
为了抓取usb的原始数据，你需要执行下面几个步骤

原文：
1. Prepare

Mount debugfs (it has to be enabled in your kernel configuration), and
load the usbmon module (if built as module). The second step is skipped
if usbmon is built into the kernel.
# mount -t debugfs none_debugs /sys/kernel/debug
# modprobe usbmon
#

译文：
1. 准备
挂载debugfs（这个必须要你的内核在配置的时候要启用这个功能），加载usbmon这个模块（如果是以模块形式编译的）。如果usbmon是直接编译在内核中的，则不需要执行第二步。
下面是对应的命令：
# mount -t debugfs none_debugs /sys/kernel/debug
# modprobe usbmon
#
————————————————————————————————
原文：
Verify that bus sockets are present.
# ls /sys/kernel/debug/usb/usbmon
0s  0u  1s  1t  1u  2s  2t  2u  3s  3t  3u  4s  4t  4u
#


Now you can choose to either use the socket '0u' (to capture packets on all
buses), and skip to step #3, or find the bus used by your device with step #2.
This allows to filter away annoying devices that talk continuously.

译文：
使用下面的命令，验证总线接口是否存在
# ls /sys/kernel/debug/usb/usbmon
0s  0u  1s  1t  1u  2s  2t  2u  3s  3t  3u  4s  4t  4u
#


2. Find which bus connects to the desired device

Run "cat /sys/kernel/debug/usb/devices", and find the T-line which corresponds
to the device. Usually you do it by looking for the vendor string. If you have
many similar devices, unplug one and compare the two
/sys/kernel/debug/usb/devices outputs. The T-line will have a bus number.
Example:

T:  Bus=03 Lev=01 Prnt=01 Port=00 Cnt=01 Dev#=  2 Spd=12  MxCh= 0
D:  Ver= 1.10 Cls=00(>ifc ) Sub=00 Prot=00 MxPS= 8 #Cfgs=  1
P:  Vendor=0557 ProdID=2004 Rev= 1.00
S:  Manufacturer=ATEN
S:  Product=UC100KM V2.00

"Bus=03" means it's bus 3. Alternatively, you can look at the output from
"lsusb" and get the bus number from the appropriate line. Example:

Bus 003 Device 002: ID 0557:2004 ATEN UC100KM V2.00

3. Start 'cat'

# cat /sys/kernel/debug/usb/usbmon/3u > /tmp/1.mon.out

to listen on a single bus, otherwise, to listen on all buses, type:

# cat /sys/kernel/debug/usb/usbmon/0u > /tmp/1.mon.out

This process will be reading until killed. Naturally, the output can be
redirected to a desirable location. This is preferred, because it is going
to be quite long.

4. Perform the desired operation on the USB bus

This is where you do something that creates the traffic: plug in a flash key,
copy files, control a webcam, etc.

