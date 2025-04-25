# Airspy 硬件系统框图

以下是 Airspy 软件定义无线电 (SDR) 设备的系统框图，展示了主要组件及其互连关系。

## 系统总体框图

```mermaid
graph TD
    ANT[天线输入] --> |RF信号| BIAS[偏置电源<br>P2_13控制]
    BIAS --> R820T[R820T调谐器<br>P1_14控制电源]
    
    SI5351C[SI5351C<br>时钟发生器] --> |CLK0| R820T
    SI5351C --> |CLK1| RTC[LPC4370 RTC]
    SI5351C --> |CLK7| CLKIN[LPC4370<br>GP_CLKIN<br>PF_4]
    
    R820T --> |IF信号| ADCHS[LPC4370<br>高速ADC]
    
    subgraph LPC4370
        CLKIN --> PLL[PLL系统]
        PLL --> |PLL1| CPU[M4/M0内核]
        PLL --> |PLL0AUDIO| ADCHS
        PLL --> |PLL0USB| USB[USB控制器]
        
        CPU --> |控制| I2C0[I2C0]
        CPU --> |控制| I2C1[I2C1]
        CPU --> |控制| SPI[SPI接口]
        CPU --> |控制| GPIO[GPIO控制]
        
        ADCHS --> |DMA| MEM[内存缓冲区]
        MEM --> USB
    end
    
    I2C0 --> SI5351C
    I2C1 --> R820T
    SPI --> FLASH[W25Q80BV<br>SPI闪存]
    GPIO --> LED[状态LED<br>P1_17]
    
    USB --> PC[计算机]
```

## 多核处理架构

```mermaid
graph TD
    subgraph LPC4370
        M4[Cortex-M4内核<br>主处理器] --> |共享内存| M0[Cortex-M0内核<br>USB处理]
        M4 <--> |事件触发| M0
        
        M4 --> ADCHS[高速ADC控制]
        M4 --> DMA[DMA控制]
        M4 --> DSP[数据处理<br>打包]
        
        M0 --> USB[USB通信]
        M0 --> CMD[命令处理]
        M0 --> R820T[R820T控制]
    end
```

## 时钟系统

```mermaid
graph TD
    XTAL[12MHz晶振] --> |启动时| CGU[时钟生成单元]
    SI5351C[SI5351C] --> |CLK7| GPCLKIN[GP_CLKIN<br>PF_4]
    GPCLKIN --> CGU
    
    CGU --> PLL0USB[PLL0USB<br>480MHz]
    CGU --> PLL0AUDIO[PLL0AUDIO<br>可变频率]
    CGU --> PLL1[PLL1<br>可变频率]
    
    PLL0USB --> USBCLK[USB时钟]
    PLL0AUDIO --> ADCCLK[ADCHS时钟]
    
    PLL1 --> M4CLK[M4内核时钟]
    PLL1 --> PERIPHCLK[外设时钟]
    PLL1 --> APB1[APB1总线<br>I2C0]
    PLL1 --> APB3[APB3总线<br>I2C1]
```

## 信号处理流程

```mermaid
graph LR
    ANT[天线] --> R820T[R820T调谐器]
    R820T --> |IF信号| ADCHS[高速ADC]
    ADCHS --> |DMA| BUFFER[内存缓冲区]
    BUFFER --> PACK[可选数据打包]
    PACK --> USB[USB传输]
    USB --> PC[计算机]
```

## 电源控制系统

```mermaid
graph TD
    USB[USB电源] --> MAIN[主电源]
    MAIN --> MCU[LPC4370]
    MAIN --> CLKGEN[SI5351C]
    
    MCU --> |P1_14| R820T_PWR[R820T电源控制]
    R820T_PWR --> |高电平开启| R820T[R820T调谐器]
    
    MCU --> |P2_13| BIAS_PWR[天线偏置电源控制]
    BIAS_PWR --> |高电平开启| BIAS[天线偏置]
    
    MCU --> |P1_17| LED_CTRL[LED控制]
    LED_CTRL --> |高电平点亮| LED[状态LED]
```

## 注意事项

1. 图中仅显示主要组件和连接，实际硬件可能包含更多细节
2. 部分引脚连接在代码中未明确指定，可能需要通过硬件检查确认
3. 时钟配置可根据不同的采样率和工作模式动态调整
4. 多核处理架构使用共享内存和事件触发机制进行通信
