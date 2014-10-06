---
layout: post
title: "autotools使用"
category: [tools]
---

日常编译的时候经常用到 ./configure, make, make install 这样的编译安装步骤。交叉编译时就在./configure时指定编译器和目标板平台。一直使用这样的方式，都习以为常了。

今天在交叉编译zmq时碰到了一点问题，czmq在./configure时不管怎么指定地址都找不到-lzmq的位置，最后还是强制修改了./configure才通过并生成了Makefile，最后完成编译安装操作。

趁着这个契机好好学习一下autotools这一套东西。

翻阅[GNU的automake文档](http://www.gnu.org/software/automake/manual/html_node/index.html#Top)时，其中推荐了快速上手的[tutorial](https://www.lrde.epita.fr/~adl/autotools.html)

后面的这个tutorial文档比较旧了，其中有些描述已经过时了。

## 为什么要开发autotools这一套工具？

不同平台提供的库函数名、库以及头文件都可能有差异，在代码中如何处理这些问题呢？有几种办法：
* 代码中使用#if/#else/#endif预处理
* 使用替代的宏和函数

预定义会使代码变得碎片化，不好管理。
手动添加#define，以及在Makefile里添加-D,-I和-l编译选项，这两个动作都很麻烦。

为了可移植性和统一的编译方式，人们开始写一些shell脚本来猜测软件包的配置方式。这之后，configure脚本就成为了GNU发布的软件包里的标配文件了。

## 编译操作

configure脚本会检查软件对系统函数、库和工具和依赖。GNU发布的软件都会有一致的编译方式：

./configure
make
make install

标准的编译目标：
* make all - 只输入`make`命令时执行的就是all目标，编译所有的东西，包括程序、库、文档等。
* make install - 安装编译结果。
* make install-strip - `make install`之后再strip程序或库中的符号表
* make uninstall - 卸载，与`make install`相反
* make clean - 清除编译生成的中间件和结果，与`make all`相反
* make distclean - `make clean`基础上再删除所有./configure生成的文件
* make check - 执行软件包自带的test测试，没有的话就算了。
* make installcheck - 检查已安装的程序或库
* make dist - 打包软件，创建PACKEAGE-VERSION.tar.gz

./configure会自动检查系统设置，我们也可以在其执行时指定变量，使用`./configure --help`可以列出所有可指定的参数，其中常见的变量有：
* CC - C编译器
* CFLAGS - C编译器执行时带的参数
* CXX - C++编译器
* CXXFLAGS - C++编译器执行时带的参数
* LDFLAGS - 链接参数 
* CPPFLAGS - C/C++预编译参数

例如：
	./configure --prefix=/usr/local CC=gcc-3 \
			     CPPFLAGS=-I$HOME/usr/include \
			     LDFLAGS=-L$HOME/usr/lib

我们也可以在另一个目录执行`./configure`，Makefile生成在这个执行的目录。例如，在源码目录下创建一个build目录来做编译动作：

	mkdir build
	cd build
	../configure
	make

这样编译时生成的中间文件都位于build目录下了。

## 交叉编译

./configure`命令时指定下面三个参数：
	* --build - 软件编译的平台，一般我们在PC上编译，这里就是i386-linux/i686-linux/x86_64之类的
	* --host - 软件将要运行的平台，比如--host=arm-linux表示软件将运行于arm上
	* --target - 在构建编译器工具链时指定这个参数，表示工具链生成结果所运行的平台。


## 编译一个程序

[这里有个tutorial](http://mij.oltrelinux.com/devel/autoconf-automake/)

* AM_INIT_AUTOMAKE([options])

其中的[options参数](http://www.gnu.org/software/automake/manual/automake.html#Options)传递给automake命令的参数，automake在解析Makefile.am时使用这些参数。

在旧版本中这个宏的参数格式是： `AM_INIT_AUTOMAKE(PACKAGE, VERSION, [NO-DEFINE])`。使用这种旧格式在automake时会有warnning，不建议使用。


## [autogen.sh](https://www.sourceware.org/autobook/autobook/autobook_43.html)

当我们编辑了configure.ac文件后，必须记得要重新运行`aclocal`命令。然后再使用`autoconf`生成新的configure脚本，`autoheader`生成新的config.h，`automake`生成新的Makefine.in。

这些操作实现烦琐，不如写一个脚本来完成这些事情。



## 参考
[autobook](https://www.sourceware.org/autobook/autobook/autobook_toc.html#SEC_Contents) - 介绍使用autoconf, automake, libtool的书
