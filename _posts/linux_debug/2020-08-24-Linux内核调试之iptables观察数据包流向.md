---
layout:     post
title:      Linux内核调试之iptables观察数据包流向
subtitle:    Linux内核调试之iptables观察数据包流向
date:       2020-08-24
author:     Albert
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
   - Blog
   - linux
   - debug
---

# Linux内核调试之iptables观察数据包流向

​	众所周知，iptables的每个规则都会记录经过该规则的数据包的个数，如下图所示：

![](https://github.com/cclinuxer/cclinuxer.github.io/blob/master/img/image-20200823202154612.png?raw=true)

​	那么我们基于此原则，可以对我们想要观察的数据包创建一条规则，规则的执行是-j ACCEPT. 那么该条规则就会对我们想要观察的数据包进行匹配，我们可以在netfilter框架中的任意怀疑丢包的地方创建规则，**如果数据包经过了该规则，那么前面的数据包个数就会递增，说明这条规则之前没有将该数据包丢弃**。这种方法在缩小丢包排查范围来说，十分有效。

iptables调试方法如下：

##### 1、对指定数据包先用iptable进行标记：

​	例如：下面表示对来自192.168.0.100 在prerouting链进行打标记操作,或者修改ip的tos：

> ​	iptables -t mangle -A PREROUTING -p icmp -s 192.168.0.100  -j  TOS   --set-tos 0x10
>
> ​	iptables -t mangle -A PREROUTING -p icmp -s 192.168.0.100  -j  MARK   --set-mark 100

##### 2、对在定链进行匹配，看数据包是否增加

> ​	iptables -t filter -I INPUT 2 -m mark --mark 100 -j ACCEPT

​	如果这条规则的数据包有所增加，那么数据包是过了该规则的，说明前面的规则没有丢包，那么可以继续加规则，逐步缩小排查范围。

**3、iptables -nxvL**

​	通过iptables -nxvL来查看我们指定的规则数据包是否有增加，如果有增加，说明数据包到了这个规则。

这种技巧在排查数据或者数据流向时候很重要，可以缩小问题范围，然后再到具体的函数，或者hook点中进一步排查问题

​	对于二层桥上的数据包，我们同样可以使用ebtables来创建规则，原理和iptables是一样的。