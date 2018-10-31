---
title: SIM 卡驻网 APDU 信息解读
layout: post
date: 2018/02/07 17:16:40
tags : eSIM
---
### 初识 APDU
查看当前 SIM 卡驻在哪个网络上的方式比较多，本篇文章以学习 APDU 的角度从底层来看待这个问题。其实并没有文档明确说明能通过 APDU 来查询驻网信息，主动发 AT 指令除外。但是一旦驻网成功，Modem 一定会通过 APDU 来把 mccmnc 写入到 SIM 卡文件系统中。

SIM 卡驻网信息主要在三个文件中体现，分别是：

* EF_LOCI（fid: 0x6F7E）
* EF_PSLOCI（fid: 0x6F73）
* EF_EPSLOCI（fid: 0x6FE3）

Modem 的驻网信息会通过 APDU 更新到 SIM 卡文件系统中，以 EF_EPSLOCI 为例，更新指令大概如下：

Command APDU:

00 A4 08 04 04 7F FF <font color=Orange>6F E3</font> 00

Response APDU:

62 17 82 02 41 21 83 02 6F E3 8A 01 05 8B 03 6F 06 01 80 02 00 12 88 01 F0 90 00

Command APDU:

00 D6 00 00 12 0B F6 64 F0 00 03 68 CA C7 E4 5C CB <font color=Red>64 F0 00</font> <font color=Blue>24 7C</font> <font color=Purple>00</font>

Response APDU:

90 00

上例中，橙色部分是更新文件 ID ，红色部分是 mccmnc (翻译：46000)，蓝色部分是小区信息，紫色部分代表状态码，00 代表 UPDATED，即驻网成功。

### 深入 APDU 结构与 SIM 卡文件系统
Modem 想要读一个文件或者写一个文件都要先选择这个文件，然后再执行动作。Command APDU 都是由 Modem 发起的，Response APDU 则是 SIM 卡处理完的结果。

Command APDU 由 cla, ins, p1, p2, lc, data, le(optional) 几个部分组成，le 可以为空。

例：

(00 : cla) (A4 : ins) (08 : p1) (04 : p2) (04 : lc) (7F FF 6F E3 : data) (00 : le)

其中 ins = A4 意思是 SELECT，ins = D6 是 UPDATE_BINARY。

其中 lc 代表数据长度，注意这里是16进制的。

再来看 Response APDU，Response APDU 由 data (optional) 和 sw 两部分组成，data 可以为空。

例：

(62 17 82 02 41 21 83 02 6F E3 8A 01 05 8B 03 6F 06 01 80 02 00 12 88 01 F0 : data) (90 00 : sw)

其中 sw = 90 00 代表一切正常。

那么我们回过头来看刚刚的例子：

这 4 条 APDU 的意思就是 Modem 先选择 EF_EPSLOCI (6FE3) 文件，SIM 卡返回 FCP 信息，Modem 再发指令更新数据，SIM 卡更新之后返回成功，数据的内容是：

0B F6 64 F0 00 03 68 CA C7 E4 5C CB 64 F0 00 24 7C 00。

其中 64 F0 00 就是 mccmnc，最后一位 00 就代表状态码。具体的文件规则可以从文档 [ts_131102](http://www.etsi.org/deliver/etsi_ts/131100_131199/131102/12.09.00_60/ts_131102v120900p.pdf) 中 EF_EPSLOCI 文件的结构表查到。

整条数据的意思就是驻网在 46000 成功。

### 举一反三
知道了上述信息，那我们就能从更新 LOCI 文件的 APDU 中获取到驻网信息。
<br/>
例子1：

Command APDU:

00 A4 08 04 04 7F FF 6F 7E 00

Response APDU:

62 17 82 02 41 21 83 02 6F 7E 8A 01 05 8B 03 6F 06 01 80 02 00 0B 88 01 58 90 00

Command APDU:

00 D6 00 00 0B FF FF FF FF 64 F0 00 FF FE 00 03

Response APDU: 90 00

翻译：更新 EF_LOCI 文件，尝试驻网在 46000 状态是 03 ，03 代表 Location Area not allowed， 即驻网失败。
<br/>
例子2：

Command APDU:

00 A4 08 04 04 7F FF 6F 7E 00

Response APDU:

62 17 82 02 41 21 83 02 6F 7E 8A 01 05 8B 03 6F 06 01 80 02 00 0B 88 01 58 90 00

Command APDU:

00 D6 00 00 0B 10 80 56 A7 64 F0 00 24 7C 00 00

Response APDU:

90 00

翻译：更新 EF_LOCI 文件，尝试驻网在 46000 状态是 00 ，00 代表 updated， 即驻网成功。
<br/>
例子3：

Command APDU:

00 A4 08 04 04 7F FF 6F 7E 00

Response APDU:

62 17 82 02 41 21 83 02 6F 7E 8A 01 05 8B 03 6F 06 01 80 02 00 0B 88 01 58 90 00

Command APDU:

00 D6 00 00 0B FF FF FF FF 64 F0 40 FF FE 00 01

Response APDU: 90 00

翻译：更新 EF_LOCI 文件，尝试驻网在 46004 状态是 01，01 代表 not updated， 即驻网失败。

<br/>
