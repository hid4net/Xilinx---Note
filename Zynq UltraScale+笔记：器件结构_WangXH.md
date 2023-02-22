
Zynq UltraScale+总结：器件结构
----

相关文档:
分类      | 文档名称                                    | 文档编号
:-------- | :------------------------------------------ | :-------
基本结构  | **Technical Reference Manual**              | UG1085
^         | Zynq UltraScale+ Devices Register Reference | UG1087
^         | CoreLink GIC-400 Generic Interrupt Controller (r0p1) Technical Reference Manual | ARM::DDI0471B
^         | PrimeCell Generic Interrupt Controller (PL390) (r0p0) Technical Reference Manual| ARM::DDI0416B

--------------------------------------------------------------------------------
# 1. 总体结构
> ![UG1137_Fig1](/assets/UG1137_Fig1.png)
> ![UG1085_Fig1-1](/assets/UG1085_Fig15-1.png)

## 1.1. 基本结构
* **PS**
    * **Cortex-A53 APU** (application processing unit) (r0p4)
        * ARMv8, 双核/四核, EL0~3, FPU/NEON
        * ACP, ACE (AXI coherency extension), SCU, L2 cache
    * **Cortex-R5F RPU** (real-time processing unit) (r1p3)
        * ARMv7, 双核, 支持动态分支预测
        * 128KB TCM, 32/64/128-bit AXI 接口与 PL 互通 (实现低延迟应用)
        * 支持错误检测, 支持锁步进
    * Mali-400 GPU (graphics processing unit)
        * 1 个 几何处理器, 2 个像素处理器, 64KB L2 cache, OpenGL ES 1.1/2.0, OpenVG 1.1, 支持抗锯齿
        > 仅 EG 和 EV 子系列有, CG 子系列没有
    * VCU (Video control unit)
        * 视频编解码, 支持 HEVC H.265 和 AVC H.264, 支持压缩/解压缩
        > 仅 EV 子系列有
    * RF (Radio frequency)
        * 最多 16 通道的 RF-ADCs 和 RF-DACs
        > 仅 RFSoC 子系列有
* **PL**
    * PL System Monitor
    * Interlaken
    * 100G 以太网
    * GTH/GTY
    * PCI-E 3.1/4.0
    * VCU (仅 EV 器件)
    * DisplayPort 音视频接口
    * RF I/O 子系统 (仅 RFSoC 器件)
* **PMU** (platform management unit)
    * 三模冗余处理器
    * 管理电源、复位、错误
    * 初始化 PS 和 PL
        * PMU ROM 预启动代码初始化系统
        * CSU (configuration security unit) ROM 运行第一阶段的 boot loader
            * FSBL 运行开始后, CSU 进入后配置阶段, 监视各种源的篡改信号
    * 支持多种安全, 测试和调试特性
        * 支持安全启动和非安全启动
        * 支持加密镜像, 支持认证镜像, 支持加密且认证的镜像
        * PL 配置文件可以加密
* 总线与 DMA
    * AMBA 互联
    * FPD DMA, LPD DMA
* 系统中断
    * SPI, PPI, SGI
    * 异构处理器间 IPI (Inter-processor interrupts)
* DDR 控制器
    * 支持 DDR3, DDR3L, DDR4, LPDDR4
    * 最大支持 2 rank, 支持 ECC, 支持 32/64 模式
* OCM: 256KB, ECC
* 外设与IO
    * 高速外设与串行 I/O
        > ![UG1085_Fig1-3](/assets/UG1085_Fig1-3.png)
        * PS 高速外设与串行 I/O
            * 5 个高速外设
                * USB3.0 控制器 (1 个): 5 Gbps
                * 串行 GMII 接口: 支持 1 Gb/s SGMII
                * PCI-E 2.1
                * SATA 3.1
                * DisplayPort (只能作为源端, 最大支持 4k x 2k 分辨率)
            * 4 通道 GTR 收发器, 由 5 个高速外设共享
        * PL 高速外设与串行 I/O
            * 3 种高速外设
                * PCI-E 3.1/4.0: Gen4 x8 和 Gen3 x16
                * 100G 以太网: 100G MAC/PCS
                * Interlaken: 150 Gb/s
            * GTH/GTY 收发器
                * GTY: 32.75 Gb/s, 支持 25G+ backplane
    * FPD 存储器外设: QSPI, NAND, SD/SDIO/eMMC
    * LPD 低速外设: SPI, CAN, I2C, UART, SYSMON
    * MIO and EMIO
        * PS: ~ 78 MIO
        * PS vs PL: EMIO

## 1.2. 电源域
> ![UG1085_Fig1-2](/assets/UG1085_Fig1-2.png)
* 4 个电源域
    * Low-power domain (LPD)
    * Full-power domain (FPD)
    * PL power domain (PLPD)
    * Battery power domain (BPD)
* 可以相互隔离
    * 由 LPD 中的电源管理单元 PMU 管理
    * 当某个电源域的某个供电意外掉电时, 会自动被隔离

--------------------------------------------------------------------------------
# 2. 信号, 接口, 管脚
> ![UG1085_Fig2-1](/assets/UG1085_Fig2-1.png)

## 2.1. 专用管脚
* 电源
    > ![UG1085_Tab2-1-1](/assets/UG1085_Tab2-1-1.png)
    > ![UG1085_Tab2-1-2](/assets/UG1085_Tab2-1-2.png)
* 时钟, 复位, 配置
    > ![UG1085_Tab2-2-1](/assets/UG1085_Tab2-2-1.png)
    > ![UG1085_Tab2-2-2](/assets/UG1085_Tab2-2-2.png)
* JTAG 接口
    > ![UG1085_Tab2-3](/assets/UG1085_Tab2-3.png)
* MIO
    > ![UG1085_Tab2-4](/assets/UG1085_Tab2-4.png)
* PS GTR 串行通道
* DDR I/O

## 2.2. PS-PL 信号和接口
* PS-PL 电平转换
    * PMU 通过控制寄存器 **PMU_GLOBAL.REQ_PWRUP_INT_EN** 进行管理
* 处理器通信
    > ![UG1085_Tab2-5](/assets/UG1085_Tab2-5.png)
* 系统错误信号
    > ![UG1085_Tab2-6](/assets/UG1085_Tab2-6.png)
* MIO-EMIO 信号和接口
    > ![UG1085_Tab2-7](/assets/UG1085_Tab2-7.png)
* 其他各种信号和接口
    > ![UG1085_Tab2-8](/assets/UG1085_Tab2-8.png)
* 专用的流接口
    * GEM: IEEE 1588 流接口 (FIFO 接口)
    * DisplayPort 媒体接口
        > ![UG1085_Tab2-9](/assets/UG1085_Tab2-9.png)
* 时钟信号
    > ![UG1085_Tab2-10](/assets/UG1085_Tab2-10.png)
* 计时器信号
    > ![UG1085_Tab2-11](/assets/UG1085_Tab2-11.png)
* 系统调试信号和接口
    > ![UG1085_Tab2-12](/assets/UG1085_Tab2-12.png)

## 2.3. PS-PL AXI 接口
> ![UG1085_Tab2-13](/assets/UG1085_Tab2-13.png)

--------------------------------------------------------------------------------
# 3. 应用处理器 APU
> 参见 *ARM Cortex-A53 总结_WangXH.md*

> ![UG1085_Fig3-1](/assets/UG1085_Fig3-1.png)

## 3.1. 2/4 核 Cortex-A53
* 2/4 核 个 Cortex-A53, EL0~3, VFP4/NEON, cryptography
* L1 Cache: 32KB I-Cache, 32KB D-Cache, ECC
* L2 Cache: 1MB, 位于 CCI 一致性域, ECC
* 128 位 ACE 主接口
* CCI (Cache-coherent interconnect)
    * 双主控: APU 和 PL
* ACP
* TrustZone
* 计时器
* 调试和跟踪
    * Arm v8 debug features in each core
    * ETMv4 instruction trace unit for each core
    * CoreSight™ cross-trigger interface (CTI)
    * CoreSight cross-trigger matrix (CTM)
    * Debug ROM

## 3.2. APU 电源管理
* 电源岛
    > ![UG1085_Tab3-1](/assets/UG1085_Tab3-1.png)
    * 每个电源岛都可以独立的控制
* 四种电源模式
    * 正常模式
    * 待命模式
        * WFI
        * WFE
        * L2 的 WFI: 当所有内核都 WFI 时, L2 也进入 WFI
    * 单一内核关断模式
    * 内核簇关断模式

## 3.3. 时钟和复位
* 时钟子系统提供两个时钟给 APU
* 每个 APU 内核可以单独复位
    * 复位源:
        * FPD 看门狗: FPD_SWDT
        * 写寄存器
            1. 使能 PMU 的复位请求中断: 写 1 到 `PMU_GLOBAL.REQ_SWRST_INT_EN` 寄存器的某个或某几个 [APUx] 位
            1. 触发中断请求: 写 1 到 `PMU_GLOBAL.REQ_SWRST_TRIG` 寄存器的某个或某几个 [APU{0:3}] 位
        * FPD 系统复位: FPD_SRST
            * 主要应对软件调试

## 3.4. 系统寄存器
> ![UG1085_Tab3-2](/assets/UG1085_Tab3-2.png)

## 3.5. SMMU (实现系统内存虚拟化)
> ![UG1085_Fig3-7](/assets/UG1085_Fig3-7.png)
* 主要用于虚拟化, 主要包括两级
    1. 用于操作系统, 由 hypervisor 管理, VA (虚拟地址) → IPA (中间物理地址)
    1. 用于程序, 由 OS 管理, IPA → PA (物理地址)

--------------------------------------------------------------------------------
# 4. 实时处理器 RPU
> ![UG1085_Fig4-1](/assets/UG1085_Fig4-1.png)

## 4.1. 双核 Cortex-R5F
* 特征
    * ARM v7-R, VFPv3 (单双精度), 动态分支预测, 返回栈
    * L1 Cache: 32KB I-Cache w/ECC, 32KB D-Cache w/ECC
    * TCM: 128KB w/ ECC
    * 中断:
        * 延时很低
        * 支持非屏蔽快中断
    * MPU
    * 支持锁步进模式
        * RPU 锁步进模式下, 出现不匹配错误后将继续运行, 直到 PMU 介入, 或系统错误中断 RPU 操作
        * RPU 中没有专门的逻辑来处理不匹配
    * 性能监测 PMU, 调试与跟踪
    * BIST (Built-in self-test) 以检测永久损坏
    * 看门狗
* 管脚配置
    > ![UG1085_Tab4-1](/assets/UG1085_Tab4-1.png)
* 独立/锁步进
    * Split: 两个核独立工作, 性能模式
    * Lock: 锁步进, 安全模式
        * 从 RESET 异常向量入口开始设置
        > ![UG1085_Fig4-2](/assets/UG1085_Fig4-2.png)

## 4.2. 错误检测和校正
* 支持 ECC, 单位错误纠正, 双位错误检错
* 中断注入机制: 模拟产生中断触发
    * 160 个共享外设中断接口 (SPI), 都支持中断注入

## 4.3. 电源管理
> ![UG1085_Tab4-3](/assets/UG1085_Tab4-3.png)

## 4.4. 异常向量指针 EVP
* 异常向量表的基地址
    * `SCTRL.V` 控制异常入口地址:
        * 0 = 0x0000_0000 (LOVEC);
        * 1 = 0xFFFF_0000 (HIVEC), 指向 OCM
    * 默认指向高地址, FSBL 应该将其改为低地址, 不建议用户将其改回高地址
        * 高地址指向 OCM

## 4.5. 紧密耦合内存 TCM (Tightly Coupled Memory)
> ![UG1085_Fig4-4](/assets/UG1085_Fig4-4.png)

* 超低延迟, 不经过 Cache
* 每个内核 128KB TCM (64 位宽), 32位 ECC
    * ATCM: 64KB, 一般存储中断和异常代码
    * BTCM: 64KB (或 32KB + 32KB), 一般存储密集处理的数据
    > ATCM 和 BTCM 可同时访问
* 独立模式:
    * 每个内核独占 64KB ATCM 和 64KB BTCM
    * 外部 AXI 主设备可以访问TCM
* 锁步进模式:
    * 联合占用 128 KBATCM 和 128KB BTCM
    > ![UG1085_Fig4-5](/assets/UG1085_Fig4-5.png)
* 地址映射
    > ![UG1085_Tab4-5](/assets/UG1085_Tab4-5.png)
    > ![UG1085_Fig4-6](/assets/UG1085_Fig4-6.png)

--------------------------------------------------------------------------------
# 5. 图形处理器 GPU
> ![UG1085_Fig5-1](/assets/UG1085_Fig5-1.png)

## 5.1. Mali-400 MP2
* 结构
    * 1 个几何处理器: geometry processor (GP)
    * 2 个像素处理器: pixel processors (PP)
    * 共享 64KB L2 cache
    * GP 和每个 PP 都有独立的 MMU
    * 128 位 AXI 总线接口
* 特征
    * 支持 OpenGL ES 1.1 和 2.0
    * 支持 OpenVG 1.1
    * SIMD 引擎
        * 32 位浮点算法
        * 4 路 32 位指令通知运行
    * 顶点加载 DMA
    * 可容忍高数据延迟
    * 4x 和 16x 抗锯齿
    * 纹理尺寸最高 4096 x 4096 像素
    * 爱立信纹理压缩 (Ericsson texture compression, ETC) 可以减少内存带宽
    * 多种纹理格式
        * RGBA 8888, 565, 1556
        * Mono 8 和 Mono 16
        * YUV
    * 渲染引擎自动负载均衡
    * 支持第一级虚地址转换 (VA → IPA)
* 电源域 (3 个)
    * 控制寄存器, L2 cache, GP
    * PP 0
    * PP 1
* 时钟域
    * 时基: GPU_REF_CLK
    * **GPU_REF_CTRL** 寄存器控制 PLL 源和时钟频率

## 5.2. 使用
* 使用 PetaLinux 配置 GPU

--------------------------------------------------------------------------------
# 6. 平台管理单元 PMU

## 6.1. 功能
* 预启动 (上电复位初始化完成后→ CSU 复位前) 初始化
    * 清除 PMU RAM
    * 通过 PS::SYSMON 监测电源电平, 确保 CSU 和 LPD 的其他部分能正常工作
    * 初始化 PLL 的默认配置和可能的旁路
    * 执行必要的扫描清零和 MBIST (内存检测)
    * 捕获错误并产生信号 (错误 ID 可通过 JTAG 读出)
    * 解除 CSU 复位, 使 CSU 开始运行
* 电源管理
    * 在 APU 和 RPU 启动前充当代理, 并在唤醒请求后, 初始化它们的上电和重启
    * 在 APU/RPU 休眠期间, 管理电源, 执行唤醒请求
    * 随时维护系统电源、隔离墙控制和状态
        * 管理的电源
            * APU: Core[3:0], L2
            * GPU: PP[0:1]
            * RPU: Core, TCM0A, TCM0B, TCM1A, TCM1B
            * OCM: bank[3:0]
            * USB[1:0]
            * FPD, PL 电源域
        * 管理的隔离墙
            * FPD, LPD, PL (PCAP 除外)
    * 休眠期间数据保留控制
        * RAM: L2, TCM, OCM
        * DDR
    * 保存用户关心的值到全局寄存器, SRST 不丢失
* 复位处理
    * 根据触发机制, 复位各模块
    * 管理的复位
        * PS 整体, LPD 整体, FPD 整体, PL 整体
        * APU 整体或单独某个 core
        * GPU 整体或某个 PP
        * RPU 整体
        * SIOU: PCIe, SATA, DP, GEM[3:0], USB[1:0]
        * IOU 整体
        * CSU 复位 (仅能由 PMU 控制)
* 系统错误处理
    * 处理的错误
        * 电源错误: VCC_PSINTLP, VCC_PSINTFP, VCC_PSAUX, VCCO_PSDDR, VCC_PSIO[3:0]
        * 时钟错误和 PLL 错误
        * FPD/LPD
            * 看门狗错误
            * 温度错误
            * 总线超时错误
        * RPU
            * 一般错误
            * 锁步进错误
            * 硬件错误: 可纠正和不可纠正的 ECC 错误
        * PMU
            * 在预启动阶段运行 BootROM 时出错
            * ROM 运行服务时出错
            * 用户固件报告的错误
            * PMU 硬件错误
                * ROM 验证错误
                * 三模冗余错误
                * RAM ECC 不可纠错的错误
                * 寄存器地址访问错误
            * 执行预启动序列时出错, 严重, 可能表示硬件故障
        * CSU
            * 在启动过程中运行 BootROM 时出错, 包括 bitstream 认证错误
                * 错误代码被记录
            * 任一错误, 包括 ROM 认证错误
            * 看门狗错误
        * OCM/DDR ECC 错误
        * XMPU/XPPU 错误
        * PL 发送的错误信号
    * 系统复位不会清除错误状态 (**PMU_GLOBAL.ERROR_STATUS_1/2**), 只能 WTC (write to clear)
    * 可被禁用 (**PMU_GLOBAL.ERROR_EN_1/2**)
    * 可选的处理方式
        * 置位 PS_ERROR_OUT 信号
        * 产生 PMU 中断
        * 产生系统复位 (SRST)
        * 产生上电复位 (POR)
    * 错误信号可路由到 PL
* 运行内置测试和功能可靠性测试
    * MBIST, MBISR, SAFETY
* PS-PL 总线的逻辑隔离
    * M_AXI_HPM{0, 1}_FPD
    * M_AXI_HPM0_LPD

### 6.1.1. 扫描清零和内置测试
* 扫描清零
    * 每个电源岛和电源域都有扫描清零引擎
    * PMU 和 CSU 也有扫描清零引擎
        * PMU 扫描清零仅在上电复位时, 由复位状态机执行
        * PMU 的本地和全局寄存器使用自清零, 不使用扫描清零
            * 部分寄存器可以保留 SRST 之前的值
        * CSU 扫描清零仅能被 PMU 触发
    * 只能由 PMU 和 CSU 控制, 其他主控可以发出请求
        * CSU 只能在安全锁定时, 才能请求扫描清零, 无需检查执行结果
    * 执行完成后触发 PMU 中断
    * 相关控制寄存器:
        * PMU 本地寄存器: **LOGCLR_TRIG**, **LOGCLR_ACK**
* MBIST: memory built-in self test
    * 可通过设置 eFUSE 以控制是否零初始化 LPD 和 FPD 寄存器
    * 清除并检查 LPD 和 FPD 中的 RAM (其他部件可以工作), 相关 RAM 包括
        * APU, RPU 内核
        * CAN, GEM, USB
        * GPU, PCIe, SIOU
        * PS-PL AXI 接口
    * 相关控制寄存器:
        * PMU 全局寄存器: **MBIST_RST**, **MBIST_PG_EN**, **MBIST_SETPU**, **MBIST_DONE**, **MBIST_GOOD**
        * PMU 本地寄存器: **MBIST_RST**, **MBIST_PG_EN**, **MBIST_SETPU**, **MBIST_DONE**, **MBIST_GOOD**
* MBISR: memory built-in self repair
    * 相关控制寄存器:
        * PMU 本地寄存器: **MBISR_CNTRL**, **MBISR_STATUS**

### 6.1.2. 电源模式
* 电池供模式
    * 断电时保存关键信息
        * BBRAM (Battery-backed RAM) 存储安全配置的密钥
        * 实时时钟 RTC (Real-time clock)
* 低功耗运行模式
    * 覆盖模块: PMU, RPU, CSU, IOP, 除 SATA 和 DP 外的所有外设
    > ![UG1085_Tab6-1](/assets/UG1085_Tab6-1.png)
* 全功耗运行模式

## 6.2. 结构
> ![UG1085_Fig6-2](/assets/UG1085_Fig6-2.png)
### 6.2.1. 处理器
* MicroBlaze, 三模冗余, 无 Cache
* MicroBlaze
    * 5 级流水线, AXI 互联, 小端, 32 位程序计数器, 支持独占访存, 硬件算术移位
    * 无硬件乘除法, 不支持快中断
    * 支持容错
    * 支持调试

### 6.2.2. ROM & RAM
* ROM
    * 存储固有启动代码、中断向量、服务例程
    * 启动前初始化
    * 启动后提供电源管理、错误管理、检查与修复等任务
* RAM
    * 128KB, 32 位 ECC
    * 运行用户定制的 PMU 固件, 存储运行数据
    * PMU 和外部主控都可访问
        * 访问优先级
            1. PMU 处理器数据访存
            2. PMU 处理器取指
            3. 外部访问
        * 外部主控只能在 PMU 休眠时才可访问 RAM
    * 仅支持 word 写操作, 不支持 byte 写操作 (可通过读-改-写间接实现)

### 6.2.3. 全局寄存器 & 本地寄存器
* 全局寄存器 (允许 PMU 和外部主控访问)
    * 电源、隔离墙、复位的控制与状态
    * 错误捕获, 报告, 处理
    * 内置测试
    * 休眠期间数据保留控制
    * PS-PL 总线逻辑隔离
* 本地寄存器 (仅允许 PMU 访问)

### 6.2.4. 中断
* 23 个中断, 其中 4 个来自 IPI
> ![UG1085_Tab6-11-1](/assets/UG1085_Tab6-11-1.png)
> ![UG1085_Tab6-11-2](/assets/UG1085_Tab6-11-2.png)

### 6.2.5. IO 模块
* GPI
    * GPI 0: PMU 专用, 容错状态寄存器
    * GPI 1: 监测唤醒请求、电源管理请求
        > ![UG1085_Tab6-6](/assets/UG1085_Tab6-6.png)
    * GPI 2: 监测电源控制请求、复位请求
        > ![UG1085_Tab6-7-1](/assets/UG1085_Tab6-7-1.png)
        > ![UG1085_Tab6-7-2](/assets/UG1085_Tab6-7-2.png)
    * GPI 3: 监测 PL 输入
* GPO
    * GPO 0: 描述 PMU 特征
        > ![UG1085_Tab6-8-1](/assets/UG1085_Tab6-8-1.png)
        > ![UG1085_Tab6-8-2](/assets/UG1085_Tab6-8-2.png)
    * GPO 1: 控制 MIO 信号
        > ![UG1085_Tab6-9](/assets/UG1085_Tab6-9.png)
    * GPO 2: PMU 产生的请求和确认
        > ![UG1085_Tab6-10](/assets/UG1085_Tab6-10.png)
    * GPO 3: 输出到 PL
* PIT (programmable interval timer)
    * 4 个, 32 位可编程间隔计数器

### 6.2.6. 接口
* 32 位 AXI 主接口 → LPD
    * 可访问 PS 部分资源: SLCR 寄存器, IPI 模块
* 32 位 AXI 从接口 ← LPD
    * 可访问 PMU 全局寄存器和 PMU RAM
    * 可以设置为一致性访问 (**PMU_GLOBAL.GLOBAL_CNTRL.Coherent**)
        * 被当作 "device, buffered"
    * 仅允许安全访问
* 电源控制接口 → PS 中所有的电源岛
* 唤醒接口 ←
    * 中断
    * 器件复位控制接口
    * MBIST, MBISR 控制接口
    * 各种其他接口, 包括电源监测接口和几个 MIO
    * 错误捕获和传递接口
* 错误接口 vs PL
    * PL 错误 → PMU
        * pmu_pl_err, 4 个, 输入
    * PMU 错误信号 → PL 和 JTAG
        * pmu_error_to_pl, 47 个, 输出
* MIO
    * 输入: MIO[31:26] ↔ GPI1[15:10]
    * 输出: MIO[37:32] ↔ GPO1[5:0]
        * MIO[32]: 控制 FPD::VCC_PSINTFP 电源, 高有效
        * MIO[33]: 控制 PL::VCCINT 电源, 高有效

### 6.2.7. 时钟与复位
* 时钟
    * SysOsc 时钟, 180 MHz±15%
    * 由 PS::SYSMON 内环形振荡器产生
    * V~CC_PSAUX~ 上电后才输出
* 复位
    * 上电复位: POR (内部和外部)
        * 清除 PMU 所有状态
        * 所有电源都打开, 隔离墙都关闭
        * 运行扫描和 BIST, 清除 LPD 和 FPD 的功能
    * 系统复位 SRST
        * 仅复位 PMU 处理器子系统、PMU 互联、一部分本地寄存器和全局寄存器
        * 大部分寄存器的状态保留
        * PS 电源状态保留

## 6.3. 操作
* 操作 PMU
    * 主控写 PMU 全局寄存器 → 产生中断 → PMU 处理
    * 如果出错, 查看 **PMU_GLOBAL.ERROR_STATUS_2.PMU_SERVICE**
* 电源管理
    * 平常 APU 和 RPU 是电源的主控, 当它们被关闭时, PMU 管理电源
    * 上电:
        * PMU控制: 写寄存器 **PMU_GLOBAL.REQ_PWRUP_TRIG** 和对应的中断控制寄存器
    * 断电:
        * APU/RPU 控制
        * PMU 控制: 写寄存器 **PMU_GLOBAL.REQ_PWRDWN_TRIG** 和对应的中断控制寄存器
    * 唤醒
        * 方式
            * 固定唤醒: 直接唤醒, 通过外设中断或计时器中断
            * 按需唤醒: 通过 PMU 操作
            * 唤醒代码: 在 PMU.RAM 中运行用户编写的代码
        * 唤醒源
            * APU/RPU
            * MIO: 连接外部事件, 外部以太网 PHY, 外部 CAN PHY 等
            * USB
            * 以太网: PHY 或 MAC
            * 实时时钟 RTC
            * DAP
            * GIC 代理: 中断路由到 PMU
    * 深度睡眠
        > ![UG1085_Tab6-14](/assets/UG1085_Tab6-14.png)
        * 可通过 GPI 或 RTC 唤醒 (也可以通过 USB 或 PHY 唤醒, 功耗多些)
    * 隔离墙
        * 信号被钳位到特定值
* 复位请求
    * PMU控制: 写寄存器 **PMU_GLOBAL.REQ_SWRST_TRIG**

## 6.4. PMU 编程
* 另一主控通过 IPI0 触发 PMU 进入睡眠模式, 加载代码, 唤醒后切换到 RAM 执行
    > ![UG1085_Fig6-6](/assets/UG1085_Fig6-6.png)

--------------------------------------------------------------------------------
# 7. 功能可靠性
* 保障系统正常运行, 免受以下错误的影响
    * 硬件错误: 制造缺陷
    * 随机错误: 老化、环境

## 7.1. 特征
* 单点错误检测
    * ECC 保护: OCM, PMU-RAM, CSU-RAM, RPU L1 cache, TCM
        * 地址解码错误检测
        * 独立的 RAM 来存储 ECC 诊断信息和数据
        * 4:1 或更大的存储位交错
    * 每次启动时, 对 CSU BootROM 进行哈希认证 (SHA3)
    * RPU 锁步进
    * PMU 和 CSU 三模冗余
    * XMPU 和 XPPU 保护内存空间
    * 看门狗: LPD 和 FPD 提供看门狗
* 一般性故障 (common cause failures, CCF) 检测
    * 系统监测: 电压、温度、时钟频率
    * 错误管理: 由 PMU 处理, 作为中断 (可路由到 PL), 错误状态寄存器
        * RPU 锁步进模式下, 出现不匹配错误后将继续运行, 直到 PMU 介入, 或系统错误中断 RPU 操作
        * RPU 中没有专门的逻辑来处理不匹配
    * 通过 PMU 激活 CCF 监测: MBIST, 扫描清零, 复位, 电源控制
    * 挂起保护: 局部复位时清除未完成的传输
    * 亚稳态错误: 跨时钟时使用冗余触发器
    * 老化错误: 大的片上 on-chip variation, OCV 裕度, 缓解老化效应
* 潜伏性故障检测
    * 启动时 LBIST (logic BIST) 检查 XMPU, 锁步进, ECC 检查器
        * 永久控制开关: **eFUSE.LBIST_EN**
    * 在启动时, 通过 MBIST 在全速率下检查 LPD 中所有的存储器
        * 大多数存储器可在运行时按需检测
    * 通过软件测试库 (STL) 评估 TCM、OCM、PMU RAM、R5F lockstep、PMU、XMPU、XPPU、时钟、电压、温度的功能和状态
* 隔离检测
    * 支持将 LPD 与系统的其他部分隔离
    * 灵活的复位管理 (由 PMU 管理)
        * 允许处理器进行冗余处理
        * LDP, FPD, PL, PS 有各自的复位
    * 独立的电源域
        * LPD, FPD, PL
    * PL 接口上内置 AXI 超时
* 其他检测
    * DDR 接口支持 ECC
    * APU L1 D-Cache, L2 cache 支持 ECC
    * APU L1 I-Cache 支持奇偶校验
    * QoS 管理
    * PL 和 APU 多核可以提供冗余处理
    * 通过 MBIST 测试 FPD 的内存
    * PL 可靠特性
        * HFT 通道兼容性
        * 错误 logging
        * PS 复位时, PL 可以保持活跃

## 7.2. 软件测试库
* LPD 中的可靠子系统寄存器 (多数模块都有)
    * 可以静态更新, 或不定时的更新
    * 通过与 Golden 副本比较, 检测错误
* RPU 中的 GIC
    * 注入中断, 检测响应
* LPD 中的计数器
    * 周期性检查 TTC 和 SWDT 的状态
* LPD 存储器中的多位错误
    * 通过读 LPD 存储器, 触发纠错错误, 然后擦除, 防止多位错误累积
* LPD DMA
    * 方法1: 周期性检查 STL 函数
    * 方法2: 对 DMA 传输的数据进行 CRC 校验
* 外设
    * 回环测试: 以太网, CAN, UART, SPI, I2C
    * 使用 MBIST 检测 I/O buffer
    * 关键 I/O 使用冗余
    * 在外设主接口上使用 XPPU (Peripheral protection unit)

--------------------------------------------------------------------------------
# 8. 系统监测器
> ![UG1085_Fig9-1](/assets/UG1085_Fig9-1.png)
> ![UG1085_Fig9-6](/assets/UG1085_Fig9-6.png)
> 参见 *Ultrascale+系列FPGA总结：基本结构_WangXH.md*

## 8.1. 功能
* SYSMONE4: PS SYSMON, PL SYSMON
* 电压监测: PS/PL 内部, PS I/O banks, PL 外部输入
* 温度监测: RPU, APU, PL
* 警报: 测量值, 极值
* 采样: 顺序采样, 低速采样
* 自动统计: 最大/最小值, 平均值
* 16 位数字输出 (但 ADC 是 10 位)
* 控制通道
    * PS SYSMON: APB 接口
    * PL SYSMON: PL JTAG, TAP controller, I2C, APB DRP, PL DRP

> ![UG1085_Tab9-1-1](/assets/UG1085_Tab9-1-1.png)
> ![UG1085_Tab9-1-2](/assets/UG1085_Tab9-1-2.png)

## 8.2. 操作
* 可靠性考虑
    * 设置操作极限: 温度, 电压
    * 实时监测: 温度, 电压

--------------------------------------------------------------------------------
# 9. 启动和配置

## 9.1. 基本情况
* 启动过程由 PMU 和 CSU 共同完成
* 主要阶段
    1. 硬件复位 (仅 POR 复位): 硬件状态机控制
        * 零初始化 PMU 寄存器
        * 可选: 执行 LBIST
            * 永久开关: **eFUSE.LBIST_EN**
        * 认证 PMU ROM
        * 释放 PMU 复位
    1. 预配置阶段: PMU 运行 PMU ROM
        1. 可选: 零初始化 LPD 寄存器
            * 永久开关: **eFUSE.LPD_SC**
        1. 可选: 零初始化 FPD 寄存器
            * 永久开关: **eFUSE.FPD_SC**
        1. 零初始化 PMU RAM, 并读回以确保成功
        1. 零初始化 PMU TLB
        1. 使用 PS SYSMON 检查 LPD, AUX, 专用 I/O 的供电电压
        1. 零初始化 CSU, LPD, FPD 中的内存
        1. 通过 SHA-3/384 引擎向 CSU 发送 ROM 代码, 并计算密码校验, 与器件中的 Golden 副本对比
            * 如果 bif 文件启用了验证功能, 则验证 CSU ROM 的完整性
        1. 解除 CSU 复位
        1. PMU 进入服务模式
    1. 配置阶段: CSU 运行 CSU ROM, APU 或 RPU 运行 FSBL
        1. 初始化 OCM
        1. 通过读启动模式寄存器判断启动模式
            * 在 POR 阶段由 PMU 固件捕获
        1. 运行 Boot ROM (CSU 的一部分代码), 根据 Boot header 配置系统
        1. Boot header 验证后, 加载 PS FSBL 到 OCM (安全或非全模式), 由 RPU 或 APU 运行
        1. 加载用户定制的 PMU 固件到 PMU RAM
            * 多数系统中必须存在 PMU 固件
    1. 后配置阶段:
        * CSU 监测篡改事件
        * CSU 提供硬件支持: 认证文件、通过 PCAP 配置 PL、存储和管理安全密钥、解密文件...
* 启动模式
    启动模式 | 模式管脚[3:0] | 管脚位置 | CSU 模式 | 描述
    :------------- | :--: | :--------: | :----: | :---
    PS JTAG        | 0000 | JTAG       | Slave  | PSJTAG 接口, PS 专用管脚
    Quad-SPI (24b) | 0001 | MIO[12:0]  | Master | 24 位地址 (QSPI24)
    Quad-SPI (32b) | 0010 | MIO[12:0]  | Master | 32 位地址 (QSPI32)
    SD0 (2.0)      | 0011 | MIO[25:21, 16:13] | Master |SD 2.0
    NAND           | 0100 | MIO[25:09] | Master | 需要 8 位数据总线
    SD1 (2.0)      | 0101 | MIO[51:43] | Master | SD 2.0
    eMMC (1.8V)    | 0110 | MIO[22:13] | Master | eMMC (v4.5) at 1.8V
    USB0 (2.0)     | 0111 | MIO[52:63] | Slave  | USB 2.0 only
    PJTAG (MIO #0) | 1000 | MIO[29:26] | Slave  | PJTAG connection 0 option
    PJTAG (MIO #1) | 1001 | MIO[15:12] | Slave  | PJTAG connection 1 option
    SD1 LS (3.0)   | 1110 | MIO[51:39] | Master | SD 3.0 with a required SD 3.0 compliant voltage level shifter
* Golden 镜像搜素
    > ![UG1085_Fig11-1](/assets/UG1085_Fig11-1.png)
    * 支持搜索镜像: 验证 boot header (识别码, 校验和)
    * 镜像可以位于每个 32KB 处, 每次搜索地址增加 32KB
    * 启动镜像搜索限制
        启动模式              | 搜索限制
        :-------------------- | :--:
        QSPI: 24 位单芯片     | 16 MB
        QSPI: 24 位并行双芯片 | 32 MB
        QSPI: 32 位           | 256 MB
        QSPI: 32 位并行双芯片 | 512 MB
        NAND                  | 128 MB
        SD/eMMC               | 8,191 files
        USB                   | 1 file
* 回滚
    > ![UG1085_Fig11-2](/assets/UG1085_Fig11-2.png)
    * 在 FSBL 中实现, 将另一个 boot 序号写到 **CSU.csu_multi_boot** 寄存器, 然后触发系统复位 (非 POR 复位)
    * 系统复位后, 从 **CSU.csu_multi_boot** * 32KB 处加载镜像
    * 如果未找到有效的, 继续查找

## 9.2. 启动镜像的格式
> ![UG1085_Fig11-3](/assets/UG1085_Fig11-3.png)
> ![UG1085_Tab11-4](/assets/UG1085_Tab11-4.png)
> ![UG1085_Tab11-5-1](/assets/UG1085_Tab11-5-1.png)
> ![UG1085_Tab11-5-2](/assets/UG1085_Tab11-5-2.png)

## 9.3. CSU Boot ROM 错误代码
* 错误代码记录在 **PMU_GLOBAL.CSU_BR_ERR** 寄存器中
* bit[15:8] 记录第一个镜像的错误代码
* bit[7:0] 记录最后一个镜像的错误代码
* 详见 UG1085::Table 11‐9

## 9.4. PL 配置过程
* FSBL 通过 CSU 中的 PCAP 配置 PL
* 配置过程
    1. 初始化 PCAP 接口
    1. 通过 CSU DMA 和 PCAP 写 bitstream
    1. 等待 PL Done 状态

--------------------------------------------------------------------------------
# 10. 安全性
* 配置文件加密和认证
* 硬件加密加速器
* 存储密钥的安全方法
* 检测和响应篡改事件

## 10.1. 器件和数据安全性

### 10.1.1. CSU (Configuration Security Unit)
> ![UG1085_Fig12-1](/assets/UG1085_Fig12-1.png)
* 功能
    * 安全启动
    * 篡改监控和响应
    * 安全密钥存储和管理
    * 硬件加速加密
* 通过 XPPU 访问 CSU
* 安全处理器模块 (secure processor block, SPB)
    * 三模冗余处理器 (MircoBlaze), 用户无法访问
    * 128KB ROM, 由 SHA3 验证完整性
    * 32KB RAM, ECC
    * PUF(physically unclonable function)
        * 用于生成设备唯一密钥的
* 加密接口模块 (crypto interface block, CIB)
    * 安全数据流路由
        > ![UG1085_Tab11-6](/assets/UG1085_Tab11-6.png)
        > ![UG1085_Tab11-7](/assets/UG1085_Tab11-7.png)
    * DMA
        * 在内存和流外设 (AES, SHA, PCAP) 之间移动数据
            * 可访问的内存: OCM, TCM, DDR
        * 简单的双通道 DMA, 每个通道有 128×32-bit FIFO
    * AES-GCM-256: 对称加密
        * 解密镜像
    * SHA-3/384
        * 产生 384 位数字摘要 (digset)
        * 和 RSA 一起用于认证镜像
    * RSA: 非对称加密
        * 支持 3 中密钥长度: 2048, 3072, 4096 位
            * 启动时仅支持 4096 位
        * 和 SHA-3 一起用于认证镜像
    * PCAP (processor configuration access port)
        * 配置 PL 的唯一的接口
        * 有 PCAP 隔离墙, 需要在配置前禁用
* 时钟
    * 启动时使用内部时钟
    * 启动后, CIB 可以切换到更快的 PLL 时钟

### 10.1.2. 篡改监控和响应
* 监视对象
    * PS SYSMON, PL SEU, JTAG 切换, MIO 输入
        > ![UG1085_Tab12-2](/assets/UG1085_Tab12-2.png)
    * 外部事件
        > ![UG1085_Tab12-3](/assets/UG1085_Tab12-3.png)
* 响应
    * 擦除 BBRAM、安全锁定、复位、中断
    * 每个 **csu_tamper_x** 寄存器有 4 个控制位, 决定如何处理篡改事件
        > ![UG1085_Tab12-4](/assets/UG1085_Tab12-4.png)
    * 某种响应一旦被设置, 就不能取消, 只能新增
        * 除非 POR 复位
    * 两级响应: **csu_tamper_trig** 和 **csu_tamper_0** 配合
        * 第一级: 篡改事件触发中断
        * 第二级: 中断响应处理事件, 最后通过 **csu_tamper_trig** 和 **csu_tamper_0** 触发锁定

### 10.1.3. 锁定
* 非安全锁定
* 安全锁定
    * 发生篡改事件, 且设置响应为安全锁定
    * 安全启动失败
        * 永久开关: **eFUSE.SEC_LK**
* 安全锁定后执行下列操作
    1. 将 MIO 设置为三态
    2. 零初始化 AES 密钥, 并复位 AES-GCM
    3. 复位 APU
    4. 复位 RPU
    5. 禁用 SRST 管脚
    6. 启用 LPD/FPD 隔离
    7. 启用 JTAG 安全门控
    8. 切换 PROG_B 到 PL, 清除 PL 配置
    9. 让 PMU 执行它的锁定
    10. PMU 对 LPD, FPD 和 PMU 执行 MBIST
    11. PMU 等 PL 完成清除工作
    12. PMU 让所有模块复位
    13. PMU 对 LPD 和 FPD 执行 SCAN 清零 (如果 LPD_SC 和 FPD_SC eFUSEs 被编程)
    14. 置位安全锁定完成位
    15. 可选 (通过 eFUSE 禁用)
        a. PMU 设置启动模式为 JTAG
        b. PMU 触发内部 POR
        c. PS 重启, 启用 BSCAN 功能

### 10.1.4. 密钥管理
* 密钥类型
    > ![UG1085_Tab12-5](/assets/UG1085_Tab12-5.png)
    * 片上存储:
        * BBRAM: 仅支持未加密格式密钥 (红色密钥)
        * eFUSE: 支持未加密格式密钥 (红色密钥), 模糊格式密钥 (灰色密钥), 加密格式密钥 (黑色密钥)
    * 引导镜像中
        * 模糊格式密钥 (例如, 使用器件家族代码加密)
        * 加密格式密钥 (例如, 使用 PUF KEK 加密)
* 根据 boot header 或 eFUSE 选择
    > ![UG1085_Fig12-2](/assets/UG1085_Fig12-2.png)
* 模糊格式密钥
    > ![UG1085_Fig12-4](/assets/UG1085_Fig12-4.png)
* 加密格式密钥
    > ![UG1085_Fig12-5](/assets/UG1085_Fig12-5.png)
    * 使用 PUF 生成的 KEK 对密钥进行强加密
        * 所有器件的密钥相同, 因此加密的镜像也相同
        * 每个设备的 KEK 是唯一的
            * 制造过程中固有的, 随机的, 不可控的变化形成的芯片独特的特征
            * PUF 电路利用该特征生成芯片独有的密钥
        * PUF KEK 只有芯片自己知道, 用户无法读取

### 10.1.5. 测试接口保护
* JTAG 接口保护
    > ![UG1085_Fig12-7](/assets/UG1085_Fig12-7.png)
* 编程 **eFUSE.JTAG_DIS** 后 JTAG 只能使用 `BYPASS` 和 `IDCODE` 命令

### 10.1.6. 清除 PL
* PCAP 用于配置 PL
    * **CSU.pcap_prog [pcfg_prog_b]** 可用于擦除 PL
    * **CSU.pcap_status** 可用于检查是否擦除成功
* POR 复位会擦除 PL
* 程序可以按需擦除 PL
    * 由 PROG_GATE 电路控制
        * 控制寄存器 **PMU_GLOBAL.PS_CNTRL** 中的 *PROG_GATE*  和 *PROG_ENABLE* 位控制
        * 也可以由 **eFUSE.PROG_GATE[2:0]** 永久改变

### 10.1.7. 器件 DNA ID
* 每个芯片都有唯一的 96 位 DNA 识别码
* 存在 PL 和 PS 中, 值可能不一样
    * 推荐使用 PL 的 DNA ID, PS 的可能会变
* 获取 DNA ID
    * PL::DNA ID
        * 方法1: PL `DNA_PORTE2` 原语
        * 方法2: JTAG 读取
    * PS::DNA ID
        * 只读寄存器: **EFUSE.DNA_x**

### 10.1.8. 禁用错误输出
* 通过 **eFUSE.ERR_DIS** 永久禁用 JTAG 读取错误状态寄存器 **PS_ERROR_STATUS**

### 10.1.9. 安全的非易失存储 eFUSE
* PS::eFUSE
    * 相关函数: "xilskey_efuseps_zynqmp_input.h", "xilskey_puf_registration.h"
    > 详见 UG1085:: Table 12-13
* PL::eFUSE
    * 128 位, 通过 `FUSE_USER_128` 寄存器访问

## 10.2. 安全启动
> ![UG1085_Fig12-8](/assets/UG1085_Fig12-8.png)
* 两种安全启动模式
    * 硬件信任根: 使用带有可选加密的非对称身份验证来提供机密性、完整性以及引导和配置文件的身份验证
    * 仅加密安全引导: 不使用非对称身份验证, 但需要所有加载的配置必须使用 AES-GCM 进行加密和认证
* 硬件根信任安全启动
    > ![UG1085_Fig12-10](/assets/UG1085_Fig12-10.png)
* 仅加密安全引导
    > ![UG1085_Fig12-15](/assets/UG1085_Fig12-15.png)
* 安全启动镜像的格式
    > ![UG1085_Fig12-16](/assets/UG1085_Fig12-16.png)

--------------------------------------------------------------------------------
# 11. 实时时钟 RTC
> ![UG1085_Fig7-1](/assets/UG1085_Fig7-1.png)
> ![UG1085_Fig7-2](/assets/UG1085_Fig7-2.png)
* 32 位秒计数器 (136 年), 16 位滴答计数器, 4 位小数计数器, 带校准
* 中断 (可路由到 PL): 闹钟中断, 周期秒中断
* 校准: 4 位小数计数器, 每 16 秒校准一次
    * 校准方式: 基于小数计数器展宽 tick 计数器的清零信号 (实际多跳几个 RTC 时钟周期)
        1. 根据实际频率偏差调整 tick 分频值
            > 如 32768 Hz + 50 ppm (实际频率 32769.6384 Hz), 则 tick 分频器填入 32768
            > 如 32768 Hz - 50 ppm (实际频率 32766.3616 Hz), 则 tick 分频器填入 32765
        1. 在小数计数器中填入 **(频率偏差 * 16) 的整数近似值 - 1**
            > 如 32768 Hz + 50 ppm (实际频率 32769.6384 Hz), 则小数计数器填入 round(0.6384 * 16) - 1 = 9
            > 如 32768 Hz - 50 ppm (实际频率 32766.3616 Hz), 则小数计数器填入 round(0.3616 * 16) - 1 = 5
* 断电时由 V~CC_PSBATT~ 供电

--------------------------------------------------------------------------------
# 12. PS 时钟和复位

## 12.1. 时钟
> ![UG1085_Fig37-1](/assets/UG1085_Fig37-1.png)

### 12.1.1. PLL
> ![UG1085_Fig37-2](/assets/UG1085_Fig37-2.png)
* LPD
    * I/O PLL (IOPLL)
    * RPU PLL (RPLL)
* FPD
    * APU PLL (APLL)
    * Video PLL (VPLL)
    * DDR PLL (DPLL)

### 12.1.2. 时钟生成器
* APU MPCore (唯一)
* DDR 内存控制器 (唯一)
* RPU MPCore (基本时钟生成器, 带 1 个分频器和 2 个时钟使能)
* 基本时钟生成器 (带 2 个分频器)
* 基本时钟生成器 (带 1 个分频器)

### 12.1.3. 时钟监视器
* 用一个时钟作为参考, 测量另一个时钟的频率
    * 不监测占空比, 抖动, 质量
* 通过计数方式监测

## 12.2. 复位
> ![UG1085_Fig38-1](/assets/UG1085_Fig38-1.png)
* 两级处理
    1. 由 LPD 中的复位寄存器控制
    1. 交由 PMU 控制
* POR 过程: 略
* 复位源
    > ![UG1085_Tab38-1-1](/assets/UG1085_Tab38-1-1.png)
    > ![UG1085_Tab38-1-2](/assets/UG1085_Tab38-1-2.png)
* 复位原因寄存器
    > ![UG1085_Tab38-2](/assets/UG1085_Tab38-2.png)
* 仅复位 PS
    * 通过 PMU 操作
        * 阻止 CSU 复位 PL
        * 触发 PS 复位
* 系统级软件复位
    * 每个模块都有复位, 操作相关寄存器
    * 也可以请求 PMU 执行复位
* 调试复位
* PL 逻辑复位: 可以使用 PMU 的通用输出管脚
* PL 配置复位:
    * 默认受 PS 系统复位的影响, 也可选择不受其影响
    > ![UG1085_Tab38-4](/assets/UG1085_Tab38-4.png)

## 12.3. 使用
* Vivado 中设置时钟
* 时钟和复位控制寄存器
    * **CRF_APB**
    * **CRL_APB**
--------------------------------------------------------------------------------
# 13. 中断
> ![UG1085_Fig13-1](/assets/UG1085_Fig13-1.png)
* RPU GIC
* APU GIC
* GIC 代理
    * 连接 SPI 中断
    * 或逻辑后产生中断

## 13.1. 系统中断
<!-- > 详见 UG1085::Table 13-1 -->
> ![UG1085_Tab13-1-1](/assets/UG1085_Tab13-1-1.png)
> ![UG1085_Tab13-1-2](/assets/UG1085_Tab13-1-2.png)
> ![UG1085_Tab13-1-3](/assets/UG1085_Tab13-1-3.png)
> ![UG1085_Tab13-1-4](/assets/UG1085_Tab13-1-4.png)
> ![UG1085_Tab13-1-5](/assets/UG1085_Tab13-1-5.png)

## 13.2. APU 中断
> ![UG1085_Fig13-3](/assets/UG1085_Fig13-3.png)
* ARM PL-400, GICv2
* 支持虚拟化
* SGI
* PPI
    > ![UG1085_Tab13-4](/assets/UG1085_Tab13-4.png)
* SPI
* 虚拟化扩展

## 13.3. RPU 中断
> ![UG1085_Fig13-2](/assets/UG1085_Fig13-2.png)
* ARM PL-390, GICv1
* SGI: 16×
    * 写中断号到寄存器并指定目标 RPU
    <!-- * 边沿触发, 敏感类型不可变 -->
* SPI: 约 160×
    * 可以指向任意 RPU, 但只能由一个 RPU 处理
    * PL SPI 的敏感类型可变, 其它不可变
* 无 PPI
* 优先级
    * 分配优先级
    * 优先级相同时, 优先响应中断号小的

## 13.4. PMU 中断
* 本地中断
* 通过 GIC 代理 APU/RPU 中断
    * 管理所有链接到 GIC SPI 的中断, 经过规约或后, 作为 PMU 的外部中断

## 13.5. CSU 中断
* 由 CSU ROM 管理, 封闭系统

## 13.6. IPI (Inter-Processor Interrupt)
* 异构多处理器之间传递消息
* 结构: IPI 中断结构和消息 buffer
    * 11 个中断结构
        > ![UG1085_Fig13-5](/assets/UG1085_Fig13-5.png)
        * 4 个通道指向 PMU
            * IPI0 专用于使 PMU 进入休眠
        * 7 个通道指向 RPU 0, RPU 1, APU多核, PL[3:0]
    * 8 组消息 buffer (对应 8 个目标对象)
        > ![UG1085_Fig13-6](/assets/UG1085_Fig13-6.png)
        * 7 组可分配, 用于 RPU 0, RPU 1, APU多核, PL[3:0]
        * 1 组专用于 PMU
        * 每组有 8 个请求和 8 个响应 buffer (对应 8 个目标处理器)
            * 数据结构需要双方软件协定, 硬件不管
            > ![UG1085_Tab13-3](/assets/UG1085_Tab13-3.png)
        * 每个 buffer 有 32B
    * 中断结构和消息 buffer 受 XPPU 保护
* 工作流程
    * 发送方写请求消息并触发中断, 接收方写回应消息并清中断, 发送方读 **OBS** 寄存器以接收响应
        * 发送 IPI 通信
            1. 写 32B 请求到合适的消息 buffer
            1. 写 1 到中断触发寄存器 **TRIG** 与目标处理器对应的位
            1. 可选: 读 **OBS** 寄存器以验证中断是否被发送
            1. 判断中断是否被处理
                * 方法 1: 轮询 **OBS** 直到状态位被清除
                * 方法 2: 接收中断
        * 接收 IPI 通信
            1. 准备接收消息请求
                * 方法 1: 启用中断
                * 方法 2: 轮询状态
            1. 接收到中断后, 可选: 写 32B 回应到合适的消息 buffer
            1. 通知发送方 "中断已被处理"
                * 方法 1: 清中断状态
                * 方法 2: 发送 IPI 中断
    * 发送方可发送多个中断请求, 可以使用不同的通信协议与每个目标通信
        * 可以发给自己
    > ![UG1085_Fig13-4](/assets/UG1085_Fig13-4.png)

## 13.7. GIC 代理中断
* 由 PMU 处理, 在 RPU/APU 不工作时处理中断
    * 链接所有 SPI 中断
    * 经过规约或后, 作为 PMU 的外部中断
* 寄存器: **LPD_GIC_PROXY**

--------------------------------------------------------------------------------
# 14. 计时器和计数器
> ![UG1085_Fig14-1](/assets/UG1085_Fig14-1.png)
* 计时器
    * APU 计时器
        * 多核全局计时器 (系统级私有计时器)
        * 每个核的计时器
            * 物理私有计时器
            * 虚拟私有计时器
    * 4 个三计时/计数器: LPD::TTC
    * 系统看门狗
        * FPD_SWDT (SWDT1)
        * LPD_SWDT (SWDT0)
        * CSU_SWDT: 位于CSU/PMU 互联

## 14.1. APU 多核计时器/计数器
* 详见 ARM Cortex-A53 相关文档
    > ![UG1085_Fig14-1](/assets/DDI0487G_Fig-D11-1.png)
* 计数器由 **IOU_SCNTRS** 寄存器组控制



---------------- Next: Page 347 ----------------
--------------------------------------------------------------------------------








--------------------------------------------------------------------------------
# 15. 系统地址
> ![UG1085_Fig10-1](/assets/UG1085_Fig10-1.png)

* SCLR
    基地址      | 名称        | 安全访问 | 描述
    :---------- |:--------------- |:---: |:----
    0xFD1A_0000 | CRF_APB         | XMPU | FPD 时钟和复位
    0xFD5C_0000 | APU             | XMPU | APU 控制
    0xFD61_0000 | FPD_SLCR        | XMPU | FPD 全局 SLCR
    0xFD69_0000 | FPD_SLCR_SECURE | Yes  | FPD 全局 SLCR, 用于 PCIe, SATA, 和其他协议的 TrustZone 设置
    0xFF18_0000 | IOU_SLCR        | XPPU | IOU SLCR, 用于 MIO 管脚配置
    0xFF24_0000 | IOU_SECURE_SLCR | Yes  | IOU SLCR, 用于 AXI 读写保护配置
    0xFF26_0000 | IOU_SCNTRS      | Yes  | 系统时间戳生成器
    0xFF41_0000 | LPD_SLCR        | XPPU | LPD SLCR
    0xFF4B_0000 | LPD_SLCR_SECURE | Yes  | LPD SLCR (TrustZone)
    0xFF5E_0000 | CRL_APB         | XPPU | LPD 时钟和复位
    0xFF9A_0000 | RPU             | XPPU | RPU 控制
    0xFD6E_0000 | CCI_GPV         | Yes  | CCI_GPV (CCI400, 参数)
    0xFD70_0000 | FPD_GPV         | Yes  | FPD_GPV (参数)
* APU/RPU 私有寄存器 (中断)
    基地址                    | 描述
    :------------------------ |:----
    0xF900_0000 ~ 0xF900_1FFF | GIC 分发器
    0xF900_2000 ~ 0xF900_2FFF | GICC 接口
* LPD 外设地址
    基地址                   | 描述
    :----------------------- |:----
    0xFF00_0000, 0xFF01_0000 | UART0, UART1
    0xFF02_0000, 0xFF03_0000 | I2C0, I2C1
    0xFF04_0000, 0xFF05_0000 | SPI0, SPI1
    0xFF06_0000, 0xFF07_0000 | CAN0, CAN1
    0xFF0A_0000              | GPIO
    0xFF0B_0000, 0xFF0C_0000, 0xFF0D_0000, 0xFF0E_0000  | GEM0, GEM1, GEM2, GEM3
    0xFF0F_0000              | QSPI
    0xFF10_0000              | NAND
    0xFF16_0000, 0xFF17_0000 | SD0, SD1
    0xFF99_0000              | IPI 消息缓存
    0xFF9D_0000, 0xFF9E_0000 | USB0, USB1
    0xFFA5_0000, 0xFFA5_0800, 0xFFA5_0C00   | System monitor (AMS, PSSYSMON, PLSYSMON)
    0xFFCB_0000              | CSU_SWDT, 系统看门狗 (csu_pmu_wdt)
* FPD 外设地址
    基地址      | 描述
    :---------- |:----
    0xFD0C_0000 | SATA registers (HBA, vendor, port-0/1 control)
    0xFD0E_0000 | AXI PCIe bridge
    0xFD0E_0800 | AXI PCIe ingress {0:7}
    0xFD0E_0C00 | AXI PCIe egress {0:7}
    0xFD0F_0000 | AXI PCIe DMA {0:7}
    0xFD3D_0000 | SIOU slave access ports
    0xFD40_0000 | PS GTR transceivers
    0xFD48_0000 | PCIe attributes
    0xFD4A_0000 | DisplayPort controller
    0xFD4B_0000 | GPU
    0xFD4C_0000 | DisplayPort DMA
* LPD 系统寄存器
    基地址      | 描述
    :---------- |:----
    0xFF30_0000 | Inter-processor interrupts (IPI)
    0xFF11_0000, 0xFF12_0000, 0xFF13_0000, 0xFF14_0000 | TTC0, TTC1, TTC2, TTC3
    0xFF15_0000 | LPD_SWDT, 系统看门狗 (swdt0)
    0xFF98_0000 | XPPU (Xilinx peripheral protection unit)
    0xFF9C_0000 | XPPU_Sink
    0xFF9B_0000 | PL_LPD (S_AXI_LPD)
    0xFFA0_0000 | OCM 互联
    0xFFA1_0000 | LPD-FPD 互联
    0xFFA6_0000 | RTC
    0xFFA7_0000 | OCM_XMPU
    0xFFA8_0000 | LPD_DMA channels {0:7}
    0xFFC8_0000 | CSU_DMA
    0xFFCA_0000 | Configuration and security unit (CSU)
    0xFFCD_0000 | Battery-backed RAM (BBRAM) 控制和数据
* LPD 系统寄存器
    基地址      | 描述
    :---------- |:----
    0xFD00_0000 | DDR_XMPU{0:5}
    0xFD07_0000 | DDR controller
    0xFD08_0000 | DDR PHY
    0xFD09_0000 | DDR QoS control
    0xFD0B_0000 | Arm for DDR
    0xFD36_0000 | HPC0 (S_AXI_HPC0_FPD)
    0xFD37_0000 | HPC1 (S_AXI_HPC1_FPD)
    0xFD38_0000 | HP0 (S_AXI_HP0_FPD)
    0xFD39_0000 | HP1 (S_AXI_HP1_FPD)
    0xFD3A_0000 | HP2 (S_AXI_HP2_FPD)
    0xFD3B_0000 | HP3 (S_AXI_HP3_FPD)
    0xFD49_0000 | Arm for CCI
    0xFD4D_0000 | FPD_SWDT, system watchdog timer (swdt1)
    0xFD50_0000 | FPD_DMA channels {0:7}
    0xFD5D_0000 | FPD_XMPU
    0xFD4F_0000 | XMPU_Sink (FPD)
    0xFD5E_0000 | CCI_REG register set wrapper: debug enables
    0xFD5F_0000 | SMMU_REG (中断, 电源, 控制)
    0xFD6E_0000 | CCI_GPV (CCI400, 参数)
    0xFD70_0000 | FPD_GPV (参数)
    0xFD80_0000 | SMMU_GPV (SMMU500, 参数)
    0xFE00_0000 | IOU_GPV (参数)
    0xFE10_0000 | LPD_GPV (参数)

--------------------------------------------------------------------------------
# 16. 总线: PS 互联与系统保护单元
> ![UG1085_Fig15-1](/assets/UG1085_Fig15-1.png)

## 16.1. 结构
* **互联路由**: ARM NIC-400, LPD/FPD 有各自的互联路由
* **CCI** (Cache Coherent Interconnect): Cache 一致性互联, ARM CCI-400
* **SMMU** (System Memory Management Unit): 系统内存管理单元, 用于虚地址转换, 也可用于地址隔离
* **XMPU** (Xilinx Memory Protection Unit): Xilinx 内存保护单元: DDR, OCM, FPD 从设备
* **XPPU** (Xilinx Peripheral Protection Unit): Xilinx 外设保护单元: IOP 从端口, SIOU 从端口, QSPI 存储
* **QoS** (Quality of Service): 实现 AXI 传输优先级
* **APM** (AXI Performance Monitor): AXI 性能监测, 收集总线传输相关指标
* **ATB** (AXI Timeout Block): AXI 超时模块, 防止从设备无响应导致主控锁定
* **AIB** (AXI Isolation Block): AXI 隔离模块, 用于隔离总线
* PS-PL 总线
    * **S_AXI_HPC[0:1]_FPD** 和 **S_AXI_HP[0:3]_FPD**: 高性能 AXI 从接口, PL 作主控
    * **M_AXI_HPM0/1_FPD**: 低延迟 AXI 主接口, 访问 PL
    * **S_AXI_ACE_FPD**: 两路 AXI 一致性扩展从接口, PL 作主控
    * **S_AXI_ACP_FPD**: 一致性加速器从接口, PL 作主控
    * **S_AXI_LPD**: 低功耗域 AXI 从接口, PL 作主控
    * **M_AXI_HPM0_LPD**: 低功耗域 AXI 主接口, 访问 PL

## 16.2. SMMU
* 包含 1 个 TCU (Translation Cache Unit) 和 6 个 TBU (Translation Buffer Unit)
* 虚拟化与二级地址转换
* 地址转换隔离 (本地模式, 非虚拟化场景)
    * 限定 DMA 的访问空间
    * 例如: PL 通过 SMMU 访问 PS 时, SMMU 可以限定 PL 能访问 PS 中哪些内存区域

## 16.3. XMPU
> ![UG1085_Tab16-5](/assets/UG1085_Tab16-5.png)
* 特征
    * 主接口支持毒输出, 支持访问违例中断 (地址和权限), 支持寄存器写保护
    * 内存分区保护: 限制一个或多个主控的访问
    * DDR 有 6 个 XMPU, 1 MB 颗粒度
    * FPD 有 1 个 XMPU, 4 KB 颗粒度
    * OCM 有 1 个 XMPU, 4 KB 颗粒度
    * 检查安全/非安全访问
* 区间
    * 每个 XMPU 可设置 16 个区间
        * 参数: 起始地址, 终止地址,
        * 区间号越高, 优先级越高
    * 区间被启用后才进行检查
    * 区间检查步骤
        1. 是否落在某个区间
        1. 主控的 ID 是否被允许, 即 (incoming_MID & MID_Mask == MID_Value & MID_Mask)
        1. 检查安全模式和读写权限
    * 如果没有设置任何区间, 或地址未匹配也设置的区间, 将使用默认操作: 运行或投毒
        > ![UG1085_Fig16-3](/assets/UG1085_Fig16-3.png)
        * 两种方式投毒:
            * 毒属性: 转发访问请求, 但加上毒属性
                * 从设备忽略写操作, 对于读操作返回 0, 并可选产生 DECERR 或中断
                * 仅 DDR 和 OCM 的 XMPU 支持毒属性
            * 毒地址: 替换访问请求的高位地址
                * 将产生数据中止异常或中断
                * FPD XMPU 只能使用毒地址策略, 毒地址指向 XMPU_SINK (0xFD4F_0000)
        * 相关寄存器: XMPU CTRL [PoisonCfg] 和 XMPU POISON [ATTRIB]
* 错误处理
    * 发生错误时, 对错误的访问请求投毒, 记录地址、主控 ID、违例标志, (可选) 产生中断
        * 仅有第一个访问违例会被记录
        * 读写同时违例, 只有写会被记录
* 配置
    * 可以在启动时设置
        * 在 Vivado 中设置, FSBL 根据用户设置自动配置
    * 可以通过安全主控按需设置
    * 寄存器锁定后, 只能在 POR 复位后才能设置

## 16.4. XPPU
> ![UG1085_Fig16-4](/assets/UG1085_Fig16-4.png)
* 保护 LPD 的外设和 SLCR, 以及异构处理器间中断 IPI
* 保护机制
    * 主控 ID 列表 (20个条目): 限定哪些主控可访问
        > ![UG1085_Tab16-9](/assets/UG1085_Tab16-9.png)
    * 区间权限列表 (400个条目): 被允许的主控可访问哪些地址空间
        * 128 个条目用于 IPI 消息缓存, 每条地址空间: 32B
        * 256 个条目用于外设从接口, 每条地址空间: 64KB
        * 16 个条目用于外设从接口, 每条地址空间: 1MB
        * 1 个条目用于 QSPI, 地址空间: 512MB
        > ![UG1085_Tab16-10](/assets/UG1085_Tab16-10.png)
        > ![UG1085_Tab16-11](/assets/UG1085_Tab16-11.png)
* 错误处理
    * 未被允许, XPPU 设置毒位, 并将访问转发到 XPPU_SINK, 产生中断或 SLVERR
    > ![UG1085_Tab16-12](/assets/UG1085_Tab16-12.png)

## 16.5. QoS
* ArQoS[3:0], AwQoS[3:0]
    * 0xF 表示最高优先级
* 策略
    * NIC400: 根据优先级仲裁, 如果优先级相同, 仲裁为最近授权
    * PL 相关的 QoS
        * 静态设置: 设置寄存器 **AFIFM.xx** 的值
        * 动态设置: PL 控制 QoS 位
* QoS 子系统
    * AXI 互联上的 QoS-400 调节器
        * 34 个调节器及控制寄存器
            > 详见 UG1085::Figure15-1 和 Table 15-6
        * 可以限制最大未决传输的数量
        * 可以限制命令发射的速率
        * CCI400 中的 QoS 虚拟网络
            * 低延迟 (Low Lantency, LL) 传输, 也即高优先级传输, 和高通量 (Best Effort, BE) 传输分开
                * 基于 AxQoS 区分
            * 示例
                > ![UG1085_Fig15-4](/assets/UG1085_Fig15-4.png)
    * AXI 超时单元 (ATB)
    * AXI QoS 虚拟网络通道 (QVN 网络)
* 相关寄存器
    * **FPD_GPV**, **LPD_GPV**, **IOU_GPV**, **CCI400**, **DDRC**, **DDR_QOS_CTRL**, **AFIFM**

## 16.6. APM
> ![UG1085_Tab15-2](/assets/UG1085_Tab15-2.png)
* 基于 *Xilinx AXI Performance Monitor(PG037)* , 无错误记录和 AXI Stream 接口
    > ![PG037_Fig1-1](/assets/PG037_Fig1-1.png)
* 周期性统计信息
* 可统计的信息
    * 读写传输计数, 读写字节计数, 写次数, 读写延迟, 空闲计数, Valid/Last 计数, ...
* 相关寄存器
    * **APM_CCI_INTC**, **APM_INTC_OCM**, **APM_LPD_FPD**, **APM_DDR**

## 16.7. ATB
* 防止从设备无响应导致主控锁定
> ![UG1085_Fig15-2](/assets/UG1085_Fig15-2.png)
> ![UG1085_Tab15-1](/assets/UG1085_Tab15-1.png)
* 控制寄存器: **LPD_SLCR** 或 **FPD_SLCR**

## 16.8. AIB
* 隔离 AMBA 互联
    * 一般在断电之前运行
    * 不隔离时, 对总线时透明的
* PMU 发出隔离命令后, AIB 不传输新的访问请求, 忽略传输请求导致超时, 或产生 SLVERR
* 控制寄存器 **lpd_slcr_aib.ISO_AIB{AXI, APB}_TYPE/REQ/ACK**

## 16.9. 系统保护: TrustZone
* Xilinx 将 AxPORT[1] 作为第 41 位地址, 用于区分安全或非安全访问
    * 因此安全访问不能访问非安全内存
* 安全性配置
    * 安全从设备阻止非安全主控访问
        * 从设备由 XPPU 和 XMPU 保护
        * 一些系统控制寄存器必须使用安全访问
    * DDR 和 OCM 可以分为安全区域和非安全区域
        * 每个区域的大小可配置 (DDR 对齐到 1MB, OCM 对齐到 4KB)
        * 由 XMPU 保护
    * 主控的安全/非安全类型
        * 固定类型: 安全, 或非安全
        * 可编程: 由寄存器控制
        * 动态: 例如 PS-PL 互联
    * 处理器启动后是安全模式
    * RPU 不支持 TrustZone, RPU 到 APU 的传输可以配置为安全访问或非安全访问
    * RPU 启动时的安全模式可配置, 默认为安全模式
    > ![UG1085_Tab16-2-1](/assets/UG1085_Tab16-2-1.png)
    > ![UG1085_Tab16-2-2](/assets/UG1085_Tab16-2-2.png)

## 16.10. 系统保护: 写保护寄存器
* 一旦写保护, 不能修改, 除非 POR 复位
    * CRF_APB, CRL_APB
    * XMPU, XPPU
    * VCU_SLCR
    * EFUSE

--------------------------------------------------------------------------------
# 17. 总线: DMA

--------------------------------------------------------------------------------
# 18. 存储: DDR
> ![UG1085_Fig17-2](/assets/UG1085_Fig17-2.png)

## 18.1. 基本特征
* 内部数据接口:
    * 6 个 AXI 接口
        * #0: 64 位宽
        * #1~5: 128 位宽
    * APM
    * XMPU
* DDR 控制器
    * 调度
        * QoS
            * 视频/同步访问: 支持读/写 QoS
            * 低延迟访问: 支持读 QoS
            * 大数据量访问: 支持读/写 QoS
        * 64 个读缓存和 64 个写缓存, 基于 CAM
        * 支持乱序运行
    * 支持 ECC
        * 32/64 位宽
        * 在 ECC 模式下支持高效的"读-该-写"操作
        * 自动记录可纠正和不可纠正的错误
        * 发生不可纠错错误时, 支持对写数据投毒
        * LPDDR3 不支持
* 外部物理接口
    * 支持 DDR3, DDR3L, LPDDR3, DDR4, LPDDR4
        * 执行初始化, 训练等
    * 可以从 SODIMM/UDIMM/RDIMM 内存条的读取 SPD 信息 (关键时序参数)
    * 支持双 Rank
    * 支持自动低功耗
        * 基于内存传输
        * 断电, 停止时钟, 自刷新
            * RDIMM 不支持停止时钟
        * LPDDR3 和 LPDDR4 不支持自动低功耗
    * 通过软件可以显式更新 SDRAM 的模式寄存器
    * 在 DDR3/4 上支持 2tCK 命令时序 (2T 时序)
        * 当 PCB 布局设计不合理或电源质量有问题时, 可解决时序问题
    > ![UG085_Tab17-1](/assets/UG1085_Tab17-1.png)
* 时钟和复位
    * 只能和 FPD 一起复位
        * 寄存器: **PMU_GLOBAL.GLOBAL_RESET[FPD_RST]**

## 18.2. 结构

### 18.2.1. QoS 控制器
* 基于优先级, **AxQoS[3:0]** 值越大, 优先级越高
* 传输类型
    * 视频/同步传输: 实时, 固定带宽, 固定的最大延时
        * 可以接受低优先级和长延时, 但最大延时不能超过设定值
    * 低延时传输 (LL): 低延时, 高优先级
        * 一般分配最高优先级, 只能被超时的视频类传输超越
    * 最大努力传输 (BE): 高延时, 低优先级
        * 一般分配最低优先级
    * DDR 读按 3 种类型分配优先级
        * 最大努力传输 (BER), 视频/同步传输 (VPR), 低延迟传输 (LL)
    * DDR 写按 2 种类型分配优先级
        * 最大努力传输 (BEW), 视频/同步传输 (VPW)
* #1 和 #2 组成 QoS 虚拟网络通道 (QVN), 同时支持多种传输类型, 其它通道仅支持 1 种传输类型
* 调度
    > ![UG1085_Fig17-3](/assets/UG1085_Fig17-3.png)
    * QoS 控制器确保在任何时候都有足够的 CAM 用于视频类传输
        * 监控 CAM 水平, 超过设定阈值时, 阻断 XPI 数据接口仲裁器上非视频类传输
            * 当 CAM 水平低于阈值, 发送控制信号到 DDR 控制器, 阻断所有 BE 传输
    * 可以指定 6 个 AXI 端口的传输类型, 寄存器 **DDR_QOS_CTRL.PORT_TYPE**
        * AXI 端口 #0: 默认 LL
        * AXI 端口 #1/2: 红端口默认为 LL, 蓝端口默认为 BE
        * AXI 端口 #3/4/5: 默认为视频/同步传输
    * 可以启用/禁用每个 AXI 端口的阻断控制, 寄存器 **DDR_QOS_CTRL.PORTn_LPR_CTRL/PORTn_HPR_CTRL/PORTn_WR_CTRL**
    * 全局 CAM 阈值寄存器 **DDR_QOS_CTRL.RD_LPR_THRSLD/RD_HPR_THRSLD/WR_THRSLD**
* 中断: IRQ 144
    * 可纠正 ECC 错误 [DDR_ECC_CORERR]
    * 不可纠正 ECC 错误
    * DFI 初始化完成 [DFI_INIT_COMP]
    * DDR 模式寄存器设置 (MRS) 导致的 DFI 校验错误 [DFI_ALT_ERR_FTL]
    * DFI 校验错误计数器达到最大值 [DFI_ALT_ERR_MAX]
    * DFI 接口上的校验错误或 CRC 错误 [DFI_ALT_ERR]
    * 性能计数器中断 [PC_COPY_DONE]
    * 无效的 DDR 控制器寄存器访问 [INV_APB]

### 18.2.2. DDR 内存控制器
> ![UG1085_Fig17-4](/assets/UG1085_Fig17-4.png)
* 4 个主要模块
    * AXI 端口接口 (AXI port interface, XPI)
    * 端口仲裁器 (PA)
    * DDR 控制器 (DDRC)
    * APB 寄存器
* XPI
    * 将 AXI burst 传输转换为 DDR 读写请求, 并转发到仲裁器
    * 反之亦然
* PA
    1. 读写仲裁:
        * 合并读写操作, 保持在读/写状态, 除非反向的请求超时
        * 读优先于写
        * 每个端口的每个方向都有老化计数器, 实现超时管理
    1. 读写优先级
        * 读通道可被设置为: HPR > LPR/VPR
            * VPR 超时前, 被当作 LPR
            * VPR 超时后, 优先级高于 HPR, 也高于写操作
        * 写通道可被设置为: BEW = VPW
            * VPW 超时前, 被当作 BEW
            * VPW 超时后, 优先级高于 BEW
            * 同时超时, 轮询
    1. 端口命令优先级
        * 基于每个端口的 AXI AxQoS 信号
        * 端口命令仲裁等级低于超时
        * 对于读取, 端口命令仲裁等级也低于 HPR/LPR-VPR 总裁等级
    1. 轮询仲裁
        * 经过所有以上仲裁后, 使用轮询仲裁, 从 #0 端口开始轮询
    * 仲裁掩码
        * 由 QoS 控制器负责
* DDRC
    * 包含 CAM, 缓存 DDR 操作命令
    * 基于优先级, bank/rank 状态和 DDR 时序, 优化调度, 并将命令发送给 PHY
    * 地址映射
* ECC

### 18.2.3. DDR 物理层
> ![UG1085_Fig17-8](/assets/UG1085_Fig17-8.png)

## 18.3. 编程
* 大多数寄存器在初始化时设置好了, 不需要更改
* 部分动态寄存器可以更改
    * 刷新相关寄存器
* 准动态寄存器
    * Group 1: registers that can be written when no Read/Write traffic is present at the DFI.
    * Group 2: registers that can be written in self-refresh, DPD, and MPSM modes.
    * Group 3: registers that can be written when the controller is empty.
    * Group 4: registers that can be written depending on MSTR.frequency_mode and the MSTR2.target_frequency
* 功耗控制
    * 功耗控制方法
        * 预充电断电
        * 自刷新
        * 深度断电
        * 最大省电
        * 禁用时钟
    * 操作相关寄存器
        * 断电: **PWRCTL[powerdown_en]**=1
        * 自动自刷新: **PWRCTL[selfref_en]**=1
        * 软件控制自刷新: **PWRCTL[selfref_sw]**=1
        * 禁用时钟: **PWRCTL[en_dfi_dram_clk_disable]**
* DDR 初始化
    * PHY 初始化
    * DRAM 初始化
        * 数据训练

--------------------------------------------------------------------------------
# 19. 存储: OCM
> ![UG1085_Fig18-1](/assets/UG1085_Fig18-1.png)

## 19.1. 特征
* 256KB, 256 位宽, ECC
* 内部实现了读-改-写
* 内部仲裁: 轮询
* 8 个独占访问监视器
* XMPU (4KB × 64) 和 Trustzone
* 4 个独立的电源岛和数据保持

## 19.2. 操作
* 通过 XMPU 管理哪些主控可以访问 OCM
* 管理 ECC 功能和错误

--------------------------------------------------------------------------------
# 20. 存储: QSPI

--------------------------------------------------------------------------------
# 21. 存储: SD/SDIO/eMMC

--------------------------------------------------------------------------------
# 22. LPD 外设: MIO-EMIO

--------------------------------------------------------------------------------
# 23. LPD 外设: UART

--------------------------------------------------------------------------------
# 24. LPD 外设: I2C

--------------------------------------------------------------------------------
# 25. LPD 外设: SPI

--------------------------------------------------------------------------------
# 26. LPD 外设: CAN

--------------------------------------------------------------------------------
# 27. FPD 外设: PS-GTR 收发器

--------------------------------------------------------------------------------
# 28. FPD 外设: GEM

--------------------------------------------------------------------------------
# 29. FPD 外设: PCI-E

--------------------------------------------------------------------------------
# 30. FPD 外设: USB

--------------------------------------------------------------------------------
# 31. FPD 外设: SATA

--------------------------------------------------------------------------------
# 32. FPD 外设: DispalyPort

--------------------------------------------------------------------------------
# 33. PS-PL AXI

--------------------------------------------------------------------------------
# 34. PL 外设

--------------------------------------------------------------------------------
# 35. 系统测试和调试

