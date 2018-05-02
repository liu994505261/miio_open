# 米家BLE广播协议 - Mi Beacon

[TOC]

## 概述

Mi Beacon属于MIOT蓝牙协议的一部分，规范了基于蓝牙4.0及以上设备的广播格式，使得小米智能家庭APP以及MIUI能进行识别以及设备间可以进行联动。

## 定义

只要含有如下指定信息的广播报文，即可认为是符合了小米广播格式：

- Advertising中«Service Data»(0x16)含有Mi Service(0xFE95)
- Scan Response中«Manufacturer Specific Data»(0xFF)含有小米公司识别码(0x038F)

无论是在Advertising中，还是Scan Response中，均采用统一格式定义，详见下一章节。对于Scan Response，建议不添加可选的MAC和Capability字段，以容纳更多的Object数据。

## 格式

|      名称      |  类型  | 长度  | 是否强制 |     说明       |
|:-------------:|:------:|:----:|:-------:|:-------------:|
| Frame Control | Bitmap |  2   |   是    |  控制位        |
| Product ID    | U16    |  2   |   是    | 产品唯一ID      |
| Frame Counter | U8     |  1   |   是    | 序号(用于去重)   |
| MAC Address   | U8     |  6   |   否    | 设备MAC地址     |
| Capability    | U8     |  1   |   否    | 设备能力        |
| Object        | U8     |  N   |   否    | 事件或属性      |
| Wifi MAC Address | U8  |  2   |   否    | Wifi MAC的后两字节(小端)|
| Random Num/高位Frame Counter|U8|3|否     | 若使用加密则为必选字段 |

### Frame Control

|    编号  |   名称         |            说明       |
|:--------:|:-------------:|:--------------------:|
|  0       | Time Protocol | 置位表示请求授时        |
|  1 ~ 2   | (Reserved)    |        保留           |
|  3       | Encrypted     | 置位表示该包已加密      |
|  4       | MAC           | 置位表示包含MAC地址     |
|  5       | Capability    | 置位表示包含Capability |
|  6       | Object        | 置位表示包含Object     |
|  7 ~ 8   | (Reserved)    |        保留           |
|  9       | Binding Confirm| 置位表示是一个绑定确认包|
|  10      | Secure Auth   | 置位表示设备支持安全芯片认证|
|  11      | Secure Login  | 置位表示使用对称加密登陆，否则使用非对称加密登陆|
|  12 ~ 15 | Version       | 设备版本号             |

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
|  1 ~ 2   |  MESH Ability |                        |
|  3 ~ 4   |  Binding Ability |                     |
|  5 ~ 7   |  (Reserved)   |      保留               |