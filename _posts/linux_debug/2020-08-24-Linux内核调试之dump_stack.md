---
layout:     post
title:      linux内核调试之dump_stack
subtitle:   linux内核调试之dump_stack
date:       2020-08-24
author:     Albert
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
   - Blog
   - linux
   - debug
---

# linux内核调试之dump_stack

​	在实际的环境中，系统异常的时候，我们有时候需要知道，是哪一个进程调用了系统调用，或者是谁下发了配置到内核，这个时候可以通过dump_stack来打印调用栈。

```c
   struct task_struct *my_parent = current->parent;
   if((dev!=NULL)&&(dev->name!=NULL)){
			printk(" ifconfig devname=%s current=%s pid=%d   parent_pid=%d\n",dev->name,
                current->comm, current->pid, my_parent->pid);
    }
    dump_stack();
	if (net_ratelimit()){
		char *mac_str=NULL;
		struct task_struct *my_parent = current->parent;
	printk("huangjie edit current=%s pid=%d	parent_pid=%d\n",current->comm, current->pid, my_parent->pid);
			dump_stack();
    }
```

​	这里只是我比较喜欢用的，用于查看是哪些应用在配置系统状态, 比如开关网络设备，配置网络参数等。当然还有其他的一些工具诸如ftrace、jprobe、ebpf后面我会开新的issue来详细介绍