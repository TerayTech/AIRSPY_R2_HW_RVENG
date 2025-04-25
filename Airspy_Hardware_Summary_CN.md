# Airspy 硬件架构概述

## 简介

Airspy 是一款高性能软件定义无线电 (SDR) 接收器，基于 NXP LPC4370 微控制器和 Rafael Micro R820T 调谐器。本文档提供了 Airspy 硬件架构的概述，作为详细文档的入口指南。

## 文档索引

本项目包含以下文档：

1. **Airspy_Hardware_Summary_CN.md** - 本文档，提供概述和索引
2. **Airspy_IO_Pin_Distribution_CN.md** - 基本 IO 引脚分布信息
3. **Airspy_IO_Pin_Distribution_Detailed_CN.md** - 详细的 IO 引脚和系统配置信息
4. **Airspy_Block_Diagram_CN.md** - 系统框图和可视化图表

## 硬件规格

- **微控制器**：NXP LPC4370（ARM Cortex-M4/M0/M0+多核处理器）
- **RF 调谐器**：Rafael Micro R820T
- **频率范围**：24MHz - 1.8GHz
- **采样率**：可配置，最高可达 10MSPS
- **分辨率**：12位 ADC
- **接口**：USB 2.0 高速
- **闪存**：W25Q80BV 1MB SPI 闪存
- **时钟源**：SI5351C 可编程时钟发生器

## 硬件版本

Airspy 有两种主要硬件版本：

1. **Airspy NOS** - 包含 R820T 和 SI5351C
2. **Airspy MINI** - 仅包含 R820T，时钟配置不同

## 系统架构亮点

### 多核处理

- **M4 内核**：负责高速 ADC 采样和数据处理
- **M0 内核**：负责 USB 通信和命令处理
- 核间通信通过共享内存和事件触发机制实现

### 关键 IO 引脚

- **R820T 电源控制**：P1_14 (GPIO1[7])
- **天线偏置电源**：P2_13 (GPIO1[13])
- **状态 LED**：P1_17 (GPIO0[12])
- **I2C1 接口**（R820T）：P2_3 (SDA) 和 P2_4 (SCL)
- **外部时钟输入**：PF_4 (来自 SI5351C)

### 时钟系统

- 复杂的 PLL 系统，支持多种采样率
- SI5351C 为系统提供多路时钟信号
- 动态时钟配置，根据工作模式优化性能和功耗

### 信号路径

1. 天线输入 → R820T 调谐器 → IF 信号 → LPC4370 ADCHS
2. ADCHS → DMA → 内存缓冲区 → 数据处理 → USB 传输

## 逆向工程注意事项

1. **RF 信号路径**：需要特别注意阻抗匹配和信号完整性
2. **时钟配置**：时钟信号对系统性能至关重要
3. **电源时序**：R820T 和其他组件有特定的电源时序要求
4. **未明确指定的引脚**：部分引脚在代码中未明确指定，可能需要通过硬件检查确认

## 参考资料

- Airspy 官方文档：https://airspy.com/
- GitHub 仓库：https://github.com/airspy/firmware
- LPC4370 数据手册：https://www.nxp.com/
- R820T 数据手册：http://superkuh.com/rtlsdr.html

## 后续步骤

1. 查阅 **Airspy_IO_Pin_Distribution_CN.md** 获取基本引脚分布
2. 参考 **Airspy_IO_Pin_Distribution_Detailed_CN.md** 了解详细系统配置
3. 使用 **Airspy_Block_Diagram_CN.md** 中的图表理解系统架构
4. 根据这些文档进行 PCB 逆向工程设计
