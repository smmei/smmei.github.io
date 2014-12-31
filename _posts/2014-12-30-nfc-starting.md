
## NFC 相关标准

* ISO 14443 - RFID卡标准
* ISO 7816 - 接触式IC卡标准
* ISO 15693 - 
* ISO 18092 - NFC标准

### Mifare, FeliCa

* Mifare Classic -> ISO 14443-3 TYPE A
* Mifare DESFire -> ISO 14443-4 TYPE A

* FeliCa -> JIS:X6319-4

* Pboc 国内常见的支付卡

## NFC 数据格式

1. NFC-A  -> ISO 14443-3A
2. NFC-B  -> ISO 14443-3B
3. NFC-F  -> JIS 6319-4
4. NFC-V  -> ISO 15693
5. IsoDep -> ISO 14443-4
6. Ndef
7. Ndef-Formatable

# ISO/ICE14443A 帧格式

通迅起始 + 数据(+奇偶校验位) + 通信结束

每8个数据位后面有一个奇偶校验位。LSB，低位先传

NFC-A 有三种帧：
1. 短帧，用于通信初始化(Wake-Up)
2. 标准帧，用于数据交换
3. bit-oriented SSD 帧，used for collision resolution.

### 短帧

通信开始 + 7个数据位 + 通信结束

### 标准帧

由三部分组成：

1. 通信开始
2. n * (8个数据位 + 1个奇偶校验位), n >= 1
3. 能信结束


## ISO/14443A 命令与应答
