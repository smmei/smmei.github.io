---
layout: post
title: "linux 网络设备驱动"
category: linux
---

- 数据包发送

dev_queue_xmit() -> struct net_device -> hard_start_xmit -> hw

- 数据包接收

hw -> interrupt handler -> struct net_device -> netif_rx()

编写网络设备驱动功能层相关函数，填充net_device结构体，并最终注册到内核中。

## struct net_device

每个接口由一个 struct net_device 描述

### net_device 分配

```
struct net_device *alloc_netdev(int sizeof_priv, const char *name,
				void (*setup)(struct net_device *));
```
- sizeof_priv : 驱动程序私有数据大小，这个函数总共会分配`sizeof(struct net_device) + sizeof_priv` 大小的内存。
- name : 接口名。
- setup : 初始化函数。

在alloc_netdev的基础上，为不同类型的接口封装了许多函数，如：

```
以太网设备   : alloc_etherdev <linux/etherdevice.h>
光纤通道设备 : alloc_fcdev    <linux/fcdevice.h>
FDDI设备     : alloc_fddidev  <linux/fddidevice.h> 
令牌环设备   : alloc_trdev    <linux/trdevice.h> 
```

### 初始化

在向网络子系统注册之前，要完全初始化net_device结构。

1. ether_setup() 
```
void ether_setup(struct net_device *dev)
{
	dev->header_ops		= &eth_header_ops;
	dev->type		= ARPHRD_ETHER;
	dev->hard_header_len 	= ETH_HLEN;
	dev->mtu		= ETH_DATA_LEN;
	dev->addr_len		= ETH_ALEN;
	dev->tx_queue_len	= 1000;	/* Ethernet wants good queues */
	dev->flags		= IFF_BROADCAST|IFF_MULTICAST;
	dev->priv_flags		|= IFF_TX_SKB_SHARING;

	memset(dev->broadcast, 0xFF, ETH_ALEN);
}
```

### 设备注册
```
int register_netdev(struct net_device *dev)
```

### net_device 成员描述

* const struct net_device_ops *netdev_ops;

* const struct header_ops *header_ops; /* Hardware header description */


## struct sk_buff

socket buffer，套接字缓冲区，用于在linux网络子系统各层之间传递数据。

include/linux/skbuff.h


