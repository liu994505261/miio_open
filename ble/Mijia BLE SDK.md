# Mijia BLE SDK

## 简介
Mijia BLE SDK是在蓝牙芯片原厂SDK基础上，加上小米 Profile后封装得到，目的是方便合作伙伴进行二次开发。具有如下功能：
- 支持小米蓝牙广播协议：mi beacon
- 支持 Mi Service，负责智能家庭APP与设备间的安全连接及认证
- 示例工程：Mi Demo，烧录到开发板后可配合小米智能家庭APP直接看效果，对应产品名称：小米蓝牙开发板


## 支持平台：
- Dialog 14580
- NXP 9020 / 9022
- TI CC254X / CC264X
- Nordic 51822
- Marvell 8777


## OTA
关于固件升级，Mijia BLE SDK中没有提供统一的实现，只提供设备的版本号规范。智能家庭APP会根据版本号判断是否有新固件需要升级，具体的升级逻辑由产品自己实现。
 - 版本号示例：1.0.1_16,  sdk版本号_应用版本号，应用版本号不要超过4个字符
 - OTA流程说明
	 1. 智能家庭APP与设备连接后会自动读取设备当前版本号
	 2. 与云端最新版本进行比较，若有新版本则通知 插件
	 3. 由插件来决定什么时候提示用户进行升级

	 
## 安全连接/认证
Mi Serivce在蓝牙基础连接上建立一个安全认证的流程，用于防止中间人攻击以及重放攻击。这一步骤已经内置在SDK中，设备一旦与APP建立了蓝牙连接后，会自动开始安全认证，安全认证的结果会回调到应用层,建议开发者在收到auth_fail的消息后能主动断开蓝牙连接。

```C
int miss_auth_fail_ind_handler(ke_msg_id_t const msgid,
                                    struct miss_authFail_ind const *param,
                                    ke_task_id_t const dest_id,
                                    ke_task_id_t const src_id)
{
    /* TODO: Add application layer handler here */
    app_disconnect();
    LOG_DEBUG("app_task auth fail\r\n");
    return (KE_MSG_CONSUMED);
}

int miss_bond_succ_ind_handler(ke_msg_id_t const msgid,
                                    struct miss_bondSucc_ind const *param,
                                    ke_task_id_t const dest_id,
                                    ke_task_id_t const src_id)
{
    /* TODO: Add application layer handler here */
    LOG_DEBUG("app_task bond succ\r\n");
    return (KE_MSG_CONSUMED);
}
```


## 长连接
有些产品需要跟手机维持长连接，MIUI已经对Mi Profile做了支持，可以在插件中将需要长连接的产品设置到系统中。
