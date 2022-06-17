---
title: 闸机开发设备调试-Android人脸机
categories: -技术
tags: 
- Android
- Kotlin
- 硬件
date: 2022-05-12 17:43
---

# 闸机开发设备调试-Android人脸机

Android人脸机控制闸机需要接入主控同时接入单片机进行开发

## Android设备接口

- DC12V电源

- USB接口

- 网口
- 开闸信号
- 4口端子

  1. 12V

  2. GND

  3. WG-D1输入

  4. WG-D输入

- 5口端子
  1. 232-RX
  2. 232-TX
  3. WG-D1输出
  4. WG-D0输出
  5. GND

## 接线

### 开发调试

Android人脸设备通电，连接USB口

### 开闸


Android人脸设备->单片机

- 开闸接入单片机K1和GND

  

### 过闸

过闸信号由闸机反馈给人脸设备，通过转接模块转接入RS232接口

**闸机接转接模块：**

- 引脚①12V接入转换模块VCC，

- 引脚④GND计入转接模块GND

- 引脚⑤左计数接入转接模块IN1

- 引脚⑥右计数接入转接模块IN2    

**转接模块接Android设备**

- com ②TXD 接入5口端子的②RS232-TX
- com③RXD接入5口端子的①RS232-RX
- com⑤GND接入5口端子的⑤GND







