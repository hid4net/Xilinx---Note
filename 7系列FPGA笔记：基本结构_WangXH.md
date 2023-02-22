
<h1 style="text-align:center">Xilinx 7系列FPGA：基本结构</h1>

--------------------------------------------------------------------------------

相关文档:
| 分类      | 文档名称                                                       | 文档编号 |
| :-------- | :------------------------------------------------------------- | :------- |
| 基本结构  | **Configuration User Guide**                                   | UG470    |
| ^         | **SelectIO Resources User Guide**                              | UG471    |
| ^         | **Clocking Resources User Guide**                              | UG472    |
| ^         | **Memory Resources User Guide**                                | UG473    |
| ^         | **Configurable Logic Block User Guide**                        | UG474    |
| ^         | **DSP48E1 Slice User Guide**                                   | UG479    |
| 收发器    | GTX/GTH Transceivers User Guide                                | UG476    |
| ^         | GTP Transceivers User Guide                                    | UG482    |
| ADC       | XADC Dual 12-Bit 1 MSPS Analog-to-Digital Converter User Guide | UG480    |
| 封装和PCB | Packaging and Pinout Product Specification                     | UG475    |
| ^         | PCB Design Guide                                               | UG483    |

--------------------------------------------------------------------------------
# 1. CLB
> Configurable Logic Block User Guide (UG474)

* 6-input LUT, 双 5-input
    > FPGA逻辑资源的计算方法: 以 4-input LUT 为计量单位
* 每个Bank 50 个 CLB
* 每个 CLB 包含两个 slice: 2 个 SLICEL, 或 1 个 SLICEL + 1 个 SLICEM
    > ![UG474_Fig1-1](/assets/UG474_Fig1-1.png)
    > ![UG474_Fig2-2](/assets/UG474_Fig2-2.png)
    > ![UG474_Fig2-3](/assets/UG474_Fig2-3.png)
    > ![UG474_Fig2-4](/assets/UG474_Fig2-4.png)
* 每个 SLICEM 可以实现多种分布式 RAM (同步写, 异步读)
    * 单端口: 32 x 1, 64 x 1, 128 x 1, 256 x 1
    * 简单双端口: 32 x 6, 64 x 3
    * 双端口: 32 x 1, 64 x 1, 128 x 1
    * 四端口: 32 x 2, 64 x 1
* 每个 SLICEM 可以实现 1~128 拍的移位寄存器链
    * 支持动态长度读出, 支持静态长度读出
* 每个 slice 可以实现: 4个 4:1 MUX, 2个 8:1 MUX, 1个 16:1 MUX
    * 每个 CLB 可以实现 3 种 MUX: F7AMUX (LUT A+B), F7BMUX (LUT C+D), F8MUX
* 内建进位链

--------------------------------------------------------------------------------
# 2. 时钟资源
> Clocking Resources User Guide (UG472)

## 2.1. Overview
* 32 个全局时钟树: 整个芯片
    * 由 32 个 BUFG 驱动
    * 每个 Bank 最多可引入12个全局时钟, 由 BUFH 驱动
* I/O 和局部时钟树: 由 BUFIO 或 BUFR 驱动
    * 可引出到上边或下边最多3个 Bank, 由 BUFMR 驱动
    * 每个 Bank 最多支持4个I/O 时钟和4个局部时钟
* 时钟管理单元: MMCM, PLL
    * 每个 Bank 都有1个
![UG472_Fig1-1](/assets/UG472_Fig1-1.png)
![UG472_Fig1-4](/assets/UG472_Fig1-4.png)
![UG472_Fig1-5](/assets/UG472_Fig1-5.png)

* 互联性
    * 全局时钟输入管脚 MRCC: 2个/Bank
        * 源: 外部时钟 (==单端信号时, 必须用 _P 管脚输入, 此时 _N 管脚只能用作普通 I/O==)
        * 驱动内部: 4个BUFIO, 4个 BUFR, 2个 BUFMR, 1个 CMT, 上下2个CMT(间接), 16个 BUFG, 水平方向的 BUFH
    * 局部时钟输入管脚 SRCC: 2个/Bank
        * 源: 外部时钟 (==单端信号时, 必须用 _P 管脚输入, 此时 _N 管脚只能用作普通 I/O==)
        * 驱动内部: 4个BUFIO, 4个 BUFR, 1个 CMT, 上下2个CMT(间接), 16个 BUFG, 水平方向的 BUFH
    * BUFIO
        * 源: MRCC, SRCC, MMCM, BUFMR
        * 驱动内部: ILOGIC, OLOGIC
    * BUFR
        * 源: MRCC, SRCC, MMCM, BUFMR, 内部互联
        * 驱动内部: CMT, BUFG (不推荐)
    * BUFMR:
        * 源: MRCC, 某些GT时钟, 内部互联 (不推荐)
        * 驱动内部: CMT, BUFG (不推荐)
    * BUFG:
        * 源: MRCC, SRCC, CMT, 某些GT时钟, BUFR (不推荐), 内部互联 (不推荐), BUFG
        * 驱动内部: CMT, 某些GT时钟, BUFG, BUFH, 内部信号, I/O,
    * BUFH:
        * 源: MRCC, SRCC, CMT, BUFG, 某些GT时钟, 内部互联 (不推荐)
        * 驱动内部: CMT, 某些GT时钟, BUFG
    * CMT:
        * 源: MRCC, SRCC, BUFG, 某些GT时钟, BUFR, CMT
        * 驱动内部: BUFG, BUFIO, BUFR, BUFH, CMT

## 2.2. 时钟输入
* 全局时钟输入管脚 MRCC
    * 2个/Bank
    * 单端信号时, 必须用 _P 管脚输入, 此时 _N 管脚只能用作普通 I/O
* 局部时钟输入管脚 SRCC
    * 2个/Bank
    * 单端信号时, 必须用 _P 管脚输入, 此时 _N 管脚只能用作普通 I/O

## 2.3. 全局时钟资源
* 时钟树
* BUFG
    * glitch free
    * 原语: `BUFGCTRL`, `BUFG`, `BUFGCE`, `BUFGCE_1`, `BUFGMUX`, `BUFGMUX_1`, `BUFGMUX_CTRL`

## 2.4. 局部时钟资源
* BUFR 和 BUFIO
    * 支持源同步系统

## 2.5. CMT
* MMCM 和 PLL
    * 频率综合, 相位调整, 动态重配
    * MMCM 支持动态相位调整
    * PLL 是 MMCM 的子集
* 配合 BUFG 可以实现参考时钟切换
    > ![UG472_Fig3-2](/assets/UG472_Fig3-2.png)
    > ![UG472_Fig3-3](/assets/UG472_Fig3-3.png)

> In **Virtex-6** the **MMCM** - Mixed Mode Clock Manager - was introduced. This is a PLL with some small part of a DCM tacked on to do fine phase shifting (that's why its mixed mode - the PLL is analog, but the phase shift is digital). Thus the MMCM can do everything the PLL can do plus the phase shifting from the DCM. The V6 only had MMCMs.
> In the **7 series**, they have a combination of **PLLs and MMCMs**. Mostly this is so that there are more cells available for use (the PLLs are smaller, so they take less room on the FPGA die). ==Furthermore the PLLs are tightly bound to the I/O structures that are used for DDRx-SDRAM memory controllers (via the MIG)==.
> As for the number of them, that is determined by the size of the device. Look at the Product Table for the device you are using - it will tell you what is in the CMT (Clock Management Tile) and how many of them are available in your device.

--------------------------------------------------------------------------------
# 3. IO
> SelectIO Resources User Guide (UG471)

## 3.1. SelectIO 资源
> ![UG471_Tab1-1](/assets/UG471_Tab1-1-1.png)
> ![UG471_Tab1-1](/assets/UG471_Tab1-1-2.png)

* HP I/O 和 HR I/O
    * HP I/O: 应对高速存储、芯片到芯片接口, 最高 1.8V
    * HR I/O: 宽范围I/O标准, 最高 3.3V, 不支持 DCI, 不支持 ODELAY
* 50 IOs / Bank, 最边上两个只能用作单端信号
* V~REF~
    * 如果使用外部 V~REF~, 多功能管脚必须连到合适电压
    * 如果使用内部 V~REF~, 需要使能 `INTERNAL_VREF` 属性
        * 整个 Bank 只能设置一个电压值
        * 电压值必须是支持的 I/O 标准的参考值
        * 可选的电压值包括 (不是所有的 Bank 都能用): 0.60, 0.675, 0.75, 0.90
            * 如: `INTERNAL_VREF_BANK15 = 0.90`
* DCI (digitally controlled impedance)
    * 仅存在于 HP I/O bank 中
    * 多功能管脚 VRN (上拉到 V~CCO~) 和 VRP (下拉到 GND), 阻值与特征阻抗相同, 或2倍于特征阻抗 (与电平标准相关), 内部DCI会校准到外部电阻值
    * 支持源端匹配
        * R 匹配
            > ![UG471_Fig1-8](/assets/UG471_Fig1-8.png)
            * 支持的标准: LVDCI_15, LVDCI_18, HSLVDCI_15, HSLVDCI_18, HSUL_12_DCI, and DIFF_HSUL_12_DCI
        * R/2 匹配
            > ![UG471_Fig1-9](/assets/UG471_Fig1-9.png)
            * 支持的标准: LVDCI_DV2_15 and LVDCI_DV2_18
    * 支持终端匹配
        * 戴维南终端匹配 (DC电平为 V~CCO~/2)
            > ![UG471_Fig1-11](/assets/UG471_Fig1-11.png)
            > ![UG471_Tab1-2](/assets/UG471_Tab1-2.png)
    * DCI 校准
        * 在芯片上电启动时进行, DCI 调整会阻塞 DONE
        * 可通过 **DCIRESET** 原语复位 (仅当 `DCIUpdateMode = Quiet` 时可用)
        * **DCIRESET** 的 LOCKED 有效后, DCI 可用
        * 配置选项
            * `Match_cycle`: 使 FPGA 在 DCI 校准后才开始 startup
            * `DCIUpdateMode`: DCI 校准模式
                * `AsRequired` (默认): 芯片初始化时校准, 根据需要动态调节, Xilinx 强烈建议此选项
                * `Quiet`: 芯片初始化时校准一次, **DCIRESET** 复位时调节
    * 特殊情况
        * 如果 Bank 14 和 15 的多功能管脚使用 DCI, 需要在设计中添加 DCIRESET, 且复位后才能使用 DCI
            * 多功能管脚在芯片启动过程中会忽略 DCI 校准
    * DCI 级联
        * 同一列的 Bank 才能用级联
        * DCI 级联可以作用到整列的 Bank (不能跨越硅中介层)
        * 级联 Bank 的 V~CCO~ 和 V~REF~ 需相同
        * 使用: `set_property DCI_CASCADE {slave_banks} [get_iobanks master_bank]`
* HR Bank 使用内部电阻实现戴维南终端匹配
    * 电阻未经校准, 可能不准
    * 戴维南等效电阻值可以是: 40 Ω, 50 Ω, 60 Ω
    * 通过设置 IN_TERM 约束实现, 如: `NET "pad_net_name" IN_TERM = "UNTUNED_SPLIT_50"`
* 差分信号匹配
    * 使用内建 100Ω 电阻
    * 由 DIFF_TERM 属性决定
        * `DIFF_TERM = TRUE`
        * `DIFF_TERM = FALSE` (默认)
* 原语
    * 单端信号:
        * IBUF, IBUF_IBUFDISABLE, IBUF_INTERMDISABLE, IBUFG
        * IOBUF, IOBUF_DCIEN, IOBUF_INTERMDISABLE
        * OBUF, OBUFT
    * 差分信号:
        * IBUFDS, IBUFDS_DIFF_OUT, IBUFDS_DIFF_OUT_IBUFDISABLE, IBUFDS_DIFF_OUT_INTERMDISABLE, IBUFDS_IBUFDISABLE, IBUFDS_INTERMDISABLE, IBUFGDS, IBUFGDS_DIFF_OUT
        * IOBUFDS, IOBUFDS_DCIEN, IOBUFDS_DIFF_OUT, IOBUFDS_DIFF_OUT_DCIEN, IOBUFDS_DIFF_OUT_INTERMDISABLE, IOBUFDS_INTERMDISABLE
        * OBUFDS, OBUFTDS
* 属性和约束:
    * 基本属性: LOC, IOSTANDARD, IBUF_LOW_PWR, SLEW, DRIVE, PULLUP/PULLDOWN/KEEPER
    * 阻抗匹配: DCI_CASCADE, DIFF_TERM
    * 内部 V~REF~: INTERNAL_VREF
    * 其他: VCCAUX_IO
* 支持的 I/O 标准
    > 参见 UG471::Table 1-55
    * LVTTL
    * LVCOMS:
        * LVCMOS12, LVCMOS15, LVCMOS18, LVCMOS25, LVCMOS33
        * LVDCI, LVDCI_DV2, HSLVDCI
    * HSTL, 差分HSTL, SSTL, SSTL(18,15,135,12), 差分SSTL(18,15,135,12), SSTL(18,15,135,12) (T_DCI)
    * HSUL_12, 差分HSUL
    * LPDDR
    * LVDS, LVDS_25
        * LVDS 用于 HP bank, 若使用内部差分终端匹配 (DIFF_TERM = TRUE), V~CCO~ 只能是1.8V
        * LVDS_25 用于 HR bank, 若使用内部差分终端匹配 (DIFF_TERM = TRUE), V~CCO~ 只能是2.5V
    * RSDS, Mini-LVDS, PPDS, TMDS, BLVDS

## 3.2. SelectIO 逻辑资源
* 支持: 3态输出, 寄存输出, DDR, SAME_EDGE/SAME_EDGE_PIPELINED DDR输入, DDR 三态输出, SAME_EDGE DDR输出, IDELAY, ODELAY (HR Bank 不支持)
    > ![UG471_Fig2-1](/assets/UG471_Fig2-1.png)
    > ![UG471_Fig2-2](/assets/UG471_Fig2-2.png)

### 3.2.1. ILOGIC
* 基本结构: 组合逻辑路径 (直连内部逻辑) + DFF(DDR) + ZHOLD(仅存在与 HR Bank)
    * ILOGICE2, 位于 HP bank
        > ![UG471_Fig2-3](/assets/UG471_Fig2-3.png)
    * ILOGICE3, 位于 HR bank
        > ![UG471_Fig2-4](/assets/UG471_Fig2-4.png)
* ZHOLD: 在 system synchronous 应用中补偿保持时间
    > 时钟从 pin -> MMCM -> BUFG -> 全局网络, 会引入延时 (大约 3ns), 时钟相对往后移
    > 数据/信号从 pin 进入全局网络时, 通过 ZHOLD 引入延时 (自动校准到内部时钟插入延时), 保证数据/信号与时钟对齐
    * HR Bank 有 ZHOLD, HP Bank 没有 (可以手动用 IDELAY 实现延时)
    * ZHOLD会被自动调用, 除非:
        * IOB 的时钟源不是全局网络而是 MMCM 或 PLL
        * XDC 中使用了 IOBDELAY
        * XDC 中禁用 ZHOLD (`set_property IOBDELAY NONE [get_cells ibuf_inst_name]`)
* IDDR:
    * OPPOSITE_EDGE 模式
        > ![UG471_Fig2-5](/assets/UG471_Fig2-5.png)
    * SAME_EDGE 模式
        > ![UG471_Fig2-6](/assets/UG471_Fig2-6.png)
    * SAME_EDGE_PIPELINED 模式
        > ![UG471_Fig2-7](/assets/UG471_Fig2-7.png)

### 3.2.2. IDELAY
> ![UG471_Fig2-11](/assets/UG471_Fig2-11.png)
* 31 拍, 必须例化 IDELAYCTRL
* 时钟频率 (MHz): 200 ± 10, 300 ± 10, 400 ± 10

### 3.2.3. IDELAYCTRL
> ![UG471_Fig2-14](/assets/UG471_Fig2-14.png)
* 持续性的校正每个 IDELAY/ODELAY 的延时, 以减低工艺、电压和温度的影响
    * 时钟和 IDELAY/ODELAY 一样
    * 修正完成后 RDY 拉高
* 如果使用了 IDELAYE2 或 ODELAYE2, 必须例化 IDELAYCTRL
* 每个 Bank 一个, 在IOB 的中心位置

### 3.2.4. OLOGIC
* 基本结构: 组合逻辑路径 + DFF(DDR) + 3态控制(组合/寄存)
    ![UG471_Fig2-17](/assets/UG471_Fig2-17.png)
* ODDR:
    * OPPOSITE_EDGE 模式
        > ![UG471_Fig2-18](/assets/UG471_Fig2-18.png)
    * SAME_EDGE 模式
        > ![UG471_Fig2-19](/assets/UG471_Fig2-19.png)
    * 可以用 ODDR 输出差分时钟

### 3.2.5. ODELAY (只存在于 HP Bank)
> ![UG471_Fig2-25](/assets/UG471_Fig2-25.png)
* 31 拍, 必须例化 IDELAYCTRL
<!-- * 时钟频率 (MHz): 200 ± 10, 300 ± 10, 400 ± 10 -->

## 3.3. 高级 SelectIO 逻辑资源
> ![UG471_Fig3-3](/assets/UG471_Fig3-3.png)
### 3.3.1. ISERDESE2
> ![UG471_Fig3-1](/assets/UG471_Fig3-1.png)
> ![UG471_Tab3-3](/assets/UG471_Tab3-3.png)
* 串并转换
    * SDR 支持: 2 ~ 8 bit
    * DDR 支持: 4, 6, 8 bit
        * 两个级联可实现: 10, 14 bit
        > ![UG471_Fig3-8](/assets/UG471_Fig3-8.png)
* 支持 strobe-based 存储器接口:
    * 网络接口: DDR模式的串并转换
    * DDR3/QDR 接口: 用于 MIG, 由 MIG 工具设置
        > ![UG471_Fig3-5](/assets/UG471_Fig3-5.png)
        > 波形图
        > ```wavedrom
        > { signal:[
        >     {name:"CLK",     wave:"LHLHLHLHLHLH", phase: 0.5},
        >     {name:"OCLK",    wave:"HLHLHLHLHLHL"},
        >     {name:"CLKDIV",  wave:"l.H.l.H.l.H.l"},
        >     {name:"D",       wave:"x34563456x..", data: "A1 A2 B1 B2 C1 C2 D1 D2", phase: 1},
        >     [ "Stage1",
        >         {name:"FF0", wave:"x3.5.3.5.x..", data: "A1 B1 C1 D1", phase: 0.5},
        >         {name:"FF1", wave:"x.4.6.4.6.x.", data: "A2 B2 C2 D2", phase: 0.5},
        >     ],
        >     ["Stage2",
        >         {name:"FF2", wave:"x.3.5.3.5.x.", data: "A1 B1 C1 D1"},
        >         {name:"FF3", wave:"x..4.6.4.6.x", data: "A2 B2 C2 D2"},
        >         {name:"FF4", wave:"x...3.5.3.5.", data: "A1 B1 C1 D1"},
        >         {name:"FF5", wave:"x...4.6.4.6.", data: "A2 B2 C2 D2"},
        >     ],
        >     ["Stage3",
        >         {name:"FF6", wave:"x.....6...6..", data: "B2 D2"},
        >         {name:"FF7", wave:"x.....5...5..", data: "B1 D1"},
        >         {name:"FF8", wave:"x.....4...4..", data: "A2 C2"},
        >         {name:"FF9", wave:"x.....3...3..", data: "A1 C1"},
        >     ],
        > ], }
        > ```
        > OCLK 和 CLKDIV 必须对齐, CLK 和 OCLK 的相位根据 MIG 的校准决定
    * 异步/过采样接口: 过采样技术, 最高支持 4 倍采样
        > ![UG471_Fig3-7](/assets/UG471_Fig3-7.png)
        > 波形图
        > ```wavedrom
        > { signal:[
        >     {name:"CLK",       wave:"lH.l.H.l.H.l.H.l"},
        >     {name:"CLKB",      wave:"hl.H.l.H.l.H.l.H"},
        >     {name:"OCLK",      wave:"l.H.l.H.l.H.l.H."},
        >     {name:"OCLKB",     wave:"H.l.H.l.H.l.H.l."},
        >     {name:"DDLY",      wave:"x333344445555x..", data: "A1 A2 A3 A4 B1 B2 B3 B4 C1 C2 C3 C4", phase: 0.6},
        >     [ "Stage1",
        >         {name:"FF1  ", wave:"x3...4...5...x..", data: "A1 B1 C1"},
        >         {name:"FF2  ", wave:"x..3...4...5...x", data: "A3 B3 C3"},
        >         {name:"FF3  ", wave:"x.3...4...5...x.", data: "A2 B2 C2"},
        >         {name:"FF4  ", wave:"x...3...4...5...", data: "A4 B4 C4"},
        >     ],
        >     ["Stage2",
        >         {name:"FF1' ", wave:"x....3...4...5..", data: "A1 B1 C1"},
        >         {name:"FF2' ", wave:"x.....3...4...5.", data: "A3 B3 C3"},
        >         {name:"FF3' ", wave:"x....3...4...5..", data: "A2 B2 C2"},
        >         {name:"FF4' ", wave:"x.....3...4...5.", data: "A4 B4 C4"},
        >     ],
        >     ["Stage3",
        >         {name:"FF1''", wave:"x........3...4..", data: "A1 B1 C1"},
        >         {name:"FF2''", wave:"x........3...4..", data: "A3 B3 C3"},
        >         {name:"FF3''", wave:"x........3...4..", data: "A2 B2 C2"},
        >         {name:"FF4''", wave:"x........3...4..", data: "A4 B4 C4"},
        >     ],
        > ], }
        > ```
        > CLK: 0°, OCLK: 90°, CLKB: 180°, OCLKB: 270°

        > 如果额外增加一个超采样模块, 并使用 45°时钟, 可以实现 8 倍超采样
* Bitslip 模式
    * 仅在 NETWORKING 模式下适用
    * 在网络应用中用于 word-alignment, 通过 Bitslip 识别重复的给定数据
    * 每次 Bitslip, 接收数据会调整顺序
        > ![UG471_Fig3-11](/assets/UG471_Fig3-11.png)
        > ![UG471_Fig3-12](/assets/UG471_Fig3-12.png)

        > 实际上数据被丢弃, 只是因为数据是重复的, 看起来像是左移或右移

### 3.3.2. OSERDESE2
> ![UG471_Fig3-13](/assets/UG471_Fig3-13.png)
* 并串转换:
    * 一级可最多实现 8:1, 两级可实现 10:1 或 14:1
    * 支持 SDR 和 DDR
        * OSERDESE2 SDR 支持 2, 3, 4, 5, 6, 7, 8 bit
        * OSERDESE2 DDR 支持 4, 6, 8, 10, 14 bit
    * 三态控制:
        * 最多 4 个信号 T[1:4], 不能级联
* DDR3 模式
    * 三态信号 (T1-T4) 可以控制一串数据中哪些输出
> ![UG471_Fig3-17](/assets/UG471_Fig3-17.png)
> ![UG471_Fig3-18](/assets/UG471_Fig3-18.png)

### 3.3.3. IO_FIFO
> ![UG471_Fig3-19](/assets/UG471_Fig3-19.png)
* 4个/Bank
* 常用于连接 IOLOGIC, 也可以用作 fabric FIFO
    * IN_FIFO 可以连接 ILOGIC, 从 ILOGIC 接收 4 bit 数据, 向 fabric 输出 4/8 bit 数据
    * OUT_FIFO 可以连接 OLOGIC, 从 fabric 接收 4/8 bit 数据, 向 OLOGIC 输出 4 bit 数据
    * 两种模式
        * 4 × 4 模式 (1:1)
        * 4 × 8 / 8 × 4 模式 (1:2 / 2:1)
* IO_FIFO 有 768 bit 存储资源
    * 12 组 4 bit 或 10 组 8 bit
    * 深度: 9 (1 输入寄存器 + 7 深度 + 1 输出寄存器)
* 常用于: 跨时钟域, 2:1 serializer/deserializer

## 3.4. FPGA上电后IO的默认状态
### 3.4.1. 普通IO
* 普通IO包括两部分：
    1. 该封装中所有的通用IO引脚。
    1. 当前所选择的模式下没有使用到的所有功能复用管脚。
* 配置完成之前
    * 在7系列以后的器件中, 这些引脚的状态是根据PUDC_B引脚
* 配置完成之后
    * FPGA就进入正常工作的模式了。在配置完成之后, 普通引脚可以分为以下两种：
        1. 工程设计中使用的IO, 即在UCF或者XDC中有明确约束的IO。
        1. 其余没有使用, 也没有约束的IO。（称为Unassigned Pins）
    * 首先, 对于第一种情况, 由于已经在设计中明确设定了这些引脚的设置, 包括方向、电平、驱动能力等等, 所以在配置完成之后, 这些引脚的状态已经被设置为了预设的状态
    * 对于没有约束的IO, 又复杂一些了
    * 在Vivado中也有相同的设置, 必须在实现完成之后, 打开Implementation Design之后选择bitstream Settings, 其中在"Configure additional bitstream settings"中的Configuration栏中
    * 也可以在约束文件中设置
        ```verilog
        set_property BITSTREAM.CONFIG.UNUSEDPIN Pulldown [current_design]
        set_property BITSTREAM.CONFIG.UNUSEDPIN Pullup [current_design]
        set_property BITSTREAM.CONFIG.UNUSEDPIN Pullnone [current_design]
        ```
### 3.4.2. 专用IO
* 所有的专用配置引脚全部位于Bank0, 包括CFGBVS、M[2:0]、TCK、TMS、TDI、TDO、PRORAM_B、INIT_B、DONE以及CCLK。专用引脚的含义就是无论在配置过程中还是配置完成之后, 这些引脚无论在什么阶段都只用于配置。
* 所以对于这些引脚考虑相对比较简单, 分为输入信号和输出信号。输入信号的状态始终保持LVCMOS电平标准, 电压值为VCCO（输入信号为什么也有 电平标准要求, 需要与Input buffer的供电电压相匹配, 见图 2）。输出信号的状态始终保持LVCMOS电平标准, 电压为VCCO, 12mA drive, fast slew rate。

### 3.4.3. 功能复用IO
* 相比于其他引脚, 功能复用引脚的情况是最复杂的, 这些管脚包括与配置相关的PUDC_B、EMCCLK、CSI_B、CSO_B、DOUT、 RDWR_B、D00_MOSI、D01_DIN、D[00-31]、A[00-28]、FCS_B、FOE_B、FEW_B、ADV_B、RS0以及 RS1；以及与System Monitor相关的AD0P至AD15P、AD0N至AD15N, I2C_SDA以及I2C_SCLK。
* 为了说清楚功能复用管脚在不同阶段的状态, 将复用管脚分为以下几类：
    1. 在当前所选择的功能中使用到的功能复用管脚, 例如在选择BPI配置时的D[00-31]和A[00-28]。
    1. 在当前所选择的功能中没有使用到的功能复用管脚。例如在选择SPI配置时的D[00-31]和A[00-28]。
    1. 在完成配置之前需要作为输出或者双向, 总之有可能向外输出信号的管脚, 例如I2C_SDA和I2C_SCLK。
* 配置完成之前
    * 在FPGA上电至配置完成之前的这段时间内, 前面列出的第一类引脚, 即在当前所选择的功能中使用到的功能复用管脚, 状态等同于专用配置IO管脚。输 入信号的状态始终保持LVCMOS电平标准, 电压值为VCCO。输出信号的状态始终保持LVCMOS电平标准, 电压为VCCO, 12mA drive, fast slew rate。
    * 前面列出的第二类引脚, 即在当前所选择的功能中没有使用到的功能复用管脚, 视同于普通IO, 其状态受到HSWAPEN或者PUDC_B信号的控制, 决定是高阻还是连接弱上拉电阻。
    * 第三类引脚的情况比较复杂, 目前所知的只有I2C_SDA和I2C_SCLK, 这两个信号会在配置完成前出现一些不确定的状态。所以如果FPGA的IO还有富余, 并且要求所有连接到外部的引脚有确定的状态, 那么最好不要使用这两个引脚作为连接外设的IO。
* 配置完成之后
    * 在FPGA配置完成之后, 前面提到的三种引脚会被分成另外三类：
    * 第一类是在用户设计中明确配置了需要保留的功能引脚, 例如对于配置相关的引脚设置了Persist option属性, 这种情况下这些引脚会继续保持之前与配置相关的功能, 其状态为输入信号的状态始终保持LVCMOS电平标准, 电压值为VCCO。输出信 号的状态始终保持LVCMOS电平标准, 电压为VCCO, 12mA drive, slow slew rate。再例如设计中使用了SYSMON, I2C_SDA和I2C_SCLK引脚则继续保持DRP I2C的功能。
    * 第二类是在用户设计配置中没有要求保留其特殊功能, 这些引脚在配置完成之后会变成普通IO, 且在用户设计中没有使用到的IO。这些IO相当于Unassigned IO, 如前所述, 这些IO在配置完成之后的状态受到相应设置的影响, 可以是上拉、下拉或者Floating。
    * 第三类是在用户设计配置中没有要求保留其特殊功能, 这些引脚在配置完成之后会变成普通IO, 并且在用户设计中使用到的IO。这些IO的状态由用户设 计控制, 会在XDC或者UCF中设定。如果没有设定就会按照默认的状态, 输入端口默认状态为LVCMOS电平标准, 电压值为VCCO；输出信号默认状态为 保持LVCMOS电平标准, 电压为VCCO, 12mA drive, slow slew rate。

--------------------------------------------------------------------------------
# 4. BRAM
> Memory Resources User Guide (UG473)

## 4.1. Block RAM
* RAMB36E1: 36 Kbits, 也可作为 2 个单独的 18 Kbits
* 支持 simple dual-port (SDP) 模式, 可以在两侧使用不同的数据宽度
* 输出端口支持锁存和寄存模式, 有内部寄存器(可以优化时序)
    > ![UG473_Fig1-5](/assets/UG473_Fig1-5.png)
* 两个 36 Kbits 可以直接级联 (无需 CLB 资源), 形成一个 64K × 1 的RAM
* 支持 ECC
* 未使用的 BRAM 处于 power down 状态, 以 18K 为最小单位

## 4.2. 内建 FIFO
* 支持标准模式 和 first-word fall-through 模式
* 支持同步和异步
> ![UG473_Fig2-2](/assets/UG473_Fig2-2.png)

## 4.3. 内建 ECC
* RAMB36E1 在双端口模式下可以配置为 512 × 64 RAM, 额外的 8 bit 用作校验
    * FIFO36E1 也支持 ECC
* ECC 解码器输出两个状态位: SBITERR, DBITERR

--------------------------------------------------------------------------------
# 5. DSP
> DSP48E1 Slice User Guide (UG479)

* DSP 资源
    * 每个 DSP tile = 2 个 DSP, 每个 DSP tile 的高度 = 5 个 CLB = 1 个 Block RAM
        > ![UG479_Fig2-3](/assets/UG479_Fig2-3.png)
    * 每列 10 个 tile (20 个 DSP)

## 5.1. 结构
> ![UG479_Fig1-1](/assets/UG479_Fig1-1.png)
> ![UG479_Fig2-1](/assets/UG479_Fig2-1.png)
> ![UG479_Fig2-2](/assets/UG479_Fig2-2.png)

## 5.2. 特征
* 数学运算 (二进制补码)
    * 25 位预加器: $D \pm A$
        * 其中 A, D 都可以通过设置 INMODE 而变为 0, 因此输出可以变为 $0, D, \pm A$
    * 25 × 18 位乘法器
        * 43 位结果会被扩展到 48 位
        * 可动态旁路 (设置 `USE_MULT = NONE`)
        * 配合 C 端口, 可以实现乘-加、乘-减、乘-舍入
    * 48 位加减法
        * 3 输入 (使用乘法器时变为 2 输入)
            > ![UG479_Tab2-10](/assets/UG479_Tab2-10.png)
            * $X \in \{0, M, P, A:B \}$
            * $Y \in \{0, M, 48'hFFFFFFFFFFFF, C \}$
            * $Z \in \{0, PCIN, P, C, (PCIN >> 17), (P >> 17) \}$
        * SIMD
            * 双 24 位加, 减, 累加操作, 两个独立的 CARRYOUT 信号
            * 4 个 12 位加, 减, 累加操作, 4 个独立的 CARRYOUT 信号
            > ![UG479_Fig2-16](/assets/UG479_Fig2-16.png)
    * 48 位累加器, 也可用作同步上/下计数器
    * 桶形移位
        * 17 位右移, 用于产生更准确的结果
    * 进位
        * 支持舍入
        * 支持级联, 以实现更宽的加减法
        * 进位输入
            > ![UG479_Fig2-11](/assets/UG479_Fig2-11.png)
            > ![UG479_Tab2-11](/assets/UG479_Tab2-11.png)
        * 进位输出
            * CARRYOUT[3] 在使用乘法器或 3/4 输入加减法器时是无效的
            > ![UG479_Tab2-12](/assets/UG479_Tab2-12.png)
    * MACC 符号位
        * MULTSIGNIN 和 MULTSIGNOUT 内部级联
* 48 位逻辑
    * 两输入: X, Z
    * 按位逻辑: AND, OR, NOT, NAND, NOR, XOR, XNOR
    * 动态选择功能: ALUMODE
    > ![UG479_Tab2-13](/assets/UG479_Tab2-13.png)
* 模式检测
    > ![UG479_Fig2-17](/assets/UG479_Fig2-17.png)
    * 比较器, 产生标志信号
        * 计数器终点 (计数器自动复位)
        * 上下溢出
            > ![UG479_Fig2-18](/assets/UG479_Fig2-18.png)
        * 收敛舍入/对称舍入
* 流水线
    * 输入寄存器: A1, A2, B1, B2, C, D
    * 预加器结果寄存器 (AD)
    * 乘法结果寄存器 (M)
    * 最终结果寄存器 (P)
    * XOR 结果寄存器, 进位输出寄存器, 模式检测结果寄存器
    * 每个寄存器有独立的时钟只能和同步复位管脚, 可选极性
* 级联
    * A, B 级联
    * CARRYCASCIN 和 CARRYCASCOUT 内部级联, 支持 96 bit 累计, 加, 减操作
    * MULTSIGNIN 和 MULTSIGNOUT 内部级联, 支持 96 bit 乘加 (MACC) 操作
    * 48 位 P 总线, 支持 SIMD 加法级联
    > 例如: 在 FIR 中, 将运算结果级联到下一级的输入
* 可以动态配置运算功能
    * INMODE [4:0]
        * 双 A, B 寄存器
        * 预加器控制, 加减控制
        * pre-adder add-sub control as well as mask gates for pre-adder multiplexer functions
    * OPMODE[6:0]
        * 控制 X, Y, Z
    * ALUMODE[3:0]
        * 逻辑功能控制
        * 累加器加减控制

## 5.3. 设计
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

## 6.1. 配置接口
* JTAG
    * M[2:0] = 101
* Master SPI
    * M[2:0] = 001
    * x1, x2, x4
    * CCLK 作为输出, 频率可在 Vivado 配置选项中选择
    > ![UG470_Fig2-14](/assets/UG470_Fig2-14.png)
* 其他配置方式:
    * Master-Serial
    * Slave-Serial
    * Master SelectMAP (parallel) (x8 and x16)
    * Slave SelectMAP (parallel)  (x8, x16, and x32)
    * Master Byte Peripheral Interface (BPI) flash (x8 and x16), using parallel NOR flash
* 配置管脚 (仅考虑 JTAG 和 Master SPI)
    * 专用信号, 位于Bank 0
        | 分类  |     Pin     |  I/O   | 注意                                                                                        |
        | :---: | :---------: | :----: | :------------------------------------------------------------------------------------------ |
        | JATG  |    TCK_0    |   I    | 官方使用 2mm 间距的插针                                                                     |
        |   ^   |    TDI_0    |   I    | ^                                                                                           |
        |   ^   |    TDO_0    |   O    | ^                                                                                           |
        |   ^   |    TMS_0    |   I    | ^                                                                                           |
        |  CFG  | PROGRAM_B_0 |   I    | 复位配置, 低有效, 最好用 RC POR 复位电路(或复位芯片), 上拉(4.7k)                            |
        |   ^   |  INIT_B_0   | IO(OD) | 表明配置存储器开始初始化, 需要上拉 (4.7k)                                                   |
        |   ^   |   DONE_0    |   IO   | 是否配置成功, 高有效, 最好通过 NMOS 驱动一个 LED, 上拉(330)                                 |
        |   ^   |  M[2:0]_0   |   I    | 配置模式, Master_SPI 模式为 001, 串联电阻需 ≤ 1 kΩ                                          |
        |   ^   |  CFGBVS_0   |   I    | 如果与配置相关的 BANK 0, 14, 15 是 2.5/3.3 V供电, 需要拉到 2.5/3.3 V; 如果是 1.8V, 拉到 Gnd |
        | Flash |   CCLK_0    |   IO   | 配置时钟                                                                                    |
    * 多功能信号, 位于 Bank 14
        | 分类  |   Pin    |  I/O  | 注意                                            |
        | :---: | :------: | :---: | :---------------------------------------------- |
        | Flash | D00 - D4 |  IO   | 配置 Flash 的数据接口, 排序最好与 QSPI 保持一致 |
        |   ^   |   FCS    |   O   | 配置 Flash 的片选信号, 上拉 ≤ 4.7 kΩ            |
        | Init  |  PUDC_B  |   I   | 0 = 使能内部上拉电阻, 1 = 禁用; 不能 float      |
    * 在 Vivado 配置选项中设置 `CONFIG_VOLTAGE` 或 `CFGBVS`, 以便检查

## 6.2. DRP
* FPGA内部很多组件支持 DRP 端口, 可通过 fabric 动态更改工作参数

## 6.3. 配置过程
* 配置文件
    * `.bit`: JTAG 调试用
    * `.BIN, .MCS, .HEX`: 配置到 Flash
* 配置顺序
    > ![UG470_Fig5-2](/assets/UG470_Fig5-2.png)
    * 上电启动
        * 可使用 INIT_B 或 PROGRAM_B 延长初始化过程
        * 在此期间, Flash 需要准备好
    * 加载 Bitstream
    * Startup 步骤
        * 8-phase
            * 1-6 phase 用户可调整
                * MMCM锁定,
                * DCI匹配完成
                * 释放 DONE 管脚, 默认 4th-phase
                * 拉低 Global 3-State (GTS), 以使能 I/O, 默认 5th-phase
                * 置位 Global Write Enable (GWE) 以使能 RAM 和寄存器, 默认 6th-phase
            * 7th-phase: 置位 End Of Startup (EOS)
        * 可以通过 STARTUPE2 原语, 控制 Startup 或获取其状态

## 6.4. 安全性
* 每个 FPGA 都有一个唯一的设备识别码 (Device DNA), 不可更改
    * 可通过 JTAG 从外部读取
    * 也可通过原语 DNA_PORTE 从内部读取
* 配置文件加密
    * AES
        * Vivado 产生密钥和加密的 bitstream
        * FPGA 内部通过专用 RAM (需要备用电池) 或 eFUSE 存储密钥, 只能通过 JTAG 写入, 不能回读
* 禁止回读配置数据
    * 在 Vivado 中通过 "Edit Device Properties" 对话框设置
        * NONE: 可以回读
        * Level1: 禁止回读
        * Level2: 禁止回读和重配置
    * 通过置位 PROGRAM_B 管脚或重新上电, 可以去除回读限制

## 6.5. Vivado 配置选项
* 可通过 "Edit Device Properties" 对话框设置配置相关的选项, 将保存在 XDC 文件中
    > ![Vivado-DeviceProperties.png](/assets/Vivado-DeviceProperties.png)
* "Edit Device Properties" 对话框打开方式:
    1. 打开综合或布局布线设计 -> Menu::Tools -> Edit Device Properties
        > ![Vivado-DeviceProperties-Edit-1.png](/assets/Vivado-DeviceProperties-Edit-1.png)
    1. 打开综合或布局布线设计 -> Setting -> Edit Device Properties
        > ![Vivado-DeviceProperties-Edit-2-1.png](/assets/Vivado-DeviceProperties-Edit-2-1.png)
        > ![Vivado-DeviceProperties-Edit-2-2.png](/assets/Vivado-DeviceProperties-Edit-2-2.png)

