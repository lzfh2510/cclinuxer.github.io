---
layout:     post
title:      Linux内核调试之netfilter函数定位
subtitle:   寻找是哪个netfilter函数丢了网络数据包
date:       2020-08-24
author:     Albert
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
   - Blog
   - linux
   - debug
---

# Linux内核调试之`netfilter`函数定位

​		在分析网络协议栈的问题的时候，经常出现`netfilter`钩子函数丢包的情况，特别是我们自己在内核中注册了大量钩子函数的时候，然而钩子函数是通过遍历链表的方式执行，这个时候我们无法知道是哪一个具体函数丢弃了数据包，这个时候我们可以先打印出函数指针的地址，然后通过`addr2line`工具将该地址转换为字符串，然后通过`grep`工具找到该函数定义的地方，从而定位问题。

## 一、打印出函数地址

​         一般我们在遍历`netfilter`函数的地方加下打印，打印出函数地址：

```c
static unsigned int nf_iterate(struct list_head *head,
			       struct sk_buff **skb,
			       int hook,
			       const struct net_device *indev,
			       const struct net_device *outdev,
			       struct list_head **i,
			       int (*okfn)(struct sk_buff *),
			       int hook_thresh)
{
	/*
	 * The caller must not block between calls to this
	 * function because of risk of continuing from deleted element.
	 */
	list_for_each_continue_rcu(*i, head) {
		struct nf_hook_ops *elem = (struct nf_hook_ops *)*i;

		if (hook_thresh > elem->priority)
			continue;

		/* Optimization: we don't need to hold module
                   reference here, since function can't sleep. --RR */
		switch (elem->hook(hook, skb, indev, outdev, okfn)) {
		case NF_QUEUE:
			return NF_QUEUE;

		case NF_STOLEN:
			return NF_STOLEN;

		case NF_DROP:
		    printk("-----debug_nf------address=0x%p\n",(*elemp)->hook);
			return NF_DROP;

		case NF_REPEAT:
			*i = (*i)->prev;
			break;

#ifdef CONFIG_NETFILTER_DEBUG
		case NF_ACCEPT:
			break;

		default:
			NFDEBUG("Evil return from %p(%u).\n", 
				elem->hook, hook);
#endif
		}
	}
	return NF_ACCEPT;
}
```

`printk("-----debug_nf------address=0x%p\n",(*elemp)->hook);`

## 二、根据函数地址找到具体函数

### 2.1、addr2line方式

​	`/opt/toolchain-aarch64_cortex-a53_gcc-5.2.0_musl-1.1.16/bin/aarch64-openwrt-linux-addr2line 0xffffffc0005d5864 -f -e vmlinux.debug`
​	`-e`后面的参数`vmlinux.debug` 可以替换成其他带符号表的文件，如果文件较多，可以写一个shell脚本遍历出来，从而分析特定的钩子函数来判定为什么会导致丢包

### 2.2 、 获取内核符号地址或符号名

​	请参考我的另外一篇博客：[获取内核符号地址或符号名](https://cclinuxer.gitee.io/2020/09/%E8%8E%B7%E5%8F%96%E5%86%85%E6%A0%B8%E7%AC%A6%E5%8F%B7%E5%92%8C%E5%87%BD%E6%95%B0%E5%90%8D/)。