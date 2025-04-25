# Airspy 硬件 IO 引脚分布详细文档

本文档提供了 Airspy 软件定义无线电 (SDR) 设备的详细 IO 引脚分布，用于 PCB 逆向工程。Airspy 基于 NXP LPC4370 微控制器，该微控制器包含 ARM Cortex-M4 和 Cortex-M0 内核。

## 系统架构概述

Airspy 硬件由以下主要组件组成：

1. **LPC4370 微控制器** - 主处理器，包含 M4 和 M0/M0+ 内核
2. **R820T 调谐器** - 用于 RF 接收，频率范围 24MHz-1.8GHz
3. **SI5351C 时钟发生器** - 为系统提供时钟信号
4. **W25Q80BV SPI 闪存** - 1MB 容量，用于固件存储
5. **USB 接口** - 高速 USB 2.0，用于与计算机通信
6. **LED 和控制引脚** - 用于状态指示和控制

## 多核架构

Airspy 使用 LPC4370 的多核架构进行任务分配：

- **M4 内核**：负责高速 ADC 采样、数据处理和打包
- **M0 内核**：负责 USB 通信、命令处理和 R820T 调谐器控制
- **M0+ 内核**：在某些配置中使用（可选）

核间通信通过共享内存和事件触发机制实现。

## 引脚分布详情

### 关键控制引脚

| 引脚功能 | LPC4370 引脚 | 描述 |
|---------|-------------|------|
| R820T 电源控制 | P1_14 (GPIO1[7]) | 控制 R820T 调谐器的电源，高电平开启 |
| BIAST 电源控制 | P2_13 (GPIO1[13]) | 控制天线偏置电源，高电平开启 |
| LED1 控制 | P1_17 (GPIO0[12]) | 控制状态 LED，高电平点亮 |

### I2C 接口

#### I2C0 - 用于 SI5351C 时钟发生器

| 功能 | LPC4370 引脚 | 描述 |
|-----|-------------|------|
| I2C0 SDA | 未在代码中明确指定 | SI5351C 数据线，I2C 地址 0x60 |
| I2C0 SCL | 未在代码中明确指定 | SI5351C 时钟线，频率约 375kHz-400kHz |

#### I2C1 - 用于 R820T 调谐器

| 功能 | LPC4370 引脚 | 描述 |
|-----|-------------|------|
| I2C1 SDA | P2_3 | R820T 数据线，I2C 地址 0x1A |
| I2C1 SCL | P2_4 | R820T 时钟线，频率约 400kHz |

### SPI 接口 - 用于 W25Q80BV 闪存

| 功能 | LPC4370 引脚 | 描述 |
|-----|-------------|------|
| SSP0 MISO | P3_6 | 主机输入，从机输出 |
| SSP0 MOSI | P3_7 | 主机输出，从机输入 |
| SSP0 SCK | P3_3 | 时钟信号 |
| SSP0 SSEL | P3_8 (GPIO5[11]) | 片选信号，低电平有效 |
| FLASH_HOLD | P3_4 (GPIO1[14]) | 闪存保持信号 |
| FLASH_WP | P3_5 (GPIO1[15]) | 闪存写保护信号 |

### 时钟信号

| 功能 | LPC4370 引脚 | 描述 |
|-----|-------------|------|
| 外部时钟输入 | PF_4 | 来自 SI5351C 的时钟输入 (CGU_SRC_GP_CLKIN) |

### ADC 接口

LPC4370 包含高速 ADC (ADCHS)，用于采样来自 R820T 调谐器的 IF 信号。ADCHS 具有以下特性：

- 12位分辨率
- 最高采样率可达 80MSPS
- 通过 DMA 将数据传输到内存
- 支持多种采样率配置
- 支持数据打包模式，提高 USB 传输效率

### SI5351C 时钟配置

SI5351C 时钟发生器配置如下：
- **CLK0** -> R820T 时钟 (XTAL_I)，提供 R820T 的参考时钟
- **CLK1** -> LPC4370 RTC 32KHz，提供实时时钟
- **CLK2** -> SGPIO 时钟 (预留，未来使用)
- **CLK3-6** -> 未使用
- **CLK7** -> LPC4370 主时钟，为微控制器提供主时钟

### 引导配置引脚

| 功能 | LPC4370 引脚 | 描述 |
|-----|-------------|------|
| BOOT0 | P1_1 (GPIO0[8]) | 引导配置位 0 |
| BOOT1 | P1_2 (GPIO0[9]) | 引导配置位 1 |
| BOOT2 | P2_8 (GPIO5[7]) | 引导配置位 2 |
| BOOT3 | P2_9 (GPIO1[10]) | 引导配置位 3 |

### SCU 引脚配置

LPC4370 的 SCU (System Control Unit) 负责引脚功能配置。代码中定义了大量引脚配置，以下是部分关键配置：

```c
#define SCU_CONF_FUNC0 (SCU_CONF_EPUN_DIS_PULLUP | SCU_CONF_FUNCTION0)
#define SCU_CONF_FUNC1 (SCU_CONF_EPUN_DIS_PULLUP | SCU_CONF_FUNCTION1)
#define SCU_CONF_FUNC2 (SCU_CONF_EPUN_DIS_PULLUP | SCU_CONF_FUNCTION2)
#define SCU_CONF_FUNC4 (SCU_CONF_EPUN_DIS_PULLUP | SCU_CONF_FUNCTION4)
#define CLK_IN_FUNC1 (SCU_CLK_IN | SCU_CONF_FUNCTION1)
```

大多数引脚配置为禁用上拉电阻，并设置为特定功能。

## 时钟系统详解

Airspy 使用复杂的时钟系统，主要包括：

### 1. PLL 配置

- **PLL0USB**：配置为 480MHz，用于 USB 接口
- **PLL0AUDIO**：用于 ADCHS 时钟，根据采样率动态配置
- **PLL1**：用于 M4/M0 内核、外设、APB1 和 APB3 总线时钟
  - 低速模式：用于降低功耗
  - 高速模式：用于高性能处理

### 2. 时钟源

- **XTAL**：12MHz 晶振，启动时使用
- **GP_CLKIN**：来自 SI5351C 的外部时钟，作为主要时钟源
- **IRC**：内部 RC 振荡器，12MHz，用于启动

### 3. 时钟分配

- **BASE_M4_CLK**：M4 内核时钟
- **BASE_PERIPH_CLK**：外设时钟
- **BASE_APB1_CLK**：APB1 总线时钟，连接 I2C0
- **BASE_APB3_CLK**：APB3 总线时钟，连接 I2C1
- **BASE_ADCHS_CLK**：ADCHS 时钟，可配置为不同采样率

## 信号路径

Airspy 的信号路径如下：

1. 天线输入 -> (可选 BIAST 偏置) -> R820T 调谐器
2. R820T 调谐器 -> IF 信号 (通常为 4.5MHz 或 5MHz) -> LPC4370 ADCHS
3. ADCHS -> DMA -> 内存缓冲区
4. 数据处理（可选打包）-> USB 传输

## 电源系统

- **R820T 电源**：由 P1_14 控制，高电平开启
- **天线偏置电源**：由 P2_13 控制，高电平开启
- **系统电源管理**：代码中包含多种电源优化措施
  - 未使用的时钟和外设被禁用
  - 支持低功耗模式

## 硬件版本

代码支持两种硬件版本：
1. **Airspy NOS** - 包含 R820T 和 SI5351C
2. **Airspy MINI** - 仅包含 R820T，时钟配置不同

## GPIO 配置表

代码中定义了完整的 GPIO 配置表，包含 161 个引脚配置。以下是部分关键配置：

```c
const gpio_conf_t gpio_conf[GPIO_CONF_NB] =
{
  { P0_0, SCU_CONF_FUNC0 },
  { P0_1, SCU_CONF_FUNC0 },
  ...
  { P1_14, SCU_CONF_FUNCTION0 }, // PIN_EN_R820T
  ...
  { P1_17, SCU_CONF_FUNCTION0 }, // PIN_EN_LED1
  ...
  { P2_3, SCU_CONF_FUNC4 }, // I2C1_SDA
  { P2_4, SCU_CONF_FUNC4 }, // I2C1_SCL
  ...
  { P2_13, SCU_CONF_FUNCTION0 }, // PIN_EN_BIAST
  ...
  { PF_4, CLK_IN_FUNC1 }, // CGU_SRC_GP_CLKIN Clock Input from SI5351C
  ...
}
```

## 固件更新机制

Airspy 支持通过 USB 进行固件更新，使用以下机制：

1. W25Q80BV SPI 闪存用于存储固件
2. 支持扇区擦除和页编程
3. 支持从 ROM 到 RAM 的代码执行

## 注意事项

1. 部分引脚在代码中未明确指定，可能需要通过硬件检查确认
2. 系统使用复杂的时钟配置，确保时钟信号正确连接
3. 逆向工程时应特别注意 RF 信号路径的阻抗匹配
4. R820T 调谐器需要特定的电源时序和初始化
5. SI5351C 时钟配置对系统性能至关重要

## 参考资料

本文档基于对 Airspy 固件代码的详细分析，包括以下关键文件：
- common/airspy_core.c
- common/airspy_core.h
- common/r820t.c
- common/si5351c.c
- airspy_m4/airspy_m4.c
- airspy_m4/adchs.c
- airspy_m0/airspy_m0.c
- airspy_m0/airspy_usb_req.c

更多信息请参考 Airspy 官方文档和 GitHub 仓库：https://github.com/airspy/firmware
