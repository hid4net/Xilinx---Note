
Xilinx 7系列FPGA总结：高速收发器结构
---
相关文档:
分类      | 文档名称                                                       | 文档编号
:-------- | :------------------------------------------------------------- | :-------
基本结构  | Configuration User Guide                                       | UG470
^         | SelectIO Resources User Guide                                  | UG471
^         | Clocking Resources User Guide                                  | UG472
^         | Memory Resources User Guide                                    | UG473
^         | Configurable Logic Block User Guide                            | UG474
^         | DSP48E1 Slice User Guide                                       | UG479
收发器    | **GTX/GTH Transceivers User Guide**                           | UG476
^         | **GTP Transceivers User Guide**                               | UG482
ADC       | XADC Dual 12-Bit 1 MSPS Analog-to-Digital Converter User Guide | UG480
封装和PCB | Packaging and Pinout Product Specification                     | UG475
^         | PCB Design Guide                                               | UG483


--------------------------------------------------------------------------------
# 1. Overview
支持：
* PCI-E 1.1/2.0/3.0
* 10GBASE-R
* SATA
* 其他

## 1.1. 基本结构

### 1.1.1. Quad
![GTX Transceiver Quad Configuration](/assets/GTX%20Transceiver%20Quad%20Configuration.png)

### 1.1.2. GTXE2_Channel
![GTXE2_CHANNEL Primitive Topology](/assets/GTXE2_CHANNEL%20Primitive%20Topology.png)

--------------------------------------------------------------------------------
# 2. 共性

## 2.1. 时钟输入结构
![Reference Clock Input Structure](/assets/Reference%20Clock%20Input%20Structure.png)

## 2.2. 时钟选择
* 可用时钟：
    1. Local RefClk Pin pairs：GTREFCLK0 or GTREFCLK1
    1. 上边的Quad的时钟：GTSOUTHREFCLK0 or GTSOUTHREFCLK1
    1. 下边的Quad的时钟：GTNORTHREFCLK0 or GTNORTHREFCLK1
* 注意：FPGA内部逻辑时钟不能用于GTX/GTH,只保留用作内部测试

## 2.3. CPLL
* 频率范围较低；
* GTX：1.6 GHz to 3.3 GHz
* GTH：1.6 GHz to 5.16 GHz

## 2.4. QPLL
* 频率范围较宽；
* GTX：
    * Lower Band 5.93–8.0 GHz
    * Upper Band 9.8–12.5 GHz
* GTH
    * 8.0-13.1 GHz

## 2.5. 复位和初始化
* 自动顺序复位：
    * 状态机控制，自动执行下一单元复位
    * 复位完成后
        * TXRESETDONE
        * RXRESETDONE
* 手动局部复位

## 2.6. 低功耗
* 控制信号：
    * TXPD
    * RXPD
    * CPLLPD
    * QPLLPD

## 2.7. Loopback
* Near-end：
    * Near-End PMA
    * Near-End PCS
* Far-end
    * Far-End PMA
    * Far-End PCS

## 2.8. DRP

## 2.9. Digital Monitor

--------------------------------------------------------------------------------
# 3. Transmitter

## 3.1. TX Interface
* 时钟域：TXUSRCLK2
* 相关时钟：TXUSRCLK
    * PCS时钟
    * 关系与内部-外部数据宽度有关

## 3.2. TX 8B/10B Encoder
* 用于DC-balance

## 3.3. TX Gearbox
* 64B/66B and 64B/67B 编码

## 3.4. TX Buffer和TX Buffer Bypass
* 在PCS（时钟域 TXUSRCLK）和PMA（时钟域 XCLK）之间
* 推荐使用TX Buffer
* 和TX Buffer Bypass相对，只能二选一
    * 高级特性，需要额外逻辑和时钟约束
    * 低延迟

## 3.5. TX Pattern Generator
* 产生伪随机序列
    * PRBS-7 1 + X6 + X7
    * PRBS-15 1 + X14 + X15
    * PRBS-23 1 + X18 + X23
    * PRBS-31 1 + X28 + X31
    * PCI Express Compliance Pattern
    * Square Wave with 2 UI period
    * Square Wave with 16 UI, 20 UI, 32 UI or 40 UI period
* 与RX Pattern Checker匹配

## 3.6. TX Polarity Control
* 控制信号：TXPOLARITY

## 3.7. TX Fabric Clock Output Control

## 3.8. TX Phase Interpolator PPM Controller
* 相位微调

## 3.9. TX Configurable Driver
* Differential voltage control
* Pre-cursor and post-cursor transmit pre-emphasis
* Calibrated termination resistors

## 3.10. TX Receiver Detect
* PCI-E需要

## 3.11. TX Out-of-Band
* SATA需要

--------------------------------------------------------------------------------
# 4. Receiver


## 4.1. RX Analog Front End
* Configurable RX termination voltage
* Calibrated termination resistors

## 4.2. RX Out-of-Band
* SATA需要

## 4.3. RX Equalizer
* 均衡器：与信道衰减、反射、预加重等有关
* LPM模式：
    * 功耗低，但功能简单
    * 适用于
    * 线速率低于11.2 GHz
    * 且 段路径
    * 且 奈奎斯特频率信道损失≤ 12 dB
* DFE模式：
    * 复杂
    * 适用于
    * 中长路径
    * 信道损失 ≥ 8 dB

## 4.4. RX CDR
* 时钟和数据恢复

## 4.5. RX Fabric Clock Output Control


## 4.6. RX Margin Analysis
* 接收信号质量、眼图分析

## 4.7. RX Polarity Control
* 控制信号：RXPOLARITY

## 4.8. RX Pattern Checker
* 检查随机序列，与TX Pattern Generator匹配

## 4.9. RX Byte and Word Alignment
* Comma对齐

## 4.10. RX 8B/10B Decoder
* 用于DC-balance

## 4.11. RX Elastic Buffer和RX Buffer Bypass
* 在PCS（时钟域 RXUSRCLK）和PMA（时钟域 XCLK）之间
* 推荐使用RX Elastic Buffer

## 4.12. RX Clock Correction


## 4.13. RX Channel Bonding
* 多个Lane绑定：
    * XAUI
    * PCI-E

## 4.14. RX Gearbox
* 64B/66B and 64B/67B解码

## 4.15. RX Interface
* 时钟域：RXUSRCLK2
* 与RXUSRCLK相关：关系与内部-外部数据宽度有关

--------------------------------------------------------------------------------
# 5. 电路设计指导

## 5.1. Pins
* 时钟输入：
    * MGTREFCLK0P，MGTREFCLK0N
    * MGTREFCLK1P，MGTREFCLK1N
* 数据输入：
    * MGTXRXP[3:0]/MGTXRXN[3:0]
    * MGTHRXP[3:0]/MGTHRXN[3:0]
* 数据输出：
    * MGTXTXP[3:0]/MGTXTXN[3:0]
    * MGTHTXP[3:0]/MGTHTXN[3:0]
* 终端电阻校正偏压
    * MGTAVTTRCAL
* 终端电阻校正参考电压
    * MGTRREF
* Quad内部模拟电路电源
    * MGTAVCC
    * 1.0 V
* 收发器模拟电源
    * MGTAVTT
    * 1.2 V
* QPLL辅助电源
    * MGTVCCAUX
    * 1.8 V

## 5.2. 参考时钟
* AC耦合
    * LVDS
    * LVPECL

## 5.3. 供电和滤波


## 5.4. PCB设计检查列表


