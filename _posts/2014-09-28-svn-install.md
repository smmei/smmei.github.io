---
layout: post
title: "subverion安装"
category: tools
---

## 安装apr, apr-util

到[这个地址](http://archive.apache.org/dist/apr/)下载

先装apr, 再装apr-util

	./configure
	make
	sudo make install

默认安装到 /usr/local/apr 目录中

## 安装neon

[neon](http://www.webdav.org/neon/)可以为svn提供http的服务，否则用svn从http的地址checkout源码时会发生错误

	svn: E170000: Unrecognized URL scheme for 'http:xxx'

下载代码后，默认开启ssl，默认使用openssl，编译安装：

	./configure --with-ssl
	make
	sudo make install

## 安装subversion

从[github](https://github.com/apache/subversion)下载对应的release

	./configure --with-apr=/usr/local/apr --with-apr-util=/usr/local/apr --with-neno=/usr/local
	make
	sudo make install

可能会指定要求某个版本的neon，配置的时候要注意看下

