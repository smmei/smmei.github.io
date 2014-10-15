---
layout: post
title: "OpenWrt升级脚本sysupgrade分析"
categories: openwrt
tags:
---

之前分析时写的，再贴到这里来。

## /sbin/sysupgrade

### 用法：
sysupgrade [<升级选项>...] <image file 或 URL>
sysupgrade [-q] [-i] <备份选项> <file>

### 升级选项：
* -d <delay> 重启前等待 delay 秒
* -f <config> 从 .tar.gz (文件或链接) 中恢复配置文件
* -i 交互模式
* -c 保留 /etc 中所有修改过的文件
* -n 重刷固件时不保留配置文件
* -T | --test 校验固件 config .tar.gz，但不真正烧写
* -F | --force 即使固件校验失败也强制烧写
* -q 较少的输出信息
* -v 详细的输出信息
* -h 显示帮助信息

### 备份选项：
* -b | --create-backup <file>
	把sysupgrade.conf 里描述的文件打包成.tar.gz 作为备份，不做烧写动作
* -r | --restore-backup <file>
	从-b 命令创建的 .tar.gz 文件里恢复配置，不做烧写动作
* -l | --list-backup
	列出 -b 命令将备份的文件列表，但不创建备份文件
	
### 举例

```
sysupgrade recovery.bin
```
使用recovery.bin来升级，如果校验成功的话，会将其写入firmware分区

## 实现

### 解析选项参数

```sh
# parse options
while [ -n "$1" ]; do
	case "$1" in
		-i) export INTERACTIVE=1;;
		-d) export DELAY="$2"; shift;;
		-v) export VERBOSE="$(($VERBOSE + 1))";;
		-q) export VERBOSE="$(($VERBOSE - 1))";;
		-n) export SAVE_CONFIG=0;;
		-c) export SAVE_OVERLAY=1;;
		-b|--create-backup) export CONF_BACKUP="$2" NEED_IMAGE=1; shift;;
		-r|--restore-backup) export CONF_RESTORE="$2" NEED_IMAGE=1; shift;;
		-l|--list-backup) export CONF_BACKUP_LIST=1; break;;
		-f) export CONF_IMAGE="$2"; shift;;
		-F|--force) export FORCE=1;;
		-T|--test) export TEST=1;;
		-h|--help) export HELP=1; break;;
		-*)
			echo "Invalid option: $1"
			exit 1
		;;
		*) break;;
	esac
	shift;
done

export CONFFILES=/tmp/sysupgrade.conffiles
export CONF_TAR=/tmp/sysupgrade.tgz

export ARGV="$*"
export ARGC="$#"
```
解析选项。$ARGV是参数列表，$ARGC是参数个数。
选项中有-d, -b, -r, -f时，由于这些选项都要带一个参数，所以使用了shift去减少$ARGV和$ARGC的值。

CONFFILES 和 CONF_TAR 是两个临时文件，后面会用到。

* `sysupgrade openwrt.bin` --> ARGV="openwrt.bin", ARGC=1
* `sysupgrade -b config.backup` --> ARGV为空，ARGC=0

### 判断参数合法

```sh
[ -n "$ARGV" -a -n "$NEED_IMAGE" ] && {
	cat <<-EOF
		-b|--create-backup and -r|--restore-backup do not perform a firmware upgrade.
		Do not specify both -b|-r and a firmware image.
	EOF
	exit 1
}
```

如果sysupgrade附带参数-b或-r时，则$NEED_IMAGE=1，否则为空。当$NEED_IMAGE=1时，我们希望ARGV是空的，否则就是出错，则输出帮助信息，并退出。 例如:

- `sysupgrade -b config.backup`，此时$NEED_IMAGE=1, ARGV为空，合法
- `sysupgrade -b config.backup openwrt.bin`，此时$NEED_IMAGE=1, ARGV为"openwrt.bin"，参数多了，错误。

```sh
# hooks
sysupgrade_image_check="platform_check_image"
[ $SAVE_OVERLAY = 0 -o ! -d /overlay/etc ] && \
	sysupgrade_init_conffiles="add_uci_conffiles" || \
	sysupgrade_init_conffiles="add_overlayfiles"
```
* 带-c参数，且"/overlay/etc"目录存在 --> sysupgrade_init_conffiles="add_overlayfiles"
* 否则 --> sysupgrade_init_conffiles="add_uci_conffiles"

这里会影响要备份的配置文件

```
include /lib/upgrade
```
包含lib/upgrade目录下的所有文件

```
[ "$1" = "nand" ] && nand_upgrade_stage2 $@
```
命令指定nand时，则调用nand_upgrade_stage2函数，例如`sysupgrade nand openwrt.bin`。 暂时使用spi flash，不讨论这里。

### sysupgrade -l

```
add_uci_conffiles() {
	local file="$1"
	( find $(sed -ne '/^[[:space:]]*$/d; /^#/d; p' \
		/etc/sysupgrade.conf /lib/upgrade/keep.d/* 2>/dev/null) \
		-type f 2>/dev/null;
	  opkg list-changed-conffiles ) | sort -u > "$file"
	return 0
}

if [ $CONF_BACKUP_LIST -eq 1 ]; then
	add_uci_conffiles "$CONFFILES"
	cat "$CONFFILES"
	rm -f "$CONFFILES"
	exit 0
fi
```

列出一份文件列表，放入/tmp/sysupgrade.conffiles，打印出来，然后删掉。文件列表：

```
find $(sed -ne '/^[[:space:]]*$/d; /^#/d; p' /etc/sysupgrade.conf /lib/upgrade/keep.d/* 2>/dev/null) -type f 2>/dev/null

opkg list-changed-conffiles
```

### `sysupgrade -b /tmp/backup.tgz`

```
do_save_conffiles() {
	local conf_tar="${1:-$CONF_TAR}"

	[ -z "$(rootfs_type)" ] && {
		echo "Cannot save config while running from ramdisk."
		ask_bool 0 "Abort" && exit
		return 0
	}
	run_hooks "$CONFFILES" $sysupgrade_init_conffiles
	ask_bool 0 "Edit config file list" && vi "$CONFFILES"

	v "Saving config files..."
	[ "$VERBOSE" -gt 1 ] && TAR_V="v" || TAR_V=""
	tar c${TAR_V}zf "$conf_tar" -T "$CONFFILES" 2>/dev/null

	rm -f "$CONFFILES"
}

if [ -n "$CONF_BACKUP" ]; then
	do_save_conffiles "$CONF_BACKUP"
	exit $?
fi
```

-b 如果指定打包文件时，$CONF_BACKUP 为那个文件名。则此时按如下流程来生成备份文件：

```
do_save_conffiles
  -> sysupgrade_init_conffiles
     -> add_uci_conffiles
        -> tar czf /tmp/backup.tgz -T /tmp/sysupgrade.conffiles
	-> rm -f /tmp/sysupgrade.conffiles
```

### restore

```
if [ -n "$CONF_RESTORE" ]; then
	if [ "$CONF_RESTORE" != "-" ] && [ ! -f "$CONF_RESTORE" ]; then
		echo "Backup archive '$CONF_RESTORE' not found."
		exit 1
	fi

	[ "$VERBOSE" -gt 1 ] && TAR_V="v" || TAR_V=""
	tar -C / -x${TAR_V}zf "$CONF_RESTORE"
	exit $?
fi
```

`sysupgrade -r config.tgz` --> tar -C / -xzf config.tgz，解压，覆盖到/目录下

### image check

```
for check in $sysupgrade_image_check; do
	( eval "$check \"\$ARGV\"" ) || {
		if [ $FORCE -eq 1 ]; then
			echo "Image check '$check' failed but --force given - will update anyway!"
			break
		else
			echo "Image check '$check' failed."
			exit 1
		fi
	}
done
```

* sysupgrade openwrt.bin --> 检查bin文件：platform_check_image openwrt.bin
* lib/upgrade/platform.sh 中定义platform_check_image函数，取.bin文件的头部，检查magic number

### backup

```
if [ -n "$CONF_IMAGE" ]; then
	case "$(get_magic_word $CONF_IMAGE cat)" in
		# .gz files
		1f8b) ;;
		*)
			echo "Invalid config file. Please use only .tar.gz files"
			exit 1
		;;
	esac
	get_image "$CONF_IMAGE" "cat" > "$CONF_TAR"
	export SAVE_CONFIG=1
elif ask_bool $SAVE_CONFIG "Keep config files over reflash"; then
	[ $TEST -eq 1 ] || do_save_conffiles
	export SAVE_CONFIG=1
else
	export SAVE_CONFIG=0
fi
```

在升级时会先保存配置文件到/tmp/sysupgrade.tgz
1. -f 指定配置文件
2. 交互模式 ask_bool 可以获取输入值，如果不在交互模式，则ask_bool的第一个参数就是默认值。
$SAVE_CONFIG默认为1, 这里调用do_save_conffiles保存当前系统的配置文件。

### upgrade

```
if [ -n "$(rootfs_type)" ]; then
	v "Switching to ramdisk..."
	run_ramfs '. /lib/functions.sh; include /lib/upgrade; do_upgrade'
else
	do_upgrade
fi
```

1. rootfs_type = "overlayfs" ，执行第一个逻辑
2. run_ramfs, 在/tmp/root下安装一个临时ramdisk，最后再执行do_upgrade
3. do_upgrade -> platform_do_upgrade -> 

`get_image "$1" | mtd -j "$CONF_TAR" write - "firmware"`

mtd工具在写入时，会把$CONF_TAR文件整合进入jffs2分区，可以看到打印信息：

```
Appending jffs2 data from /tmp/sysupgrade.tgz to firmware...
```

