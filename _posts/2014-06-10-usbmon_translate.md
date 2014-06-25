---
layout: post
title: usbmon.txt 翻译
description: usbmon.txt的翻译
keywords: usbmon
---

usbmon.txt 的英文原文链接：<a href="https://www.kernel.org/doc/Documentation/usb/usbmon.txt">
https://www.kernel.org/doc/Documentation/usb/usbmon.txt </a>
<!--<div style="text-indent: 2em">-->

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
    usbmon是linux内核中用来抓取USB总线上的输入或输出数据的工具。 这个功能类似于网络监视工具
	tcpdump或Ethereal抓取socket的封包。我们期望能有一个工具usbdump或USBMon 来检验通过usbmon
	抓取到的原始数据。usbmon 的请求是通过外围设备驱动向主USB控制器发送的。所以，如果主USB控
	制器比较繁忙的话,则通过usbmon抓取到的数据可能和总线处理的数据不完全一致。这种情况在
	tcpdump中也会出现。现在提供两种类型的输出接口，文本类型和二进制类型。二进制的API在/dev 下
	字符设备上是可以使用的。这也是一个ABI接口。文本输出API在2.6.35版本以后就不建议使用了，
	但是为了方便，依然可以使用。
</pre>
<pre style="word-wrap: break-word; white-space: pre-wrap;>
原文：
* How to use usbmon to collect raw text traces

Unlike the packet socket, usbmon has an interface which provides traces
in a text format. This is used for two purposes. First, it serves as a
common trace exchange format for tools while more sophisticated formats
are finalized. Second, humans can read it in case tools are not available.

翻译：
*如何使用usbmon抓取USB的原始数据
和socket的封包不同，usbmod提供了一个将原始数据以文本格式输出的接口。这样做有两个好处。首先它
可以作为一个转换工具，将复杂的格式转换成简单的文本格式。其次可以不借助任何工具就可以阅读。
</pre>
<pre style="word-wrap: break-word; white-space: pre-wrap;>
原文：
To collect a raw text trace, execute following steps.
译文：
为了抓取usb的原始数据，你需要执行下面几个步骤
</pre>
<pre style="word-wrap: break-word; white-space: pre-wrap;>
原文：
1. Prepare

Mount debugfs (it has to be enabled in your kernel configuration), and
load the usbmon module (if built as module). The second step is skipped
if usbmon is built into the kernel.
\# mount -t debugfs none_debugs /sys/kernel/debug
\# modprobe usbmon
\#

Verify that bus sockets are present.
\# ls /sys/kernel/debug/usb/usbmon
0s  0u  1s  1t  1u  2s  2t  2u  3s  3t  3u  4s  4t  4u
\#

Now you can choose to either use the socket '0u' (to capture packets on all
buses), and skip to step \#3, or find the bus used by your device with step \#2.
This allows to filter away annoying devices that talk continuously.

译文：
1. 准备
挂载debugfs（这个必须要你的内核在配置的时候要启用这个功能），加载usbmon这个模块
（如果是以模块形式编译的）。如果usbmon是直接编译在内核中的，则不需要执行第二步。
下面是对应的命令：
\# mount -t debugfs none_debugs /sys/kernel/debug
\# modprobe usbmon
\#

使用下面的命令，验证总线接口是否存在
\# ls /sys/kernel/debug/usb/usbmon
0s  0u  1s  1t  1u  2s  2t  2u  3s  3t  3u  4s  4t  4u
\#
现在如果你选择'0u'这个接口的话，就表示抓取所有usb总线上的数据，
这样就可以直接进行步骤三了，你也可以进行步骤二找到你的设备所在的总线。
这样就可以过滤掉那些令人烦恼不断通信的设备的干扰了。
</pre>

<pre style="word-wrap: break-word; white-space: pre-wrap;>
原文：
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

译文：
2.找到设备所在的总线
运行“cat /sys/kernel/debug/usb/devices” 找以T开头的行，下面是该总线上一个对应的设备的信息。
通常你可以通过匹配供应商（生产厂商）的名字来找到你的设备。
如果你有多个类似的设备，拔掉一个，然后比较/sys/kernel/debug/usb/devices的内容在拔
掉前后有什么变化。以T开头的那一行中有设备所在的总线号。
样例：
T:  Bus=03 Lev=01 Prnt=01 Port=00 Cnt=01 Dev#=  2 Spd=12  MxCh= 0
D:  Ver= 1.10 Cls=00(>ifc ) Sub=00 Prot=00 MxPS= 8 #Cfgs=  1
P:  Vendor=0557 ProdID=2004 Rev= 1.00
S:  Manufacturer=ATEN
S:  Product=UC100KM V2.00
BUS=03 表示这个是总线3.
或者你也可以通过使用“lsusb” 来获取设备对应的总线号。
样例：
Bus 003 Device 002: ID 0557:2004 ATEN UC100KM V2.00
</pre>
<pre style="word-wrap: break-word; white-space: pre-wrap;>
原文：
3. Start 'cat'

\# cat /sys/kernel/debug/usb/usbmon/3u > /tmp/1.mon.out
to listen on a single bus, otherwise, to listen on all buses, type:
\# cat /sys/kernel/debug/usb/usbmon/0u > /tmp/1.mon.out

This process will be reading until killed. Naturally, the output can be
redirected to a desirable location. This is preferred, because it is going
to be quite long.
译文：
3.开始抓取数据
如果单独抓取一条bus上的数据，用下面的命令（记住和step2 相对应）
\# cat /sys/kernel/debug/usb/usbmon/3u > /tmp/1.mon.out
要抓取所有总线上的数据，请使用下面的命令
\# cat /sys/kernel/debug/usb/usbmon/0u > /tmp/1.mon.out
这个进程将一直在读取数据，知道你将这个进程kill掉。同时，这个数据可以被重定向到你想输出
的位置（某个文件）。因为数据可能很多，推荐你的将输出重定向到一个文件。
</pre>
<pre style="word-wrap: break-word; white-space: pre-wrap;>
原文：
4. Perform the desired operation on the USB bus

This is where you do something that creates the traffic: plug in a flash key,copy files,
control a webcam, etc.
译文：
4. 执行你想要执行的USB操作
现在你可以做你要做的事情了，创建一个USB通信。
比如插入一个USB密匙，复制一个文件或者控制一个摄像头。
</pre>
<pre style="word-wrap: break-word; white-space: pre-wrap;>
原文：
5. Kill cat

Usually it's done with a keyboard interrupt (Control-C).

At this point the output file (/tmp/1.mon.out in this example) can
be saved,sent by e-mail, or inspected with a text editor. In the
last case make sure that the file size is not excessive for your
favourite editor.
译文：
停止抓包：
一般来说，通过（Control-C)组合键，就可以发送一个中断信号停止当前的抓包进程。
此时输出文件中已经保存了抓取到的内容，如本例中/tmp/1.mon.out.

</pre>
<pre style="word-wrap: break-word; white-space: pre-wrap;>
原文：
* Raw text data format

Two formats are supported currently: the original, or '1t' format, and
the '1u' format. The '1t' format is deprecated in kernel 2.6.21. The '1u'
format adds a few fields, such as ISO frame descriptors, interval, etc.
It produces slightly longer lines, but otherwise is a perfect superset
of '1t' format.

If it is desired to recognize one from the other in a program, look at the
"address" word (see below), where '1u' format adds a bus number. If 2 colons
are present, it's the '1t' format, otherwise '1u'.

Any text format data consists of a stream of events, such as URB submission,
URB callback, submission error. Every event is a text line, which consists
of whitespace separated words. The number or position of words may depend
on the event type, but there is a set of words, common for all types.

Here is the list of words, from left to right:

- URB Tag. This is used to identify URBs, and is normally an in-kernel address
  of the URB structure in hexadecimal, but can be a sequence number or any
  other unique string, within reason.

- Timestamp in microseconds, a decimal number. The timestamp's resolution
  depends on available clock, and so it can be much worse than a microsecond
  (if the implementation uses jiffies, for example).

- Event Type. This type refers to the format of the event, not URB type.
  Available types are: S - submission, C - callback, E - submission error.

- "Address" word (formerly a "pipe"). It consists of four fields, separated by
  colons: URB type and direction, Bus number, Device address, Endpoint number.
  Type and direction are encoded with two bytes in the following manner:
    Ci Co   Control input and output
    Zi Zo   Isochronous input and output
    Ii Io   Interrupt input and output
    Bi Bo   Bulk input and output
  Bus number, Device address, and Endpoint are decimal numbers, but they may
  have leading zeros, for the sake of human readers.

- URB Status word. This is either a letter, or several numbers separated
  by colons: URB status, interval, start frame, and error count. Unlike the
  "address" word, all fields save the status are optional. Interval is printed
  only for interrupt and isochronous URBs. Start frame is printed only for
  isochronous URBs. Error count is printed only for isochronous callback
  events.

  The status field is a decimal number, sometimes negative, which represents
  a "status" field of the URB. This field makes no sense for submissions, but
  is present anyway to help scripts with parsing. When an error occurs, the
  field contains the error code.

  In case of a submission of a Control packet, this field contains a Setup Tag
  instead of an group of numbers. It is easy to tell whether the Setup Tag is
  present because it is never a number. Thus if scripts find a set of numbers
  in this word, they proceed to read Data Length (except for isochronous URBs).
  If they find something else, like a letter, they read the setup packet before
  reading the Data Length or isochronous descriptors.

- Setup packet, if present, consists of 5 words: one of each for bmRequestType,
  bRequest, wValue, wIndex, wLength, as specified by the USB Specification 2.0.
  These words are safe to decode if Setup Tag was 's'. Otherwise, the setup
  packet was present, but not captured, and the fields contain filler.

- Number of isochronous frame descriptors and descriptors themselves.
  If an Isochronous transfer event has a set of descriptors, a total number
  of them in an URB is printed first, then a word per descriptor, up to a
  total of 5. The word consists of 3 colon-separated decimal numbers for
  status, offset, and length respectively. For submissions, initial length
  is reported. For callbacks, actual length is reported.

- Data Length. For submissions, this is the requested length. For callbacks,
  this is the actual length.

- Data tag. The usbmon may not always capture data, even if length is nonzero.
  The data words are present only if this tag is '='.

- Data words follow, in big endian hexadecimal format. Notice that they are
  not machine words, but really just a byte stream split into words to make
  it easier to read. Thus, the last word may contain from one to four bytes.
  The length of collected data is limited and can be less than the data length
  reported in the Data Length word. In the case of an Isochronous input (Zi)
  completion where the received data is sparse in the buffer, the length of
  the collected data can be greater than the Data Length value (because Data
  Length counts only the bytes that were received whereas the Data words
  contain the entire transfer buffer).

Examples:

An input control transfer to get a port status.

d5ea89a0 3575914555 S Ci:1:001:0 s a3 00 0000 0003 0004 4 <
d5ea89a0 3575914560 C Ci:1:001:0 0 4 = 01050000

An output bulk transfer to send a SCSI command 0x28 (READ_10) in a 31-byte
Bulk wrapper to a storage device at address 5:

dd65f0e8 4128379752 S Bo:1:005:2 -115 31 = 55534243 ad000000 00800000 80010a28 20000000 20000040 00000000 000000
dd65f0e8 4128379808 C Bo:1:005:2 0 31 >

翻译：
*原始文本数据格式

现在支持两种版本的数据格式。原始数据格式和‘1t’或‘1u’格式。‘1t’格式是在内核2.6.21中
使用的，而‘1u’格式增加了一些字段，比如ISO帧描述，时间间隔等等。除了每条记录变得略长
之外，可以说是对‘1t’格式的完美补充。
如果想要区分程序中使用的是'1u'还是‘1t’格式，可以根据“address“字段（见下文）。”lu“的
数据格式有bus总线的标号，如果这边显示的是两个冒号，则是'lu'格式。
任何文本格式的数据都是由一系列的事件构成的，比如 URB提交，URB回调，提交错误等。
每一个事件都是一条文本记录，通过空格来划分字段。字段数据的长度或者位置是根据事件类
型的不同而定义的。但是有一些字段适用于所有的类型的事件。

下面就是这些字段的列表，自左向右顺序排列：

- URB 标志. 这个是用来标志USB 请求块地址的，通常是一个十六进制的内核地址用来
表示URB结构体数据所在的地址，但是也可以是一个序列号者任何具有唯一性的字符串，只
要具有一定的合理性。

- 时间戳（微秒级的，以十进制表示）。这个时间戳的精度取决于可以使用的时钟。所以它可能
远远达不到微秒级的精度（例如：使用jiffes这个时间变量来实现）。

- 事件类型。
  这个类型涉及的是事件的格式而不是URB的类别。可用类型有下面三种：S-提交，C-回调，
  E-提交错误

- “地址“ （前身是”管道“）。它主要包含四个字段，通过冒号来分离：URB类型和方向，
总线号，设备地址，端点号。
类型和方向通过2个字节来编码表示，具体方式如下：
	Ci Co 控制类数据的输入和输出
	Zi Zo 同步数据的输入和输出
	li lo 中断输入和输出
	Bi Bo 批量（bulk）传输模式下的输入和输出
总线号，设备地址和端点号都是十进制的数据。为了方便阅读，他们前面可能有0进行位置
补全。

- URB状态字。这个可以是一个字母或者是以冒号分隔的几个数字：URB 状态，间隔，
起始帧和错误计数.
和”地址“字不同，所有用来保存状态的字段都是可选择的。时间间隔的输出只是针对于中断
和同步模式的URBs。只有同步模式的URBs才会打印出起始帧。错误计数只有在同步回调的时候
才会打印出来。
这个状态字段是一个十进制数，有时是负的，此时它代表的URB的“状态”字段。此字段对于提
交事件是没有意义的，但是至少可以帮助脚本来解析文件。当发生错误时，该字段会包含错误
代码。
在提交一个控制数据包的时候，这个字段包含一个 Setup 标签来替代组号。这是很容易辨别
Setup标签是否存在，因为它不是一个数字。这个字段，主要是用来处理读到的数据的长度
（除了同步模式的URBs）。如果他们在读取长度或同步描述符之前，发现别的东西，像一个字母,这个就是Setup包，

- Setup 包。如果存在这个字段的话，它是由五个字组成了的：每一条都是由
bmRequestType，bRequest，wValue，WINDEX，wLength 构成，这些在USB2.0规范中有说明。
如果Setup 标签设置为s,则这些字（段）都是可以安全解码的。否则setup
包虽然存在，但是没有捕捉，这个字段只是包含填充字符。

- 同步帧描述符和描述符本身的数量。如果一个同步传输事件有一组描述符，
描述符的总数先打印，然后是每个描述符一个字，最多5个字。该字由3个以冒号隔开的十进
制数构成，分别为状态，偏移量和长度。对于提交事件，报告初始长度。对于回调事件，
报实际长度。

- 数据长度。对于提交事件，这是请求的长度。对于回调事件，这是实际的长度。

- 数据标签。该usbmon可能并不总是捕获数据，即使长度为非零的时候。
只有当这个标签是'='的时候才表示抓取的数据是显示的。

- 抓取的数据,以大端机格式的十六进制表示。注意他们不是以字为单位的，只不过是一个字节流被以
字位分割以方便阅读。 因此，最后一个字可以包含一到四个字节。
由于收集的数据的长度是有限，所以得到的数据的长度可以小于 Data Length 字段中
报告的数据长度。在一个同步输入（Zi）完成后，而收到的字符有在缓冲器中滞留的情况下，所收集的数据的长度可以大于Data Length 字段中报告的数据长度（因为Data
Length 字段数据长度只计算收到的数据，而数据字的字节包含整个传输缓冲区）。

样例：

一个获得端口状态的输入控制传输记录
d5ea89a0 3575914555 S Ci:1:001:0 s a3 00 0000 0003 0004 4 <
d5ea89a0 3575914560 C Ci:1:001:0 0 4 = 01050000

通过批量传输31个字节来发送一个SCSI命令0x28给储存设备在5号地址
dd65f0e8 4128379752 S Bo:1:005:2 -115 31 = 55534243 ad000000 00800000
80010a28 20000000 20000040 00000000 000000
dd65f0e8 4128379808 C Bo:1:005:2 0 31 >

</pre>
<pre style="word-wrap: break-word; white-space: pre-wrap;>

* Raw binary format and API

The overall architecture of the API is about the same as the one above,
only the events are delivered in binary format. Each event is sent in
the following structure (its name is made up, so that we can refer to it):

struct usbmon_packet {
	u64 id;			/*  0: URB ID - from submission to callback */
	unsigned char type;	/*  8: Same as text; extensible. */
	unsigned char xfer_type; /*    ISO (0), Intr, Control, Bulk (3) */
	unsigned char epnum;	/*     Endpoint number and transfer direction */
	unsigned char devnum;	/*     Device address */
	u16 busnum;		/* 12: Bus number */
	char flag_setup;	/* 14: Same as text */
	char flag_data;		/* 15: Same as text; Binary zero is OK. */
	s64 ts_sec;		/* 16: gettimeofday */
	s32 ts_usec;		/* 24: gettimeofday */
	int status;		/* 28: */
	unsigned int length;	/* 32: Length of data (submitted or actual) */
	unsigned int len_cap;	/* 36: Delivered length */
	union {			/* 40: */
		unsigned char setup[SETUP_LEN];	/* Only for Control S-type */
		struct iso_rec {		/* Only for ISO */
			int error_count;
			int numdesc;
		} iso;
	} s;
	int interval;		/* 48: Only for Interrupt and ISO */
	int start_frame;	/* 52: For ISO */
	unsigned int xfer_flags; /* 56: copy of URB's transfer_flags */
	unsigned int ndesc;	/* 60: Actual number of ISO descriptors */
};				/* 64 total length */

These events can be received from a character device by reading with read(2),
with an ioctl(2), or by accessing the buffer with mmap. However, read(2)
only returns first 48 bytes for compatibility reasons.

The character device is usually called /dev/usbmonN, where N is the USB bus
number. Number zero (/dev/usbmon0) is special and means "all buses".
Note that specific naming policy is set by your Linux distribution.

If you create /dev/usbmon0 by hand, make sure that it is owned by root
and has mode 0600. Otherwise, unpriviledged users will be able to snoop
keyboard traffic.

The following ioctl calls are available, with MON_IOC_MAGIC 0x92:

 MON_IOCQ_URB_LEN, defined as _IO(MON_IOC_MAGIC, 1)

This call returns the length of data in the next event. Note that majority of
events contain no data, so if this call returns zero, it does not mean that
no events are available.

 MON_IOCG_STATS, defined as _IOR(MON_IOC_MAGIC, 3, struct mon_bin_stats)

The argument is a pointer to the following structure:

struct mon_bin_stats {
	u32 queued;
	u32 dropped;
};

The member "queued" refers to the number of events currently queued in the
buffer (and not to the number of events processed since the last reset).

The member "dropped" is the number of events lost since the last call
to MON_IOCG_STATS.

 MON_IOCT_RING_SIZE, defined as _IO(MON_IOC_MAGIC, 4)

This call sets the buffer size. The argument is the size in bytes.
The size may be rounded down to the next chunk (or page). If the requested
size is out of [unspecified] bounds for this kernel, the call fails with
-EINVAL.

 MON_IOCQ_RING_SIZE, defined as _IO(MON_IOC_MAGIC, 5)

This call returns the current size of the buffer in bytes.

 MON_IOCX_GET, defined as _IOW(MON_IOC_MAGIC, 6, struct mon_get_arg)
 MON_IOCX_GETX, defined as _IOW(MON_IOC_MAGIC, 10, struct mon_get_arg)

These calls wait for events to arrive if none were in the kernel buffer,
then return the first event. The argument is a pointer to the following
structure:

struct mon_get_arg {
	struct usbmon_packet *hdr;
	void *data;
	size_t alloc;		/* Length of data (can be zero) */
};

Before the call, hdr, data, and alloc should be filled. Upon return, the area
pointed by hdr contains the next event structure, and the data buffer contains
the data, if any. The event is removed from the kernel buffer.

The MON_IOCX_GET copies 48 bytes to hdr area, MON_IOCX_GETX copies 64 bytes.

 MON_IOCX_MFETCH, defined as _IOWR(MON_IOC_MAGIC, 7, struct mon_mfetch_arg)

This ioctl is primarily used when the application accesses the buffer
with mmap(2). Its argument is a pointer to the following structure:

struct mon_mfetch_arg {
	uint32_t *offvec;	/* Vector of events fetched */
	uint32_t nfetch;	/* Number of events to fetch (out: fetched) */
	uint32_t nflush;	/* Number of events to flush */
};

The ioctl operates in 3 stages.

First, it removes and discards up to nflush events from the kernel buffer.
The actual number of events discarded is returned in nflush.

Second, it waits for an event to be present in the buffer, unless the pseudo-
device is open with O_NONBLOCK.

Third, it extracts up to nfetch offsets into the mmap buffer, and stores
them into the offvec. The actual number of event offsets is stored into
the nfetch.

 MON_IOCH_MFLUSH, defined as _IO(MON_IOC_MAGIC, 8)

This call removes a number of events from the kernel buffer. Its argument
is the number of events to remove. If the buffer contains fewer events
than requested, all events present are removed, and no error is reported.
This works when no events are available too.

 FIONBIO

The ioctl FIONBIO may be implemented in the future, if there's a need.

In addition to ioctl(2) and read(2), the special file of binary API can
be polled with select(2) and poll(2). But lseek(2) does not work.

* Memory-mapped access of the kernel buffer for the binary API

The basic idea is simple:

To prepare, map the buffer by getting the current size, then using mmap(2).
Then, execute a loop similar to the one written in pseudo-code below:

   struct mon_mfetch_arg fetch;
   struct usbmon_packet *hdr;
   int nflush = 0;
   for (;;) {
      fetch.offvec = vec; // Has N 32-bit words
      fetch.nfetch = N;   // Or less than N
      fetch.nflush = nflush;
      ioctl(fd, MON_IOCX_MFETCH, &fetch);   // Process errors, too
      nflush = fetch.nfetch;       // This many packets to flush when done
      for (i = 0; i < nflush; i++) {
         hdr = (struct ubsmon_packet *) &mmap_area[vec[i]];
         if (hdr->type == '@')     // Filler packet
            continue;
         caddr_t data = &mmap_area[vec[i]] + 64;
         process_packet(hdr, data);
      }
   }

Thus, the main idea is to execute only one ioctl per N events.

Although the buffer is circular, the returned headers and data do not cross
the end of the buffer, so the above pseudo-code does not need any gathering.
</pre>
