# EMS软件设计书学习指标

## 常用缩写称呼

- EMS  Energy management system 能源调度系统

- PE Power Electronics 电力电子设备

- BMS Battery Management System 电池管理系统 

- 从DSP 离并网切换控制器

- aGate EMS 配电盘，智能负载的集合

- aPower BMS+电池

- MPPT Max Power Point Tracking 最大功率点追踪软件（aPower2.0S才有）：实现与PE及MSA-C的信息交互，实现PV能量以最大功率流入aPower，实现光伏功率调度，实现buck-boost变换器的控制，实现自身故障保护，如漏电流保护、拉弧保护、绝缘阻抗过低保护、光伏过、欠压及光伏反接保护等；

- | Abbreviations缩略语 | Full spelling 英文全名         | Chinese explanation 中文解释             |
  | ------------------- | ------------------------------ | ---------------------------------------- |
  | FHP                 | Franklin Home Power            | 富兰瓦时储能系统                         |
  | MSA                 | Meter Socket Adapter           | 电表插座适配器                           |
  | MSA-C               | MSA Controller                 | MSA控制器                                |
  | CMS                 | Cloud management system        | 云平台软件                               |
  | EMS                 | Energy Management System       | 能量管理系统                             |
  | PCS                 | Power Conversion System        | 储能变换系统                             |
  | V2L                 | Vehicle-to-Load                | 电动汽车交流输出供载                     |
  | PV                  | Polar Voltage                  | 光伏板或太阳能电池板                     |
  | MDP                 | Main Distribution Panel        | 主配电盘                                 |
  | SCI                 | Serial Communication Interface | 串行通信接口                             |
  | MPU                 | Micro Processor Unit           | 微处理器单元                             |
  | OS                  | Operating System               | 操作系统                                 |
  | PE                  | Power Electronics              | 电力电子设备，这里指aPower的功率变换部分 |
  | BMS                 | Battery Management System      | 电池管理系统                             |
  | BMU                 | Battery Management Unit        | 电池管理单元（主控）                     |
  | SOC                 | State of Charge                | 电池荷电状态                             |
  | SOH                 | State of health                | 电池健康状态                             |
  | CAN                 | controller area network        | 控制器局域网络                           |
  | SPI                 | Serial Peripheral Interface    | 串行外设接口                             |
  | SCI                 | Serial Communication Interface | 串行通信接口                             |
  | RS485               | Recommended Standard 485       | 通信设备间采用差分信号传输的SCI          |
  | ARM                 | Advanced RISC Machine          | ARM处理器                                |
  | DSP                 | Digital Signal Processor       | 数字信号处理器                           |
  | PV                  | Polar Voltage                  | 光伏板或太阳能电池板                     |
  | AFE                 | Analog Front End               | 模拟前端                                 |

## 三个业务软件分工

- mian  ems控制主程序
- AWS  ems与云平台通讯
- 本地APP 本地调试程序

## 设备联网的三种方式

- wifi
- 4G
- 网线

## 三大调度模式

### 自发自用

* 主要特征式通过存储光伏减少从市电取电，充分利用光伏发电给家庭负载供电
* 并网情况下，光伏优先供载，多余的能量充电；光伏能量不足以供载时，电池放电补充，尽量少从市电取电和馈网。在预留SOC设置为100%时，可以当做备电模式使用，可以根据用户设置对aPower进行补电。

### TOU

- 根据用户设置的峰平谷时间段来进行不同的调度，帮助用户节省电费。主要用于峰平谷差价较大的情况。
- 具体策略见p38页

### 紧急备电

- 设备可以从市电和光伏取电给aPower充电，将电池SOC保存在较高水平（100%）。优势是可以最大限度保证用户家庭用电。

## EMS和PE，BMS的通讯方式

- 均为CAN总线通讯，采用重新定义的29位扩展帧数据格式。
- EMS与BMS通讯有两种方式，方案①只与PE组成内部CAN，与EMS通讯需通过PE转发，PE在内部CAN上收到BMS发送的数据报文后，要向系统CAN转发。PE在系统CAN上收到目标地址的低6位与之匹配时，也要向内部CAN转发；l 方式②直接连接系统CAN，不需PE转发报文。

## EMS与从DSP的通讯方式

- 通讯方式为UART，参考规范**INFINITE_IBG设备DSP 与ARM_串口通信协议** 

- 通讯波特率暂定19200 主从工作模式，大端模式，主机（Host）发送数据帧到从机(Slave)，从机(Slave)收到数据帧后返回响应帧，在通信超时时间内，主机(Host)接收不到从机(Slave)响应帧或者响应帧错误，认为本次通信失败。

## CAN协议文档

- 协议数据单元(PDU)，扩展帧29位重新定义 优先级 保留位 PDU功能码，目的地址，源地址和数据域。
- 地址分配：用于保证消息标识符的唯一性以及表面消息的来源。目的地址分配方式，高4bit得到设备类型，低4bit得到设备多个设备站号。源地址分配与目标地址分配相同，但广播帧不作为源地址。
- 通讯策略：同一台INFINITE设备中PE和BMS的地址是关联的，相差0x40(64)。
- 各类设备控制字和命令字格式

## 从DSP串口协议文档

- 描述串口通讯配置，协议基本数据帧格式

# 学习指标

## 自发自用模式调度策略，紧急备电模式策略描述

- 同上，详见P37页

## CAN协议文档中，EMS,PE,BMS地址如何表示

- 通过扩展帧的bit15~bit12确定设备类型，x000表示PE，x100表示BMS，0x3D表示EMU

## 描述编址的过程和结果，为什么需要编址

- 同上 详见CAN协议27页

## 系统SOC的计算

- 用户可见SOC即为系统可用SOC，需要把电池可用SOC映射到系统可用SOC。
- User =Sum*100% / (95% * num)

## 文档中提到的光伏有哪些

| ***\*名称\**** | ***\*意义\****          | ***\*备注\****       |
| -------------- | ----------------------- | -------------------- |
| AGATE_PV1      | Agate光伏1              | 每个agate必备        |
| AGATE_PV2      | Agate光伏2              | 澳洲设备             |
| APBOX_LOAD1    | aPbox负载侧光伏1        | 美标设备             |
| APBOX_LOAD2    | aPbox负载侧光伏2        | 美标设备             |
| APBOX_SUB      | aPbox分路接入到aGate    | 美标设备             |
| APBOX_LINE     | Apbox市电侧光伏         | 美标设备，不需要控制 |
| METER_KIT      | NEM+光伏占用APbox485    | 美标设备，不需要控制 |
| DSP_NEM        | NEM+光伏使用DSP内置电表 | 美标设备，不需要控制 |

## 计算负载功率

## 已知光伏功率，计算光伏电量

# 进阶指标

## 描述黑启动

- 在系统不能稳定供电的情况下，仅靠光伏启动，储能系统先启动为光伏逆变器提供电源，然后光伏设备逐渐接入，带动其他负载启动。

## 油机为什么不能跟交流光伏并存

- 光伏不具备调节频率的能力，如果并存，由于负载变化，造成倒灌。

## 接入油机后，为什么aPower只能充电不能放电

- 防止电流倒灌，油机和电池同时放电的话，当负载发生突变，多余的能量将流向油机，造成损害

