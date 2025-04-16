
<h1 style="text-align:center">Xilinx Ultrascale+系列FPGA: 高速收发器</h1>

--------------------------------------------------------------------------------

| 文档名称                        | 文档编号 |
| :------------------------------ | :------- |
| **GTH Transceivers User Guide** | UG576    |
| **GTY Transceivers User Guide** | UG578    |
| **GTM Transceivers User Guide** | UG581    |

--------------------------------------------------------------------------------
# 1. GTH Transceivers

## 1.1. overview
* GT Quad
    * 每个 Quad 有 4 个 channel 和 1 个 COMMMON
    > ![UG576_Fig1-1](/assets/UG576_Fig1-1.png)
* Channel
    > ![UG576_Fig1-2](/assets/UG576_Fig1-2.png)

## 1.2. Shared Features

### 1.2.1. 参考时钟
* input/output 专用时钟管脚
    * 支持输入、输出模式, 不能动态切换
    * 输入模式
        > ![UG576_Fig2-1](/assets/UG576_Fig2-1.png)
    * 输出模式
        > ![UG576_Fig2-2](/assets/UG576_Fig2-2.png)
        > ![UG576_Fig2-3](/assets/UG576_Fig2-3.png)
* 参考时钟源
    * 本 Quad 的两个专用时钟输入管脚: `GTREFCLK[1:0]`
    * 南 (下) 、北 (上) 两个相邻 Quad 的专用时钟输入管脚
    > ![UG576_Fig2-4](/assets/UG576_Fig2-4.png)

### 1.2.2. PLL
* CPLL
    * 每个 channel 中有 1 个
    * 范围: 2.0 ~ 6.25 GHz
* QPLL
    * 每个 Quad 中有 2 个, 位于 `COMMMON` 子模块中
    * QPLL0: ==9.8–16.375 GHz==
    * QPLL1: ==8.0–13.0 GHz==
    * 可动态调整频率
* 每个 Quad 中, 可以动态切换 CPLL 和 QPLL

### 1.2.3. 复位和初始化
> ![UG576_Fig2-16](/assets/UG576_Fig2-16.png)
* 两种复位:
    * 初始化复位
        * 完全复位, 上电和配置后执行
        * 正常操作时, 仍可通过 `GTTXRESET` 和 `GTRXRESET` 重新初始化
    * 组件复位
        * 复位特定子模块, 复位信号包括:
            * TX: `TXPMARESET`, `TXPCSRESET`
            * RX: `RXPMARESET`, `RXDFELPMRESET`, `EYESCANRESET`, `RXPCSRESET`, `RXBUFRESET`, `RXOOBRESET`
* 复位信号: 高有效, 异步, 持续至少 1 个参考时钟周期
* 复位模式
    * TX: 仅支持顺序复位
        > ![UG576_Fig2-19](/assets/UG576_Fig2-19.png)
    * RX: 支持顺序复位、单独复位
        > ![UG576_Fig2-24](/assets/UG576_Fig2-24.png)

### 1.2.4. 低功耗
* 收发低功耗: `TXPD` 和 `RXPD`
* PLL 低功耗: `CPLLPD`, `QPLL{0/1}PD`

### 1.2.5. 回环
> ![UG576_Fig2-27](/assets/UG576_Fig2-27.png)

### 1.2.6. 动态重配
* 控制端口: `DRP`

### 1.2.7. 数字监控
* 监控回环质量, 可借助 `IBERT` IP 核实现监控

## 1.3. 发送端
> ![UG576_Fig3-1](/assets/UG576_Fig3-1.png)

### 1.3.1. 发送接口
* 接口时钟: `TXUSRCLK2`
* 接口数据: `TXDATA`
    * 支持 2/4/8 byte, 实际宽度依赖 `TX_DATA_WIDTH` 和 `TX_INT_DATAWIDTH` 属性, 以及 `TX8B10BEN` 端口设置



## 1.4. 接受端



--------------------------------------------------------------------------------
# 2. GTY Transceivers

--------------------------------------------------------------------------------
# 3. GTY Transceivers

