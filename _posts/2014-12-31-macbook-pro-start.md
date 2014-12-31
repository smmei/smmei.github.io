
今天是2014年的最后一天。公司的人走的特别早。正好留我一人在公司，趁这点安静
的时间好好把电脑整一整。

MacBook Pro 到手两周多，现在使用都基本顺手了。现在还有唯一让我小不爽的就是
默认使用的文件系统是不区分大小写的。这在编译大型工程的时候很可能会碰到问题。
像目前我编译OpenWrt的时候就要手动创建一个20G的磁盘映像文件，然后挂载上来。
再到其中进行各种操作。虽然可以使用，但一想到这点还是非常不爽。

比如说万一要编译几个工程的话，那之前创建的磁盘映像文件很可能就不够用。

万一以后还要编译类似Android工程，又得创建一个磁盘映像文件。

目前的文件系统里我在终端里使用TAB键来自动补全时，由于文件系统大小写不敏感，
导致很容易补全为错误的目录。

于是纠结之后，我决定完全重装系统，并默认使用区分大小写的文件系统。

---

### 制作U盘安装盘

最近刚入手一个32G 的U盘，正好派上用场。

* 用磁盘工具格式化为“Mac OS扩展（日志）”格式的分区。
* 下载最新的安装包 "Install OS X Yosemite.app", 复制出 `/Applications/Install\
OS\ X\ Yosemite.app/Contents/Resources/createinstallmedia` 文件
* 使用该命令制作U盘

```
% sudo ./createinstallmedia --volume /Volumes/SAM\'S\ USB --applicationpath
% /Applications/Install\ OS\ X\ Yosemite.app --nointeration
Ready to start.
To continue we need to erase the disk at /Volumes/SAM'S USB.
If you wish to continue type (Y) then press return: Y
Erasing Disk: 0%... 10%... 20%... 30%...100%...
Copying installer files to disk...
Copy complete.
Making disk bootable...
Copying boot files...
Copy complete.
Done.
```


