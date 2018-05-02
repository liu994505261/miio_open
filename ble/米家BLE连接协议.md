# 米家BLE连接协议 - Mi Service

## 简介

本文档规范了小米智能家庭中BLE设备所使用的通信协议，所有需要连入小米智能家庭APP的BLE设备需要兼容此协议规定的广播及服务格式。

## 主要功能

该协议打通了BLE设备到小米智能家庭APP的数据链路，支持分享设备的信息到手机及云端，还规范了不同BLE设备间进行互通互联互操作的基础。此外，通过小米智能家庭APP，更多的设备间联动规则可由用户自定义，用户具有更多的发挥空间。

## 基础

### BLE设备等级

根据设备是否具有连接能力，可以将设备分为两个等级：

- 等级1的设备只具备发送广播报文的能力，因此其不需要做非常复杂的交互。
- 等级2的设备具有建立蓝牙链接的能力，可以做更多的工作。

### Mi Service定义

小米规定的Service UUID为0xFE95，各Charcteristics定义如下：

| UUID | Size | Properties | Description |
|:----:|:----:|:----------:|:-----------:|
|0x0001|  12  |Write Notify|  Token      |
|0x0002|  2   | Read       | Product ID  |
|0x0004|  10  | Read       | Version     |
|0x0005|  20  |Write Notify| WIFI Config |
|0x0010|  4   | Write      | Authentication |
|0x0013|  20  | Read Write | Device ID   |
|0x0014|  12  | Read       | Beacon Key  |
|0x0015|  20  |Write Notify| Device List |
|0x0016|  20  |Write Notify| Security Auth |

### Version定义

version格式定义为 x.x.x_xxxx，其中下划线后由生态链公司定义并管控，下划线前由小米定义，如下所示：

- 标准型(版本号1.x.x)使用混淆算法实现认证，源代码不开放，由小米实现，协议调整或修复代码bug后，均需更新小版本号。
- 增强型(版本号2.x.x)使用加密算法套件实现认证，协议及源码开源，当且仅当协议调整或升级时更新小版本号

### Application Service定义

| UUID | Size | Properties | Description |
|:----:|:----:|:----------:|:-----------:|
|0xFFE0|  4   | Read Write | LED Control |
|0xFFE1|  4   |  Notify    |   Button    |

## 协议

### 前提

为了和小米智能家庭APP相连以及和小米其余产品互联互通，BLE设备需要符合如下格式：

- 在设备的广播数据(adv data)或扫描响应(scan rsp)中加入Mi Service UUID以及Service Data字段。
- 具有连接能力的BLE设备，需要实现基于Mi Service的安全认证流程。

### 安全

小米智能家庭蓝牙协议的安全机制可以分为两部分考虑。第一部分是蓝牙设备到手机APP之间的安全，这一部分主要由安全认证机制保证；第二部分是设备与设备通过广播进行交互时，报文的安全。这一部分若设备具有建立连接的能力，可以先建立连接以交换密钥，然后对广播报文进行加密。

### 适配

米家提供蓝牙设备与米家APP之间标准认证库，用于在蓝牙设备端创建Mi Service并进行认证交互，各接口功能如下：

|           函数名            |             功能             |              说明              |
| :-------------------------: | :--------------------------: | :----------------------------: |
|   mible_server_info_init    |        初始化设备信息        |    参数包括产品ID、版本号等    |
| mible_server_miservice_init |       初始化Mi Service       | 应在设备上电并加载协议栈后调用 |
|   mible_service_init_cmp    | Mi Service初始化完成回调函数 |      弱定义，根据需要实现      |
|       mible_connected       |         连接回调函数         |      弱定义，根据需要实现      |
|     mible_disconnected      |       断开连接回调函数       |      弱定义，根据需要实现      |
| mible_bonding_evt_callback  |       认证结果回调函数       |     用于通知应用层绑定结果     |

设备第一次与米家APP连接时，进入绑定流程，此函数返回绑定成功或失败的结果；绑定成功后(设备与用户完成绑定，可在APP中查看)，设备每次与APP建立连接，都进行登录流程，此函数返回登录成功或失败的结果。

需要注意的是：

- 建立连接后，在未绑定或登录成功前，设备端不能允许任何对设备的操作，如读写Characteristics
- 设备重新绑定成功后，需立即清除之前用户的全部配置信息

