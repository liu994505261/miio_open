# Mi Beacon协议介绍

### 概述


Mi Beacon属于MIOT蓝牙协议的一部分，规范了基于蓝牙4.0及以上设备的广播格式，使得小米智能家庭APP以及MIUI能进行识别以及设备间可以进行联动。

### 定义

只要含有如下指定信息的广播报文，即可认为是符合了Mi Beacon广播格式

- advertising中«Service Data»（0x16）含有Mi Service（UUID：0xFE95）
- scan response中«Manufacturer Specific Data»（0xFF）含有小米公司识别码（ID：0x038F）

无论是在advertising中，还是scan response中，均采用统一格式定义。

<注> 所有数据均为**小端格式**

<注> **v5版本的Mi Beacon禁止在scan response中添加有效object数据**


### Service Data 或 Manu Data 格式

| Field | Name                    | 类型   | 长度（byte） | 必备(M) / 可选(O) | 说明                                            |
| :---: | ----------------------- | ------ | ------------ | ----------------- | ----------------------------------------------- |
|   1   | Frame Control           | Bitmap | 2            | M                 | 控制位                                          |
|   2   | Product ID              | U16    | 2            | M                 | 产品ID，每类产品唯一                            |
|   3   | Frame Counter           | U8     | 1            | M                 | 序号，用于去重                                  |
|   4   | MAC Address             | U8     | 6            | C.1               | Mac地址                                         |
|   5   | Capability              | U8     | 1            | C.1               | 设备能力                                        |
|   6   | I/O capability          | U8     | 2            | C.2               | I/O能力                                         |
|   7   | Object                  | U8     | n            | C.1               | 触发事件或广播属性                              |
|   8   | Random Number           | U8     | 3            | C.1               | 如果加密则为必选字段，随机数3字节，用于加密报文 |
|   9   | Message Integrity Check | U8     | 4            | C.1               | 如果加密则为必选字段，MIC四字节                 |

C.1 : 根据Frame Control字段定义确定是否包含

C.2 : 根据Capability字段确定是否包含

<注> 如果包含的数据过多，超出Beacon长度，建议分多个Beacon广播。例如第一包广播Object，第二包广播MAC。

<注> 如果有多个Object，建议分多个Beacon广播。


### Frame Control 字段定义

|  Bit  | Name               | Description                                                                              |
| :---: | ------------------ | ---------------------------------------------------------------------------------------- |
|   0   | Reserved           | 保留                                                                                     |
|   1   | Reserved           | 保留                                                                                     |
|   2   | Reserved           | 保留                                                                                     |
|   3   | isEncrypted        | 1，该包已加密，0，该包未加密                                                             |
|   4   | MAC Include        | 1，包含固定的MAC地址，0，不包含MAC地址（如设备需要兼容 iOS, 这一位强制为1）              |
|   5   | Capability Include | 1，包含Capability，0，不包含Capability。设备未绑定前，这一位强制为1                      |
|   6   | Object Include     | 1，包含Object，0，不包含Object                                                           |
|   7   | Mesh               | 1，包含Mesh，0，不包含Mesh                                                               |
|   8   | Reserved           | 保留                                                                                     |
|   9   | bindingCfm         | 1，需要被绑定，0，不需要被绑定（当周边有多个相同设备时，用此位来表示哪个设备需要被绑定） |
|  10   | Secure Auth        | 1，设备支持安全芯片认证，0，不支持                                                       |
|  11   | Secure Login       | 1，使用对称加密登陆，0，使用非对称加密登陆（目前只支持0）                                |
| 12~15 | version            | 版本号（当前为5）                                                                        |

<注> Reserved位必须填0

### Capability 字段格式

|  Bit  | Name        | Description                               |
| :---: | ----------- | ----------------------------------------- |
|   0   | Reserved    | 保留                                      |
|   1   | Reserved    | 保留                                      |
|   2   | Reserved    | 保留                                      |
|  3~4  | BondAbility | 0，无绑定，1，前绑定，2，后绑定，3，Combo |
|   5   | I/O         | 1，包含I/O Capability字段                 |
|  6~7  | Reserved    | 保留                                      |

<注> BondAbility字段表明当周边有多个相同设备时如何确定要绑定哪个设备。无绑定：用户自主选择或基于RSSI选择。前绑定：先扫描，设备发确认包后(bindingCfm in Frame Control)进行连接。后绑定：扫描后直接连接，设备通过震动等方式确认。Combo：针对Combo芯片。

### I/O Capability

| Byte  | Name                | Description                          |
| :---: | ------------------- | ------------------------------------ |
|   0   | Base I/O capability | 0-3:基础输入能力; 4-7:基础输出能力     |
|   1   | Reserved            | 保留                                 |

Base I/O capability类型可分为Input/Output两类，细分类型具体表示如下：

| Bit  | Description         |
| :---: | ------------------- |
|  0  | 设备可输入 6 位数字 |
|  1  | 设备可输入 6 位字母 |
|  2  | 设备可读取 NFC tag  |
|  3  | 设备可识别 QR Code  |
|  4  | 设备可输出 6 位数字 |
|  5  | 设备可输出 6 位字母 |
|  6  | 设备可生成 NFC tag  |
|  7  | 设备可生成 QR Code  |

### Object

请参考[米家BLE Object定义](https://github.com/MiEcosystem/miio_open/blob/master/ble/%E7%B1%B3%E5%AE%B6BLE%20Object%E5%AE%9A%E4%B9%89.md)。
