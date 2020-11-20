---
layout:     post
title:     linux中定位数据包异常导致kernel panic
subtitle:  linux中定位数据包异常导致kernel panic
date:       2020-08-30
author:     Albert
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
   - Blog
   - Linux
   - debug
---

# Linux中定位数据包异常导致kernel panic

​	在网络问题的排查中，我们经常遇到，某个数据包导致异常发生，或者说这个导致异常的数据包是谁发出的，具体的报文格式是怎么样的，这里可以通过在内核中将导致异常的整个数据包dump出来，然后用wireshark再来具体分析是哪一个主机发出的数据包导致路由器异常。

######     1、在内核中dump出异常代码段的数据包

   

```
			printk("------fix--me--drop-package--in--dst_outputl\n");
			if (skb)
			{
			             char *buf = skb->data;
			             int len = skb->len;
			             int i;
			             printk("[%s:%d]Packet length = %#4x\n", __FUNCTION__, __LINE__, len);
			             for (i = 0; i < len; i++){
			                     if (i % 16 == 0) printk("%#4.4x", i);
			                     if (i % 2 == 0) printk(" ");
			                     printk("%2.2x", ((unsigned char *)buf)[i]);
			                    if (i % 16 == 15) printk("\n");
			             }
			             printk("\n\n\n\n");
			}
			dump_stack();
```

###### 2、tcpdump配合wireshark找出熟悉的数据包

​    同时用tcpdump抓取对应接口的数据包，通过在wireshark中搜索包含**异常数据包的16进制**的数据包。从而匹配出异常数据包。从而做进一步分析。

###### 3、定位异常，并打印调用栈进一步分析