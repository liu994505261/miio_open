# Mi Beacon协议介绍

### 概述


Mi Beacon属于MIOT蓝牙协议的一部分，规范了基于蓝牙4.0及以上设备的广播格式，使得小米智能家庭APP以及MIUI能进行识别以及设备间可以进行联动。本文档定义了Mi Beacon的格式以及进行了实用举例。

### 定义

只要含有如下指定信息的广播报文，即可认为是符合了小米广播格式（mi Beacon）

- advertising中«Service Data»（0x16）含有mi service（UUID：0xFE95）
- scan response中«Manufacturer Specific Data»（0xFF）含有小米公司识别码（ID：0x038F）

无论是在advertising中，还是scan response中，均采用统一格式定义。对于scan response，建议不添加可选的MAC和Capability字段，以容纳更多的Object数据。

<注> 所有数据均为 小端格式


### Service Data 或 Manu Data 格式


| Field  | Name                         | 类型   | 长度（byte） | 必备(M)/可选(O) | 说明                                                         |
| ------ | ---------------------------- | ------ | ------------ | --------------- | ------------------------------------------------------------ |
| **1**  | Frame Control                | Bitmap | 2            | M               | 控制位。                                                     |
| **2**  | Product ID                   | U16    | 2            | M               | 产品ID,每类产品唯一                                          |
| **3**  | Frame Counter                | U8     | 1            | M               | 序号，用于去重                                               |
| **4**  | MAC Address                  | U8     | 6            | C.1             | Mac地址，是否包含取决于frame control                         |
| **5**  | Capability                   | U8     | 1            | C.1             | 设备能力，是否包含取决于frame contrl                         |
| **6**  | WiFi MAC Address             | U8     | 2            | C.2             | 如果 Capability.BondAbility 是 Combo 类型WiFi MAC的后两字节（小端） |
| **7**  | I/O capability               | U8     | 2            | C.2             | 如果 Capability.IO_capability 是 1                           |
| **8**  | Object                       | U8     | n            | C.1             | 触发的事件或广播的属性                                       |
| **9**  | Random Num/高位Frame Counter | U8     | 3            | C.1             | 如果加密则为必选字段增强型：随机数3字节，用于加密的报文标准型：高位Frame Counter |





### Frame Control 字段定义


| Bit       | Name               | Description                                                  |
| --------- | ------------------ | ------------------------------------------------------------ |
| **0**     | Time Protocol      | 1 请求授时，0 无                                             |
| **1**     | reserved           | 保留                                                         |
| **2**     | reserved           | 保留                                                         |
| **3**     | isEncrypted        | 1，该包已加密，0，该包未加密                                 |
| **4**     | MAC Include        | 1，包含固定的MAC地址，0，不包含MAC地址（如设备需要兼容 iOS, 这一位强制为1） |
| **5**     | Capability Include | 1，包含capability，0，不包含capability，当设备重置后，这一位强制为1 |
| **6**     | Object Include     | 1，包含Object，0，不包含Object                               |
| **7**     | Mesh*              | 1，包含Mesh，0，无                                           |
| **8**     | reserved           | 保留                                                         |
| **9**     | bindingCfm         | 1，是一个绑定确认包，0，不是绑定的确认包                     |
| **10**    | Secure Auth        | 1，设备支持安全芯片认证；0，不支持                           |
| **11**    | Secure Login       | 1，使用对称加密登陆，0，使用非对称加密登陆                   |
| **12~15** | version            | 版本号（当前为4）                                                   |


### Capability 字段格式


| Bit     | Name        | Description                                                  |
| ------- | ----------- | ------------------------------------------------------------ |
| **0**   | Connectable | 1，设备有建立连接能力，0，设备没有建立连接能力               |
| **1**   | Centralable | 1，设备有做central device能力，0，设备没有该能力             |
| **2**   | Encryptable | 1，设备有加密mibeacon能力，0，设备没有该能力                 |
| **3~4** | BondAbility | 0，  不使用MIOT选择绑定类型，1，使用MIOT前绑定方式选择绑定类型，2， 使用MIOT后绑定方式选择绑定类型，3，使用Combo型绑定 |
| **5**   | I/O         | 1，包含 I/O capability 字段                                  |
| **6~7** | Reserved    | 保留                                                         |

无绑定：用户自主选择或基于RSSI选择。

连接前确认，再绑定：先扫描，设备发确认包后进行连接。

连接后确认，再绑定：扫描后直接连接，设备通过震动等方式确认。

