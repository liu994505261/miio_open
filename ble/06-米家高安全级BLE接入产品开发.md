# 米家高安全级BLE接入产品开发

*本文用于指导产品厂商利用已经支持米家高安全级BLE接入的芯片开发产品*

<br/>

## 接入米家

高安全级BLE接入方式的软件SDK，由三部分组成：芯片原厂SDK，米家高安全级认证库，米家智能门锁示例Demo。同时必须外接安全芯片。具体步骤如下：

1. [米家高安全级认证库](https://github.com/MiEcosystem/mijia_ble_libs)是私有工程，产品开发者首先需联系小米（yangfan23@xiaomi.com）增加获取高安全级认证库的权限。
2. 根据芯片选型选择对应的分支，下载芯片厂商提供的SDK，各分支下有对应的下载地址链接或下载方式。
3. 下载[米家智能门锁示例demo](https://github.com/MiEcosystem/mijia_ble_secure)
   - 按照各分支下的README说明，进入到芯片原厂SDK的指定目录
   - 命令行执行`git clone -b xxx(对应分支) --recursive https://github.com/MiEcosystem/mijia_ble_secure.git`。建议使用git clone --recursive命令，直接网站上下载会发现有些文件找不到，这是因为submodule没有下载成功
4. 联系小米（yangfan23@xiaomi.com）申请开通“小米蓝牙安全开发板”白名单。
5. 按照`main.c`文件中I2C引脚定义连接开发板和安全芯片。
6. 按照各分支下README说明中的路径导入工程，编译固件，下载固件测试开发板。
7. 打开米家APP，开启蓝牙，发现附近的设备，点击“小米蓝牙安全开发板”。在连接过程中出现6位数字PIN码，在固件上输入这6个数字（小米蓝牙安全开发板需要在Segger JLink RTT Viewer环境下输入）。登陆成功后，APP会有相应提示。进入插件，点击“开关闪灯”可以模拟开锁和关锁，同时开发板会广播带有相应事件Object的MiBeacon。这时也可以看到开发板上LED灯有响应。同时RTT Viewer也可以打印出开发板log。
8. APP管理者可以分享钥匙给其他小米账号用户。小米蓝牙开发板暂时不支持此功能。
9. 重置设备，目前需要两步
    - APP删除设备
    - 固件上长按重置按钮

已支持芯片平台列表：其他分支仅供米家内部测试使用

| 芯片品牌     | 芯片型号 | SDK                                                                |
| :----------- | :------- | :----------------------------------------------------------------- |
| Nordic       | 51       | https://github.com/MiEcosystem/mijia_ble_secure/tree/nordic_legacy |
| Nordic       | 52       | https://github.com/MiEcosystem/mijia_ble_secure/tree/nordic        |
| Silicon Labs | BG13     | https://github.com/MiEcosystem/mijia_ble_secure/tree/silabs        |

## 米家高安全级认证库介绍

开发者首先需要了解[米家普通BLE接入产品开发](https://github.com/MiEcosystem/miio_open/blob/master/ble/04-%E7%B1%B3%E5%AE%B6%E6%99%AE%E9%80%9ABLE%E6%8E%A5%E5%85%A5%E4%BA%A7%E5%93%81%E5%BC%80%E5%8F%91.md)流程。

产品开发过程中需了解的宏定义

| 宏名                      | 作用                                        |
| :------------------------ | :------------------------------------------ |
| MI_BLE_ENABLED            | 高安全级BLE接入产品必须打开这个宏              |
| HAVE_MSC                  | 0，无安全芯片；1，MJSC安全芯片；2，MJA1安全芯片 |
| MI_LOG_ENABLED            | 打印调试log，调试时候可以打开此宏            |
| MI_ASSERT                 | MI_LOG_ENABLED打开时才能使用，打印error信息 |
| CUSTOMIZED_MI_CONFIG_FILE | 定义开发者自己的头文件                      |
| PRODUCT_ID                | 必须与在小米IoT开发者平台创建的产品ID相同    |

<注> MJSC为第一代安全芯片，已停产。MJA1为第二代安全芯片。

修改`mi_config.h`文件中的PRODUCT_ID定义

```
#ifndef PRODUCT_ID
#ifdef MI_BLE_ENABLED
#define PRODUCT_ID  0x01CF
#else
#define PRODUCT_ID  0x0379
#endif
#endif

// pid = 0x01CF代表小米蓝牙安全开发板，仅用于测试。在真正的产品开发中，pid必须与在小米IoT开发者平台上创建的产品ID相同。
```

`main.c`文件中定义了开发板的I2C引脚用来连接安全芯片，以Nordic芯片为例：

```
#define MSC_PWR_PIN                     23   // enable MSC chip
const iic_config_t iic_config = {
        .scl_pin  = 24,
        .sda_pin  = 25,
        .freq = IIC_100K
};
```

产品开发过程中需要了解的API

| 函数名 | 功能 | 说明 |
| :----- | :--- | :--- |
| mi_scheduler_init        | 调度器初始化        | 工作间隔应小于 BLE 最小连接间隔 |
| mi_schd_process          | 调度器处理函数       |在main loop或者task执行 |
| mi_schd_event_handler    | 事件回调函数         |处理事件响应   |
| mi_session_encrypt       | 加密函数             |登陆成功后，设备和手机之间建立了加密通道，负责对传输数据加密   |
| mi_session_decrypt       | 解密函数             |负责对传输数据解密  |
| mibeacon_obj_enque       | 事件/属性信息上报    | 搭配蓝牙网关使用，开发者调用mibeacon_obj_enque将要发送的object放入发送队列，之后libs自动发送带有object的MiBeacon。**不允许**开发者调用系统广播函数广播MiBeacon。**必须采用此接口**，小米在其中定义了重发次数和重发间隔，确保事件可以被网关收到     |

关于API的具体定义在mijia_ble_libs README中有更详细的描述。https://github.com/MiEcosystem/mijia_ble_libs/blob/master/readme.md

安全芯片测试

调用 mi_scheduler_start(SYS_MSC_SELF_TEST)可以对安全芯片完成自测试，如果测试通过, mi_schd_event_handler会收到事件SCHD_EVT_MSC_SELF_TEST_PASS。

## 更多

米家高安全级BLE接入必须使用安全芯片，第二代安全芯片MJA1有两款供客户选择，主要参数区别如下：

| 参数                     | MJA1-HCIW  | MJA1-SXIW  |
|:------------------------ |:---------- |:---------- |
| 睡眠功耗(uA)             | <0.5       | <1         |
| 空闲功耗(uA)             | <10        | < 460      |
| 典型运行功耗（mA）       | 1          | 10         |
| 最大峰值电流（mA）       | 3          | 18         |
| 典型签名用时（ms)        | 38         | 78         |
| 典型验签用时（ms）       | 69         | 152        |
| 典型生成密钥对用时（ms） | 32         | 66         |
| 典型秘钥协商用时（ms）   | 41         | 130        |
| 工作温度范围（℃）      | -40 ~ +85  | -40 ~ +105 |
| ESD                   | 8KV（HBM） | 5KV（HBM） |
|       包装             |  卷带包装  |    卷带包装   |
|     安全认证          | 国内EAL4+      | 国际CC EAL5+    |

两颗芯片pin-to-pin，主要应用区别是温度范围和认证不同。出国内市场产品建议量产使用MJA1-HCIW，海外市场建议使用MJA1-SXIW。请根据产品定义与销售区域选择合适产品。

请详细阅读MJA1规格书，按照参考电路设计使用。米家一次可免费提供20PCS芯片样品、2块开发板，供开发调试使用。更多需求请与小米供应链联系采购。供应链联系方式：刘亚妹 liuyamei@xiaomi.com

## FAQ

Q: 有问题怎么办？

A: 关于产品定义或[小米IoT开发者平台](https://iot.mi.com/)的问题，请联系米家产品经理。技术问题请区分是芯片开发的问题还是米家接入的问题。如果是芯片开发的问题，请联系厂商，如果是米家接入的问题，请搜索[米家高安全级接入示例demo](https://github.com/MiEcosystem/mijia_ble_secure)相关issue，看是否有类似的问题。如果没有，请提交新issue。

 Q: 如何查看log？

 A: 设备端查看log，请下载Segger JLink RTT Viewer。连接成功可直接查看log。APP端首先需安装[debug apk](https://github.com/MiEcosystem/NewXmPluginSDK/blob/master/%E7%B1%B3%E5%AE%B6%E8%B0%83%E8%AF%95APK%E4%B8%8B%E8%BD%BD%E5%9C%B0%E5%9D%80.md)。然后查找文件管理 -> 手机 -> Android -> data -> com.xiaomi.smarthome -> files -> log -> miio-bluetooth log
