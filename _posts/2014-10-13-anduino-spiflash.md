---
layout: post
title: "Arduino 烧写SPI FLASH"
categories: tools
---

看到这篇文章：[LINUX 下离线烧写 SPI 闪存](http://blog.dword1511.info/?p=4107)。里面提到在Linux下使用Arduino来烧写SPI Flash，这正是我想要的。顺着这篇文章查阅里面的相关工具的资料，然后立刻上淘宝买了几片SPI Flash和座子，明后天东西一到就可以实验了。在这里先记录一下这套工具的使用。

## flashrom

[flashrom](http://flashrom.org/Flashrom) 是一个检测、读、写、校验和擦除flash芯片的工具。最初开发用于烧写主板、网卡、显卡、存储器上的 BIOS/EFI/coreboot/firmware/optionROM 等固件。

如今的主板把BIOS都存储在可编程的FLASH芯片里，这些FLASH芯片多种多样，比如，有不同的：存储空间大小、速率、通迅接口、封装等。

封装是跟烧录程序最没关系的，这里列一些常见封装：[图片见这里](http://flashrom.org/Technology)

* DIP32: Dual In-line Package, 32 pins
* PLCC32: Plastic Leaded Chip Carrier, 32 pins
* DIP8: Dual In-line Package, 8 pins
* SO8/SOIC8: Small-Outline Integrated Circuit, 8 pins
* TSOP: Thin Small-Outline Package, 32, 40, or 48 pins
* BGA: Ball Grid Array

通迅总线有：

* Parallel 并行总线
* LPC: Low Pin Count
* FWH: Firmware Hub，是一种特殊的LPC总线
* SPI: Serial Peripheral Interface 串行接口

这里我要烧写的是路由器上的SPI Flash，属于SO8/SOIC8封装，SPI接口。

### flashrom 安装

```sh
$ svn co svn://flashrom.org/flashrom/trunk flashrom
$ cd flashrom
$ make
$ sudo make install
```

### flashrom 使用

flashrom里支持一个叫 [Serprog](http://www.flashrom.org/Serprog) 的协议。在该页面的介绍里最后一个"serprog-stm32vcp by Chi Zhang"就是最上面提到的那篇文章的作者dword。

Serprog 是一个通过串口操作 FLASH 的协议。对于我的情况来说，流程是这样的：

* flashrom 就是在PC端向串口写命令或数据，PC的串口与 Arduino 的串口相连接
* Arduino 接收来自串口的数据，解析它，然后通过SPI接口输出。Arduino 的SPI接口连接到FLASH的SPI
* 这样 flashrom 就可以在PC端控制向串口写数据，进而控制Arduino的SPI输出，最终实现对SPI FLASH进行读、写、擦除操作

```sh
./flashrom -p serprog:dev=/dev/ttyACM0:2000000
```

## Arduino固件

Arduino 固件的描述以及连线方法详见[这里](http://www.flashrom.org/Serprog/Arduino_flasher)

下载并编译固件：

```sh
sudo apt-get install gcc-avr
sudo apt-get install avr-libc
sudo apt-get install avrdude

git clone git://gitorious.org/gnutoo-personal-arduino-projects/serprog-duino.git
cd serprog-duino && make && make upload
```

前三句是下载一些avr的编译器和标准库，以及一个avr烧录工具[AVRDUDE - AVR Downloader/UploaDEr](http://www.nongnu.org/avrdude/)。Arduino连接上PC之后，`make upload`将编译好的固件烧到avr单片机里。


## avr 开发环境

经过这么一番查找资料，发现开发avr的单片机程序真方便。gcc-avr 里包含了一整套编译avr程序的编译链工具，avr-libc安装之后，在 /usr/lib/avr/ 目录下又有一系列的头文件和库可以使用。如果熟悉了这一套东西之后，再完整读一遍AVR单片机的数据手册，写一个单片机程序估计也就是分分钟的事情。自从大学毕业后就再也没写过单片机程序了，以前写过51、义隆、Heltek单片机，但回想一下那套工具真是各种不顺手。今天发现了avr的这一套工具之后，竟然让我有了重新好好学习一下Arduino的冲动。

加油吧。


