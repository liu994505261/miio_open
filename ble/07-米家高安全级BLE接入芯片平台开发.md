# 米家高安全级 BLE 产品接入指南
<!> 本文用于指导 **固件开发者** 实现高安全级 BLE 产品接入

## 准备工作
### 软件环境：
开发环境搭建: 
* IDE/toolchains (根据芯片 SDK 要求进行安装，如遇问题可咨询芯片原厂 FAE)
* [JLink](https://www.segger.com/downloads/jlink/)
* [Git](https://git-scm.com/downloads)

### 硬件环境：
硬件平台可以选用芯片原厂的开发板，或者直接使用在研产品 PCBA。推荐，优先使用原厂开发板环境进行功能验证，米家提供开发板 Demo Project，完成功能验证后再移植到在研产品 PCBA。
Demo Project GitHub 地址如下：（请按照每个平台分支下 README 操作指引，进行下载）

| 芯片厂商      | 芯片平台        | Demo Project                                                       |
| :----------- | :-------       | :----------------------------------------------------------------- |
| Nordic       | 51 Series      | https://github.com/MiEcosystem/mijia_ble_secure/tree/nordic_legacy |
| Nordic       | 52 Series      | https://github.com/MiEcosystem/mijia_ble_secure/tree/nordic        |
| Silicon Labs | BG13           | https://github.com/MiEcosystem/mijia_ble_secure/tree/silabs        |

安全认证需要外接认证芯片。有关米家认证芯片的详细资料以及样片申请 (请联系产品接口人)。

## 代码集成
(*如果开发者直接采用 Demo project 进行开发，可以跳过本节内容*)

Demo Project 代码结构：

![高安全级 BLE 产品软件框架](./pics/secure-auth.png)

如上图所示，高安全级 BLE 产品接入需要用到以下代码：
* SoC SDK & 协议栈
* [mijia ble api](https://github.com/MiEcosystem/mijia_ble_api)
* [mijia ble libs](https://github.com/MiEcosystem/mijia_ble_libs.git)

其中，SoC stack & SDK 由芯片原厂提供，米家会推荐已验证的 SDK 版本；
mijia ble api & libs 由米家提供，以源码形式托管在 GitHub 上。开发者可以通过 git submodule 方式将米家代码集成到自己仓库中。mijia ble api 为开源仓库，无需申请权限即可访问。mijia ble libs 为私有仓库，需申请访问权限。（请联系产品接口人）

### 集成方法如下：（将以 nordic 为例，示例如何具体操作）
1.下载芯片原厂 SDK。(SDK 版本根据 mijia ble api 要求进行选择)
```
示例：从 nordic 官网下载 [SDK 15.2.0](https://www.nordicsemi.com/Software-and-Tools/Software/nRF5-SDK/Download#infotabs)
```

2.按照芯片原厂 SDK 说明文档，新建 BLE 空工程。(如果已创建工程，可以跳过此步)
```
示例：nordic BLE 空工程在 `SDK\examples\ble_peripheral\ble_app_template` 目录下，复制一份 ble_app_template 并重命名，比如叫 secure_demo
```

3.在新建的 BLE 工程目录下创建 [Git](https://git-scm.com/) 本地仓库。(如果已有本地仓库，可以跳过此步)
```
示例：
$ git init
```

4.添加 mijia ble api 和 mijia ble libs 到本地仓库。添加 mijia ble api 时需选择对应芯片平台的分支。
```
示例：
$ git submodule add -b nordic https://github.com/MiEcosystem/mijia_ble_api.git
$ git submodule add -b master https://github.com/MiEcosystem/mijia_ble_libs.git
$ git commit -am "add submodules mijia ble api & libs"
```

5.添加 mijia ble api 和 mijia ble libs 路径下源码到 BLE 工程中，并将路径加入工程的 `include path`。mijia ble api 会使用到 SoC SDK 中的 BLE，TIMER，IIC，FLASH 等功能，需要开发者将这些功能模块添加到工程中。 （详细资源占用情况及配置方法，请参考对应芯片分支下的 README？）。添加下列路径下代码。
```
mijia_ble_libs
├── common
├── cryptography
│   └── mja1
├── mijia_profiles
├── secure_auth
└── third_party
    ├── SEGGER_RTT
    ├── mbedtls
    ├── micro-ecc
    └── pt
```

>注意，libs 使用的 `./third_party/` 下的开源代码（例如， mbedtls，RTT），可能在 SoC SDK 中已经包含。当工程编译出现重复定义问题时，开发者需要去掉 libs 中重复文件以解决冲突。（例如，原厂工程中已经包含 SEGGER RTT 文件，就无需再添加 `./third_party/SEGGER_RTT` 源码）

6.创建一个配置文件，在文件中添加下列宏定义，然后将 `CUSTOMIZED_MI_CONFIG_FILE=<your_config.h>` 添加到工程 `Preprocesser symbols`中；也可以直接将以下宏定义添加到工程的 `Preprocesser symbols` 配置中；

| 宏名                      | 作用                                         |
| :------------------------ | :------------------------------------------- |
| CUSTOMIZED_MI_CONFIG_FILE | 自定义的 config 头文件                        |
| HAVE_MSC                  | 外接认证芯片型号 1，MJSC系列；2，MJA1系列       |
| MI_BLE_ENABLED            | 使能高安全级 BLE 接入                        |
| MI_LOG_ENABLED            | 打印调试log，调试时候可以打开此宏            |
| MI_ASSERT                 | MI_LOG_ENABLED 打开时才有效，进行 assert 检查 |
| PRODUCT_ID                | 必须与在小米IoT开发者平台创建的产品ID相同   |
| DEVELOPER_VERSION         | 开发者固件版本号                            |

*<注1> MJSC为第一代米家认证芯片，已停产。MJA1为第二代米家认证芯片。* <br>
*<注2> 如果缺省宏定义，会采用 mi_config.h 中默认值。例如，不定义 PRODUCT_ID，则默认为 0x01CF 小米蓝牙安全开发板，此产品需白名单权限才可以发现。（如需此 PID 进行快速测试，请申请白名单权限）*

## 使用米家库
(*进行以下步骤前请先阅读 mijia ble libs [**使用手册**](https://github.com/MiEcosystem/mijia_ble_libs/blob/master/readme.md)*)

1.芯片运行环境初始化（包括但不限于芯片电源，时钟，GPIO，TIMER，IIC，BLE stack）。

2.使用 `mibeacon_data_set()` 生成 `mibeacon` 广播数据，并开始广播。
[示例代码](https://github.com/MiEcosystem/mijia_ble_secure/blob/c8fdaf4daedc412c7af1fdfa39fbd59e9874d27e/main.c#L521-L561)

3.实现 C 标准库 `time` 相关函数，初始化系统时间 。`time()` 可参考以下实现:
[示例代码](https://github.com/MiEcosystem/mijia_ble_secure/blob/c8fdaf4daedc412c7af1fdfa39fbd59e9874d27e/time.c#L22-L40)

4.根据米家认证芯片硬件连线，配置 IIC 引脚，实现电源管理回调，进行 `mi_scheduler` 初始化，恢复绑定信息。
[示例代码](https://github.com/MiEcosystem/mijia_ble_secure/blob/c8fdaf4daedc412c7af1fdfa39fbd59e9874d27e/main.c#L823-L830)

5.调用 `mi_service()` 创建米家服务，`lock_service()` 创建门锁服务，`stdio_service_init()` 创建透传服务。并注册相应的回调函数。
[示例代码](https://github.com/MiEcosystem/mijia_ble_secure/blob/c8fdaf4daedc412c7af1fdfa39fbd59e9874d27e/main.c#L832-L838)

6.打开米家APP，开启蓝牙，发现附近的设备，点击“小米蓝牙安全开发板”。在连接过程中出现6位数字PIN码，在固件上输入这6个数字（小米蓝牙安全开发板需要在Segger JLink RTT Viewer环境下输入）。登陆成功后，APP会有相应提示。进入插件，点击“开关闪灯”可以模拟开锁和关锁，同时开发板会广播带有相应事件Object的MiBeacon。这时也可以看到开发板上LED灯有响应。同时RTT Viewer也可以打印出开发板log。

7. 重置设备，目前需要两步
    - APP删除设备
    - 固件上长按重置按钮

已支持芯片平台列表：其他分支仅供米家内部测试使用


## 米家高安全级认证库介绍

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

米家高安全级BLE接入必须使用米家认证芯片，第二代认证芯片MJA1有两款供客户选择，主要参数区别如下：

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
| 工作温度范围（摄氏度）   | -40 ~ +85  | -40 ~ +105 |
| ESD                      | 8KV（HBM） | 5KV（HBM） |
| 包装                     | 卷带包装   | 卷带包装   |
| 安全认证                 | 国内EAL4+  | 国际EAL5+  |

两颗芯片pin-to-pin，主要应用区别是温度范围和认证不同。出国内市场产品建议量产使用MJA1-HCIW，海外市场建议使用MJA1-SXIW。请根据产品定义与销售区域选择合适产品。

请详细阅读MJA1规格书，按照参考电路设计使用。米家一次可免费提供20PCS芯片样品、2块开发板，供开发调试使用。更多需求请与小米供应链联系采购。供应链联系方式：刘亚妹 liuyamei@xiaomi.com

## FAQ

Q: 有问题怎么办？

A: 关于产品定义或[小米IoT开发者平台](https://iot.mi.com/)的问题，请联系米家产品经理。技术问题请区分是芯片开发的问题还是米家接入的问题。如果是芯片开发的问题，请联系厂商，如果是米家接入的问题，请搜索[米家高安全级接入示例demo](https://github.com/MiEcosystem/mijia_ble_secure)相关issue，看是否有类似的问题。如果没有，请提交新issue。

 Q: 如何查看log？

 A: 设备端查看log，请下载Segger JLink RTT Viewer。连接成功可直接查看log。APP端首先需安装[debug apk](https://github.com/MiEcosystem/NewXmPluginSDK/blob/master/%E7%B1%B3%E5%AE%B6%E8%B0%83%E8%AF%95APK%E4%B8%8B%E8%BD%BD%E5%9C%B0%E5%9D%80.md)。然后查找文件管理 -> 手机 -> Android -> data -> com.xiaomi.smarthome -> files -> log -> miio-bluetooth log
