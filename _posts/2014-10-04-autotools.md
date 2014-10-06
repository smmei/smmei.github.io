---
layout: post
title: "autotools使用"
category: tools
---

日常编译的时候经常用到 ./configure, make, make install 这样的编译安装步骤。交叉编译时就在执行 ./configure 命令时指定编译器和目标板平台。一直使用这样的方式，都习以为常了。 今天在交叉编译zmq时碰到了一点问题，czmq在./configure时不管怎么指定地址都找不到-lzmq的位置，最后还是强制修改了./configure才通过并生成了Makefile，最后完成编译安装操作。于是趁着这个契机好好学习一下autotools这一套东西。

翻阅[GNU的automake文档](http://www.gnu.org/software/automake/manual/html_node/index.html#Top)时，其中推荐了快速上手的[tutorial](https://www.lrde.epita.fr/~adl/autotools.html)。后面的这个tutorial文档比较旧，其中有些描述已经过时了。

## 为什么要开发autotools这一套工具？

不同平台提供的库函数名、库以及头文件都可能有差异，在代码中如何处理这些问题呢？有几种办法：

* 代码中使用`#if #els, #endif`预处理
* 使用替代的宏和函数

预定义会使代码变得碎片化，不好管理。手动添加#define，以及在Makefile里添加-D,-I和-l编译选项，这两个动作都很麻烦。

为了可移植性和统一的编译方式，人们开始写一些shell脚本来猜测软件包的配置方式。这之后，configure脚本就成为了GNU发布的软件包里的标配文件。

## 编译操作

configure脚本会检查软件对系统函数、库和工具和依赖。GNU发布的软件都会有一致的编译方式：

```bash
./configure
make
make install
```

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

./configure会自动检查系统设置，我们也可以在其执行时指定变量，`./configure --help`列出所有可配置的参数。其中通用的变量有：

* CC - C编译器
* CFLAGS - C编译器执行时带的参数
* CXX - C++编译器
* CXXFLAGS - C++编译器执行时带的参数
* LDFLAGS - 链接参数 
* CPPFLAGS - C/C++预编译参数

例如可以这样指定CC，CPPFLAGS，LDFLAGS变量：

```bash
./configure --prefix=/usr/local CC=gcc-3 \
		     CPPFLAGS=-I$HOME/usr/include \
		     LDFLAGS=-L$HOME/usr/lib
```

我们也可以在另一个路径执行`./configure`，这样Makefile就生成在这个路径下。例如，在源码目录下创建一个build目录来做编译动作：

```bash
	mkdir build
	cd build
	../configure
	make
```
这样编译时生成的中间文件都位于build目录下了。

## 交叉编译

执行 ./configure 时可以指定下面三个参数，那么在编译时就可以根据 build 和 host 是否相同来判断当前是否在交叉编译：

* --build - 软件编译的平台，一般我们在PC上编译，这里就是i386-linux/i686-linux/x86_64之类的
* --host - 软件将要运行的平台，比如--host=arm-linux表示软件将运行于arm上
* --target - 在构建编译器工具链时指定这个参数，表示工具链生成结果所运行的平台。

## 使用 automake, autoconf 工具

[这里有个tutorial](http://mij.oltrelinux.com/devel/autoconf-automake/)，里面说明了如何使用automake, autoconf工具来为一个工程生成Makefile文件。

## autogen.sh 是什么？

当我们编辑了configure.ac文件后，必须记得要重新运行`aclocal`命令。然后再使用`autoconf`生成新的configure脚本，`autoheader`生成新的config.h，`automake`生成新的Makefine.in。

这些操作实现烦琐，不如写一个脚本来完成这些事情。于是把自动执行 aclocal, automake, autoconf 的命令都放在autoge.sh脚本里。

## 参考

* [autobook](https://www.sourceware.org/autobook/autobook/autobook_toc.html#SEC_Contents) - 介绍使用autoconf, automake, libtool的书
* [autogen.sh](https://www.sourceware.org/autobook/autobook/autobook_43.html)

