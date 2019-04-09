# 普通BLE接入开发引导

平台提供BLE标准认证库，开发者能通过使用标准lib将设备接入平台。

设备需实现MiBeacon协议，并通过标准认证流程完成设备绑定功能。对于产品自身功能，需开发者自行开发。

### BLE SDK结构介绍

米家BLE SDK结构如下:

1. BLE芯片+协议栈：（开发者无需关心）

   由芯片原厂提供SDK及开发环境，并提供技术支持。

2. BLE通用接口：（开发者无需关心）

   BLE通用接口是由米家制定的、与平台无关的抽象接口，目的是为了屏蔽不同BLE芯片SDK的底层接口的差异，使得米家BLE应用可以无缝的运行在各家芯片SDK上。

   米家负责制定BLE通用接口的API名称、参数、功能，芯片原厂或者芯片代理公司负责实现。适配完成后米家会经过测试并发布在支持列表中。

3. 米家BLE应用：（开发者关心）

   米家负责实现米家BLE应用层功能，并以lib库的形式提供给开发者。开发者需在平台提供的芯片支持列表中，根据情况选择合适的芯片平台接入。

### 米家标准认证库说明

需要接入到米家系统的ble设备，连接米家app时需要先经过认证，mible_std_authen库就是用来完成这个功能，mible_std_authen_xxxx.lib 库会在蓝牙设备端建立Mi Service服务，用于认证交互。该库适用于普通安全等级的米家接入设备。

| 函数名                      | 功能                         | 说明                                                         |
| --------------------------- | ---------------------------- | ------------------------------------------------------------ |
| mible_server_info_init      | 初始化设备信息               | 初始化一下参数：<br />1、强/弱绑定关系，需与米家开放平台上强弱绑定配置一致；<br />2、产品id，需与米家开放平台上产品ID配置一致;<br />3、版本号（四位数字字符如”1234”） |
| mible_server_miservice_init | 初始化MiService              | 用于初始化MiService，在初始化阶段调用                        |
| mible_service_init_cmp      | MiService初始化完成回调函数 | 当MiService初始化完成，会调用此函数，此函数可以在应用层实现。 |
| mible_connected             | 连接回调函数                 | 当设备与app建立连接，会调用此函数，此函数可在应用层实现。    |
| mible_disconnected          | 断开连接回调函数             | 当设备与app断开连接，会调用                                  |
| mible_bonding_evt_callback  | 认证结果回调函数             | 设备第一次与米家app连接时，进入绑定流程，此函数返回绑定成功或失败的结果；绑定成功后(设备输入手机app的账号名下)，设备每次与app建立连接，都进行登录流程，此函数返回登录成功或失败的结果。【注】建立连接后，在未绑定或登录成功前，设备端不能允许任何对设备的操作，如读写特性等。 |

此外在mible_server.h文件中，还有一些参数可供应用层修改：

| 宏定义                              | 说明           |
| ----------------------------------- | -------------- |
| MIBLE_STD_SERVER_CONN_MAX_INTERVAL  | 最大连接间隔   |
| MIBLE_STD_SERVER_CONN_MIN_INTERVAL  | 最小连接间隔   |
| MIBLE_STD_SERVER_CONN_SLAVE_LATENCY | 从设备连接延迟 |
| MIBLE_STD_SERVER_CONN_SUP_TIMEOUT   | 连接超时时间   |

### 调用实例说明

示例Demo：https://github.com/MiEcosystem/mijia_ble 。

Demo中展示了如何使用米家蓝牙标准认证库，具体使用流程如下：

1. 完成必要的初始化工作，如蓝牙协议栈初始化、flash记录初始化、timer初始化等，使得米家通用蓝牙接口可以正常工作；
2. 调用 mible_server_info_init 函数，完成产品信息的初始化配置；
3. 调用 mible_server_miservice_init 函数，初始化用于米家认证 Service 和 characteristic。  Service UUID 为 0xFE95。

运行认证库的蓝牙设备为slave，当手机米家app作为master连接蓝牙设备时，会触发认证流程。 当设备第一次建立连接时，触发绑定流程。若绑定成功，则此后建立连接都会触发登录流程。建立连接后，若未成功进行绑定或登陆，超时10s后会自动断开连接。认证库开放给应用层四个回调函数，用户可以在回调函数中添加自己的应用层代码，四个回调函数如下：

1. mible_service_init_cmp 认证用的service 初始化完成回调函数；
2. mible_connected 建立连接回调函数；
3. mible_disconnected 断开连接回调函数；
4. mible_bonding_evt_callback 认证结果回调函数。

在绑定成功后，会将设备认证的相关信息存储在flash中。如果flash存储不成功会导致此后的的登录流程不成功。


***

# 高安全级BLE接入开发引导

BLE SDK结构与普通BLE接入类似，只是由米家高安全级认证库代替米家标准认证库。高安全级认证库中集成了IIC驱动用以完成于安全芯片的交互。

### 米家高安全级认证库说明

| 函数名                      | 功能                         | 说明                                                         |
| --------------------------- | ---------------------------- | ------------------------------------------------------------ |
|mi_service_init|创建小米服务|调用mi_service_init即可完成小米服务创建|
|mi_scheduler_init|初始化调度器|初始化时，需要传入调度器工作间隔，回调函数，若由安全芯片，还需要传入芯片电源管理函数以及IIC引脚信息|
|mi_scheduler_process|调度器处理函数|在main loop或者task执行|
|mi_session_encrypt|加密函数|登陆成功后，设备和手机之间建立了加密通道，mi_session_encrypt负责对传输数据加密|
|mi_session_decrypt|解密函数|负责对传输数据解密|
|mibeacon_obj_enque|设备状态/事件信息上报|搭配蓝牙网关使用，开发者调用mibeacon_obj_enque将要发送的object放入发送队列，之后libs自动发送带有object的mibeacon|

### 调用实例说明

高安全级接入示例Demo： https://github.com/MiEcosystem/mijia_ble_secure 。
