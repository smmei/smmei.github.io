# libnfc 源码简读

简读就是简单读一下。我的库有一个习惯就是边用边看其源码实现。再一次继续这个坏习惯。

## nfc_init

```
void nfc_init(nfc_context **context);
```

做了几件事：

* 创建一个 context，其中记录当前这次上下文的行为方式。比如 allow_autoscan, log_level (默认为1，即error才报错)。

* 还会到 $sysconfdir/nfc/ 目录里读配置文件 libnfc.conf 和 devices.d/ 目录里配置文件的信息。devices.d/ 目录下可以自定义设备。

* 注册设备驱动。把支持设备的驱动挂到 nfc_drivers 链表上。


```
nfc_context *context;

nfc_init(&context);
nfc_device *pnd = nfc_open(context, NULL)
nfc_initiator_init(pnd)
```

nfc_initiator_init()

 设置当前设备的属性。如工作模式、传输速率等。
 
 
nfc_initiator_poll_target(pnd, ...)

调用注册驱动的 .initiator_poll_target 回调