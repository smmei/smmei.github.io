---
layout: post
title: "极1s拆机"
categories: openwrt
---
买了一个极1s。拆机之后，焊上UART，便可以看到它里面的信息了。

![极1s主板](/assets/hi5661.jpg)

## 硬件配置

* CPU: MT7620A

* memory : DDR2 128MB, [winbond w971gg6kb](https://www.winbond.com/NR/rdonlyres/1AE99C6F-CE4E-45A5-B74D-4B1FA6CAE69C/0/W971GG6KB.pdf)

* flash : spi flash 16MB, [winbond 25q128fvfg](http://www.winbond.com/NR/rdonlyres/A5B6C30B-C174-43CF-867E-E1A2F371A07C/0/W25Q128FV.pdf)

## 分区

```sh
root@Hiwifi:/# mount
rootfs on / type rootfs (rw)
/dev/mtdblock4 on /rom type squashfs (ro,relatime)
proc on /proc type proc (rw,noatime)
sysfs on /sys type sysfs (rw,noatime)
tmpfs on /tmp type tmpfs (rw,nosuid,nodev,noatime)
tmpfs on /var type tmpfs (rw,noatime)
tmpfs on /dev type tmpfs (rw,noatime,size=512k,mode=755)
devpts on /dev/pts type devpts (rw,noatime,mode=600)
/dev/mtdblock5 on /overlay type jffs2 (rw,noatime)
overlayfs:/overlay on / type overlayfs (rw,noatime,lowerdir=/,upperdir=/overlay)
binfmt_misc on /proc/sys/fs/binfmt_misc type binfmt_misc (rw,relatime)
debugfs on /sys/kernel/debug type debugfs (rw,relatime)
none on /proc/bus/usb type usbfs (rw,relatime)
```

```sh
root@Hiwifi:/# cat /proc/mtd
dev:    size   erasesize  name
mtd0: 00030000 00010000 "u-boot"
mtd1: 00010000 00010000 "hw_panic"
mtd2: 00010000 00010000 "Factory"
mtd3: 00140000 00010000 "kernel"
mtd4: 00e40000 00010000 "rootfs"
mtd5: 005a0000 00010000 "rootfs_data"
mtd6: 00010000 00010000 "hwf_config"
mtd7: 00010000 00010000 "bdinfo"
mtd8: 00010000 00010000 "backup"
mtd9: 00f80000 00010000 "firmware"
```

* firmware分区包括 kernel, rootfs, rootfs_data
* hwf_config 这个分区用于保存配置文件。在升级脚本和/sbin/hwf-config 脚本中都可以看到对这个分区的使用
* bdinfo 里面装有公钥信息和wifi参数mac地址等，在升级过程中没看到对这个分区的使用。
* backup 里的内容看起来像是对bdinfo的备份

bdinfo里找到的公钥信息：

```
-----BEGIN RSA PUBLIC KEY-----
MIGJAoGBAK7cBjOnooyuBwJqTfXqcHnIPvxDPbm6IsEcwtDlwKDukESn5X+v8Bre
xK3zylUPu1kAIFY53x+BQjnBgatYIXsffgjmm9uHqIrJlc9v8Vh4RXgCITcc4ZvB
NBmreHQqVOFVbF5Z5XHVgTE/8dfXRqmzuuKub9MksTpfBb8bqEhbAgMBAAE=
-----END RSA PUBLIC KEY-----
```

bdinfo里的mac信息，不知道他们是不是取fac_mac字段

```
fac_mac = D4:EE:07:0A:8C:04
```

bdinfo里另外还有两段信息，偏移地址分别为 1d80 和 fd80，不知道是干什么用的。

## 进程

```sh
ps w
  PID USER       VSZ STAT COMMAND
    1 root      1680 S    init
    2 root         0 SW   [kthreadd]
    3 root         0 SW   [ksoftirqd/0]
    4 root         0 SW   [kworker/0:0]
    5 root         0 SW   [kworker/u:0]
    6 root         0 SW<  [khelper]
    7 root         0 SW   [kworker/u:1]
   58 root         0 SW   [sync_supers]
   60 root         0 SW   [bdi-default]
   62 root         0 SW<  [kblockd]
   93 root         0 SW   [kswapd0]
  142 root         0 SW   [fsnotify_mark]
  202 root         0 SW   [mtdblock0]
  207 root         0 SW   [mtdblock1]
  212 root         0 SW   [mtdblock2]
  217 root         0 SW   [mtdblock3]
  222 root         0 SW   [mtdblock4]
  227 root         0 SW   [mtdblock5]
  232 root         0 SW   [mtdblock6]
  237 root         0 SW   [mtdblock7]
  242 root         0 SW   [mtdblock8]
  247 root         0 SW   [mtdblock9]
  262 root         0 SW   [kworker/0:1]
  507 root         0 SWN  [jffs2_gcd_mtd5]
  529 root      1712 S    {rcS} /bin/sh /etc/init.d/rcS S boot
  532 root      1672 S    logger -s -p 6 -t sysinit
  534 root      1688 S    /bin/ash --login
  662 root         0 SW<  [crypto]
  675 root         0 SW   [khubd]
  922 root       880 S    /sbin/hotplug2 --override --persistent --set-rules-file /etc/hotplug2.rules --set-coldplug-cmd /sbin/udev
  944 root     13688 S    /usr/sbin/rsyslogd -f /etc/rsyslog.conf -i /var/run/rsyslog.pid
  955 root       888 S    /sbin/ubusd
  996 nobody     792 S    /usr/sbin/atd
 1104 root      1520 S    /sbin/netifd
 1636 root         0 SW   [RtmpCmdQTask]
 1638 root         0 SW   [RtmpWscTask]
 1704 root      1684 S    udhcpc -p /var/run/udhcpc-apcli0.pid -s /lib/netifd/dhcp.script -f -t 0 -i apcli0
 2289 nobody    7316 S    /usr/sbin/pdnsd --daemon -p /var/run/pdnsd.pid
 2505 root      1172 S    /usr/sbin/dnsmasq -C /var/etc/dnsmasq.conf -u root
 2656 root      1156 S    /usr/sbin/dropbear -P /var/run/dropbear.1.pid -p 22
 2772 nobody    2224 S    /usr/sbin/htpdate -4 -d -l -s -t -T 1390214218 -u nobody -D www.qq.com www.baidu.com www.youku.com www.ta
 2832 root      2004 S    /usr/bin/fcgi-cgi
 2852 root      5344 S    nginx: master process /usr/sbin/nginx
 2853 nobody    6000 S    nginx: worker process
 2856 root      1756 S    {inet_chk.sh} /bin/sh /usr/bin/inet_chk.sh
 2862 root      3896 S    /usr/lib/cmagent/cmagent
 2872 root      1748 S    {hidaemon} /bin/sh /sbin/hidaemon
 3007 root      1684 S    /usr/sbin/crond -c /etc/crontabs -l 9
 3030 root      1684 S    {wisp_chk.sh} /bin/sh /usr/bin/wisp_chk.sh
 9847 root      1664 S    sleep 10
 9882 root      1664 S    sleep 5
 9888 root      1664 S    sleep 3
 9889 root      1672 R    ps w
 ```

 * htpdate 时间同步
 * hidaemon 

## 系统版本信息
```
root@Hiwifi:/# cat /etc/openwrt_release 
DISTRIB_ID="OpenWrt"
DISTRIB_RELEASE="Bleeding Edge"
DISTRIB_REVISION="0.9005.5699s"
DISTRIB_CODENAME="barrier_breaker"
DISTRIB_TARGET="ralink/mt7620"
DISTRIB_DESCRIPTION="OpenWrt Barrier Breaker 0.9005.5699s"

root@Hiwifi:/# cat /etc/openwrt_version 
0.9005.5699s
```

## cpuinfo
```
root@Hiwifi:/tmp# cat /proc/cpuinfo 
system type             : Ralink MT7620A ver:2, eco:3
machine                 : Hiwifi Wireless HC5661 Board
processor               : 0
cpu model               : MIPS 24KEc V5.0
BogoMIPS                : 385.84
wait instruction        : yes
microsecond timers      : yes
tlb_entries             : 32
extra interrupt vector  : yes
hardware watchpoint     : yes, count: 4, address/irw mask: [0x0ffc, 0x0ffc, 0x0ffb, 0x0ffb]
ASEs implemented        : mips16 dsp
shadow register sets    : 1
kscratch registers      : 0
core                    : 0
VCED exceptions         : not available
VCEI exceptions         : not available
```

## 启动信息
```
U-Boot 1.1.3 (Aug  8 2014 - 11:40:28)

Board: Ralink APSoC DRAM:  128 MB
enable ephy clock...done. rf reg 29 = 5
SSC disabled.
******************************
Software System Reset Occurred
******************************
spi_wait_nsec: 14 
spi device id: ef 40 18 0 0 (40180000)
find flash: w25q128
*** Warning - bad CRC, using default environment

============================================ 
ASIC 7620_MP (Port5<->None)
DRAM component: 1024 Mbits DDR, width 16
DRAM bus: 16 bit
Flash component: SPI Flash
Date:Aug  8 2014  Time:11:40:28
============================================ 
 1  0 
## Booting image at bc050000 ...
   Image Name:   HC5661
   Image Type:   MIPS Linux Kernel Image (lzma compressed)
   Data Size:    1288395 Bytes =  1.2 MB
   Load Address: 80000000
   Entry Point:  80000000
   Verifying Checksum ... OK
   Uncompressing Kernel Image ... OK

Starting kernel ...

[    0.000000] Linux version 3.3.8 (build@DEV-9-1-243) (gcc version 4.6.4 20121210 (prerelease) (Linaro GCC 4.6-2012.12) ) #1 Mon Aug 25 14:08:47 CST 2014
[    0.000000] 
[    0.000000]  The CPU feqenuce set to 580 MHz
[    0.000000] 
[    0.000000]  MIPS CPU sleep mode enabled.
[    0.000000]  PCIE: bypass PCIe DLL.
[    0.000000]  PCIE: Elastic buffer control: Addr:0x68 -> 0xB4
[    0.000000]  disable all power about PCIe
[    0.000000] CPU revision is: 00019650 (MIPS 24KEc)
[    0.000000] Determined physical RAM map:
[    0.000000]  memory: 08000000 @ 00000000 (usable)
[    0.000000] Initrd not found or empty - disabling initrd
[    0.000000] Zone PFN ranges:
[    0.000000]   Normal   0x00000000 -> 0x00008000
[    0.000000] Movable zone start PFN for each node
[    0.000000] Early memory PFN ranges
[    0.000000]     0: 0x00000000 -> 0x00008000
[    0.000000] Built 1 zonelists in Zone order, mobility grouping on.  Total pages: 32512
[    0.000000] Kernel command line:  board=HC5661 console=ttyS1,115200 rootfstype=squashfs,jffs2
[    0.000000] PID hash table entries: 512 (order: -1, 2048 bytes)
[    0.000000] Dentry cache hash table entries: 16384 (order: 4, 65536 bytes)
[    0.000000] Inode-cache hash table entries: 8192 (order: 3, 32768 bytes)
[    0.000000] Primary instruction cache 64kB, VIPT, 4-way, linesize 32 bytes.
[    0.000000] Primary data cache 32kB, 4-way, PIPT, no aliases, linesize 32 bytes
[    0.000000] Writing ErrCtl register=000374e0
[    0.000000] Readback ErrCtl register=000374e0
[    0.000000] Memory: 126228k/131072k available (2558k kernel code, 4844k reserved, 576k data, 308k init, 0k highmem)
[    0.000000] SLUB: Genslabs=9, HWalign=32, Order=0-3, MinObjects=0, CPUs=1, Nodes=1
[    0.000000] NR_IRQS:128
[    0.000000] MTK/Ralink System Tick Counter init... cd:802ffae0, m:214748, s:32
[    0.000000] console [ttyS1] enabled
[    0.010000] Calibrating delay loop... 385.84 BogoMIPS (lpj=1929216)
[    0.070000] pid_max: default: 32768 minimum: 301
[    0.070000] Mount-cache hash table entries: 512
[    0.080000] NET: Registered protocol family 16
[    0.080000] MIPS: machine is Hiwifi Wireless HC5661 Board
[    0.090000] gpiochip_add: registered GPIOs 0 to 23 on device: MT7620-GPIO0
[    0.090000] gpiochip_add: registered GPIOs 24 to 39 on device: MT7620-GPIO1
[    0.100000] gpiochip_add: registered GPIOs 40 to 71 on device: MT7620-GPIO2
[    0.100000] gpiochip_add: registered GPIOs 72 to 72 on device: MT7620-GPIO3
[    1.300000] PCIE0 no card, disable it(RST&CLK)
[    1.310000] bio: create slab <bio-0> at 0
[    1.320000] PCI host bridge to bus 0000:00
[    1.320000] pci_bus 0000:00: root bus resource [mem 0x20000000-0x2fffffff]
[    1.330000] pci_bus 0000:00: root bus resource [io  0x10160000-0x1016ffff]
[    1.330000] Switching to clocksource Ralink external timer
[    1.340000] NET: Registered protocol family 2
[    1.340000] IP route cache hash table entries: 1024 (order: 0, 4096 bytes)
[    1.350000] TCP established hash table entries: 4096 (order: 3, 32768 bytes)
[    1.360000] TCP bind hash table entries: 4096 (order: 2, 16384 bytes)
[    1.360000] TCP: Hash tables configured (established 4096 bind 4096)
[    1.370000] TCP reno registered
[    1.370000] UDP hash table entries: 256 (order: 0, 4096 bytes)
[    1.380000] UDP-Lite hash table entries: 256 (order: 0, 4096 bytes)
[    1.390000] NET: Registered protocol family 1
[    1.630000] RT3xxx EHCI/OHCI init.
[    1.650000] squashfs: version 4.0 (2009/01/31) Phillip Lougher
[    1.660000] JFFS2 version 2.2 (NAND) (SUMMARY) (LZMA) (RTIME) (CMODE_PRIORITY) (c) 2001-2006 Red Hat, Inc.
[    1.670000] msgmni has been set to 246
[    1.670000] io scheduler noop registered
[    1.680000] io scheduler deadline registered (default)
[    1.680000] Serial: 8250/16550 driver, 2 ports, IRQ sharing disabled
[    1.690000] serial8250: ttyS0 at MMIO 0x10000500 (irq = 37) is a 16550A
[    1.700000] serial8250: ttyS1 at MMIO 0x10000c00 (irq = 12) is a 16550A
[    1.720000] brd: module loaded
[    1.720000] deice id : ef 40 18 0 0 (40180000)
[    1.730000] chip info: w25q128(ef 40180000)
[    1.730000] mtd name = raspi, size = 0x01000000 (16M) erasesize = 0x00010000 (64K)
[    1.740000] raspi: squash filesystem found at block 25
[    1.760000] Creating 9 MTD partitions on "raspi":
[    1.760000] 0x000000000000-0x000000030000 : "u-boot"
[    1.770000] mtdchar alloc 0x10000 memory success
[    1.770000] 0x000000030000-0x000000040000 : "hw_panic"
[    1.780000] 0x000000040000-0x000000050000 : "Factory"
[    1.790000] 0x000000050000-0x000000190000 : "kernel"
[    1.790000] 0x000000190000-0x000000fd0000 : "rootfs"
[    1.800000] mtd: partition "rootfs" set to be root filesystem
[    1.810000] mtd: partition "rootfs_data" created automatically, ofs=A30000, len=5A0000 
[    1.810000] 0x000000a30000-0x000000fd0000 : "rootfs_data"
[    1.820000] 0x000000fd0000-0x000000fe0000 : "hwf_config"
[    1.830000] 0x000000fe0000-0x000000ff0000 : "bdinfo"
[    1.830000] 0x000000ff0000-0x000001000000 : "backup"
[    1.840000] 0x000000050000-0x000000fd0000 : "firmware"
[    1.850000] Stopped WatchDog Timer.
[    1.850000] Ralink APSoC Hardware Watchdog Timer
[    1.860000] TCP cubic registered
[    1.870000] NET: Registered protocol family 17
[    1.870000] 8021q: 802.1Q VLAN Support v1.8
[    1.880000] Freeing unused kernel memory: 308k freed
-HiWiFi initrd-
- preinit -
Press the [f] key and hit [enter] to enter failsafe mode
keypressed: 1
- regular preinit -
[    7.020000] JFFS2 notice: (506) jffs2_build_xattr_subsystem: complete building xattr subsystem, 1 of xdatum (0 unchecked, 0 orphan) and 51 of xref (0 dead, 39 orphan) found.
switching to jffs2
- init -

Please press Enter to activate this console. [    9.220000] Uniform Multi-Platform E-IDE driver
[    9.390000] NET: Registered protocol family 10
[    9.600000] SCSI subsystem initialized
[    9.740000] usbcore: registered new interface driver usbfs
[    9.740000] usbcore: registered new interface driver hub
[    9.750000] usbcore: registered new device driver usb
[   10.500000] 
[   10.500000] 
[   10.500000] === pAd = c04dc000, size = 1056192 ===
[   10.500000] 
[   10.510000] <-- RTMPAllocTxRxRingMemory, Status=0
[   10.510000] <-- RTMPAllocAdapterBlock, Status=0
[   10.520000] AP Driver version-2.7.1.6
[   10.610000] device-mapper: ioctl: 4.22.0-ioctl (2011-10-19) initialised: dm-devel@redhat.com
[   10.910000] loop: module loaded
[   11.070000] PPP generic driver version 2.4.2
[   11.110000] PPP MPPE Compression module registered
[   11.130000] IPv6 over IPv4 tunneling driver
[   11.240000] exFAT: Core Version 1.2.4
[   11.250000] exFAT: FS Version 1.2.4
[   11.270000] GRE over IPv4 demultiplexor driver
[   11.360000] GRE over IPv4 tunneling driver
[   11.440000] ide-gd driver 1.18
[   11.690000] ip_tables: (C) 2000-2006 Netfilter Core Team
[   11.870000] NET: Registered protocol family 24
[   12.050000] ehci_hcd: USB 2.0 'Enhanced' Host Controller (EHCI) Driver
[   12.070000] rt3xxx-ehci rt3xxx-ehci: Ralink EHCI Host Controller
[   12.080000] rt3xxx-ehci rt3xxx-ehci: new USB bus registered, assigned bus number 1
[   12.120000] rt3xxx-ehci rt3xxx-ehci: irq 18, io mem 0x101c0000
[   12.140000] rt3xxx-ehci rt3xxx-ehci: USB 0.0 started, EHCI 1.00
[   12.140000] usb usb1: New USB device found, idVendor=1d6b, idProduct=0002
[   12.150000] usb usb1: New USB device strings: Mfr=3, Product=2, SerialNumber=1
[   12.160000] usb usb1: Product: Ralink EHCI Host Controller
[   12.160000] usb usb1: Manufacturer: Linux 3.3.8 ehci_hcd
[   12.170000] usb usb1: SerialNumber: rt3xxx
[   12.170000] hub 1-0:1.0: USB hub found
[   12.180000] hub 1-0:1.0: 1 port detected
[   12.280000] nf_conntrack version 0.5.0 (1977 buckets, 7908 max)
[   12.580000] PPTP driver version 0.8.5
[   12.930000] xt_time: kernel timezone is -0000
[   13.540000] nf_conntrack_rtsp v0.6.21 loading
[   13.560000] nf_nat_rtsp v0.6.21 loading
[   13.700000] ip6_tables: (C) 2000-2006 Netfilter Core Team
[   14.290000] ohci_hcd: USB 1.1 'Open' Host Controller (OHCI) Driver
[   14.310000] rt3xxx-ohci rt3xxx-ohci: RT3xxx OHCI Controller
[   14.320000] rt3xxx-ohci rt3xxx-ohci: new USB bus registered, assigned bus number 2
[   14.330000] rt3xxx-ohci rt3xxx-ohci: irq 18, io mem 0x101c1000
[   14.390000] usb usb2: New USB device found, idVendor=1d6b, idProduct=0001
[   14.400000] usb usb2: New USB device strings: Mfr=3, Product=2, SerialNumber=1
[   14.400000] usb usb2: Product: RT3xxx OHCI Controller
[   14.410000] usb usb2: Manufacturer: Linux 3.3.8 ohci_hcd
[   14.410000] usb usb2: SerialNumber: rt3xxx-ohci
[   14.420000] hub 2-0:1.0: USB hub found
[   14.420000] hub 2-0:1.0: 1 port detected
[   14.530000] Initializing USB Mass Storage driver...
[   14.530000] usbcore: registered new interface driver usb-storage
[   14.540000] USB Mass Storage support registered.
[   14.650000] fuse init (API version 7.18)
[   20.700000] device eth2.1 entered promiscuous mode
[   20.700000] device eth2 entered promiscuous mode
[   20.720000] br-lan: port 1(eth2.1) entered forwarding state
[   20.720000] br-lan: port 1(eth2.1) entered forwarding state
[   21.370000] device eth2.2 entered promiscuous mode
[   21.750000] ADDRCONF(NETDEV_CHANGE): eth2.1: link becomes ready
[   22.720000] br-lan: port 1(eth2.1) entered forwarding state
[   24.400000] RX DESC a7228000  size = 4096
[   24.520000] default ApCliAPSDCapable[0]=0
[   24.720000] Key1Str is Invalid key length(0) or Type(0)
[   24.730000] Key2Str is Invalid key length(0) or Type(0)
[   24.740000] Key3Str is Invalid key length(0) or Type(0)
[   24.740000] Key4Str is Invalid key length(0) or Type(0)
[   24.760000] EntryLifeCheck=256
[   24.770000] 1. Phy Mode = 9
[   24.770000] 2. Phy Mode = 9
[   24.770000] eeFlashId: 0x7620
[   24.770000] 2.4G power value = 1010
[   24.780000] E2PROM: D0 target power=0xff20 
[   24.780000] E2PROM: 40 MW Power Delta= 0 
[   24.790000] 3. Phy Mode = 9
[   24.790000] AntCfgInit: primary/secondary ant 0/1
[   24.790000] Initialize RF Central Registers for E2 !!!
[   24.800000] Initialize RF Central Registers for E3 !!!
[   24.810000] Initialize RF Channel Registers for E2 !!!
[   24.810000] Initialize RF Channel Registers for E3 !!!
[   24.820000] Initialize RF DCCal Registers for E2 !!!
[   24.820000] Initialize RF DCCal Registers for E3 !!!
[   24.830000] D1 = 2, D2 = 9, CalCode = 16 !!!
[   24.830000] RT6352_Temperature_Init : BBPR49 = 0x2
[   24.840000] RT6352_Temperature_Init : TemperatureRef25C = 0xfffffff8
[   24.850000] Current Temperature from BBP_R49=0xfffffff8
[   24.850000] RT6352_TssiTableAdjust: upper_bound = 0x7F decimal: 127
[   24.860000] RT6352_TssiTableAdjust: lower_bound = 0xFFFFFF80 decimal: -128
[   24.860000] *** RT6352_TssiTableAdjust: G Tssi[-7 .. +7] = -128 -1 -1 -1 -128 -1 -1
[   24.860000]  - -1 - -1 -1 -1 -1 -1 -1 -1, offset=-8, tuning=0
[   24.880000] RT6352_TssiTableAdjust: G Tssi[-7 .. +7] = -128 -9 -9 -9 -128 -9 -9 - -9 - -9 -9 -9 -9 -9 -9 -9, offset=-8, tuning=0
[   24.890000] mp_temperature=0x00, step = +7
[   24.890000] E2PROM: G Tssi[-7 .. +7] = -128 -9 -9 -9 -128 -9 -9 - -9 - -9 -9 -9 -9 -9 -9 -9, offset=-8, tuning=0
[   24.910000]  TX BW Filter Calibration !!!
[   25.040000]  RX BW Filter Calibration !!!
[   25.270000] LOFT Calibration Done!
[   25.280000] IQCalibration Start!
[   25.290000] IQCalibration Done! CH = 0, (gain= 1, phase=3d)
[   25.300000] IQCalibration Start!
[   25.310000] IQCalibration Done! CH = 1, (gain= f, phase= 2)
[   25.320000] TX IQ Calibration Done!
[   25.360000] bAutoTxAgcG = 0
[   25.360000] MCS Set = ff ff 00 00 01
[   25.440000] Main bssid = d4:ee:07:0a:8c:04
[   25.450000] <==== rt28xx_init, Status=0
[   25.450000] 0x1300 = 00064320
[   25.470000] device ra0 entered promiscuous mode
[   25.470000] br-lan: port 2(ra0) entered forwarding state
[   25.480000] br-lan: port 2(ra0) entered forwarding state
[   26.300000] Simple address matching list for Netfilter - HiWiFi
[   27.480000] br-lan: port 2(ra0) entered forwarding state
[   34.410000] [NOTICE] <HWF-Kproxy> (hwf_kproxy_init:1733) kproxy hook prioriy -101
[   34.420000] [NOTICE] <HWF-Kproxy> (hwf_kproxy_init:1751) Module is inserted
[   34.670000] [NOTICE] <HWF-SQOS> (hwf_sqos_lan_init:120) lan: 192.168.199.1, lan1: 0.0.0.0
[   34.680000] [NOTICE] <HWF-SQOS> (hwf_sqos_init:186) Module is inserted
[   34.990000] [NOTICE] <HWF-SQOS> (hwf_sqos_qdisc_init:343) tc direction  1
[   35.000000] [NOTICE] <HWF-SQOS> (hwf_sqos_qdisc_init:344) tc queue len max 4000
[   35.010000] [NOTICE] <HWF-SQOS> (hwf_sqos_qdisc_init:343) tc direction  0
[   35.020000] [NOTICE] <HWF-SQOS> (hwf_sqos_qdisc_init:344) tc queue len max 4000
[   39.470000] device eth2.2 left promiscuous mode
[   39.570000] ethernet: port0 linkdown

BusyBox v1.19.4 (2014-08-25 13:58:48 CST) built-in shell (ash)
Enter 'help' for a list of built-in commands.

***********************************************************
              __  __  _              _   ____  _   TM
             / / / / (_) _      __  (_) / __/ (_)
            / /_/ / / / | | /| / / / / / /_  / / 
           / __  / / /  | |/ |/ / / / / __/ / /  
          /_/ /_/ /_/   |__/|__/ /_/ /_/   /_/   
                  http://www.hiwifi.com/
***********************************************************
root@Hiwifi:/#
```

---

## 极路由固件升级过程

参考这篇[极1S自制固件](http://www.iptvfans.cn/wiki/index.php/%E6%9E%811S%E8%87%AA%E5%88%B6%E5%9B%BA%E4%BB%B6)，可以找到如何下载固件，并解出固件。

极路由的升级过程主要部分还是OpenWrt里的sysupgrade，在里面的平台相关部分嵌入了自己定义的platform_xxx相关函数。sysupgrade 的执行过程在另一篇文章里已经分析过了，这里主要看看极路由都做了哪些自定义操作。

平台相关部分的脚本都在 /lib/upgrade/platform.sh 中，极路由在其中写了几个函数：

* platform_check_image() 用于检查固件是否有效，以及固件是否带有uboot
* platform_pre_upgrade() 执行写入操作前的准备动作，比如亮灯的方式和卸载一些模块
* platform_firmware_save() 在do_upgrade()之前把固件复制为/tmp/data/hiwifi/rom.bin，并计算md5。使用grep发现极路由在usr/lib/lua/openapi/system/下有三个lua脚本操作了rom.bin。猜想可能是极路由开放的某些API会对固件做某种操作。令人发指的是系统里几乎所有lua脚本都以bytecode形式存在。
* platform_do_upgrade() 固件烧写操作

这里主要看看其中的 `platform_check_image` 和 `platform_do_upgrade` 两个操作。

### platform\_check\_image

```sh
platform_check_image() {
	local name="HIWIFI"
	local machine=$(awk 'BEGIN{FS="[ \t]+:[ \t]"} /machine/ {print $2}' /proc/cpuinfo)
	local board=$(tw_board_name)
	local pci_devices=$(cat /proc/bus/pci/devices | wc -l)
	local magic="$(get_magic_word "$1")"
	local magic_long="$(get_magic_long "$1")"
	local magic_boot="$(get_magic_boot "$1")"
	local board_len="$(echo "$board" | wc -L)"
	local pdtname_up=$(get_board_name_up "$1" | cut -c 1-"$board_len")
	local pdtname_smt=$(get_board_name_smt "$1" | cut -c 1-"$board_len")

	[ "$ARGC" -gt 1 ] && return 1

	case "$board" in
	HC5661 | HC5761 | HB5981m | HB5981s | HC5641 | HC5663 | HC5861 |\
	HB5881 | BL-H750AC | BL-855R | HB5811)
		[ "$magic_long" != "27051956" -a "$magic" != "2705" -a "$magic_boot" != "ff00001000000000fd00001000000000" ] && {
			echo "Invalid image type."
			return 1
		}
		[ "$pdtname_up" != "$board" -a "$pdtname_smt" != "$board" ] && {
			[ "$pdtname_up" != "$name" -a "$pdtname_smt" != "$name" ] && {
				echo "Invalid image board."
				return 1
			}
		}
		[ "$magic_boot" == "ff00001000000000fd00001000000000" ] && {
			touch /tmp/img_has_boot
			echo "Image has uboot."
		}		
		return 0
		;;
	esac

	echo "Sysupgrade is not yet supported on $board."
	return 1
}
```

* get_magic_word, get_magic_long, get_magic_boot, 从.bin中取中字段，判断升级固件是否有效

* 固件是firmware(kernel+rootfs)时，取出的$magic_long为"27051956", $magic为"2705"。

* 固件为 uboot + firmware 时，取出的magic_boot为"ff00001000000000fd00001000000000"

* get_board_name_up, get_board_name_smt 从.bin中取出板型描述字串，判断固件与板子是否匹配

* 检测到固件中包含uboot时，则创建/tmp/image_has_boot。当真正执行烧写动作时，会先检查是否存在这个文件，如果存在则先升级uboot

### platform\_do\_upgrade

```sh
platform_do_upgrade() {
	local board=$(tw_board_name)

	case "$board" in
	HC5661 | HC5761 | HB5981m | HB5981s | HC5641 | HC5663 | HC5861 |\
	HB5881 | BL-H750AC | BL-855R | HB5811)
		platform_do_upgrade_hiwifi "$ARGV"
		;;
	*)
		default_do_upgrade "$ARGV"
		;;
	esac
}
```

`platform_do_upgrade_hiwifi` 定义在 /lib/upgrade/hiwifi.sh中

platform_do_upgrade_hiwifi中做了以下事情：

1. 先控制system, internet, wlan2p4三个灯的闪烁。

2. 如果固件中有uboot，则比较一下固件中uboot和本地uboot的版本，版本不同的话就烧写uboot

3. 将配置备份写入hw_config分区，然后分别烧写uboot(如果有的话)和firmware

```sh
	if [ "$SAVE_CONFIG" -eq 1 -a -z "$USE_REFRESH" ]; then
		do_conf_backup "$CONF_TAR"
		if [ -f /tmp/img_has_boot ]; then
			if [ $upgrade_boot -eq 1 ]; then
				do_upgrade_bootloader "$1"
			fi
			get_image "$1" | dd bs=2k skip=160 conv=sync 2>/dev/null | mtd -q -l -j "$CONF_TAR" write - "${PART_NAME:-image}"
		else
			get_image "$1" | mtd -q -l -j "$CONF_TAR" write - "${PART_NAME:-image}"
		fi
	else
		if [ -f /tmp/img_has_boot ]; then
			if [ $upgrade_boot -eq 1 ]; then
				do_upgrade_bootloader "$1"
			fi
			get_image "$1" | dd bs=2k skip=160 conv=sync 2>/dev/null | mtd -q -l write - "${PART_NAME:-image}"
		else
			get_image "$1" | mtd -q -l write - "${PART_NAME:-image}"
		fi
	fi
```

上面这一段是升级的关键操作，不过里面的if...else...fi写的很冗余，很好奇为什么不写成下面这样：

```sh
	if [ "$SAVE_CONFIG" -eq 1 -a -z "$USE_REFRESH" ]; then
		do_conf_backup "$CONF_TAR"
	fi

	if [ -f /tmp/img_has_boot ]; then
		if [ $upgrade_boot -eq 1 ]; then
			do_upgrade_bootloader "$1"
		fi
		get_image "$1" | dd bs=2k skip=160 conv=sync 2>/dev/null | mtd -q -l write - "${PART_NAME:-image}"
	else
		get_image "$1" | mtd -q -l write - "${PART_NAME:-image}"
	fi
```

**备份操作**

`do_conf_backup` 是升级时的备份操作。 执行 `sysupgrade` 命令时默认会先备份配置文件，放在/tmp/sysupgrade.tgz里，这里就将这个文件写入hwf_config分区

```sh
do_conf_backup() {
	mtdpart="$(find_mtd_part hwf_config)"
	[ -z "$mtdpart" ] && return 1
	mtd -q erase hwf_config
	mtd -q write "$1" hwf_config
}

do_conf_backup "$CONF_TAR"
```

**烧写 uboot**

上面的 `do_upgrade_bootloader` 就是烧写uboot。写入之后再检查，失败的话重试，最多写5遍。在下面这个函数中可以看到，uboot就是固件的头192KB。通过这个信息我们就可以从固件中提取出uboot了。

```sh
do_upgrade_bootloader() {
	local loop_min=1
	local loop_max=5

	while [[ $loop_min -le $loop_max ]];
	do
		get_image "$1" | dd bs=2k count=96 conv=sync 2>/dev/null | mtd -q -l write - "${BOOT_NAME:-image}"
		get_image "$1" | dd bs=2k count=96 conv=sync 2>/dev/null | mtd -q -l verify - "${BOOT_NAME:-image}" 2>&1 | grep -qs "Success"
		if [[ "$?" -eq 0 ]]; then
			break
		fi
		loop_min=`expr $loop_min + 1`
	done
}
```

**烧写 firmware**

烧写操作很简单，取出 firmware，然后用 mtd write 烧写。

* 对于包含uboot的固件，去掉头部320KB，剩下的都是firmware
```sh
get_image "$1" | dd bs=2k skip=160 conv=sync 2>/dev/null | mtd -q -l -j "$CONF_TAR" write - "${PART_NAME:-image}"
```

* 对于不包含uboot的固件，直接将固件写入firmware分区
```sh
get_image "$1" | mtd -q -l -j "$CONF_TAR" write - "${PART_NAME:-image}"
```

`-j "$CONF_TAR"` 是指定 mtd 命令在写入之后使用 `"$CONF_TAR"` 作恢复备份的操作。

通过上面的分析可知，升级时 hw_panic, Factory 以及 bdinfo, backup 这四个分区不会改动。事实上bdinfo这个分区写是不可写的，即使是root用户也无法写入。猜测极路由在内核里把这个分区结构的 `.write` 指针置为空，导致这个分区在内核起来之后就无法写入。不过在uboot阶段肯定是有办法改写的。

而极路由根据bdinfo中的信息来与服务器端做验证，bdinfo中的内容是出厂时写入的。这个分区一旦破坏掉会导致这台路由器不能被极路由官方认证，无法使用官方应用市场提供的插件，进而也没有办法使用手机APP管理这台路由器。

我用另一台mt7620a的机器烧写极路由的firmware，就没办法被官方认证。可以在uboot阶段烧录好bdinfo分区。不过每一台路由器可能有唯一的识别号，如果取一台极路由的bdinfo烧到另一台上，不知道能不能成功认证。按道理来说这种智能路由厂商应该防到这一招，不然别人直接硬拷贝它的SPI Flash，然后照抄它的硬件，就完全可以复制出可以使用它云服务的路由器了。

