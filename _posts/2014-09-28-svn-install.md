---
layout: post
title: "subverion安装"
category: tools
---

本来 `apt-get install subversion` 就搞定一切了。可是从http的url下checkout代码时，总是不成功。google了一下，说是要编译svn时打开http的支持。于是手动安装，记录一下步骤。

## 安装apr, apr-util

到[这个地址](http://archive.apache.org/dist/apr/)下载apr和apr-util， 先装apr, 再装apr-util:

```bash
./configure
make
sudo make install
```
默认安装到 /usr/local/apr 目录中

## 安装neon

[neon](http://www.webdav.org/neon/) 可以为svn提供http的服务，否则用svn从http的地址checkout源码时会发生错误：

```
svn: E170000: Unrecognized URL scheme for 'http:xxx'
```

下载代码后，默认开启ssl，默认使用openssl，编译安装：

```sh
./configure --with-ssl
make
sudo make install
```

## 安装subversion

从[github-subversion](https://github.com/apache/subversion)下载对应的release

```sh
./configure --with-apr=/usr/local/apr --with-apr-util=/usr/local/apr --with-neno=/usr/local
make
sudo make install
```
可能会指定要求某个版本的neon，配置的时候要注意看下

