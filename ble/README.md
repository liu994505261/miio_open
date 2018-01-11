# 蓝牙产品接入指南
---

## 目的
本文档为接入小米IoT平台的第三方蓝牙产品提供固件端对接指导。

## 接入方式
- 直接使用，小米BLE模组 
- 产品接入方自选蓝牙芯片，自主设计硬件，通过 [Mijia BLE SDK](https://github.com/MiEcosystem/miio_open/blob/master/ble/Mijia%20BLE%20SDK.md)进行接入

## 接入流程
1. 完成小米开发者账号的注册和资质认证 [小米IoT开发者平台](https://iot.mi.com/index.html)
2. 阅读[小米蓝牙产品设计规约](https://github.com/MiEcosystem/miio_open/blob/master/ble/%E5%B0%8F%E7%B1%B3%E8%93%9D%E7%89%99%E4%BA%A7%E5%93%81%E8%AE%BE%E8%AE%A1%E8%A7%84%E7%BA%A6.md)
3. 登陆小米开放平台，在“产品管理”页面添加 Demo：“小米蓝牙开发板”，进行体验。
4. 根据新产品向导进行功能填写，并自动生成蓝牙Service及对应的Mijia BLE SDK.
5. 固件应用层开发，调试。
6. APP 插件开发，并与设备联调
7. (可选) 服务器逻辑开发，并与小米服务器进行对接
8. 自主测试，提交固件，申请上线




 

