# libnfc uart 控制 pn532

pn532开发板上可以选择接口模式，确保选择的是UART

参考[这个链接](http://robospatula.blogspot.com/2014/01/configure-install-libnfc-linux-PN532-breakout-board.html)

### 安装依赖

```
sudo apt-get install libusb-0.1-4 libusb-dev libpcsclite1 libpcsclite-dev libccid pcscd
```

### 编译

```
./configure --sysconfdir=/etc --prefix=/usr --with-drivers=pn532_uart

make

sudo make install
```

### 创建配置文件


#### pn532_via_uart2usb.conf


```
mkdir -p /etc/nfc/devices.d
```

```
sudo vim /etc/nfc/devices.d/pn532_via_uart2usb.conf
```
内容如下：

```
## Typical configuration file for PN532 board (ie. microbuilder.eu / Adafruit) device
name = "Adafruit PN532 board via UART"
connstring = pn532_uart:/dev/ttyUSB0
allow_intrusive_scan = true
log_level = 3
```

connstring 这一行的冒号后面指设备名，如果在MAC下的话，会是

```
connstring = pn532_uart:/dev/tty.usbserial
```

#### libnfc.conf

```
vim /etc/nfc/libnfc.conf
```

内容如下：

```
# Allow device auto-detection (default: true)
# Note: if this auto-detection is disabled, user has to set manually a device
# configuration using file or environment variable
allow_autoscan = true

# Allow intrusive auto-detection (default: false)
# Warning: intrusive auto-detection can seriously disturb other devices
# This option is not recommended, user should prefer to add manually his device.
#allow_intrusive_scan = false

# Set log level (default: error)
# Valid log levels are (in order of verbosity): 0 (none), 1 (error), 2 (info), 3 (debug)
# Note: if you compiled with --enable-debug option, the default log level is "debug"
#log_level = 1

# Manually set default device (no default)
# To set a default device, you must set both name and connstring for your device
# Note: if autoscan is enabled, default device will be the first device available in device list.
#device.name = "microBuilder.eu"
#device.connstring = "pn532_uart:/dev/ttyUSB0"
```

### 运行 nfc

```
./util/nfc-list

./examples/nfc-poll
```

运行结果：

```
% nfc-poll
nfc-poll uses libnfc 1.7.1
NFC reader: pn532_uart:/dev/tty.usbserial opened
NFC device will poll during 30000 ms (20 pollings of 300 ms for 5 modulations)
ISO/IEC 14443A (106 kbps) target:
    ATQA (SENS_RES): 00  04
       UID (NFCID1): e4  18  6d  db
      SAK (SEL_RES): 08
nfc_initiator_target_is_present: Target Released
Waiting for card removing...done.
```