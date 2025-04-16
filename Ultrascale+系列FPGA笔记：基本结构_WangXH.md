
<h1 style="text-align:center">Xilinx Ultrascale+系列FPGA：基本结构</h1>

--------------------------------------------------------------------------------

相关文档:
| 文档名称                                | 文档编号 |
| :-------------------------------------- | :------- |
| **Configuration User Guide**            | UG570    |
| **SelectIO Resources User Guide**       | UG571    |
| **Clocking Resources User Guide**       | UG572    |
| **Memory Resources User Guide**         | UG573    |
| **Configurable Logic Block User Guide** | UG574    |
| **DSP Slice User Guide**                | UG579    |
| **System Monitor User Guide**           | UG580    |

--------------------------------------------------------------------------------
# 1. CLB
> Configurable Logic Block User Guide (UG574)

## 1.1. 结构
* 每个 Clock Region (CR) 包含 60 行 **CLB**
* 每个 CLB 包含 1 个 slice: **SLICEL** 或 **SLICEM**
* 每个 **SLICEL** 包含
    * 8 个 **LUT** 和 16 个 DFF
        * 每个 LUT 可配置为 1 个 6-input LUT 或 2 个 5-input
        * LUT 标号: A ~ H
        * DFF 标号: AQ1, AQ2 ~ HQ1, HQ2
        > ![UG574_Fig1-2](/assets/UG574_Fig1-2.png)

        > ![UG574_Fig1-3](/assets/UG574_Fig1-3.png)

        > ![UG574_Fig2-1](/assets/UG574_Fig2-1.png)
    * MUX:
        * 4 个 F7-MUX, 2 个 F8-MUX, 1 个 F9-MUX
        * 最多支持 32:1 的任意 MUX, 和最多 55:1 特定的 MUX
        > ![UG574_Fig2-3](/assets/UG574_Fig2-3.png)
    * 1 个 8-bit 进位链 (CARRY8)
        > ![UG574_Fig2-4](/assets/UG574_Fig2-4.png)
* **SLICEM** 结构基本和 SLICEL 相同, 只是 LUT 更复杂
    * 每个 LUT 可配置为 64-bit 分布式 RAM
        > ![UG574_Fig1-4](/assets/UG574_Fig1-4.png)
    * 每个 SLICEM 可以实现多种分布式 RAM
        * 同步写: 写时钟 **LCLK**, 写使能 **WE**
        * 异步或同步读
        * 单端口:
            * 32 x (1~16)-bit
            * 64 x (1~8)-bit
            * 128 x (1~4)-bit
            * 256 x (1~2)-bit
            * 512 x 1-bit
        * 双端口:
            * 32 x (1~8)-bit
            * 64 x (1~4)-bit
            * 128 x 2-bit
            * 256 x 1-bit
        * 四端口:
            * 32 x (1~4)-bit
            * 64 x (1~2)-bit
            * 128 x 1-bit
        * 八端口:
            * 64 x 1-bit
        * 简单双端口:
            * 32 x (1~14)-bit
            * 64 x (1~7)-bit
    * 每个 LUT 可配置为 32-bit 移位寄存器链 (**SRL**)
        > ![UG574_Fig2-10](/assets/UG574_Fig2-10.png)
        * 支持动态长度读出, 支持静态长度读出
        > ![UG574_Fig2-11](/assets/UG574_Fig2-11.png)
    * 每个 SLICEM 可以实现 1~256 拍的 SRL

## 1.2. 设计建议
* 每个 CLB 都有置位和复位信号, 但不要同时使用
* 有足够的 DFF, 建议使用流水线
* 多个 CLB 共享控制信号 (时钟, 时钟使能, 置位, 复位, 写使能), 因此避免仅某一个或几个 CLB 使用控制信号的设计
* 如果使用 LUT 中的 SRL, 不要使用复位, 建议最后一级使用 DFF
* 对于小型的存储器, 可以使用 CLB
* 减少置位和复位的使用, 上电时 DFF 会被初始化
* ...

--------------------------------------------------------------------------------
# 2. 时钟资源
> Clocking Resources User Guide (UG572)

## 2.1. 全局时钟输入
* 外部时钟必须通过全局时钟 (**GC**) 输入
* 每个 Bank 有 4 个 **GC** 管脚
    * 可用作单端或差分输入
    * 单端信号时, 必须用 _P 管脚输入, 此时 _N 管脚只能用作普通 I/O
* 可以直连全局时钟 buffer, **MMCM**, **PLL**
* UltraScale+ 的 HD I/O bank 有 4 个 **HDGC** 管脚, 可以直连 **BUFG**, 不能直连 **MMCM** 和 **PLL**
    * 可通过 **BUFG** 连接 **MMCM** 和 **PLL**, 需添加约束 `set_property CLOCK_DEDICATED_ROUTE FALSE [get_nets <clock_net_name>]`

## 2.2. Byte 时钟输入
* Byte-lane 时钟: **DBC** 和 **QBC**
    * 在 DDR 存储器中是 DQS
* 用于源同步系统

## 2.3. 时钟结构
> ![UG572_Fig2-1](/assets/UG572_Fig2-1.png)
> ![UG572_Fig2-2](/assets/UG572_Fig2-2.png)
> ![UG572_Fig2-3](/assets/UG572_Fig2-3.png)

* 芯片由多个 **CR** 组成
    * 每个 CR 的高度: 60 个 CLB, 24 个 DSP, 12 个 BRAM
    * IO 和 GT 插在两列 CR 之间, 最多 52 个 IO, 4 个 GT
    * 某些 CR 中还包含配置模块, SYSMON 和 PCIE
* 横向: Horizontal Clock Spine (**HCS**)
    * 在 CR 中间, 横向穿透芯片
    * 24 个路由走线
    * 24 个分布走线
        * 连接叶时钟 buffer
        * 纵横互联
* 纵向
    * 纵向跨越整个芯片
    * 24 个路由走线
    * 24 个分布走线
* 时钟分布的形式
    * 通过横向或纵向路由到 CR 中心 → 通过时钟扇出驱动分布走线 (先纵向再横向) → 通过叶时钟 buffer 驱动 CR 或 相邻的 CR
        * 真全局时钟, 器件范围的时钟网络
    * 时钟 buffer 也可以直接驱动分布走线, 以减小插入延时
* 优势
    * 时钟 Root 可以在任意一个 CR, 可以放在使用的逻辑资源的几何中心附近, 因此 skew 更小
        * 7 系列 FPGA 的时钟 Root 在芯片中心 (BUFG 扇出)
    * 不使用的横向纵向走线, buffer 可以关闭, 因此功耗更低
* XIPHY BITSLICE 中 4 个 byte 的每一个都有 6 个从 HSC 到全局管脚的互联, 因此只有 6 个全局 buffer 可以驱动每半个 I/O bank 中的 BITSLICE

## 2.4. 时钟 buffer
* 全局 buffer: **BUFGCTRL**, **BUFGCE** 和 **BUFGCE_DIV**
    * 源: GC 管脚, 同一个 PHY 的 MMCM 和 PLL, 互联
    * 驱动路由走线和分布走线
    * 每个 PHY 包含 24 个 **BUFGCE**, 8 个 **BUFGCTRL**, 4 个 **BUFGCE_DIV**, 但是同时最多只能使用 24 个
    * **BUFGCE_DIV**: 1~8 分频
* 叶 buffer: **BUFCE_LEAF**
    * 从 HSC 引入时钟, 驱动各种模块
* 高速收发器: **BUFG_GT**
    * 一般接收**IBUFDS_GTE** 的 **ODIV2** 信号
    * **BUFG_GT_SYNC** 为  **BUFG_GT** 的控制管脚提供同步信号, 无需例化, 会自动添加
* **BUFG_PS**
    * Zynq UltraScale+ MPSoC processor system (PS)

## 2.5. CMT
* 1 个 **MMCM**
    * 频率综合, 相位调整, 占空比调整, 动态重配, jitter 过滤
    * MMCM 支持动态相位调整, 支持时钟分频动态切换 (Clock Divide Dynamic Change, CDDC)
    > ![UG572_Fig3-1](/assets/UG572_Fig3-1.png)
* 2 个 **PLL**
    > ![UG572_Fig3-16](/assets/UG572_Fig3-16.png)
    * 频率综合, 相位调整, 占空比调整, 动态重配, jitter 过滤
    * 是 MMCM 的子集, 区别
        * 输出较少
        * 不支持 deskew
        * 不支持动态相位调整
        * M, D 数值范围小

--------------------------------------------------------------------------------
# 3. SelectIO
> SelectIO Resources User Guide (UG571)

## 3.1. 接口资源

### 3.1.1. 基本情况
* 3 种 I/O Bank: HPIO, HRIO, HDIO
    > ![UG571_Tab1-1-1](/assets/UG571_Tab1-1-1.png)
    > ![UG571_Tab1-1-2](/assets/UG571_Tab1-1-2.png)
    * High-performance (HP):
        * 高速, 高性能, 1.0~1.8 V
        * 支持: DCI, 内部 V~REF~, 内部差分终端匹配, IDELAY, ODELAY, IDELAYCTRL, ISERDES, OSERDES, 发送器预加重, 接收器均衡, 接收器偏压控制, 接收器 V~REF~ 扫描
        * Ultrascale+ 还支持 MIPI D-PHY 收发器
        > ![UG571_Fig1-1](/assets/UG571_Fig1-1.png)
        > ![UG571_Fig1-2](/assets/UG571_Fig1-2.png)
    * high-range (HR):
        * 宽范围I/O标准, 1.2~3.3V
        * 与 HP 相比, **不支持**: DCI, 接收器偏差控制, 接收器 V~REF~ 扫描
        * Ultrascale+ 没有此种接口
        <!-- > ![UG571_Fig1-3](/assets/UG571_Fig1-3.png) -->
        <!-- > ![UG571_Fig1-4](/assets/UG571_Fig1-4.png) -->
    * high-density (HD):
        * 低速接口, 每个 Bank 24 个, 1.2~3.3V
        * 成对的 I/O 可以支持部分差分标准(仅输入)和部分伪差分标准
        * Ultrascale 没有此种接口
* 一般每个 Bank 有 52 个 I/O, 其中 48 个可以用作单端或差分, 另外 4 个只能用作单端
    * HR mini-Bank 只有 26 个
    * HD Bank 只有 24 个

### 3.1.2. 配置中和配置后 I/O 的状态
* 配置中, 除配置所需的 Bank 外, 全部是 3 态
    * HP 默认的电平标准 IOSTANDARD = LVCMOS18, SLEW = FAST, DRIVE = 12 mA
    * HR 默认的电平标准 IOSTANDARD = LVCMOS25, SLEW = FAST, DRIVE = 12 mA
* 配置完成后, 没有配置的管脚是 3 态 + 弱上拉

### 3.1.3. 阻抗匹配与参考电压
* DCI (digitally controlled impedance)
    * 仅存在于 HP I/O bank 中
    * 根据连在 VRP 管脚上的 240Ω 对地电阻动态调节内部匹配电阻
    * 支持源端匹配
        > ![UG571_Fig1-9](/assets/UG571_Fig1-9.png)
        > ![UG571_Tab1-2](/assets/UG571_Tab1-2.png)
        * 阻值由属性 `OUTPUT_IMPEDANCE` 决定, R $\in$ {RDRV_40_40, RDRV_48_48, RDRV_60_60, RDRV_NONE_NONE}
    * 支持终端匹配
        * 戴维南终端匹配 (DC电平为 V~CCO~/2)
            > ![UG571_Fig1-11](/assets/UG571_Fig1-11.png)
            > ![UG571_Tab1-3](/assets/UG571_Tab1-3.png)
            * R 阻值由属性 `ODT` 决定, R $\in$ {RTT_40, RTT_48, RTT_60}
        * 单终端匹配
            > ![UG571_Fig1-13](/assets/UG571_Fig1-13.png)
            > ![UG571_Tab1-4](/assets/UG571_Tab1-4.png)
            * R 阻值由属性 `ODT` 决定
                * 对于 POD: R $\in$ {RTT_40, RTT_48, RTT_60}
                * 对于 HSUL_12_DCI 和 DIFF_HSUL_12_DCI: R $\in$ {RTT_120, RTT_240}
                * RTT_NONE
    * DCI 校准
        * 在芯片上电启动时进行, DCI 调整会阻塞 DONE
        * 可通过 **DCIRESET** 原语复位 (仅当 `DCIUpdateMode = Quiet` 可用)
        * **DCIRESET** 的 LOCKED 有效后, DCI 可用
        * 配置选项
            * `Match_cycle`: 使 FPGA 在 DCI 校准后才开始 startup
            * `DCIUpdateMode`: DCI 校准模式
                * `AsRequired` (默认): 芯片初始化时校准, 根据需要动态调节, Xilinx 强烈建议此选项
                * `Quiet`: 芯片初始化时校准一次, **DCIRESET** 复位时调节
    * 特殊情况
        * 如果 Bank 65 的多功能管脚使用 DCI, 需要在设计中添加 DCIRESET, 且复位后才能使用 DCI
            * 多功能管脚在芯片启动过程中会忽略 DCI 校准
    * DCI 级联
        * 同一列的 Bank 才能用级联
        * DCI 级联可以作用到整列的 Bank (不能跨越硅中介层)
        * 级联 Bank 的 V~CCO~ 和 V~REF~ 需相同
        * 使用: `set_property DCI_CASCADE {slave_banks} [get_iobanks master_bank]`
* 未校准的输入终端匹配
    * 终端匹配
        * 与 DCI 不同, 电阻未经校准, 可能不准
        * HRIO 和 HPIO: 戴维南终端匹配, 支持 HSTL 和 SSTL
            * 支持 `RTT_40`, `RTT_48`, `RTT_60`, `RTT_NONE`
        * HRIO 和 HPIO: 单终端匹配, 支持 POD 和 HSUL
            * POD 支持 `RTT_40`, `RTT_48`, `RTT_60`, `RTT_NONE`
            * HSUL 支持: `RTT_120`, `RTT_240`, `RTT_NONE`
        * 作用于输入或双向端口
        * 阻抗可选 40Ω, 48Ω, 60Ω, 通过 `OUTPUT_IMPEDANCE` 属性设置, `RDRV_40_40, RDRV_48_48, RDRV_60_60`
    * 源端匹配
        * 与 DCI 不同, 电阻未经校准, 可能不准
        * OUTPUT_IMPEDANCE: `RDRV_40_40`, `RDRV_48_48`, `RDRV_60_60`
    * HPIO 可以控制接收器的 offset
        * 最大可支持 ±35 mV 的偏移
        * 输入将被上拉到 V~REF~ (外部或内部)
        * 支持此功能的原型: **IBUFE3**, **IBUFDSE3**, **IOBUFE3** 和 **IOBUFDSE3**
        > ![UG571_Fig1-14_15](/assets/UG571_Fig1-14_15.png)
    * HPIO 支持接收器 V~REF~ 扫描 (精调 V~REF~ 电压)
        > ![UG571_Fig1-16](/assets/UG571_Fig1-16.png)
        * 支持此功能的原型: **IBUFE3** 和 **IOBUFE3**
        * 最多支持连续 13 个 I/O (1 个 byte 组)
            > ![UG571_Fig1-17](/assets/UG571_Fig1-17.png)
        * 必须开启 `INTERNAL_VREF` 属性且设置合适的电压值才能使用此功能
            * 同一个 Bank 只能使用一种 V~REF~
            * V~REF~ 不能跨 Bank 使用
            * 不能和外部 V~REF~ 混用
        * 控制方式: 可选是否由 CLB 控制
            * `set_property OFFSET_CNTRL {CNTRL_NONE|FABRIC] [get_ports port_name]`
        * 取值 (% of VCCO)
            * 设置 `VREF_CNTR` 属性
            > 参见 UG571::Table 1-11
* 差分信号匹配
    * 使用内建 100Ω 电阻
    * 由 DIFF_TERM (用在 HDL 代码中) 或 DIFF_TERM_ADV (用在 XDC 中) 属性决定
        * `DIFF_TERM = TRUE` 自动映射到 `DIFF_TERM_ADV = TERM_100`
        * `DIFF_TERM = FALSE` 自动映射到 `DIFF_TERM_ADV = TERM_NONE` (默认)
* V~REF~
    * 可以使用外部 V~REF~
        * 如果使用, 多功能管脚必须连到合适电压
        * 否则, V~REF~ 管脚通过 500Ω 或 1kΩ 电阻到地, 或悬空
    * 也可以使用内部 V~REF~, 开启 `INTERNAL_VREF` 属性或 HP bank 的 V~REF~ 扫描
        * 如果使用, V~REF~ 管脚通过 500Ω 或 1kΩ 电阻到地
        * 基于 V~CCO~
        * INTERNAL_VREF 设置:
            * 按 Bank 分配, 如设置 Bank 65 为 0.9V: `set_property INTERNAL_VREF 0.90 [get_iobanks 65]`
            * 电压值必须是支持的 I/O 标准的参考值
                * 可选的值包括 (不是所有的 Bank 都能用): 0.60, 0.675, 0.70, 0.75, 0.84, 0.90
        * 不能同时使用外部 V~REF~

### 3.1.4. 发送与接收加强
* 发送器预加重
    * 支持某些电平标准 (例如 DDR4 应用中)
    * 由属性 `ENABLE_PRE_EMPHASIS` 控制
        * `PRE_EMPHASIS = RDRV_NONE` (default)
        * `PRE_EMPHASIS = RDRV_240` (where ENABLE_PRE_EMPHASIS must be set to TRUE)
* LVDS发送器预加重
    * 由属性 `LVDS_PRE_EMPHASIS` 控制
        * `LVDS_PRE_EMPHASIS = FALSE` (Default)
        * `LVDS_PRE_EMPHASIS = TRUE` (where ENABLE_PRE_EMPHASIS must be set to TRUE)
* 接收器均衡
    * 支持某些电平标准 (例如 DDR4 应用中)
    * 由属性 `EQUALIZATION` 控制
        > ![UG571_Tab1-15](/assets/UG571_Tab1-15.png)

### 3.1.5. SelectIO 原语与属性
* 接口原语
    * 单端信号:
        * IBUF, IBUF_IBUFDISABLE, IBUF_INTERMDISABLE, `IBUFE3` (支持接收器偏压和 V~REF~ 扫描)
        * IOBUF, IOBUF_DCIEN, IOBUF_INTERMDISABLE, `IOBUFE3` (支持接收器偏压和 V~REF~ 扫描)
        * OBUF, OBUFT
    * 差分信号:
        * IBUFDS, IBUFDS_DIFF_OUT, IBUFDS_DIFF_OUT_IBUFDISABLE, IBUFDS_DIFF_OUT_INTERMDISABLE, IBUFDS_IBUFDISABLE, IBUFDS_INTERMDISABLE, `IBUFDSE3` (支持接收器偏压), `IBUFDS_DPHY` (支持 MIPI, 仅存在于 Ultrascale+)
        * IOBUFDS, IOBUFDS_DCIEN, IOBUFDS_DIFF_OUT, IOBUFDS_DIFF_OUT_DCIEN, IOBUFDS_DIFF_OUT_INTERMDISABLE, IOBUFDS_INTERMDISABLE, `IOBUFDSE3` (支持接收器偏压)
        * OBUFDS, OBUFTDS, `OBUFDS_DPHY` (支持 MIPI, 仅存在于 Ultrascale+)
    * `HPIO_VREF` (V~REF~ 扫描)
* 接口属性和约束
    * 基本属性: PACKAGE_PIN, IOSTANDARD, IBUF_LOW_PWR, SLEW, DRIVE, PULLTYPE, `DATA_RATE (仅用于功耗、时序等分析)`
    * 阻抗匹配: DCI_CASCADE, `ODT`, `OUTPUT_IMPEDANCE`, DIFF_TERM, `DIFF_TERM_ADV`
    * 内部 V~REF~: INTERNAL_VREF, `OFFSET_CNTRL`, `VREF_CNTR`
    * DQS: `DQS_BIAS`
    * 发送与接收: `PRE_EMPHASIS`, `LVDS_PRE_EMPHASIS`, `EQUALIZATION`

### 3.1.6. 支持的 I/O 标准
> 参见 UG571::Table 1-77
* 支持常用的单端和差分信号, 可选是否带阻抗匹配
* 支持 DDR 要求的各种电平标准
* 支持视频需要的 TMDS, MIPI 电平标准
> 相较于 7 系列, 增加了新型高速内存的电平标准和 LVPECL, MIPI

## 3.2. 逻辑资源
### 3.2.1. Overview
* 每个 Bank 包含 4 个 ByteGroup, 每个 ByteGroup 可以作为一组使用, 也可以拆分为两个小块
> ![UG571_Fig2-1](/assets/UG571_Fig2-1.png)
* Group 1,2 中包含 **clocks quad byte clock (QBC)** and **global clock (GC)** 管脚
* QBC: 可用于所在的小块, 或 Byte, 或整个 Bank
</br>

> ![UG571_Fig2-2](/assets/UG571_Fig2-2.png)
* BITSLICE_12 (即上半块的 BITSLICE_6) 仅支持单端信号
* 任意使用 bitslice 的单端时钟信号必须使用 BITSLICE_0, 差分时钟必须使用 BITSLICE_0 和 BITSLICE_1
* 当使用 RX_BITSLICE 或 RXTX_BITSLICE 时, Byte 间时钟可能影响 BITSLICE_0 的有效性
    * 如果使用 QBC (从一个的块到另一个 ByteGroup 的块), 源端的块必须使用 BITSLICE 0, 且其 `DATA_TYPE` 设为 `DATA`
    * 如果用于串行接收模式, 每个块必须使用 BITSLICE 0, 且其 `DATA_TYPE` 设为 `SERIAL`
* IDELAY/ODELAY 和 RX_BITSLICE/TX_BITSLICE/RXTX_BITSLICE 支持 TIME 模式, 提供更准确的延时
    * 当使用 TIME 模式时, BITSLICE_0 被用于初始化校准, 因此在初始化校准过程中, 连接到 BITSLICE_0 的信号是无效的
    > 可设置约束 `set_property UNAVAILABLE_DURING_CALIBRATION TRUE [get_ports <name>]` 以禁止报错
* 示例
> ![UG571_Fig2-3](/assets/UG571_Fig2-3.png)

### 3.2.2. Component 原型
* 简单的寄存输入和输出
* IDDRE1
    * 支持 `OPPOSITE_EDGE`, `SAME_EDGE`, `SAME_EDGE_PIPELINED` 三种模式
    > ![UG571_Fig2-4](/assets/UG571_Fig2-4.png)
    > ![UG571_Fig2-5](/assets/UG571_Fig2-5.png)
    > ![UG571_Fig2-6](/assets/UG571_Fig2-6.png)
* ODDRE1
    * 仅支持 `SAME_EDGE` 模式
    > ![UG571_Fig2-8](/assets/UG571_Fig2-8.png)
    * 支持 single 和 serialized 三态模式
    > ![UG571_Fig2-10](/assets/UG571_Fig2-10.png)
    > ![UG571_Fig2-11](/assets/UG571_Fig2-11.png)
* ISERDESE3
    > ![UG571_Tab2-5](/assets/UG571_Tab2-5.png)
    * 其他比例需要借助 FPGA 逻辑
    * 连接一个浅的 FIFO (深度为 8)
* OSERDESE3
    * SDR模式下支持 2/4 并转串
    * DDR模式下支持 4/8 并转串
    * 其他比例需要借助 FPGA 逻辑
* IDELAYE3
    > ![UG571_Fig2-16](/assets/UG571_Fig2-16.png)
    * 512 拍延迟线
    * 支持 `COUNT` 和 `TIME` 模式
        * `COUNT`: 无需 IDELAYCTRL, 未校准, DELAY_VALUE $\in [0, 511] 拍$
        * `TIME`: 需要 IDELAYCTRL, 根据电压和温度校准, DELAY_VALUE 是 xx ps, 时钟需与 IDELAYCTRL 一致
    * 支持级联
* ODELAYE3
    > ![UG571_Fig2-22](/assets/UG571_Fig2-22.png)
    * 支持 `COUNT` 和 `TIME` 模式
        * `COUNT`: 无需 IDELAYCTRL, 未校准, DELAY_VALUE $\in [0, 511] 拍$
        * `TIME`: 需要 IDELAYCTRL, 根据电压和温度校准, DELAY_VALUE 是 xx ps, 时钟需与 IDELAYCTRL 一致
    * 支持级联
* IDELAYCTRL
    > ![UG571_Fig2-25](/assets/UG571_Fig2-25.png)
    * 用于校准 IDELAYE3 和 ODELAYE3
    * 每个块共享一个 IDELAYCTRL
* Component 模式下注意事项:
    * 复位
        * 遵循指定的顺序, 参见 UG571 - Page195
    * 时钟
        * 使用全局时钟
    * 双向信号
        * 三态控制
            ![UG571_Tab2-19-1](/assets/UG571_Tab2-19-1.png)
            ![UG571_Tab2-19-1](/assets/UG571_Tab2-19-2.png)

### 3.2.3. Native 原型
* 由 Component 原型组成, 更易使用, Xilinx 提供向导 High-Speed SelectIO wizard (HSSIO-Wiz)
* RXTX_BITSLICE
    > ![UG571_Fig2-31](/assets/UG571_Fig2-31.png)
    * 输入/输出延迟线
        * 512 拍, 支持 `COUNT` 和 `TIME` 模式
        * 两种控制方式
            * BITSLICE_CONTROL via the RIU 接口
            * 内部逻辑直接控制 RXTX_BITSLICE 的信号(`CLK`, `CE`, `INC`, `LOAD`, `CNTVALUEIN[8:0]`, `CNTVALUEOU[8:0]`, `RST_DLY`, and `EN_VTC`)
        * 延迟线级联:
            * RXTX_BITSLICE 不支持级联
            * RX_BITSLICE 支持级联: 使用 TX_BITSLICE 中未用到的输出延迟线实现级联, 延时增加一倍
        * 输入延时类型: `FIXED`, `VARIABLE`, `VAR_LOAD`
    * 三态控制:
        * 两种模式:
            * 每个通道单独三态控制: 由内部逻辑产生
            * 整块串行三态控制: 时间精度对应到每个 bit
    * 接收器功能
        > ![UG571_Fig2-32](/assets/UG571_Fig2-32.png)
        * 输入延迟线总在信号路径中, 如果不延时, 设为 0
        * 解串器分为 3 段: 1:2, 2:4, 4:8
        * 源同步模式下
            * 源时钟必须连到每个块的 BITSLICE_0, 有对应的 QBC 或 DBC 输入
            * 源时钟通过 BITSLICE_0 连到 BITSLICE_CONTROL
            * BITSLICE_CONTROL 产生所需的时钟
            * BITSLICE_CONTROL 需要主时钟或参考时钟 (连到它的 PLL_CLK 输入管脚)
    * FIFO
        * 深度 = 8
        * 信号: `FIFO_RD_CLK`, `FIFO_RD_EN`, `FIFO_EMPTY`, `FIFO_WRCLK_OUT`, `Q[7:0]`/`Q[3:0]`
        * 最好在 `BITSLICE_CONTROL.VTC_RDY` 拉高后, 才启动接收
    * 发送器功能
        > ![UG571_Fig2-39](/assets/UG571_Fig2-39.png)
        * 特殊应用: 输出时钟, 例如将 `D` 设置为 10101010 或 1010 可以输出 50% 占空比的时钟
        * `OUTPUT_PHASE_90` 属性: 控制数据是否偏移
            * `TRUE`: 对齐到下降沿
            * `FALSE`: 对齐到上升沿
* RX_BITSLICE
    * RXTX_BITSLICE 的接收器
        > ![UG571_Fig2-46](/assets/UG571_Fig2-46.png)
    * 支持延迟线级联
        > ![UG571_Fig2-48](/assets/UG571_Fig2-48.png)
* TX_BITSLICE
    * RXTX_BITSLICE 的发送器
        > ![UG571_Fig2-49](/assets/UG571_Fig2-49.png)
* TX_BITSLICE_TRI
    > ![UG571_Fig2-51](/assets/UG571_Fig2-51.png)
    * 当属性 `TBYTE_CTL` = `TBYTE_IN` 时, TX_BITSLICE_TRI 有效
        > ![UG571_Fig2-55](/assets/UG571_Fig2-55.png)
* BITSLICE_CONTROL
    > ![UG571_Fig2-58](/assets/UG571_Fig2-58.png)
    * 启动和复位
        > ![UG571_Fig2-60](/assets/UG571_Fig2-60.png)
        * 详情参考: UG571::Page302
    * 时钟
        * PLL_CLK or REFCLK
        * 接收时钟
            > ![UG571_Fig2-64](/assets/UG571_Fig2-64.png)
            > ![UG571_Fig2-65](/assets/UG571_Fig2-65.png)
        * 发送时钟
            > ![UG571_Fig2-66](/assets/UG571_Fig2-66.png)
        * 块间时钟
            > ![UG571_Fig2-67](/assets/UG571_Fig2-67.png)
        * Byte Group 间时钟
            > ![UG571_Fig2-68](/assets/UG571_Fig2-68.png)
            > ![UG571_Fig2-69](/assets/UG571_Fig2-69.png)
    * 自校准 (built-in self-calibration, BISC)
        * 数字控制的校准模块, 基于 DLL 和延迟线相位检测
        > ![UG571_Fig2-72](/assets/UG571_Fig2-72.png)
        * 校准步骤
            1. 对齐
            1. 延时校准
            1. 持续的 VT 跟踪
    * 控制接口: register interface unit (RIU)
        * 借助 RIU, 可将 BITSLICE_CONTROL 当作一个处理器外设
        * 64 个可读写的 16 位寄存器
        * 功能:
            * 控制所有的输入、输出、三态
            * 控制所有的延迟线 (input, output and quarter),
            * 电压和温度跟踪
            * 时钟选项
            * 控制 BISC

## 3.3. HDIO
![UG571_Tab3-1-1](/assets/UG571_Tab3-1-1.png)
![UG571_Tab3-1-2](/assets/UG571_Tab3-1-2.png)

--------------------------------------------------------------------------------
# 4. Memory
> Memory Resources User Guide (UG573)

## 4.1. Block RAM
* 基本结构: RAMB36E2
    * 36 Kb, 最大宽度 72 bit
    * 可以拆分为两个 18 Kb, 最大宽度 36 bit
    * 支持单端口和双端口模式
        * 双端口模式下, 两个端口的宽度可以不同
    * 可以级联成更大的 RAM
    * 支持内建 FIFO
* 与 7 系列相比,
    * SDP 配置下, 写模式支持 READ_FIRST, WRITE_FIRST, NO_CHANGE
    * 更易组成大 RAM/FIFO, FIFO 更好用,
    * 增加了写地址使能, 功耗控制,
    * ...
> ![UG573_Fig1-5](/assets/UG573_Fig1-5.png)
* 读操作
    * 锁存模式: 延迟为 1 CC
    * 可选寄存器: 延迟增加 1 CC
    *
* 写操作
    * 延迟 1 CC
* 地址碰撞
    * WRITE_FIRST: 输出最新写入的数据
    * READ_FIRST: 输出之前保存的数据
    * NO_CHANGE: 保持上次读的数据
* 内建 FIFO
    * 支持标准模式 和 first-word fall-through 模式
    * 支持同步和异步
* 内建 ECC
    * 在双端口模式下可以配置为 512 × 64 RAM, 额外的 8 bit 用作校验
        * FIFO 也支持 ECC
    * ECC 解码器输出两个状态位: SBITERR, DBITERR

## 4.2. UltraRAM
* 主要特征
    * 每列包含 16 个
    * 288 Kb, 只能配置为 4 K × 72 bit
        > ![UG573_Fig2-1](/assets/UG573_Fig2-1.png)
        > ![UG573_Fig2-3](/assets/UG573_Fig2-3.png)
    * 双端口, 但仅有一个时钟
        * 不支持跨时钟操作
        * 每个端口都可以独立的读写, 但是内部 SRAM 阵列只有一个端口
            * 实际运行过程: 单个时钟周期内, A 端口动作后 B 端口再动作
        * 操作同一地址的表现:
            * A 写 B 读, A 的新数据被写入 SRAM, B 读取新数据
            * A 读 B 写, A 读取旧数据, B 的新数据被写入 SRAM
            * A 写 B 写, B 的数据被写入 SRAM, A 的数据被覆盖
            * A 读 B 读, 都读取旧数据
    * 最多4个流水级:
        * 输入: IREG_PRE
        * 输出: OREG, OREG_ECC
        * 级联路径: IREG_CAS, OREG_CAS
    * 初始值为 0, 不能自定义
    * 支持级联, 专门的接口
        * 可以级联 data, address, 和 control 信号, 但不能改变 data lines
        * 不支持动态级联
        > ![UG573_Fig2-2](/assets/UG573_Fig2-2.png)
        > ![UG573_Fig2-5](/assets/UG573_Fig2-5.png)
        > ![UG573_Fig2-7](/assets/UG573_Fig2-7.png)
        > ![UG573_Fig2-8](/assets/UG573_Fig2-8.png)
    * 支持 ECC: 单 bit 检测和纠正, 双 bit 检测
    * 支持休眠, 支持自动休眠
* 与 Block RAM 的区别
    > ![UG573_Tab2-1](/assets/UG573_Tab2-1.png)

--------------------------------------------------------------------------------
# 5. DSP
> DSP Slice User Guide (UG579)

* DSP 资源
    * 每个 DSP tile = 2 个 DSP, 每个 DSP tile 的高度 = 5 个 CLB = 1 个 Block RAM
        > ![UG579_Fig1-2](/assets/UG579_Fig1-2.png)
    * 每列 12 个 tile (24 个 DSP)
    * 可按列方向级联, 可跨 CR, 最大长度为芯片的整个列

## 5.1. 结构
> ![UG579_Fig1-1](/assets/UG579_Fig1-1.png)
> ![UG579_Fig2-1](/assets/UG579_Fig2-1.png)
> ![UG579_Fig2-2](/assets/UG579_Fig2-2.png)
* 数学运算 (二进制补码)
    * 27 位预加器
        * $D \pm A$ 或 $D \pm B$
            * 其中 A, B, D 都可以通过设置 INMODE 而变为 0, 因此输出可以变为 $0, D, \pm A, \pm B$
        * 可以优化对称滤波器应用
    * 27 × 18 补码乘法器
        * 45 位结果会被扩展到 48 位
        * 可动态旁路 (设置 `USE_MULT = NONE`)
        * 配合 C 端口, 可以实现乘-加、乘-减、乘-舍入
    * 48 位加减法
        * 4 输入 (使用乘法器时变为 3 输入)
            > ![UG579_Tab2-7](/assets/UG579_Tab2-7.png)
            * $W \in \{0, P, RND, C \}$
            * $X \in \{0, M, P, A:B \}$
            * $Y \in \{0, M, 48'hFFFFFFFFFFFF, C \}$
            * $Z \in \{0, PCIN, P, C, (PCIN >> 17), (P >> 17) \}$
        * 支持 SIMD
            * 双 24 位加, 减, 累加操作, 两个独立的 CARRYOUT 信号
            * 4 个 12 位加, 减, 累加操作, 4 个独立的 CARRYOUT 信号
                > ![UG579_Fig2-14](/assets/UG579_Fig2-14.png)
    * 48 位累加器, 可级联到 96 位
    * 桶形移位
        * 17 位右移, 用于产生更准确的结果
    * 进位
        * 支持舍入
        * 支持级联, 以实现更宽的加减法
        * 进位输入
            > ![UG579_Fig2-9](/assets/UG579_Fig2-9.png)
            > ![UG579_Tab2-8](/assets/UG579_Tab2-8.png)
        * 进位输出
            * CARRYOUT[3] 在使用乘法器或 3/4 输入加减法器时是无效的
            > ![UG579_Tab2-9](/assets/UG579_Tab2-9.png)
    * MACC 符号位
        * MULTSIGNIN 和 MULTSIGNOUT 内部级联
* 逻辑运算
    * 48 位逻辑
        * 双输入, 某种情况下支持 3 输入
        * 按位逻辑: AND, OR, NOT, NAND, NOR, XOR, XNOR
        * 动态选择功能: ALUMODE
        > ![UG579_Tab2-10-1](/assets/UG579_Tab2-10-1.png)
        > ![UG579_Tab2-10-2](/assets/UG579_Tab2-10-2.png)
    * 96 位 XOR
        * X XOR Z 或 X XOR Y XOR Z
        * 可用于前向错误纠正或CRC算法
        > ![UG579_Fig2-19](/assets/UG579_Fig2-19.png)
* 模式检测
    > ![UG579_Fig2-15](/assets/UG579_Fig2-15.png)
    * 比较器, 产生标志信号
        * 计数器终点 (计数器自动复位)
        * 上下溢出
            > ![UG579_Fig2-16](/assets/UG579_Fig2-16.png)
        * 收敛舍入/对称舍入
    * 96 位规约 AND/NOR
* 流水线
    * 输入寄存器: A1, A2, B1, B2, C, D
    * 预加器结果寄存器 (AD)
    * 乘法结果寄存器 (M)
    * 最终结果寄存器 (P)
    * XOR 结果寄存器, 进位输出寄存器, 模式检测结果寄存器
    * 每个寄存器有独立的时钟只能和同步复位管脚, 可选极性
* 级联
    * 可按列方向级联, 可跨 CR, 最大长度为芯片的整个列
    * A, B 级联
    * CARRYCASCIN 和 CARRYCASCOUT 内部级联, 支持 96 bit 累计, 加, 减操作
    * MULTSIGNIN 和 MULTSIGNOUT 内部级联, 支持 96 bit 乘加 (MACC) 操作
    * 48 位 P 总线, 支持 SIMD 加法级联
    > 例如: 在 FIR 中, 将运算结果级联到下一级的输入
* 动态配置运算功能
    * INMODE [4:0]
        * 双 A, B 寄存器
        * 预加器控制, 加减控制
        * pre-adder add-sub control as well as mask gates for pre-adder multiplexer functions
    * OPMODE[7:0]
        * 控制 W, X, Y, Z
    * ALUMODE[3:0]
        * 逻辑功能控制
        * 累加器加减控制

## 5.2. 设计
* 性能
    * 用好寄存器, 最好用 3 级流水线, 不使用乘法的运算最好使用 2 级流水线
* 功耗
    * 不使用乘法时, 关闭乘法器, `USE_MULT = NONE`
    * DSP 实现运算的功耗低于 fabric
        * 尽量使用专用级联
    * 尽量使用 M 寄存器
* 加法器树可通过级联进行优化
* DSP 跨列将增加功耗
* 多数情况下, 综合工具会推测是否需要 DSP, 代码建议:
    1. 在 HDL 代码中使用有符号数
    1. 将有效的数字放在操作数的高位部分, 低位部分用 0 填充
    1. 使用流水线
    1. 用 CLB 的进位链实现小的乘法, 加法和计数
    1. 使用 CLB 或 RAM 存储滤波器系数
    1. 尽量使用级联而非 fabric

--------------------------------------------------------------------------------
# 6. 配置
> Configuration User Guide (UG570)

## 6.1. 配置接口
* JTAG
    * **M[2:0]** = 101
* Master SPI
    * **M[2:0]** = 001
    * x1, x2, x4, x8(双 QSPI 芯片)
    * **CCLK** 作为输出, 频率可在 Vivado 配置选项中选择
    > ![UG570_Fig2-4](/assets/UG570_Fig2-4.png)
* 其他配置方式:
    * Master BPI (x8 and x16)
    * Slave SelectMAP (x8, x16, x32)
    * Slave-Serial
    * 不推荐的: Master-Serial, Master SelectMAP
* 配置管脚 (仅考虑 JTAG 和 Master SPI)
    * 专用信号, 位于Bank 0
        | 分类  |     Pin     |  I/O   | 注意                                                                                        |
        | :---: | :---------: | :----: | :------------------------------------------------------------------------------------------ |
        | JATG  |    TCK_0    |   I    | 官方使用 2mm 间距的插针                                                                     |
        |   ^   |    TDI_0    |   I    | ^                                                                                           |
        |   ^   |    TDO_0    |   O    | ^                                                                                           |
        |   ^   |    TMS_0    |   I    | ^                                                                                           |
        |  CFG  | PROGRAM_B_0 |   I    | 复位配置, 低有效, 最好用 RC POR 复位电路(或复位芯片), 上拉(≤ 4.7k)                          |
        |   ^   |  INIT_B_0   | IO(OD) | 表明配置存储器开始初始化, 需要上拉 (4.7k)                                                   |
        |   ^   |   DONE_0    |   IO   | 是否配置成功, 高有效, 最好通过 NMOS 驱动一个 LED, 上拉(330)                                 |
        |   ^   |  M[2:0]_0   |   I    | 配置模式, 上下拉电阻 ≤ 1 kΩ                                                                 |
        |   ^   |  CFGBVS_0   |   I    | 如果与配置相关的 BANK 0, 65 是 2.5/3.3 V供电, 需要拉到 2.5/3.3 V; 如果是 1.5/1.8V, 拉到 Gnd |
        | Flash |   CCLK_0    |   IO   | 配置时钟, 默认关闭且三态弱上拉                                                              |
        |   ^   |     FCS     |   O    | 配置 Flash 的片选信号, 上拉 ≤ 4.7 kΩ                                                        |
        |   ^   |  D00 - D04  |   IO   | 配置 Flash 的数据接口, 排序最好与 QSPI 保持一致                                             |
        | Init  |   PUDC_B    |   I    | 0 = 使能内部上拉电阻, 1 = 禁用; 不能 float, 上下拉电阻 ≤ 1 kΩ                               |
    * 多功能信号, 位于 Bank 65
        | 分类  |    Pin     |  I/O  | 注意                              |
        | :---: | :--------: | :---: | :-------------------------------- |
        | Flash |   EMCCLK   |   I   | 仅当使能外部 Master CCLK 时才需要 |
        |   ^   | DOUT_CSO_B |   O   | 仅当 QSPI 配置为菊花链时才需要    |
        |   ^   |   FCS2_B   |   O   | 仅当使用双 QSPI 时才需要          |
    * 电源管脚
        * POR_OVERRIDE: 当用户确保电源上升很快时, 可以使用此管脚减小 POR 时间 (上拉到 VCCINT), 不能悬空
        * VBATT: 密钥相关电源
    * 在 Vivado 配置选项中设置 `CONFIG_VOLTAGE` 或 `CFGBVS`, 以便检查

## 6.2. DRP
* FPGA内部很多组件支持 DRP 端口, 可通过 fabric 动态更改工作参数

## 6.3. 配置过程
* 配置文件
    * `.bit`: JTAG 调试用
    * `.BIN, .MCS, .HEX`: 配置到 Flash
* 配置顺序
    ![UG570_Fig9-2](/assets/UG570_Fig9-2.png)
    * 上电启动
        * 可使用 **POR_OVERRIDE** 缩短 POR 时间
        * 可使用 **INIT_B** 或 **PROGRAM_B** 延长初始化过程
        * 在此期间, Flash 需要准备好
    * 加载 Bitstream
    * Startup 步骤
        * 8-phase
            * 1-6 phase 用户可调整
                * MMCM锁定,
                * DCI匹配完成
                * 释放 **DONE** 管脚, 默认 4th-phase
                * 拉低 Global 3-State (GTS), 以使能 I/O, 默认 5th-phase
                * 置位 Global Write Enable (GWE) 以使能 RAM 和寄存器, 默认 6th-phase
            * 7th-phase: 置位 End Of Startup (EOS)
        * 可以通过 `STARTUPE2` 原语, 控制 Startup 或获取其状态
        * 可在 Vivado 中通过 "Edit Device Properties" 对话框设置

## 6.4. 安全性
* 每个 FPGA 都有一个唯一的设备识别码 (Device DNA), 不可更改
    * 可通过 JTAG 从外部读取
    * 也可通过 `DNA_PORTE2` 原语从内部读取
* 配置文件 AES 加密
    * Vivado 产生加密的 bitstream
    * FPGA 内部通过专用 BBRAM (需要备用电池) 或 eFUSE 存储密钥
        * 只能通过 JTAG 写入, 不能回读
* 禁止回读配置数据
    * 在 Vivado 中通过 "Edit Device Properties" 对话框设置
        * NONE: 可以回读
        * Level1: 禁止回读
        * Level2: 禁止回读和重配置
    * 通过置位 PROGRAM_B 管脚或重新上电, 可以去除回读限制

## 6.5. Vivado 配置选项
* 可通过 "Edit Device Properties" 对话框设置配置相关的选项, 将保存在 XDC 文件中
    ![Vivado-DeviceProperties.png](/assets/Vivado-DeviceProperties.png)
* "Edit Device Properties" 对话框打开方式:
    1. 打开综合或布局布线设计 -> Menu::Tools -> Edit Device Properties
        ![Vivado-DeviceProperties-Edit-1.png](/assets/Vivado-DeviceProperties-Edit-1.png)
    1. 打开综合或布局布线设计 -> Setting -> Edit Device Properties
        ![Vivado-DeviceProperties-Edit-2-1.png](/assets/Vivado-DeviceProperties-Edit-2-1.png)
        ![Vivado-DeviceProperties-Edit-2-2.png](/assets/Vivado-DeviceProperties-Edit-2-2.png)

--------------------------------------------------------------------------------
# 7. System Monitor
> System Monitor User Guide (UG580)

> ![UG580_Tab1-1-1](/assets/UG580_Tab1-1-1.png)
> ![UG580_Tab1-1-2](/assets/UG580_Tab1-1-2.png)
> ![UG580_Fig1-1](/assets/UG580_Fig1-1.png)
> ![UG580_Fig1-2](/assets/UG580_Fig1-2.png)
* ADC专用管脚
    > ![UG580_Fig1-3](/assets/UG580_Fig1-3.png)
    * **V~CCADC~**: 1.8V, 可以连至 VCCAUX, 即便不用 SYSMON 也要连到 1.8V (VCCAUX)
    * **V~CC_PSADC~**: 1.8V, 可以连至 VCC_PSAUX, 即便不用 SYSMON 也要连到 1.8V (VCC_PSAUX 或 VCCAUX)
    * **GND~ADC~, GND~PSADC~**
    * **V~REFP~, V~REFN~**: 外部 1.25V 差分电压或内部参考
    * **V~P~, V~N~**: 专用模拟差分输入
* 外部输入
    * \_AD0P/N_ ~ \_AD15P/N_: 可作为差分模拟输入
        * 如果作为模拟管脚, 将不能再用作数字 I/O
        * 在 SYSMONE4 中, 可将所有的 N 输入连在一起, P 输入作为单端输入
            * 占用的模拟输入管脚减少, 例如 16 通道的差分输入需要 32 个管脚, 而单端输入只需要 17 个管脚
            > ![UG580_Fig2-4](/assets/UG580_Fig2-4.png)
        * 不是所有的封装的 FPGA 都有全部 16 个输入
            * 部分 FPGA 只有 0, 8, 12 个通道
        * 模拟输入的电平不能超过 V~CCO~
    * 使用:
        * 在顶层代码中, 将其连到 SYSMONE1 或 SYSMONE4, 并将其 I/O 标准设置为 ANALOG 或 ANALOG_SE
        * 例 1: VAUXP0 和 VAUXN0 作为 差分输入
            ```
            set_property PACKAGE_PIN value [get_ports VAUXP0]
            set_property IOSTANDARD ANALOG [get_ports VAUXP0]
            ```
        * 例 2: VAUXP1 使用单端模式, 作为共模的 N 端需要约束
            ```
            set_property PACKAGE_PIN value [get_ports VAUXP1]
            set_property IOSTANDARD ANALOG_SE [get_ports VAUXP1]
            set_property IOSTANDARD ANALOG [get_ports VAUXN1]
            set_property PACKAGE_PIN value [get_ports VAUXN2]
            ```
        * 使用单端时, SYSMONE4 的属性 `COMMON_N_SOURCE` 必须设置
* 内部传感器
    * 温度传感器: 1个, 单极性
        * $V = 10 \times kT/q \times \ln(10)$
            * 其中: $k = 1.3806 \times 10^{-23} J/K$; T 是开尔文温度; $q = 1.6022 \times 10^{-19} C$
    * 供电电压传感器: 多个, 单极性
        * 注意供电电压被衰减了 3 倍, 反推电压时应将 ADC 值乘以 3 倍
        * 还可以多采集用户定义的电压: V~USER0~, V~USER1~, V~USER2~, V~USER3~
            * 可以被进一步衰减到 6 倍
* ADC 工作模式
    * 单极性模式:
        * P/N 的差分电压需在 0 ~ 1V 范围内, 共模电压需在 0 ~ 0.5 V 之内
            > ![UG580_Fig2-8](/assets/UG580_Fig2-8.png)
        * 量化输出: 0 ~ 0x3FFF
    * 双极性模式
        * P/N 的差分电压不能超过 ±0.5V
        * 量化输出 (补码): 0x200 ~ 0x1FF
* ADC 采样模式
    * 连续采样
        * 26 个时钟采样一个通道
        > ![UG580_Fig2-5](/assets/UG580_Fig2-5.png)
        * 设置某些寄存器, 可以在采样一次后额外加入几个时钟
    * 触发采样
        > ![UG580_Fig2-6](/assets/UG580_Fig2-6.png)
        * BUSY 变低后, 16 个 DCLK 周期后, EOC 变高 (1 DCLK 周期) 表示转换结果已经存入对应通道的状态寄存器
        * EOS 表示一串采样完成, 由自动通道序列器和自动平均的设置决定
            * 自动通道序列器: 最后一次采样完成后拉高
            * 自动平均: 全部 (16, 64, 256 次) 采样完成后拉高
* 操作模式
    * 单通道模式
        * 设置 41h 寄存器和 40h 寄存器
    * 自动通道序列
        * 自动选择下一通道进行采样
        * 支持自动平均: 16, 64, 256 samples
    * 外部 MUX
        > ![UG580_Fig4-3](/assets/UG580_Fig4-3.png)
    * 自动警告
        * 通过设置寄存器, 自动报警
* 寄存器
    > ![UG580_Fig3-2](/assets/UG580_Fig3-2.png)
    > 详见 UG580::Page 48
* 数据接口
    * DRP
        > ![UG580_Fig3-3](/assets/UG580_Fig3-3.png)
    * JTAG
    * I2C
        > ![UG580_Fig3-11](/assets/UG580_Fig3-11.png)
        * SYSMONE4 可将 I2C_SCLK, I2C_SCLK_TS, I2C_SDA, I2C_SDA_TS, SMBALERT_TS 信号引到 FPGA 逻辑
