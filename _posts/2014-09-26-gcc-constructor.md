---
layout: post
title: "gcc attribute constructor&destructor"
category: tools
---

在看openwrt里libnl-tiny这个库的时候，遇到了C里面的构造函数这个概念。

	static void __init init_msg_size(void)
	{
		default_msg_size = getpagesize();
	}

这个static函数没有显示被其它地方调用，但确用了__init修饰。__init定义在include/netlink-local.h中：

	#define __init __attribute__ ((constructor))
	#define __exit __attribute__ ((destructor))

写一个测试函数

	#include <stdio.h>
	
	#define __init __attribute__ ((constructor))
	#define __exit __attribute__ ((destructor))
	
	static __init void before(void)
	{
		printf("before\n");
	}
	
	static __exit void after(void)
	{
		printf("after\n");
	}
	
	int main(void)
	{
		printf("main\n");
		return 0;
	}

打印结果为：

	$ ./hello 
	before
	main
	after

还可以定义优先级：

	#include <stdio.h>
	
	#define __init101 __attribute__ ((constructor(101)))
	#define __init102 __attribute__ ((constructor(102)))
	#define __exit101 __attribute__ ((destructor(101)))
	#define __exit102 __attribute__ ((destructor(102)))
	
	static __init101 void before101(void)
	{
		printf("%s\n", __func__);
	}
	
	static __init102 void before102(void)
	{
		printf("%s\n", __func__);
	}
	
	static __exit101 void after101(void)
	{
		printf("%s\n", __func__);
	}
	
	static __exit102 void after102(void)
	{
		printf("%s\n", __func__);
	}
	
	int main(void)
	{
		printf("%s\n", __func__);
		return 0;
	}

打印结果为：

	$ ./hello 
	before101
	before102
	main
	after102
	after101

优先级0~100被保留，自定义的优先级从101开始。
