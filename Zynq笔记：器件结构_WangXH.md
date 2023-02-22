
<h1 style="text-align:center">Xilinx Zynq 7000：器件结构</h1>

--------------------------------------------------------------------------------
相关文档:
| 分类      | 文档名称                                   | 文档编号 |
| :-------- | :----------------------------------------- | :------- |
| 基本结构  | **Technical Reference Manual**             | UG585    |
| 封装和PCB | Packaging and Pinout Product Specification | UG865    |
| ^         | PCB Design Guide                           | UG933    |

--------------------------------------------------------------------------------
# 1. 总体结构
> ![UG585_Fig1-1](/assets/UG585_Fig1-1.png)

## 1.1. PS

### 1.1.1. APU (Application Processor Unit)
* ARM Cortex-A9 内核 (r3p0)
    * VFPv3, NEON
    * L1 Cache: 32KB I-Cache, 32KB D-Cache, 带校验
    * L2 Cache: 512 KB, 带校验
    * SCU
    * 私有定时器和看门狗: per core
    * 全局定时器: 双核共用
    * ACP
* 系统特征
    * System-Level Control Registers (**SLCRs**)
        * 管理时钟, 复位, MIO 等
    * on-chip SRAM (**OCM**)
        * 256 KB, 带校验, 双端口
        * L2级别, 但是不可 cacheable
        * 可被 CPUs, PL 和中央互联访问
    * **DMA** (PL330, r1p1)
        * 4× for PS: memory ↔ memory
        * 4× for PL: memory ↔ PL
    * **GIC**
        * CPU私有外设中断 (private peripheral interrupts, **PPI**) ×5
            * 全局定时器, 私有定时器, 私有看门狗, PL nFIQ/nIRQ
        * 软件中断 (private software generated interrupts, **SGI**) ×16
        * 共享外设中断 (shared peripheral interrupts, **SPI**):
            * 外设 → PS:
            * PL → PS: x20
        * 独立的屏蔽和极性设置
    * System watchdog timer (**SWDT**)
    * Triple timer counter (**TTC**): ×2

### 1.1.2. 内存接口
* **DDR**
* Quad-SPI, **QSPI**
* Static Memory Controller (**SMC**)
    * NAND
    * SRAM/NOR

### 1.1.3. 外设 (I/O Peripherals, IOP)
* **GPIO**
    * 54× **MIO**
    * 64× **EMIO** vs PL
* **GigE** ×2
* **USB 2.0** ×2
* **SD/SDIO** ×2
* **SPI** ×2
* **CAN** ×2
* **UART** ×2
* **I2C** ×2

### 1.1.4. PS 互联
 $\begin{aligned}
        \text{Cortex-A9} \xrightarrow{\text{I-AXI, D-AXI, CCB}} \\
        \text{PL} \xrightarrow{\text{ACP}} \end{aligned}
        > \to \text{SCU} \to \begin{cases}
            \text{OCM} \larr \text{OCM互联} \begin{cases}
                \xleftarrow{\text{AXI-HP}} \text{PL}\\
                \larr \text{中央互联} \begin{cases}
                    \larr \text{DMA}\\
                    \to \text{IOP}\\
                    \larr \text{IOP中的DMA}\\
                    \to \text{QSPI, SMC} \\
                    \xleftrightarrow{AXI-GP} \text{PL} \end{cases} \end{cases} \\
            \text{L2 Cache} \begin{cases}
                \xrightarrow{\text{M0}} \text{DDR控制器} \begin{cases}
                    \xleftarrow{\text{AXI-HP}} \text{PL}\\
                    \larr \text{中央互联} \begin{cases}
                        \larr \text{DMA}\\
                        \to \text{IOP}\\
                        \larr \text{IOP中的DMA}\\
                        \to \text{QSPI, SMC} \\
                        \xleftrightarrow{AXI-GP} \text{PL} \end{cases} \end{cases}\\
                \xrightarrow{\text{M1}} \text{中央互联} \begin{cases}
                    \larr \text{DMA}\\
                    \to \text{IOP}\\
                    \larr \text{IOP中的DMA}\\
                    \to \text{QSPI, SMC} \\
                    \xleftrightarrow{AXI-GP} \text{PL} \end{cases} \end{cases} \end{cases}$

## 1.2. PL
* CLB、BRAM、DSP、MMCM (PLL)、I/O、GTP/GTX、XADC, PCI-E

## 1.3. PS-PL互联
* EMIO
* AXI互联
    * **AXI_ACP** (64bit) ×1
    * **AXI_HP** (64bit) ×4
    * **M_AXI_GP** (32bit) ×2
    * **S_AXI_GP** (32bit) ×2
* 时钟, 复位
* 中断与事件
* DMA 外设请求接口
* DDR Arb
* PL AXI 状态
* SMC::SRAM 中断
* Debug 接口
* 配置接口: PCAP, 配置状态, 单粒子翻转, 配置接口(Program/Done/Init)

--------------------------------------------------------------------------------
# 2. 信号、接口、管脚
> ![UG585_Fig2-1](/assets/UG585_Fig2-1.png)
> PS - PL 信号的引出: 在 Vivado 中设置相关选项, 信号才可见

## 2.1. 电源管脚
> 参考 datasheet

## 2.2. PS 管脚
> ![UG585_Tab2-2](/assets/UG585_Tab2-2.png)
> ![UG585_Tab2-4](/assets/UG585_Tab2-4.png)

## 2.3. PS-PL电压转换器
* 由 **SCLR** 控制
* FSBL 阶段会自动初始化

## 2.4. PS-PL MIO-EMIO
> ![UG585_Fig2-2](/assets/UG585_Fig2-3.png)
* IOP 信号连接
    | 外设                | MIO 路由                 | EMIO 路由                              |
    | :------------------ | :----------------------- | :------------------------------------- |
    | TTC [0,1]           | 时钟输入, 波形输出       |
    | SWDT                | 时钟输入, 复位输出       |
    | SMC, Quad-SPI [0,1] | 存储器                   | x                                      |
    | SDIO [0,1]          | 50 MHz 接口              | 25 MHz 接口                            |
    | GPIO                | 54 I/O, bank[3:2]        | 64 通道 (I, O, T), bank[3:2]           |
    | USB [0,1]           | Host, device, OTG        | x                                      |
    | Ethernet [0,1]      | RGMII v2.0               | MII/GMII                               |
    | SPI [0,1]           | 50 MHz                   |
    | CAN [0,1]           | ISO 11898 -1, CAN 2.0A/B |
    | UART [0,1]          | TX/RX                    | TX, RX, DTR, DCD, DSR, RI, RTS and CTS |
    | I2C [0,1]           | SCL, SDA {0, 1}          |
    | PJTAG               | TCK, TMS, TDI, TDO       | TCK, TMS, TDI, TDO, 3-state for TDO    |

    * 使用 Vivado 分配, Vivado 自动设置相关寄存器
* IOP 接口的 IO 必须整组使用
* MIO 分配注意事项
    * MIO 和 EMIO 的速度不一定一样
    * 两个 MIO Bank 可以通过 **VMODE (MIO[8:7] 管脚)** 选择电平标准, 支持 1.8V, 2,5V, 3.3V
    * 启动模式管脚: **MIO[8:2]** 通过 20K Ohm 电阻上下拉
    * I/O 支持输入, 输出和三态
    * 如果需要从 SD 启动, 必须使用 SD/SDIO 0
    * 如果从 QSPI 启动, 必须使用 QSPI 0, 不能仅用 QSPI 1
    * **MIO[8:7]** 管脚 (**VMODE**), 仅能被配置为输出
* 默认逻辑电平
    * 当 IOP 没有连接合适的 IO 时, 会拉到默认电平, 以保护 IOP
* VREF
    * HSTL 信号 (DDR 信号使用该电平标准) 可以使用内部 VREF, 也可以使用外部

## 2.5. PS–PL AXI Interfaces
* **AXI_ACP** (64bit) ×1
* **AXI_HP** (32/64bit) ×4
* **M_AXI_GP** (32bit) ×2
* **S_AXI_GP** (32bit) ×2

## 2.6. PS-PL其他信号

### 2.6.1. 时钟和复位, PS → PL
* **FCLK [3:0]**
    * 可作为 PL 的时钟
* **FCLKCLKTRIGN [3:0]**
    * PL Clock Throttle Control, 暂不支持
* **FCLKRESETN [3:0]**
    * 寄存器: **slcr.FPGA_RST_CTRL SLCR[FPGA[3:0]_OUT_RST]**
    * 与 PS-PL 信号异步
    * 与 PL 信号异步
* 时钟、复位、其他 PS-PL 信号三者之间都是异步的
* 4个时钟、4个复位都是独立控制的

### 2.6.2. 中断
* PL → PS 中断
    * **IRQF2P[7:0]** → SPI [68:61]
    * **IRQF2P[15:8]** → SPI [91:84]
    * **IRQF2P[19:16]** → PPI: nFIQ, nIRQ (both CPUs)
* PS → PL 中断
    * **IRQP2F[27:0]**: IOP的中断 → PL

### 2.6.3. 事件信号
* Event
    * **EVENTEVENTI**: PL → PS, 从 **WFE** 中唤醒一个或两个 CPU
    * **EVENTEVENTO**: PS → PL, 某个 core 执行 **SEV** 指令后置位
* Standby
    * **EVENTSTANDBYWFE[1:0]**: PS → PL, 当 CPU 处于 **WFE** 时置位, 每位对应一个 Core
    * **EVENTSTANDBYWFI[1:0]**: PS → PL, 当 CPU 处于 **WFI** 时置位, 每位对应一个 Core
* 可用于配合ACP, 进行传输

### 2.6.4. Idle AXI: PL → PS
* 表明 PL 中没有未决的AXI传输, 用于电源管理, 不能从任何寄存器读取
* PL 信号: **FPGAIDLEN**

### 2.6.5. DDR Arb: PL → PS
* PL 信号: **DDRARB[3:0]**
* 通知 DDR 控制器中的仲裁器, AXI-HP 传输需要抢先通过

### 2.6.6. SRAM中断信号, PL → PS
* PL 信号: **EMIOSRAMINTIN**
* PL 为 SMC 产生一个中断

### 2.6.7. DMA Req/Ack
* DMA 的外设请求接口, PL 请求 PS 中的 DMA 开始传输
* 信号:
    | 类型  | 信号                    |  I/O  |
    | :---: | :---------------------- | :---: |
    | 时钟  | **DMA[3:0]ACLK**        |   I   |
    | 请求  | **DMA[3:0]DRREADY**     |   O   |
    |   ^   | **DMA[3:0]DRVALID**     |   I   |
    |   ^   | **DMA[3:0]DRTYPE[1:0]** |   I   |
    |   ^   | **DMA[3:0]DRLAST**      |   I   |
    | 确认  | **DMA[3:0]DAREADY**     |   I   |
    |   ^   | **DMA[3:0]DAVALID**     |   O   |
    |   ^   | **DMA[3:0]DATYPE[1:0]** |   O   |

## 2.7. PL IO
* 参考 datasheet

--------------------------------------------------------------------------------
# 3. APU
> ![UG585_Fig3-1](/assets/UG585_Fig3-1.png)

## 3.1. Cortex-A9
> 参见 *ARM 官方文档* 和 *[ARM Cortex-A9 总结_WangXH.md](D:/Work/Electronics/Embedded%20System/_嵌入式系统总结_WangXH/ARM%20Cortex-A9%20总结_WangXH.md)*

<!-- > ![UG585_Fig3-3](/assets/UG585_Fig3-3.png) -->
* VFPv3, NEON
* L1 Cache: 32KB I-Cache, 32KB D-Cache, 4 路组相联
* L2 Cache: 512 KB, 8 路组相联
    * 基本特征
        * 带校验, MESI 协议, 32-byte line size
        * 支持 L2-L1 exclusive 模式: 增加 L2 的可用空间和效率
        * 替换策略: 轮询或伪随机
        * 支持三种分配机制: 读分配, 写分配, 读写分配
        * Cache Lockdown: by line, by way, or by master (includes both CPU and ACP masters)
        * 16-entry 深度预取引擎
        * 支持 critical-word-first line-fill
        * multiple 256-bit line buffers
        * 转发独占请求
        * RAM Access Latency Control
            * 默认最大延迟 (111), 应设置为 001-001-010
        * Store Buffer Operation
        * 优化
            * Early Write Response
            * Pre-fetch Hints
            * Full Line of Zero Write
            * Speculative Reads of the Cortex-A9
            * Pre-fetching Operation
    * 使用
        * SDK 中的启动代码会开启和初始化 L1
            * 启动代码已启用并初始化了 latency, 启动了 prefetch
        * 中断、事件、lockdown 等需要自己设置
        * SDK 中的 BSP 库提供 API
* 内存顺序
    * 使用 MMU 设置不同内存空间的属性
        | 存储器        | 内存范围                | 属性                              |
        | ------------- | ----------------------- | --------------------------------- |
        | DDR           | 0x00000000 - 0x3FFFFFFF | Normal write-back Cacheable       |
        | PL            | 0x40000000 - 0xBFFFFFFF | Strongly Ordered                  |
        | Reserved      | 0xC0000000 - 0xDFFFFFFF | -                                 |
        | IOP           | 0xE0000000 - 0xE02FFFFF | Device Memory                     |
        | Reserved      | 0xE0300000 - 0xE0FFFFFF | Unassigned                        |
        | NAND, NOR     | 0xE1000000 - 0xE3FFFFFF | Device memory                     |
        | SRAM          | 0xE4000000 - 0xE5FFFFFF | Normal write-back Cacheable       |
        | Reserved      | 0xE6000000 - 0xF7FFFFFF | -                                 |
        | AMBA APB 外设 | 0xF8000000 - 0xF8FFFFFF | Device Memory                     |
        | Reserved      | 0xF9000000 - 0xFBFFFFFF | -                                 |
        | QSPI          | 0xFC000000 - 0xFDFFFFFF | Normal write-through cacheable    |
        | Reserved      | 0xFE000000 - 0xFFEFFFFF | -                                 |
        | OCM           | 0xFFF00000 - 0xFFFFFFFF | Normal inner write-back cacheable |
* 定制的 CP15 寄存器
    |   配置信号    |            寄存器域             |   位   | 复位值 |
    | :-----------: | :-----------------------------: | :----: | :----: |
    | MAXCLKLATENCY |   c15.Power control register    | [10:8] |  b111  |
    |    CFGEND     |           c1.SCTLR.EE           |  [25]  |   b0   |
    |    CFGNMFI    |          c1.SCTLR.NMFI          |  [27]  |   b1   |
    |    TEINIT     |           c1.SCTLR.TE           |  [30]  |   b0   |
    |    VINITHI    |           c1.SCTLR.V            |  [13]  |   b0   |
    |   CLUSTERID   |       c0.MPIDR.ClustreID        | [11:8] | b0000  |
    |    (none)     |            c0.REVIDR            | [31:0] |  0x0   |
    |    (none)     | c0.AIDR (Auxiliary ID register) | [31:0] |  0x0   |

## 3.2. APU复位

### 3.2.1. 复位功能
* 复位源:
    * 上电 (POR) 复位: **PS_POR_B**
    * 系统复位: **PS_SRST_B**
    * PS 软件复位: 控制寄存器 **slcr.PSS_RST_CTRL.SOFT_RST**
    * 看门狗复位
    * 系统调试复位: 复位效果类似于软复位, 由 JTAG 触发
    * 调试复位: 复位 Cortex-A9 中的调试单元, 包括断点和监视点, 由 JTAG 触发
* 非 POR 复位:
    * 一些寄存器的值可以不被复位
* CPU 内核复位 (仅复位内核, 不复位寄存器和外设)
    * 单个 CPU 不能复位他自己
    * 如果要复位一个 CPU 0, 另一个 CPU 或 JTAG 需按顺序执行:
        * 复位, **slcr.A9_CPU_RST_CTRL.A9_RST0** = 1
        * 停止时钟, **slcr.A9_CPU_RST_CTRL.A9_CLKSTOP0** = 1
        * 释放复位, **slcr.A9_CPU_RST_CTRL.A9_RST0** = 0
        * 开启时钟, **slcr.A9_CPU_RST_CTRL.A9_CLKSTOP0** = 0

### 3.2.2. 复位后状态
|       功能       | 复位后状态                                                                                                       |
| :--------------: | :--------------------------------------------------------------------------------------------------------------- |
|       CPU1       | 进入 **WFE**, 运行代码位于 0xFFFFFE00 to 0xFFFFFFF0, 唤醒后从 0xFFFFFFF0 开始运行                                |
|    L1 Caches     | 禁用                                                                                                             |
| L1 Caches 有效性 | 未知, 使用之前需要无效化                                                                                         |
|       MMUs       | 禁用                                                                                                             |
|       SCU        | 禁用                                                                                                             |
|     地址过滤     | 最高和最低的 1 MB 映射到 OCM, 其它映射到 L2                                                                      |
|     L2 Cache     | 禁用                                                                                                             |
|  L2 wait states  | Tag RAM and Data RAM wait states are both 7-7-7 for setup latency, write access latency, and read access latency |
| L2 Caches 有效性 | 未知, 使用之前需要无效化                                                                                         |

## 3.3. 电源管理
* CPU standby: WFI, WFE
* SCU standby: SCU的控制寄存器 **mpcore.SCU_CONTROL_REGISTER**
* L2 standby: L2的控制寄存器 **l2cpl310.reg15_power_ctrl**
* L2控制器的动态时钟门控:
    * 操作 L2 Controller Power Control register 的 Bit 1
    * L2 空闲 32个 时钟后

--------------------------------------------------------------------------------
# 4. 地址
![UG585_Tab4-1](/assets/UG585_Tab4-1.png)
* 启动后: SCU 地址过滤范围: 0x0010_0000 ~ 0xFFE0_0000, OCM 映射: 前 192 KB 位于低地址, 后 64 KB 位于高地址

## 4.1. 系统总线控制器
* CPU 和 ACP: 具有相同的地址空间, ACP 不能访问 CPU 的私有外设
* AXI_HP: 仅可访问 DDR 和 OCM
* 其他控制器: 除 CPU 寄存器外均可访问
    * DMA 控制器
    * 器件配置接口 DevC
    * 调试接口 DAP
    * 连接 S_AXI_GP[1:0] 的 PL 总线控制器
    * 带本地DMA的AHB总线主控 (Ethernet, USB, and SDIO)

## 4.2. 地址映射
> 用户不必关心具体地址, 使用 SDK 提供的设备驱动 API 进行操作
* BootROM 启动后, OCM 低 192KB 映射到低地址, 后 64K 映射到高地址
    * 如果没有修改 IP::Zynq 与此有关的设置, FSBL 不会修改此配置
* PL中自定义外设:
    * 可以在设置 IP::Zynq 时选择自动分配地址

--------------------------------------------------------------------------------
# 5. 互联
> ![UG585_Fig5-1](/assets/UG585_Fig5-1.png)

## 5.1. 特征
* $\begin{aligned} \text{Cortex-A9} \xrightarrow{\text{I-AXI, D-AXI, CCB}} \\
        \text{PL} \xrightarrow{\text{ACP}}
    \end{aligned} > \to \text{SCU} \to \begin{cases}
        \text{OCM} \larr \text{OCM互联} \begin{cases}
            \xleftarrow{\text{AXI-HP}} \text{PL}\\
            \larr \text{中央互联} \begin{cases}
                    \larr \text{DMA}\\
                    \to \text{IOP}\\
                    \larr \text{IOP中的DMA}\\
                    \to \text{QSPI, SMC} \\
                    \xleftrightarrow{AXI-GP} \text{PL}
                \end{cases}
        \end{cases} \\
        \text{L2 Cache} \begin{cases}
            \xrightarrow{\text{M0}} \text{DDR控制器} \begin{cases}
                    \xleftarrow{\text{AXI-HP}} \text{PL}\\
                    \larr \text{中央互联} \begin{cases}
                        \larr \text{DMA}\\
                        \to \text{IOP}\\
                        \larr \text{IOP中的DMA}\\
                        \to \text{QSPI, SMC} \\
                        \xleftrightarrow{AXI-GP} \text{PL}
                    \end{cases}
                \end{cases}\\
            \xrightarrow{\text{M1}} \text{中央互联} \begin{cases}
                \larr \text{DMA}\\
                \to \text{IOP}\\
                \larr \text{IOP中的DMA}\\
                \to \text{QSPI, SMC} \\
                \xleftrightarrow{AXI-GP} \text{PL}
            \end{cases}
        \end{cases}
    \end{cases}$
* 数据路径, 连通性: 详见互联框图, 或 *UG585-Table 5-{1,3}*,
* 时钟域: 详见互联框图, 或 *UG585-Table 5-2*, *UG585-Figure 5-2*
    * 设置 IP::Zynq 时使用 GUI 工具进行设置
<!-- * AXI ID: 13-bit AXI ID -->

## 5.2. QoS
* 互联的仲裁机制
    * Lv 1: 基于优先级, QoS 值越大优先级越高
    * Lv 2: 相同优先级, 基于最近授权(least recently granted, LRG)机制
* QoS (QoS-301)
    * 影响互联的仲裁机制
    * PS 中有3个 QoS
        * L2 → DDR
        * DMA → DDR
        * IOP 中的 AHB Master → 中央互联
    * 调节传输 (例如, 人为调高 PL → PS 传输的优先级)
        * 未决传输的最大数量
        * 峰值速率
        * 平均速率
        * burstiness
    * 各个端口的优先级默认相同
    * 相关寄存器: GPV (QoS301)

## 5.3. AXI HP
> ![UG585_Fig5-4](/assets/UG585_Fig5-4.png)
* 4个, 32/64 bit, 高带宽, 连至 OCM 和 DDR
* 每个端口带 2 个 FIFO (读、写, 64 bit × 128), 因此 AXI HP 接口也称为 AFI (AXI FIFO interface)
* 可编程写命令的释放阈值
* 带有 QoS
> ![UG585_Fig5-3](/assets/UG585_Fig5-3.png)

### 5.3.1. 带宽管理
* QOS优先级
    * 有两种配置方式: PS寄存器, PL信号
* FIFO 占用
    * FIFO 占用情况通过接口连接 PL, PL 可以据此动态改变优先级
* 互联 issuance 阀门
    * 可以约束其他 Master (如 CPU) 的传输
* 写 FIFO 的存储和转发
    * 写通道可以存储转发写命令, 或直通写命令
        * 如果要低延迟, 可以直通写命令
        * 如果要提高总线利用率, 可以存储转发写命令
* 32/64 位宽的选择
* 32位时 Upsizing 和 Expansion
    * Upsizing: 两个 32 位传输合并为 1 个 64 位传输
    * Expansion: 一个 32 位传输扩展为一个 64 位传输
* 32位接口的局限性
    * Burst 必须是 2 的倍数
    * 展宽的读命令优先级低

### 5.3.2. 命令交错和重排
* 读写命令都会被重排
* 可能发生交错读

### 5.3.3. 性能优化总结
1. 对于普通读写, 使用普通端口
1. 使用 32 bit 时, 有诸多限制
1. QoS 可以被 PL 控制, 也可以被 PS 控制
1. PL 可根据 FIFO 占用情况调节传输
1. ...

## 5.4. ACP
* 64 bit
* 直连 SCU, 数据可直接存放在 L2 或 OCM
* 直接操作 Cache, 比直接访问主存要快
* 仅适用于中等颗粒的加速, 例如 block 级加密加速、视频 macro-block 级处理
* 操作:
    * ACP Requests
        * ACP coherent read requests: `ARUSER[0] = 1 and ARCACHE[1] = 1`
        * ACP non-coherent read requests: `ARUSER[0] = 0 or ARCACHE[1] =0`
        * ACP coherent write requests: `AWUSER[0] = 1 and AWCACHE[1] =1`
        * ACP non-coherent write requests: `AWUSER[0] = 0 or AWCACHE[1] = 0`
* ACP Usage
    1. The CPU prepares input data for the accelerator within its local cache space.
    2. The CPU sends a message to the accelerator using one of the general purpose AXI master interfaces to the PL
    3. The accelerator fetches the data through the ACP, processes the data, and returns the result through the ACP
    4. The accelerator sets a flag by writing to a known location to indicate that the data processing is complete. Status of this flag can be polled by the processor or could generate an interrupt
* ACP Limitations
    * 不支持独占访问
    * 不支持 locked 访问
    * byte strobes 不全为 1 时需要优化
    * 通过 ACP 连续访问 OCM 时, 其他 AXI master 将没机会访问 OCM
    * 写优先级比读优先级高的 block (如 PCIe) 不能连在 ACP 上, 可能会死锁

## 5.5. AXI_GP
* 32 bit, 4 个, 2主2从
* 用于普通的数据传输

## 5.6. 回环
* PS M_AXI_GP → PL 中回环 IP 核 → AXI_HP

## 5.7. 独占访问
* 软件产生的互斥加载和存储指令, 用于实现信号量结构
* 2个A9核、ACP、AXI_HP、S_AXI_GP 的 PL 主控可以运行独占访问
* 当 Slave 包含独占访问监控器时, 才可用
    * PS 中仅有 L1 和 DDR 控制器的 4 个端口可用
    * OCM 不行

--------------------------------------------------------------------------------
# 6. 启动和配置
> ![UG585_Fig6-1](/assets/UG585_Fig6-1.png)

## 6.1. 基本过程
1. 硬件启动: 上电, 复位, PLL
    > 参见第二节
    * 如果是 POR 复位, 先执行硬件启动, 然后运行 BootROM
        * PLL 是否启用, 由 MIO[6] 控制, 反映在寄存器 `sclr.BOOT_MODE`
    * 如果是非 POR 复位, 跳过硬件启动, 立即运行 BootROM
1. Stage 0: BootROM
    * CPU 0 运行 BootROM, CPU 1 运行 `WFE`
    * 设置 APU 并运行自检, 然后读取启动模式 (寄存器 `sclr.BOOT_MODE`), 分为两类启动模式: PS Master 和 JTAG Slave
        > ![UG585_Tab6-4](/assets/UG585_Tab6-4.png)
        * BOOT_MODE 通过 MIO[6:2] 设置 (20 kΩ 电阻上下拉)
    * PS Master 启动模式
        * 从 Flash 启动, 读取 BootROM Header 并验证, 然后通常将 FSBL 复制到 OCM (FSBL 在下一阶段运行)
            * 如果启用了 Multiboot, 当一个 BootROM Header 验证不成功, 跳到下一个 32 KB 去读取 BootROM Header
            * 启动设备中的内容
                * BootROM Header (用于 BootROM)
                * FSBL/User code ELF file (用于 BootROM)
                * PL Bitstream (不是 BootROM 必需的)
                * System/Application ELF file (不是 BootROM 必需的)
        * 分为安全模式和非安全模式
            * 安全模式: 启动镜像会被 AES/HMAC 解码或认证, 然后复制到 OCM
                * AES/HMAC 位于 PL, 需要 PL 上电, 并需要调用 DevC 和 PCAP
                * 如果违规, 会 Lockdown, 只能通过 PS_POR_B 复位解除
            * 非安全模式: FSBL 可以不被复制到 OCM, 而在 QSPI 或 NOR 中运行 (execute-in-place)
    * JTAG Slave 启动模式
        * 此时 BootROM 只运行最小系统, 然后使能 JTAG
1. Stage 1: FSBL / User Code
    * 通常在 OCM 中运行, 也可设置为在 Flash 中运行
        * FSBL/User code 无论加密与否, 都不能大于192KB, 如果Execute-in-place, 无此限制
    * 可以配置为 Fallback 和 Multiboot
        * `devcfg.MULTIBOOT_ADDR [MULTIBOOT_ADDR]`
    * SDK 可以生成 FSBL, 主要工作
        1. 调用 `PS7_init()` 依次初始化
            1. MIO: 主要包括 PS_DDRIO, PS_MIO 等 IO, 相关寄存器 `sclr`
            1. PLL: 包括 ARM_PLL, DDR_PLL, IO_PLL, 相关寄存器 `sclr`
            1. Clock: 包括 CPU 和各种外设的时钟, 相关寄存器 `sclr`
            1. DDR 控制器: 相关寄存器 `ddrc`
            1. peripheral: 主要包括 PS_DDRIO, 各种外设, 相关寄存器 `sclr` 和各种外设的寄存器
            * SDK 根据 IP::Zynq 的设置自动生成代码, 具体数值可在 *SDK::xx_hw_platform_x/* 中查看
        1. 冲刷并禁用 D-Cache
        1. 注册中断并使能
        1. 测试 DDR 是否正常
        1. 初始化 PCAP, 检查 AES source key, 以准备解码镜像
        1. 设置以使复位后可以保留部分寄存器值, 获取启动模式
        1. 【如果是 JTAG 启动模式】
            1. 进行后配置 (`ps7_post_config()`)
                1. 使能 PS-PL 电平转换器
                1. 解除 PS-PL 复位信号 (FCLKRESETN)
            1. 关闭功能: 复位后可以保留部分寄存器值
            1. 交接 (`fsbl_handoff.s`)
                1. 无效化 I-Cache 和分支预测数据
                1. 禁用 I-Cache 和 MMU
        1. 【如果是 Flash 启动模式, 加载镜像】
            1. 初始化相应的外部存储器
            1. 加载镜像
                1. 根据启动模式, Multiboot 寄存器 (`devcfg.XDCFG_MULTIBOOT_ADDR_OFFSET`) 等信息, 得到 BootROM Header 的地址, 根据 BootROM Header, 得到 FSBL 的长度和 Flash 的分区表信息
                1. 将镜像从 Flash 复制到 DDR
                1. 验证, 解密
                1. 加载 Bitstream 到 PL
            1. 交接
                1. 进行后配置 (`ps7_post_config()`)
                    1. 使能 PS-PL 电平转换器
                    1. 解除 PS-PL 复位信号 (FCLKRESETN)
                1. 关闭功能: 复位后可以保留部分寄存器值
                1. 交接 (`fsbl_handoff.s`)
                    1. 无效化 I-Cache 和分支预测数据
                    1. 禁用 I-Cache 和 MMU
                    1. 跳转到 DDR 中用户程序的起始地址
    * FSBL 提供了 hooks, 可以添加额外代码
        * 加载 bitstream 前后, handoff 前, Fallback
1. Stage 2: 启动裸机系统或嵌入式操作系统
    * 运行代码
        * CPU 0: 运行代码
        * CPU 1: 启动完成后处于 `WFE`, 当接收到 Event 后 (如 `SEV` 指令 或 PL::事件接口信号), 从 0xFFFFFFF0 开始运行代码 (仅支持 ARM 指令集, 不支持 Thumb 指令集)
> -
* PL 初始化和配置
    * 启动过程独立于 PS
    * 也可以被 FSBL 或用户代码控制, 即通过 PS::DevC → PL::PCAP 为 PL 配置 Bitstream
    * PL 配置路径
        > ![UG585_Fig6-2](/assets/UG585_Fig6-2.png)
* 器件配置接口 (DevC)
    > ![UG585_Fig6-3](/assets/UG585_Fig6-3.png)
    * PCAP 桥
        * 配置 PL
        * 解码 FSBL/User code, Bitstream
    * 安全模块
        * 监视系统安全, 可产生复位
        * 监控 PL 的配置过程
        * 控制 DAP
        * 保护 BootROM 代码
    * XADC

## 6.2. 器件开机
* 供电
    * 启动模式不同, PL 供电要求也不同
    * BootROM 运行的早期阶段, 会检查 PL 是否供电
        * 如果没有, BootROM 继续运行
        * 如果上电, 清除 PL, 并等 90s, 等清除完成, 超时将 lockdown
    * 如果 BootROM 需要 PL 上电, 将最多等 90s 让其上电, 超时将关闭系统, 且不提供 error code
* 时钟和 PLL
    * 时钟管脚: PS_CLK
    * PLL: 可选是否启用
        * 如果启用, BootROM 将等待 PLL 锁定
        * 否则, 直接使用 PS_CLK, 但一些外设 (如 USB) 将不能使用
* 复位
    * 复位源
        * POR 复位: 上电复位, PS_POR_B 管脚复位
        * Non-POR 复位包括: PS_SRST_B 管脚复位, 内部复位 (写特定寄存器, 看门狗, JTAG, Debug)
    * 区别
        * POR 复位会采样 Mode_Pin, Non-POR 复位不会
        * POR 复位会复位所有寄存器, Non-POR 复位会保留部分寄存器的值
        * POR 复位可选地启动 PLL, Non-POR 复位不理会 PLL
    * 复位时序
        > ![UG585_Fig6-4](/assets/UG585_Fig6-4.png)
    * 复位原因保存在寄存器: `slcr.REBOOT_STATUS`
    * 复位效果
        ![UG585_Tab6-3](/assets/UG585_Tab6-3.png)

## 6.3. BootROM
> ![UG585_Fig6-5](/assets/UG585_Fig6-5.png)
> 芯片中内置了代码, 用户不可访问

### 6.3.1. BootROM Header
* 仅用于 Flash 启动, JTAG 启动模式不需要
* 参数表, 由 SDK 生成, 生成下载文件时放在 `.bin` 或 `.mcs` 的开头
    > ![UG585_Tab6-5-1](/assets/UG585_Tab6-5-1.png)
    > ![UG585_Tab6-5-2](/assets/UG585_Tab6-5-2.png)
* Multiboot 与 BootROM Header Search
    * 如果 BootROM Header 不正确, 地址增加 32KB, 搜索下一个
    * 可以通过写寄存器, 使 Zynq 复位, 再次进入 BootROM, 然后从指定的 BootROM Header 开始运行
    * SD 卡启动时, 仅支持 1 个 BootROM Header
    * Multiboot 应用实例: 在产品中被用于镜像文件的升级和备份, 即 IAP (In Application programming)
        * 在 Flash 中存入 2 个 BOOT.bin 镜像文件, 分别为 update image 和 golden image, 升级过程只更新 update image, 而 golden image 作为备份不被更新
        * 当 update image 出现异常, 或 update image 更新失败导致无法从 update image 启动时, 还可以切换到备份的 golden image 启动, 避免硬件宕机
        > ![zynq-multiboot实例](/assets/zynq-multiboot实例.png)

### 6.3.2. BootROM 性能
* 受很多因素影响

### 6.3.3. QSPI 启动
* 特征
    * 支持 x1,x2,x4 单器件模式
    * 支持双 SS, 8-bit 并行 IO 配置
    * 支持双 SS, 4-bit 堆叠 IO 配置
    * 支持 Execute-in-place
    * 注意
        * x4模式: 16MB, 堆叠模式下只使用 QSPI 0
        * x8模式: 32MB

### 6.3.4. SD 卡启动
* 特征
    * 支持 FAT 16/32 文件系统, 最大支持 32GB, 第一个分区需是 FAT
    * 找到 BootROM Header 后, 读 BOOT.BIN 到 OCM, 之后 CPU 从 OCM 运行
    * 不支持 BootROM Header 搜索, 不支持多启动

### 6.3.5. JTAG 启动
> ![UG585_Fig6-7](/assets/UG585_Fig6-7.png)
* 系统复位和初始化后, BootROM 执行一些初步操作, 然后 BootROM 禁止访问安全相关项目, 使能 JTAG (会根据 **sclr.BOOT_MODE[3]** 决定 JTAG 模式), 并使CPU处于 `WFE` 状态
* 使用 JTAG 启动时, 不会查找 BootROM Header, 更不会进行后续操作
* JTAG 分为三种模式:
    * 级联模式 (最常用): 从 PL JTAG 访问 DAP (PS 编程) 和 TAP (PL 编程)
    * 独立模式1 (普通): 从 PL JTAG 访问 TAP, 配置完PL后从 EMIO JTAG 访问 DAP
    * 独立模式2 (极少用): 从 PL JTAG 访问 TAP, 从 MIO PJTAG 访问 DAP

### 6.3.6. Lockdown 和 Error Codes
* Lockdown: BootROM 运行过程中出现了错误, 会用 Error Code 通知用户
* Error Code 的表示
    * INIT_B 管脚
        * 安全模式下: 所有端口禁用 (直到POR), INIT_B 用脉冲代表 Error code
            * INIT_B 一般连接一个 LED, 可以通过 LED 闪烁判断错误原因
    * JTAG访问寄存器
        * `slcr.REBOOT_STATUS [BOOTROM_ERROR_CODE]`

### 6.3.7. BootROM 完成后的状态
* APU 和 OCM状态
    * MMU, Icache, Dcache, L2 cache 禁用
    * 两个 core 都处于 supervisor 模式
    * ROM 代码不可访问
    * OCM 的低 192KB 从地址 0x0000_0000 开始, 高 64KB 从 0xFFFF_0000 开始
    * CPU 0 将跳转到 FSBL/User Code
    * CPU 1 继续保持 `WFE` 状态
* 如果是安全启动, 启动完成后 AES 可以访问, 否则不可访问

## 6.4. 器件启动和 PL 配置

### 6.4.1. 通过 PS 控制 PL
* PL 初始化
    * 通过 `devcfg.CTRL [PCFG_PROG_B]` 复位 PL
    * 通过 `devcfg.STATUS [PCFG_INIT]` PL的复位状态
    * 通过 PS::DevC → PL::PCAP 配置 PL

### 6.4.2. PS 启动时配置 PL
> ![UG585_Fig6-15](/assets/UG585_Fig6-15.png)

### 6.4.3. PS 启动后配置 PL
> ![UG585_Fig6-16](/assets/UG585_Fig6-16.png)

### 6.4.4. 注意事项
* PCAP, ICAP, JTAG 接口配置 PL 时是互斥的

## 6.5. 使用 Xilinx 工具开发
* 在线调试:
    * 如果不包含 PS 部分, 可以使用 Vivado 生成 Bitstream, 然后直接使用 JTAG 下载调试
    * 如果包含 PS 部分, 先生成 Bitstream, 加载到 SDK, 开发软件, 用 SDK 调试
        * 用 JTAG 加载 Bitstream
        * 用 JTAG 加载 `ps7_inti.tcl` 和 `<user>.elf`
* 离线下载:
    1. 生成 PL bitstream
    1. 加载到 SDK, 开发软件, 生成 `<user>.elf`
    1. 在 SDK 中, 创建 FSBL Project (使用默认即可), 生成 `<FSBL>.elf`
    1. 使用 SDK "Create Boot Image" 命令, 按顺序加载`<FSBL>.elf`, 寄存器初始化文件, bitstream, `<user>.elf`, 生成 MCS 文件
    1. 下载 MCS 到 Flash

--------------------------------------------------------------------------------
# 7. 时钟与复位

## 7.1. 时钟
> ![UG585_Fig25-1](/assets/UG585_Fig25-1.png)
* Vivado 会设置基本选项
* 3 个可编程 PLL
* 每个时钟都有门控
* CPU 时钟
    * 6:2:1 模式和 4:2:1 模式
* 互联总线时钟
* IOP 时钟
* PL 时钟:
    * FCLK[3:0]

## 7.2. 复位
> ![UG585_Fig26-1](/assets/UG585_Fig26-1.png)
> ![UG585_Fig26-2](/assets/UG585_Fig26-2.png)
* 复位源
    * 上电复位: PS_POR_B
    * 外部系统复位: PS_SRST_B
    * 系统软件复位: `sclr.PSS_RST_CTRL[SOFT_RST]`
    * 看门狗复位
        * SWDT: 系统级看门狗
        * AWDT0/AWDT1: ARM Core 专用看门狗, 可以仅复位 core, 也可以复位真个系统
    * 安全违规 lockdown
        * 只能通过 PS_POR_B 复位解除
    * Debug 复位:
        * Debug 系统复位: 系统复位
        * Debug 复位: 仅复位 SoC Debug 模块 (包括 JTAG 逻辑)
        * 不支持 JTAG 接口的 TRST 复位
    * 复位原因保存在寄存器: `slcr.REBOOT_STATUS`
* 外设复位: Xilinx SDK 提供 API
* PL 复位:
    * **FCLKRESETN[3:0]**
    * 控制寄存器 `slcr.FPGA_RST_CTRL`
    * 与 FCLK 异步

--------------------------------------------------------------------------------
# 8. 中断
> ![UG585_Fig7-1](/assets/UG585_Fig7-1.png)
> ![UG585_Fig7-2](/assets/UG585_Fig7-2.png)

* GIC 管理系统中所有的中断, 然后路由到 CPU
    * CPU 中断信号只有 2 个: nIRQ, nFIQ
* 中断源: ID 0~95
    * 软件中断 (private software generated interrupts, **SGI**) ×16
        ![UG585_Tab7-2](/assets/UG585_Tab7-2.png)
        * 通过写 SGI 中断号到 `ICDSGIR` 寄存器产生
        * 作用于当前 CPU, 或另一个 CPU, 或同时作用于两个 CPU
            * 所有的目标 CPU 都需要响应
    * CPU私有外设中断 (private peripheral interrupts, **PPI**) ×5
        ![UG585_Tab7-3](/assets/UG585_Tab7-3.png)
        * 只能触发对应的 CPU, 信号敏感类型不能改变
        * PL 的 IRQ/FIQ 信号也可以直连到 CPU
            > ![UG585_Fig7-3](/assets/UG585_Fig7-3.png)
    * 共享外设中断 (shared peripheral interrupts, **SPI**):
        ![UG585_Tab7-4-1](/assets/UG585_Tab7-4-1.png)
        ![UG585_Tab7-4-2](/assets/UG585_Tab7-4-2.png)
        * 只有 PL 中断的信号敏感类型可以更改
        * 可以设置作用于哪个 CPU, 或同时作用于两个 CPU, 但只会被一个 CPU 处理 (无需用户去 lock)
* Wait for interrupt (**WFI**):
    * CPU 处于等待状态, 等待 PL 的中断
* 中断优先级
    * 根据优先级, 将中断发射到 CPU 接口
    * 如果设置了屏蔽寄存器, 低优先级的中断将被屏蔽
    * 默认 ID 小的优先级高
    * 优先级: 用 8 bit 表示, 值越小优先级越高, 可分为两级`[7:n+1].[n:0]`, 默认为 `[7:3].[2:0]`, 小数点位置可设置
* 复位:
    * 作为 APU 的私有外设, 复位时会连同私有定时器和私有看门狗一起复位
    * 控制寄存器: `sclr.A9_CPU_RST_CTRL[PERI_RST]`
* 寄存器: `mpcore`
    * Interrupt Controller CPU (ICC):
        * CPU 接口路由和控制
        * 优先级分组设置, 优先级屏蔽
        * 中断确认: 中断开始前读取确认信息, 接收后写确认信息, 以清除此次中断
        * 中断状态: 请求由哪个 CPU 发起, 当前中断 ID, 激活和挂起的中断的最高优先级
    * Interrupt Controller Distributor (ICD):
        * ICD 控制
        * 中断使能和挂起控制: 使能, 清除使能, 挂起, 清除挂起
        * 中断状态: 哪个中断激活了
        * 中断参数: 安全状态, 优先级 (SDK 初始化为 0xA0), 目标 CPU, 敏感电平
    * `PPI_STATUS`
    * `SPI_STATUS [2:1]`
    * Software Generated Interrupts (SGI): `ICDSGIR`

--------------------------------------------------------------------------------
# 9. 存储: DDR
> ![UG585_Fig10-1](/assets/UG585_Fig10-1.png)

## 9.1. 介绍
* 支持 DDR3, DDR3L, DDR2, LPDDR-2
* 16/32 位宽 (支持 ECC), 最大 1G, 支持多芯片 (但不支持 DIMM)
    * 16b: 2 x 8b, 1 x 16b
    * 32b: 4 x 8b, 2 x 16b, 1 x 32b
* 73个专用 PS MIO, bank 502
* PL 与 DDR
    > ![UG585_Fig10-2](/assets/UG585_Fig10-2.png)
    * AXI-HP 要先经过仲裁 (包含 QoS)

## 9.2. DDRI (DDR 控制器系统接口)
> ![UG585_Fig10-3](/assets/UG585_Fig10-3.png)
* 命令
    * FIFO: 缓存地址, 长度, ID (9-bit)
    * 一个命令会被拆分成多个请求, 然后送入仲裁器
        * 多个端口之间支持乱序读
* 写端口
    * RAM: 缓存写数据, byte enable
* 读端口
    * RAM: 缓存读数据
* 支持和限制
    * 支持 INCR 和 WRAP burst, 但不支持 FIXED burst
    * burst 长度: 1 - 16, 宽度: 1,2,4,8 byte
    * 独立的读写端口, 独立的 32-bit 寻址
    * 支持 Byte, half-word 和 word 命令
    * Byte enable
    * 可用 Urgent 旁路仲裁以降低延迟, 每个端口都有 Urgent 信号
    * 仅在单一接口上支持独占访问, 接口之间不支持独占访问
    * 支持稀疏写传输 (即随机访问)
    * 支持非对齐传输
    * 任何 AXI 端口都不支持锁
    * 用 HPR 队列实现低延迟读机制

## 9.3. DDRC (DDR 控制器内核)
> ![UG585_Fig10-8](/assets/UG585_Fig10-8.png)
* 特征
    * 高效的传输机制以优化带宽和延迟
        * 包含 32 个 CAM
        * 包含 1 个 fly-by 通道, 直通 CAM, 以降低延迟
    * 高级重排引擎以最大化内存访问效率
    * 写-读地址碰撞检测
    * 遵循 AXI 顺序规则
* Row/Bank/Column 地址映射
    * 将 AXI 地址映射到 DDR 的 Row/Bank/Column
    * 通过减少页和行的切换以提高 DDR 利用率
    * 在 Xilinx Vivado 中填写正确的 DDR 信息, Xilinx SDK 会将有关信息写入相关寄存器
* 仲裁
    * 三级
        1. AXI 读写端口 (4 个读 + 4 个写) 仲裁, 机制为
            1. 老化计数器为 0 的胜出
            1. 否则, 页匹配的胜出
            1. 否则, 优先级高的 (老化计数器值小的) 胜出
            1. 否则, 轮询
        1. 读写竞争
            * 默认情况 (不使用 HPR)
                1. 如果读写端口的老化计数器都是 0, 那么上次是读, 则读操作胜出, 上次是写, 则写操作胜出
                1. 否则, 老化计数器为 0 的胜出
                1. 如果老化计数器都不是 0, 那么上次是读, 则读操作胜出, 上次是写, 则写操作胜出
            * 如果使用 HPR
                * HPR 中的读总是优先于 LPR 中的读
        1. 传输调度
            * 一直处于 **读 (写)** 状态, 直到有 **关键写 (读)** 或 **没有读 (写) 而有写 (读)请求** 时, 跳转到 **写 (读)** 状态
            * 传输状态的变换
                * Normal → Critical: 请求被挂起了很久 (`*_max_starve_x32` 个时钟周期)
                * Critical → Hard Non-Critical: 请求被执行了一段时间 (`*_xact_run_length` 次传输)
                * Hard Non-Critical → Normal: **读 (写)** 状态持续了一段时间 (`*_min_non_critical` 个时钟周期)
    * 优先级, 老化计数器, Urgent 信号
        * 带老化计数器 (向下计数器) 的 Round-Robin 机制, 优先处理等待时间长的 (老化计数器值越小表示等待的时间越长)
        * 10-bit 的优先级值作为老化计时器的初始值 (因此数值越小, 优先级越高)
        * 每个端口有 Urgent 信号, 会复位老化计数器的值为 0, 因此立即具有最高优先级
            * Urgent 源, 通过设置寄存器 `sclr.DDR_URGENT_SEL` 选择其中一个:
                * AXI 端口 (DDR port 0 除外) 中 QoS 信号的高 4 位
                * 寄存器 `sclr.DDR_URGENT`
                * PL 信号 **DDRARB[3:0]**
    * 页匹配: 指的是新的地址和上次位于同一页
    * HPR (高优先级读端口)
        * 读队列 (32 words) 可以分为 2 个组: HPR, LPR, 4 个读端口都可以分到其中一组, 默认禁用
        * 默认都被分为 LPR
        * CAM 默认只服务于 LPR, 可以控制分多少个给 HPR
    * 合并写操作
        * 向同一个地址写的操作可以合并为一个
        * 如果使能, 覆盖旧的, 只写一次
        * 如果禁用, 将新的写操作保存到一个临时 buffer, 应用流控以阻止后续传输, 等写完, 才接收新的传输
    * 信任机制
        * 挂起传输, 防止 overflow

## 9.4. DDRP (DDR 物理层)
* 包含: PHY 控制单元, DLL 单元, 读写电平逻辑
* 会自动训练 DDR 时序

## 9.5. 功能编程模型
* 在 Xilinx Vivado 中填写正确的 DDR 信息, Xilinx FSBL 自动完成初始化
    * 初始化时钟: 通过IP核配置
    * I/O buffers (DDR IOB) 初始化和校准
    * 编程 DDRC 寄存器
    * DRAM复位和初始化
    * DRAM 输入阻抗 (ODT) 校准
    * DRAM 输出阻抗 (Ron) 校准
    * DRAM Training
        * Write leveling
        * 读 DQS gate 训练
        * 读数据眼图训练

## 9.6. ECC
* 在 16-bit 模式下支持 ECC

## 9.7. 操作编程模型
* 操作模式 `mode_sts_reg.ddrc_reg_operating_mode`
    * 000: 未初始化模式
    * 001: 正常模式
    * 010: 断电模式
    * 011: 自刷新模式
    * 100 ~ 111: 只用于 LPDDR2, 表示深度掉电模式

--------------------------------------------------------------------------------
# 10. 存储: OCM
> ![UG585_Fig29-1](/assets/UG585_Fig29-1.png)

* 特征
    * 256 KB RAM, 128 KB ROM (BootROM)
        * BootROM 用户不可见
    * 128 bit 宽度
        * 为了提高效率, AXI burst 最好是 128-bit 对齐
    * 低延迟: for CPU 和 ACP
    * 内建仲裁, 固定优先级: SCU-Rd, SCU-Wr, OCM-Switch
    * 支持字节级别的校验
* 最好的传输对齐方式
    * 128-bit 对齐时, 理论上可以达到 100% 的带宽
        * 如果是偶数 burst 传输, 或两个传输可以 128 位对齐, OCM 将两个 64 位传输合并为 128 位, 从而提高访问效率
    * 不建议在 MMU 中将 OCM 设为 device 属性, 不建议使用窄的, 不可调的 ACP 访问
* 地址映射
    * 分为 4 个 64 KB, 每个都可以映射到高地址或低地址, `sclr.OCM_CFG`
    * BootROM 结束后, 前 3 个 (192 KB) 默认为低地址, 最后一个 (64 KB) 默认为高地址
    ![UG585_Tab29-1](/assets/UG585_Tab29-1.png)
    * SCU 地址过滤:
        * 如果地址过滤的范围覆盖了 OCM 范围, 即便 OCM 被映射到低地址, CPU 和 ACP 也无法访问, 其他 master 可以访问
        * BootROM 结束后, 地址映射范围是 0x0010_0000 ~ 0xFFE0_0000, 所以 CPU 和 ACP 可以访问
* 中断
    * 单 bit 校验错误
    * 多 bit 校验错误
    * 不支持的 LOCK 请求

--------------------------------------------------------------------------------
# 11. 存储: QSPI


--------------------------------------------------------------------------------
# 12. 外设: SD/SDIO


--------------------------------------------------------------------------------
# 13. 总线: DMA
> ![UG585_Fig9-1](/assets/UG585_Fig9-1.png)
> ![UG585_Fig9-2](/assets/UG585_Fig9-2.png)

## 13.1. 介绍
* 指令运行引擎
    * DMA 指令集
        * 传输指令
        * 管理指令
    * 8 条指令 Cache
        * 4-word 宽
        * cache 未命中时, 线程被卡住, 直到 cache 完成填充
        * 如果指令超过 4-byte, 将使用多个 cache
    * 代码位于系统内存中
        * DMA 可以用其 AXI master 访问
    * 灵活的碎片收集传输
    * 可以定义 AXI 传输的属性
    * 8 个通道
        * 4 个用于 PL
        * 每个通道对应一个 DMA 引擎处理器中运行的线程
            * 每个线程都有一个状态机, 允许多线程并行运行
        * 最多支持 8 个读传输和 8 个写传输
        * 仲裁: 轮询, 优先级相同
    * 最多 16 个事件信号
        * 用于通道之间, 或通道与管理器之间的交叉触发
* 读写指令队列:
    * 缓存读写传输请求
    * 深度: 各 16
* 1 个多通道 FIFO (MFIFO)
    * 缓存读写的数据
    * DMA 引擎处理器将其看作一组可变深度的, 并行的 FIFO
        * 总深度不超过 1024 byte (128 × 64 bit)
* AXI master
    * 64-bit
* 流控
    * PS 数据传输: 使用 AXI 互联
    * PL 数据传输: 使用 AXI 流控, 或 PL 外设请求接口
* 2 套控制和状态寄存器
    * 安全模式
    * 非安全模式
    * 只能选其一, 更改时需重启 DMA
* 中断 → PS/PL
    * 8 个通道中断
    * 1 个中止中断

## 13.2. 功能描述

### 13.2.1. DMA 传输
* 适用于 OCM, DDR, PL
    * PL 一般需要使用 DMAC 外设请求接口, 以控制数据流
* 一般不用于IOP, 因为无流控

### 13.2.2. AXI 传输注意事项
* 传输尺寸
    * 最大支持 64-bit
    * 超过 64-bit 时将触发一个中止信号
    * 最大 burst: 16
* 4 KB 边界
    * AXI 协议规范不允许 burst 跨越 4 KB 边界
    * 如果跨越, DMA 会将 burst 拆分成两个, 对 DMAC 通道是透明的
* Burst 类型
    * 只支持 固定地址, 增量地址
    * 不支持 wrap 地址
* 写
    * 最多可以发送 8 个未决的写
    * 在读完所需的数据之前, DMAC 不会发送写操作
* 不会产生交错的写数据 (按命令发送顺序写)
* 不支持 lock 和独占访问

### 13.2.3. DMA 管理器
* 用 APB 总线给 DMA 管理器发送指令
* 支持的命令
    * `DMAGO`: 启动指定的通道
    * `DMASEV`: 用指定的事件号, 发送事件或中断
    * `DMAKILL`: 结束线程
* 根据安全状态, 选择合适的 APB 寄存器
* 在用 `DBGCMD` 寄存器给管理器发送命令之前, 先要通过查看 `DBGSTATUS` 寄存器, 以确保 debug 处于空闲态, 否则指令被忽略
* 在发送 `DMAGO` 命令之前, 先保证内存中已经有合适的 DMA 代码
> 示例: 启动 DMA 通道线程
> 1. 创建 DMA 通道程序
> 1. 将程序存入内存
> 1. 查询 `dmac.DBGSTATUS` 寄存器, 确保管理器空闲
> 1. 写 `dmac.DBGINST0` 寄存器和 `dmac.DBGINST1` 寄存器, 共 6 个 `DMAGO` 命令
> 1. 写 0 到 `dmac.DBGCMD` 寄存器, 启动通道线程

### 13.2.4. MFIFO
* 缓存读写的数据
* DMA 引擎处理器将其看作一组可变深度的, 并行的 FIFO
    * 总深度不超过 1024 byte (128 × 64 bit)
* 支持重新对齐
    > 如: 将 0x103 的数据搬运到 0x205

### 13.2.5. 内存与 PL 之间的传输
* 外设请求接口:
    * PL 侧以此对 DMA 进行流控: 如启动 DMA 传输
    * 共有 4 个, 对应 4 个 DMA 通道
* DMA 中有 4 个外设请求 FIFO 与外设请求接口一一对应
* 外设请求接口信号
    * 时钟: DMA{3:0}_ACK
    * 请求:
        * DMA{3:0}_DRVALID
        * DMA{3:0}_DRTYPE[1:0]: 请求单传输, 请求 burst 传输, 确认 Flush 请求
        * DMA{3:0}_DRLAST
        * DMA{3:0}_DRREADY
    * 确认:
        * DMA{3:0}_DAVALID
        * DMA{3:0}_DATYPE[1:0]: 确认单传输, 确认 burst 传输, 发送 Flush 请求
        * DMA{3:0}_DALAST
        * DMA{3:0}_DAREADY
* 传输长度
    * 如果用 PL 管理长度
        * 传输线程处于等待状态, 等待 PL 发送请求
        * DMAC 无需在意传输长度
        * DAMC 根据情况运行以下指令
            * `DMALD`, `DMAST`, `DMALPEND`
            * `DMALDP<B|S>`, `DMASTP<B|S>`
                - `B`: burst 传输, `S`: 单传输
            * `DMALPEND`
    * 如果用 DMAC 管理长度
        * 单次传输的总长度必须小于 burst 传输的总长度
        * CCRn 寄存器控制传输多少个数据
        * PL 只能在 DMAC 确认 burst 传输完成后, 才能发送单传输请求
* 指令
    * `DMAWFP`: 当 DMAC 需要等待 PL 时使用
    * `DMALDP`/`DMASTP`: 当 DMAC 完成读/写传输后, 需要发送确认时使用
    * `DMAFLUSHP`: 当需要 Flush 外设请求 FIFO 时使用
        * DMAC 发送指令后, PL 需要对此进行确认, 否则 DMAC 不理会任何请求

### 13.2.6. 事件和中断
* 16 个事件信号, 前 8 个可以作为中断, 同时送往 PS 和 PL
* `MDASEV` 指令:
    * 如果 INTEN 寄存器中将某个事件/中断源设置为事件, 则产生一个事件, 对同一个事件/中断源运行 `DMAWFE` 则会清除事件
    * 如果 INTEN 寄存器中将某个事件/中断源设置为中断, 则产生一个中断

### 13.2.7. 中止
* 精确型中止: DMAC 将引起中止的地址填入寄存器
    * 通道控制寄存器中的安全违规
    * 事件的安全违规
    * PL 外设请求接口的安全违规
    * `DMAGO` 指令的安全违规
    * AXI master 接口错误
    * 运行引擎错误
* 非精确型中止: 填入寄存器中地址可能不准确
    * 数据加载错误
    * 数据存储错误
    * MFIFO 错误
    * 看门狗中止
* 中止的处理
    * 通道线程中止的处理
        * 触发中断 **IRQ #45**, **IRQP2F[28]**
        * 停止运行
        * 将引起中止的地址填入通道程序计数器
        * 不再为在读写请求队列中的命令产生 AXI 访问
        * 允许正在进行的传输完成
    * 管理器中止的处理
        * 触发中断 **IRQ #45**, **IRQP2F[28]**
    * 处理器或 PL 的动作
        * 读管理器的错误状态寄存器, 以判断管理器是否发生错误, 以及错误原因
        * 读通道的错误状态寄存器, 以判断管理器是否发生错误, 以及错误原因
        * 运行 `DMAKILL`
### 13.2.8. 安全
* 复位后, 管理器, 事件/中断源, PL 外设请求端口都是安全状态
* 管理器运行 `DMAGO` 指令时, 可以设置各个通道的安全状态
* 不同状态下, 需要控制的寄存器地址也不同, 详见 *UG585-Table 9-{6,7}*

## 13.3. DMA 指令集
* 引擎指令
    ![UG585_Tab9-14-1](/assets/UG585_Tab9-14-1.png)
    ![UG585_Tab9-14-2](/assets/UG585_Tab9-14-2.png)
* 汇编器提供的附加指令
    ![UG585_Tab9-15](/assets/UG585_Tab9-15.png)

## 13.4. 编程限制
* 考虑 4 种情况
    * 固定地址的非对齐 burst 传输
    * 大小端转换的限制
    * 在 DMA 运行的时候更新控制寄存器
    * 当 MFIFO 满时看门狗引起通道中止

--------------------------------------------------------------------------------
# 14. 外设: Timer
> ![UG585_Fig8-1](/assets/UG585_Fig8-1.png)
* 每个 CPU 各有一个私有 32-bit 计时器和 32-bit 看门狗
* 两个 CPU 共享一个 64-bit 计时器器
* PS 系统还有一个 24-bit 看门狗, 和 2 个 16-bit 三计时器/计数器

## 14.1. CPU 私有计时器和看门狗
* 32-bit, 到 0 产生中断
* 8-bit 预分频
* 可配置为 single-shot 或 auto-load
* 可配置初值
* 看门狗可触发 reset: 仅复位 CPU 或复位整个系统

## 14.2. Global Timer (GT)
* 64-bit 自动加
* 8-bit 预分频
* 到达比较值时中断, 比较值可以自动增加 (增量寄存器 > 0)
    > 可以产生周期性中断
* 寄存器只能在安全模式下复位时可设置

## 14.3. 三计时器/计数器 (TTC)
> ![UG585_Fig8-3](/assets/UG585_Fig8-3.png)
* 3 个独立的 16-bit 向上/向下计数器
    * 16-bit 预分频
    * 每个计数器对应一个中断
    * 每个计数器有 3 个匹配值寄存器
* 时钟源
    * PS: CPU_1x
    * PS: MIO_{19,31,43}, MIO_{17,29,41}
    * PL: EMIOTTC{0,1}CLKI{1,2}
* 工作模式
    * interval 模式: 从 0 增加到 interval, 或从 interval 减小到 0, 产生 interval 中断或匹配中断
    * overflow 模式: 从 0 增加到 0xFFFF, 或从 0xFFFF 减小到 0, 产生 overflow 中断或匹配中断
* 事件计时器: 测量外部信号的宽度
    * 内部有一个 16-bit 计数器, 时钟频率 CPU_1X
    * 可以设置测量正脉宽, 还是负脉宽
    * 可以设置 overflow 时停止计数还是继续计数
    * overflow 时可以产生中断
* 可以产生 PWM, 波形输出: MIO_{18,30,42}, MIO_{16,28,40}

## 14.4. 系统看门狗 (SWDT)
> ![UG585_Fig8-2](/assets/UG585_Fig8-2.png)
* 24-bit 向下计数器
* 时钟源
    * 内部 PS 总线时钟 CPU_1x
    * 内部 PL 时钟: EMIOWDTCLKI
    * 外部时钟: MIO{14,26,38,50,52}
* 可选分频: 8, 64, 512, 4096
* 超时时, 输出为其中一个或两者组合
    * 系统中断 (PS): IRQ #41
        * 可设置中断信号的宽度
    * 系统复位 (PS, PL, MIO)
        * MIO{15,27,39,51,53}
        * PL: EMIOWDTRSTO

--------------------------------------------------------------------------------
# 15. 外设: GPIO
> ![UG585_Fig14-1](/assets/UG585_Fig14-1.png)
> ![UG585_Fig14-2](/assets/UG585_Fig14-2.png)

* 54 个 MIO, 64 个 EMIO
* MIO[8:7]: VMODE, 只能做输出
* EMIO: 可以把 EMIOGPIOO 和 EMIOGPIOTN 都当作输出
* 在 Vivado 中设置好 MIO 和 EMIO 的参数, SDK 自动生成寄存器初始化代码, zynq 启动过程中自动初始化

--------------------------------------------------------------------------------
# 16. 外设: GigE
> ![UG585_Fig16-2](/assets/UG585_Fig16-2.png)
> ![UG585_Fig16-1](/assets/UG585_Fig16-1.png)

## 16.1. 介绍
* 特征
    * 兼容 IEEE 802.3-2008, 传输速率 10/100/1000 Mbps
    * 支持全双工和半双工
        * 不支持半双工千兆网
    * GMII/MII 接口到 PL, 可以进一步适配为 TBI, SGMII, 1000 Base-X, RGMII 等接口
    * MDIO 接口: 管理 PHY
    * 支持碎片收集的 DMA
    * 中断: 接收完成, 发送完成, 发生错误, 唤醒
    * 发送时自动补 0, 自动 CRC
    * 自动丢弃错误的接收帧
    * 可编程的 IPG 宽度
    * 全双工流控: 可识别 pause 帧, 也可发送 pause 帧
    * MAC 地址过滤
    * 支持 802.1Q VLAN
    * 支持 IPv4 和 IPv6
    * 支持 IP/TCP/UDP 自动校验
    * RMON/MIB 统计计数器
    * 不支持 Jumbo 帧
    * 支持 1588 v2 PTP

## 16.2. 功能描述

### 16.2.1. 发送器
* 支持全双工和半双工
    * 半双工模式下支持 CSMA/CD
        * 退避时间 (backoff): 随机延迟
        * 人为干扰信号 (jamming): 32 bit
        * 退避 16 次以上将触发错误
    * 千兆模式下 late collision 会触发异常, 并停止传输, 不再重试
* 从发送 FIFO 取数据
* 小于 60 字节时, 自动补 0
* 自动计算 CRC, 并加到帧尾
* IGP ≥ 96 bit
    * 可展宽

### 16.2.2. 接收器
* 检查报头, FCS, 对齐, 长度
* 数据被存入 FIFO 或 DMA
    * 如果发现错误, 立即停止传送并标记
* FCS 通常存储在 DMA 或 FIFO
    * 也可设置为不存储 FCS
* 一组计数器来统计信息

### 16.2.3. MAC 地址过滤
* 过滤方式:
    * 外部匹配管脚的状态
    * 指定的地址
        * 可设置 4 个
    * 利用 Hash 算法过滤
        * 计算目标地址的 Hash 值 (6-bit)
        * 根据 Hash 值设置寄存器相应的位
        > 例如 Hash 值是 34, 则 Hash_Reg[34] = 1
    * 类型 ID
        * 可设置 4 个
* 可设置接收所有的帧, 包括错误帧
* 可设置不接受 Pause 帧
* 支持 802.1Q VLAN

### 16.2.4. 唤醒
* 唤醒条件
    * magic 包
    * 符合 IP 地址的 ARP 请求
    * MAC 指定地址 1 匹配
    * 多播 Hash 匹配
* 产生唤醒中断

### 16.2.5. DMA
* DMA: 使用 packet buffer 进行传输
    * 32位宽
    * 可存多帧, 保证最大线速率
    * 更有效的利用 AHB 接口
    * 支持存储转发机制
    * 支持发送时自动计算 TCP/IP 校验和
    * 支持优先级队列
    * 发生碰撞时, 自动重新从 packet buffer 取数据
    * 自动丢弃错误的接收帧
    * 支持手动 flush RX packet
    * 当缺少 AHB 资源时, flush Rx packet
* DMA 控制器
    * 接收和发送使用各自的 buffer 描述符
        * 描述符: 内存中的 DMA 的目标地址
    * 支持 SINGLE 和 INCR (burst) 传输
        * burst 长度: 4, 8, 16
* 接收 buffer
    * 内存中的一个列表 (可以作为循环列表)
        * 2 个 word: 指示 DMA 将数据存往何处, 以及接收状态
        ![UG585_Tab16-2-1](/assets/UG585_Tab16-2-1.png)
        ![UG585_Tab16-2-2](/assets/UG585_Tab16-2-2.png)
        ![UG585_Tab16-2-3](/assets/UG585_Tab16-2-3.png)
    * 该列表的基地址必须在开始接收前被写入控制寄存器, 否则被忽略
    * 如果接收到正确帧, 数据会被传输到指定位置, 并更新接收状态, 否则, 正在使用的 buffer 被恢复 (上一个不会被恢复)
    * 可以设置当 AHB 资源不足时, 是否自动丢弃接收帧
    * 可以设置当缺少 AHB 资源时, flush Rx packet
* 发送 buffer
    * 内存中的一个列表 (可以作为循环列表)
        * 2 个 word: 指示 DMA 从何处取数据, 以及发送控制和状态
            * 发送完成后, 状态被写回 word #1
        ![UG585_Tab16-3](/assets/UG585_Tab16-3.png)
    * 被发送数据最多只能使用 128 个 buffer 去指定
    * 该列表的基地址必须在发送器被禁用或停用前写入控制寄存器, 否则被忽略
    * 如果设置为自动计算 CRC, 则同时会启用 padding
        * 否则, 不会启用 padding, 用户数据必须超过 64 字节
* packet buffer
    * GigE 中用以缓存发送或接收的数据
    * 发送
        * 4 KB, 最多支持 256 帧
    * 接收
        * 4 KB, 最多支持 256 帧

### 16.2.6. 802.3 Pause 帧
* 暂略

## 16.3. IEEE 1588
* 暂略

## 16.4. 信号和连接
* PS::MIO
    * 支持 GMII/RGMII
* PL::EMIO
    * GMII/MII 接口到 PL
    * 可以进一步适配为 TBI, SGMII, 1000 Base-X, RGMII 等接口

## 16.5. 已知的问题
* 接收时, 最后一帧有可能被卡在 RX FIFO 中无法取出
* 当 RX FIFO 中全是错误时 (如发生了 overflow), 后续的错误不会被记录
* 接收端的 DMA 设计有问题
    * 在极端情况下 DMA 带宽不够时发生
        * 例如: 接收端是小帧重负载时容易发生
    * 长时间会引起接收端死锁
    * 目前的解决办法
        * 每个一段时间 (例如 100 ms) 检查以下错误状态寄存器, 如果连续两次检查到错误, 大概率是接收端死锁了, 这时候重启一下接收端

--------------------------------------------------------------------------------
# 17. 外设: UART

--------------------------------------------------------------------------------
# 18. 外设: IIC
> ![UG585_Fig20-1](/assets/UG585_Fig20-1.png)
> ![UG585_Fig20-2](/assets/UG585_Fig20-2.png)

## 18.1. 介绍
* 支持 I2C v2
* 支持 16-byte FIFO
* 可编程的普通速率和快速率
* 支持 master 模式
    * 支持 7-bit 地址 和 10-bit 扩展地址
    * 支持拉低 SCL 到低以支持多 master
* 支持监视 slave 模式
* 支持 slave 模式
    * 支持 HOLD 以防止 overflow
    * 支持 TO 中断以避免卡死
* 中断:
    * 失去总线控制权
    * 接收 underflow
    * 发送 overflow
    * 监视的 slave 就绪
    * 传输超时
    * 传输未被确认
    * DATA 中断
    * 传输完成 (COMP 中断)

## 18.2. 功能

### 18.2.1. master 模式
* 发送
    * 当前传输完成后
        * 如果没有设置 HOLD, 立即发送 STOP 信号
        * 否则, 不会立即发送 STOP 信号, 而是持续拉低 SCL
            * 此时支持继续发送数据
            * 清除 HOLD 后, 发送 STOP 信号
    * 如果从设备回复了 NACK, 立即停止传输并产生中断
    * 如果传输中 SCL 被拉低的时间超过了超时寄存器, 产生超时中断
* 接收
    * 如果未完成的传输的尺寸 ≥ FIFO 尺寸 - 2, 当 FIFO 尺寸 == 2 时产生中断 (DATA 中断)
    * 如果未完成的传输的尺寸 < FIFO 尺寸 - 2, 当所有未决传输完成后产生中断 (COMP 中断)
    * 接受到最后一个数据后, 发送 NACK,
        * 如果没有设置 HOLD, 立即发送 STOP 信号
        * 否则, 不会立即发送 STOP 信号, 而是持续拉低 SCL
    * 如果发送地址时接收到了 NACK, 立即停止传输并产生中断
    * 如果传输中 SCL 被拉低的时间超过了超时寄存器, 产生超时中断

### 18.2.2. slave 监视模式
* 试图传输数据到从设备
* 如果发送地址后, 从设备回复了 NACK, 等一段时间 (即寄存器值有关)
* 直到收到 ACK, 发送 STOP 并产生中断

### 18.2.3. slave 模式
* 暂略

### 18.2.4. 时钟
* $\text{I2C\_SCL\_Clock} = \dfrac{\text{CPU\_1X\_Clock}}{22 * (\text{divisor\_a} + 1) * (\text{divisor\_b} + 1)}$

## 18.3. 编程
* 寄存器
    * 控制与状态: `XIICPS_CR_OFFSET`, `XIICPS_SR_OFFSET`
    * 地址, 数据: `XIICPS_ADDR_OFFSET`, `XIICPS_DATA_OFFSET`, `XIICPS_TRANS_SIZE_OFFSET`
    * 中断: `XIICPS_ISR_OFFSET`, `XIICPS_IMR_OFFSET`, `XIICPS_IER_OFFSET`, `XIICPS_IDR_OFFSET`
    * 其他: `XIICPS_SLV_PAUSE_OFFSET`, `XIICPS_TIME_OUT_OFFSET`

--------------------------------------------------------------------------------
# 19. 外设: SPI

--------------------------------------------------------------------------------
# 20. 外设: USB

--------------------------------------------------------------------------------
# 21. 外设: CAN

--------------------------------------------------------------------------------
# 22. 外设: XADC

--------------------------------------------------------------------------------
# 23. PL

## 23.1. 基本结构
* 参见 FPGA 相关文档

## 23.2. 设计
* 用途
    * 加速软件算法
    * 提高实时性
    * 可配置计算
* 性能
    * 总线带宽: 详见 *UG585-Ch.22*
    * DDR 效率: 不低于 79%
    * OCM 效率: 最大 80%

## 23.3. 测试和调试
> ![UG585_Fig23-1](/assets/UG585_Fig23-1.png)
* FTM (Fabric Trace Monitor)
    * 兼容 CoreSight

--------------------------------------------------------------------------------
# 24. 电源管理
* 设计考虑
    * 选择合适的器件
    * APU standby
    * PL低功耗控制: PL电源可以被关断
    * 时钟频率和门控
    * DDR 低功耗模式
    * 低电压 I/O
* 睡眠模式:
    * 进入:
        1. 禁用中断: `cpsid if`
        1. 配置唤醒设备
        1. 使能 L2 Cache 动态时钟门控: `l2cpl310.reg15_power_ctrl [dynamic_clk_gating_en] = 1`
        1. 使能 SCU standby 模式: `mpcore.SCU_CONTROL_REGISTER [SCU_standby_enable] = 1`
        1. 停止中央互联时钟: `slcr.TOPSW_CLK_CTRL[CLK_DIS] = 1`
        1. 使能 Cortex-A9 动态时钟门控: `cp15.power_control_register [dynamic_clock_gating] = 1`
        1. 使外部 DDR 进入 self-refresh 模式
        1. 使 PLL 进入旁路模式: `slcr.{ARM, DDR, IO}_PLL_CTRL[PLL_BYPASS_FORCE] = 1`
        1. 关闭 PLL: `slcr.{ARM, DDR, IO}_PLL_CTRL[PLL_PWRDWN] = 1`
        1. 调高时钟分频参数, 降低时钟: `slcr.ARM_CLK_CTRL[DIVISOR] = 0x3f`
        1. 运行 `WFI`
    * 退出:
        1. 恢复 CPU 时钟分频器: `slcr.ARM_CLK_CTRL[DIVISOR] = <original value>`
        1. PLL 上电: `slcr.{ARM, DDR, IO}_PLL_CTRL[PLL_PWRDWN] = 0`
        1. 等 PLL lock: `slcr.PLL_STATUS[{ARM, DDR, IO}_PLL_LOCK] == 1`
        1. 禁用 PLL 旁路模式: `slcr.{ARM, DDR, IO}_PLL_CTRL[PLL_BYPASS_FORCE] = 0`
        1. 禁用 L2 Cache 动态时钟门控: `l2cpl310.reg15_power_ctrl[dynamic_clk_gating_en] = 0`
        1. 禁用 SCU standby 模式: `mpcore.SCU_CONTROL_REGISTER[SCU_standby_enable] = 0`
        1. 启用互联时钟: `slcr.TOPSW_CLK_CTRL[CLK_DIS] = 0`
        1. 禁用 Cortex-A9 动态时钟门控: `cp15.power_control_register[dynamic_clock_gating] = 0`
        1. 使能所需外设, 包括 DDR 控制器时钟等
        1. 启用中断: `cpsie if`

--------------------------------------------------------------------------------
# 25. JTAG 和 DAP 子系统
> ![UG585_Fig27-1](/assets/UG585_Fig27-1.png)
> ![UG585_Fig27-2](/assets/UG585_Fig27-2.png)

--------------------------------------------------------------------------------
# 26. 系统测试和调试
> ![UG585_Fig28-1](/assets/UG585_Fig28-1.png)
