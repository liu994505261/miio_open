# 米家蓝牙接入介绍

​	根据产品是否支持MESH、安全等级等特性，平台提供三种不同的接入方式：普通BLE接入、高安全级BLE接入、BLE MESH接入。三种接入方式各有特点，平台会提供相应的固件端软硬件支持，完成设备的认证，帮助产品接入到米家APP中。

​	对于普通安全级别的产品，如花花草草、蓝牙温湿度传感器、米家对讲机等，可采用普通BLE方式接入，无须提前烧写预置安全秘钥或增加安全芯片，开发更便捷。

​	对于高安全级别的产品，如蓝牙锁，可采用高安全级BLE方式接入，需搭配安全芯片使用，同时配合pin code OOB认证。

> BLE MESH接入目前处于内测中，暂不对外开放。  

</br>




## 普通BLE接入

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

目前，米家设备需要发送包含 mibeacon 的可连接广播才可以被米家 App和网关发现，且高安全 设备需要将 frame control 字段中 secure auth 标识位置位。

米家安全认证依靠 BLE 连接进行数据交互，设备需要具备 mi service 才可以完成认证交互。

整个安全认证过程会用到 mi service 的以下 characteristic：

 0x0010 auth control point、 0x0016 secure auth

调用`mi_sevice_init()`  将会创建包含所需 characteristic 的 mi service


当米家 App 建立蓝牙连接并发现 mi service 后，会发起认证流程。米家认证是一个异步过程，认证结果会通过回调函数通知用户。为了兼容main loop和RTOS 两种调度方式，米家认证依靠一个软件定时器驱动。使用`mi_scheduler_init ()` 初始化米家调度器后，调度器会自动创建并维护一个tick定时器。为了不阻塞嵌入式系统，整个过程分为多个时间片在`mi_schd_process()` 中执行，tick定时器会周期唤醒 MCU 分步完成认证过程。如果没有 RTOS 调度的系统，可以在main loop 中调用 `mi_schd_process()`；有 RTOS 调度的系统，可以创建一个 thread，在thread 中接收到 tick 定时器 timeouot signal 再调用`mi_schd_process()`。



`uint32_t mi_scheduler_init(uint32_t interval, mi_schd_event_handler_t handler,`

​    `mi_kbd_input_get_t recorder, mi_msc_power_manage_t manager, const void * p_iic_config)`

`interval` 单位 ms, 推荐设置为最小连接间隔。

`handler`用户米家事件调度函数， 用于通知用户米家认证结果。

`recorder`用户键盘输入函数， 用于读取键盘输入的 6 位 PIN CODE。

`manager`用户安全芯片电源管理函数。

`p_iic_config`安全芯片 IIC 配置信息。



设备完成认证后，再次建立连接后需要进行登录鉴权，登录结果会通过回调函数通知用户。另，用户权限也可以随时通过 `get_mi_authorization()` 获取。成功登录后，用户可以调用 `mi_session_encrypt/decrypt()`函数对本次连接 BLE 数据加解密。

