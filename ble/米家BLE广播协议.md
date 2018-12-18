# 米家BLE广播协议 - Mi Beacon

## 概述

Mi Beacon属于MIOT蓝牙协议的一部分，规范了基于蓝牙4.0及以上设备的广播格式，使得小米智能家庭APP以及MIUI能进行识别以及设备间可以进行联动。

## 定义

只要含有如下指定信息的广播报文，即可认为是符合了小米广播格式：

- Advertising中«Service Data»(0x16)含有Mi Service(0xFE95)
- Scan Response中«Manufacturer Specific Data»(0xFF)含有小米公司识别码(0x038F)

无论是在Advertising中，还是Scan Response中，均采用统一格式定义，详见下一章节。对于Scan Response，建议不添加可选的MAC和Capability字段，以容纳更多的Object数据。

为保证米家APP能够快速发现设备，以及蓝牙网关能够准确判断该设备是否在周边，设备在待机状态下，发送mibeacon的间隔应小于5秒。

## 格式

|      名称      |  类型  | 长度  | 是否强制 |     说明       |
|:-------------:|:------:|:----:|:-------:|:-------------:|
| Frame Control | Bitmap |  2   |   是    |  控制位        |
| Product ID    | U16    |  2   |   是    | 产品唯一ID      |
| Frame Counter | U8     |  1   |   是    | 序号(用于去重)   |
| MAC Address   | U8     |  6   |   否    | 设备MAC地址     |
| Capability    | U8     |  1   |   否    | 设备能力        |
| I/O Capability | U8 | 2 | 否 | Capability扩展，用于标识输入、输出能力 |
| Object        | U8     |  N   |   否    | 事件或属性      |
| Random Number/高位Frame Counter |U8|3|否     | 若使用加密则为必选字段 |
| Message Integrity Check |U8|1 or 4|否 | 若使用加密则为必选字段 |
| Mesh |U8|2|否 | MESH状态及能力 |

MIC（Message Integrity Check）说明如下：

- 标准型认证协议使用长度为1的MIC
- 增强型认证协议使用长度为4的MIC

### Frame Control

|    编号  |   名称         |            说明       |
|:--------:|:-------------:|:--------------------:|
|  0       | Time Protocol | 置位表示请求授时        |
|  1 ~ 2   | (Reserved)    |        保留           |
|  3       | Encrypted     | 置位表示该包已加密      |
|  4       | MAC           | 置位表示包含MAC地址     |
|  5       | Capability    | 置位表示包含Capability |
|  6       | Object        | 置位表示包含Object     |
|  7   | Mesh |      置位表示包含Mesh   |
| 8 | (Reserved) | 保留 |
|  9       | Binding Confirm| 置位表示是一个绑定确认包|
|  10      | Secure Auth   | 置位表示设备支持安全芯片认证|
|  11      | Secure Login  | 置位表示使用对称加密登陆，否则使用非对称加密登陆|
|  12 ~ 15 | Version       | 设备版本号(当前为4) |

版本号说明如下：

- 版本号1和2不再推荐继续使用
- 版本号3表示使用标准型认证协议，版本号4表示使用增强型认证协议（类型说明详见《米家BLE连接协议》）
- 版本号5表示支持Mesh功能，增加Mesh相关字段

### Object

|    名称  |     长度       |            说明       |
|:--------:|:-------------:|:--------------------:|
|   ID     |    2          |    事件或属性ID        |
|   Length |    1          |    事件或属性长度       |
|   Data   |    N          |    事件或属性内容       |

### Capability

|    编号  |   名称         |            说明         |
|:--------:|:-------------:|:----------------------:|
|  0       |  Connectable  | 置位表示设备有建立连接能力 |
|  1   | Centralable | 置位表示设备有做Central能力 |
|  2   | Encryptable | 置位表示设备有加密能力 |
|  3 ~ 4   | BondAbility | 设备绑定类型 |
| 5 | I/O | 置位表示设备支持输入/输出能力 |
| 6 ~ 7 | (Reserved) | 保留 |

设备绑定类型分为以下四种：

- 00b：用户自主选择或基于RSSI选择
- 01b：先扫描，设备发确认包后进行连接
- 10b：扫描后直接连接，设备通过震动等方式确认
- 11b：Combo型设备专用

I/O 类型可分为 Input、Output 两类，其中 Input 和 Output 又可以细分为多个类型，具体表示如下：

| 编号 |        说明         |
| :--: | :-----------------: |
|  0   | 设备可输入 6 位数字 |
|  1   | 设备可输入 6 位字母 |
|  2   | 设备可读取 NFC tag  |
|  3   | 设备可识别 QR Code  |
|  4   | 设备可输出 6 位数字 |
|  5   | 设备可输出 6 位字母 |
|  6   | 设备可生成 NFC tag  |
|  7   | 设备可生成 QR Code  |
| 8~15 |      （保留）       |

### Mesh

| 编号 |    名称    |        说明         |
| :--: | :--------: | :-----------------: |
|  0   |   PB-ADV   | 置位表示支持PB-ADV  |
|  1   |  PB-GATT   | 置位表示支持PB-GATT |
| 2~3  |   State    |    设备当前状态     |
| 4~7  |  Version   |   Mesh协议版本号    |
| 8~15 | (Reserved) |        保留         |

设备当前状态说明如下：

- 00：未认证
- 01：未配网
- 10：未配置
- 11：可使用

