# Airspy 硬件 IO 引脚分布

本文档详细描述了 Airspy 软件定义无线电 (SDR) 设备的 IO 引脚分布，用于 PCB 逆向工程。Airspy 基于 NXP LPC4370 微控制器，该微控制器包含 ARM Cortex-M4 和 Cortex-M0 内核。

## 系统架构概述

Airspy 硬件由以下主要组件组成：

1. **LPC4370 微控制器** - 主处理器，包含 M4 和 M0 内核
2. **R820T 调谐器** - 用于 RF 接收
3. **SI5351C 时钟发生器** - 为系统提供时钟信号
4. **W25Q80BV SPI 闪存** - 用于固件存储
5. **USB 接口** - 用于与计算机通信
6. **LED 和控制引脚** - 用于状态指示和控制

## 引脚分布详情

### 关键控制引脚

| 引脚功能 | LPC4370 引脚 | 描述 |
|---------|-------------|------|
| R820T 电源控制 | P1_14 (GPIO1[7]) | 控制 R820T 调谐器的电源 |
| BIAST 电源控制 | P2_13 (GPIO1[13]) | 控制天线偏置电源 |
| LED1 控制 | P1_17 (GPIO0[12]) | 控制状态 LED |

### I2C 接口

#### I2C0 - 用于 SI5351C 时钟发生器

| 功能 | LPC4370 引脚 | 描述 |
|-----|-------------|------|
| I2C0 SDA | D6（I2C0专用引脚） | SI5351C 数据线 |
| I2C0 SCL | E6（I2C0专用引脚） | SI5351C 时钟线 |

#### I2C1 - 用于 R820T 调谐器

| 功能 | LPC4370 引脚 | 描述 |
|-----|-------------|------|
| I2C1 SDA | P2_3 | R820T 数据线 |
| I2C1 SCL | P2_4 | R820T 时钟线 |

### SPI 接口 - 用于 W25Q80BV 闪存

| 功能 | LPC4370 引脚 | 描述 |
|-----|-------------|------|
| SSP0 MISO | P3_6 | 主机输入，从机输出 |
| SSP0 MOSI | P3_7 | 主机输出，从机输入 |
| SSP0 SCK | P3_3 | 时钟信号 |
| SSP0 SSEL | P3_8 (GPIO5[11]) | 片选信号 |
| FLASH_HOLD | P3_4 (GPIO1[14]) | 闪存保持信号 |
| FLASH_WP | P3_5 (GPIO1[15]) | 闪存写保护信号 |

### 时钟信号

| 功能 | LPC4370 引脚 | 描述 |
|-----|-------------|------|
| 外部时钟输入 | PF_4 | 来自 SI5351C 的时钟输入 (CGU_SRC_GP_CLKIN) |

### ADC 接口

LPC4370 包含高速 ADC (ADCHS)，用于采样来自 R820T 调谐器的 IF 信号。具体引脚未在代码中明确指定，但 ADCHS 通过 DMA 将数据传输到内存。

### SI5351C 时钟配置

SI5351C 时钟发生器配置如下：
- CLK0 -> R820T 时钟 (XTAL_I)
- CLK1 -> LPC4370 RTC 32KHz
- CLK2 -> SGPIO 时钟 (未来使用)
- CLK7 -> LPC4370 主时钟

### 引导配置引脚

| 功能 | LPC4370 引脚 | 描述 |
|-----|-------------|------|
| BOOT0 | P1_1 (GPIO0[8]) | 引导配置 |
| BOOT1 | P1_2 (GPIO0[9]) | 引导配置 |
| BOOT2 | P2_8 (GPIO5[7]) | 引导配置 |
| BOOT3 | P2_9 (GPIO1[10]) | 引导配置 |

## 系统工作原理

1. **多核架构**：
   - M4 内核负责高速 ADC 采样和数据处理
   - M0 内核负责 USB 通信和命令处理

2. **信号路径**：
   - 天线信号 -> R820T 调谐器 -> IF 信号 -> LPC4370 ADCHS -> 数据处理 -> USB 传输

3. **时钟系统**：
   - SI5351C 为 R820T 和 LPC4370 提供时钟信号
   - 系统支持多种采样率配置

4. **电源控制**：
   - 软件可控制 R820T 电源和天线偏置电源

## 硬件版本

代码支持两种硬件版本：
1. **Airspy NOS** - 包含 R820T 和 SI5351C
2. **Airspy MINI** - 仅包含 R820T

## 注意事项

1. 部分引脚在代码中未明确指定，可能需要通过硬件检查确认
2. 系统使用复杂的时钟配置，确保时钟信号正确连接
3. 逆向工程时应特别注意 RF 信号路径的阻抗匹配

## 参考资料

本文档基于对 Airspy 固件代码的分析，包括以下关键文件：
- common/airspy_core.c
- common/airspy_core.h
- common/r820t.c
- common/si5351c.c
- airspy_m4/airspy_m4.c
- airspy_m0/airspy_m0.c

更多信息请参考 Airspy 官方文档和 GitHub 仓库：https://github.com/airspy/firmware
