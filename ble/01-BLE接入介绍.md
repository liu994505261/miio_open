# 米家蓝牙接入介绍

​根据产品是否支持MESH、安全等级等特性，平台提供三种不同的接入方式：普通BLE接入、高安全级BLE接入、BLE MESH接入。三种接入方式各有特点，平台会提供相应的固件端软硬件支持，完成设备的认证，帮助产品接入到米家APP中。

​对于普通安全级别的产品，如花花草草、蓝牙温湿度传感器、米家对讲机等，可采用普通BLE方式接入，无须提前烧写预置安全秘钥或增加安全芯片，开发更便捷。

​对于高安全级别的产品，如蓝牙锁，可采用高安全级BLE方式接入，需搭配安全芯片使用，同时配合pin code OOB认证。

> BLE MESH接入目前处于内测中，暂不对外开放。

</br>




## 普通BLE接入

通过普通BLE方式接入后，设备将获得如下能力：
1. 设备获得由米家服务器分配的唯一device id，并绑定在登陆米家APP的账号下。
2. 设备可以通过BLE GATT通路与米家APP进行数据通信，此数据通道是加密的。
3. 设备可以发送包含数据信息的Mi Beacon。小米所有BLE网关都可以接收Mi Beacon并转发给米家服务器。包含数据信息的Mi Beacon也是加密的。
4. 设备可以在同一小米账号下的多个手机上同时使用。
5. 设备可以被安全分享至另外的小米账号。
6. 设备可以通过米家App进行OTA。OTA的image文件必须经过米家服务器的签名，以确保image的安全性。

**普通BLE方式接入的设备，通过GATT与米家APP交互的数据和通过Mi Beacon与小米网关交互的数据都必须加密。**

普通BLE接入方式的软件SDK，由三部分组成：芯片原厂SDK，米家标准认证库，米家示例Demo。

芯片原厂SDK：原厂官网下载，在米家示例Demo里面有原厂的链接。

米家示例Demo： https://github.com/MiEcosystem/mijia_ble 。

已支持芯片平台列表：

| 芯片品牌 | 芯片型号 | SDK |
| :--- | :--- | :--- |
| Nordic | 51/52 | https://github.com/MiEcosystem/mijia_ble/tree/Nordic |
| Nordic | 52（SDK15.2） | https://github.com/MiEcosystem/mijia_ble/tree/Nordic_SDK15.2 |
| Dialog | DA14585 | https://github.com/MiEcosystem/mijia_ble/tree/Dialog |
| NXP | QN908x  /  QN902x| https://github.com/MiEcosystem/mijia_ble/tree/NXP |
| Cypress | 20706 | https://github.com/MiEcosystem/mijia_ble/tree/cypress |
| ST | BlueNRG-1/2 | https://github.com/MiEcosystem/mijia_ble/tree/ST |
| Silicon Labs | BG13 | https://github.com/MiEcosystem/mijia_ble/tree/Silabs |
| Chipsea | CST92F30 | https://github.com/MiEcosystem/mijia_ble/tree/Chipsea |

平台提供各芯片demo见表中链接。

lib文件申请请发送BLE SDK使用申请邮件，详见[《开发者反馈指引》](https://iot.mi.com/new/guide.html?file=11-%E5%B8%B8%E7%94%A8%E4%BF%A1%E6%81%AF/01-%E5%BC%80%E5%8F%91%E8%80%85%E5%8F%8D%E9%A6%88%E6%8C%87%E5%BC%95)。




## 高安全级BLE接入

通过此类接入可以提供更高的安全等级
1. 必须使用小米安全芯片，安全芯片可以保证设备是小米认证过的设备。
2. 每次绑定都会生成一个**新的**长期密钥(根密钥)，每次连接都会生成一个**新的**会话密钥。
3. 高安全级接入可以应对重放攻击、窃听攻击、中间人攻击、垃圾桶攻击、暴力破解等一系列攻击手段，最大程度保证设备安全。

高安全级接入示例Demo： https://github.com/MiEcosystem/mijia_ble_secure 。

已支持芯片平台列表：

| 芯片品牌 | 芯片型号 | SDK |
| :--- | :--- | :--- |
| Nordic | 51 | https://github.com/MiEcosystem/mijia_ble/tree/Nordic |
| Nordic | 52（SDK15.2） | https://github.com/MiEcosystem/mijia_ble/tree/Nordic_SDK15.2 |
| Silicon Labs | BG13 | https://github.com/MiEcosystem/mijia_ble/tree/Silabs |
