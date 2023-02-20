Zynq总结：设计流程
---

--------------------------------------------------------------------------------
# 1. 基本设计流程

## 1.1. 固件设计
1. 在 Vivado 中新建或打开 Block Design
1. 添加 IP: **ZYNQ7 Processing System**, 并设置参数
    * 可以加载开发板预定制的参数表
    * 设置必要的参数
        * PS-PL 配置
            * UART, PL AXI 总线空闲, PS-PL 调试接口, FTM, Event, 地址, 时钟触发与复位, AXI-GP, AXI-HP, ACP, DMA
        * 外设 I/O 配置
            * MIO, EMIO 信号连接
        * MIO 配置
            * 电平标准, 速度, 上拉, 方向, 极性
        * 时钟配置
        * DDR 配置
            * DDR 类型, 位宽, ECC, burst, 时钟频率等
            * 芯片信息: 位宽, 容量, 速度, 地址宽度, 时序
            * 训练信息: 训练方式, 信号延时 (与 PCB 走线相关)
            * 高级设置
                * 读写优先级
                * 是否开启 HPR 以及 HPR 相关参数
            * DDR 三级仲裁相关参数
        * 中断
        > 与硬件紧密相关, 参见 *UG585* 和 *Zynq总结: 器件结构_WangXH*
1. 添加基本外设 IP
    * **Processing System Reset**
    * **AXI Interconnect**
    > 这些外设也可以被自动添加
1. 添加应用需要的 IP, 如
    * **AXI-Stream FIFO**
    * **AXI Data FIFO**
    * **AXI GPIO**
    * **AXI DMA**
    * **AXI Multichannel DMA**
    * **AXI DataMover**
    * 用户设计的 IP
1. 使用 **Address Editor** 分配地址
1. 验证 Block Design, 并生成 HDL wrapper
1. 设计 PL 部分
1. 在顶层设计文件中合并 Block Design 和 PL 设计
1. 综合, 实现, 生成 bitstream
1. 导出 Hardware
    * 向 SDK 输出一个 Handoff 文件 (`xx.hdf`)

## 1.2. 软件设计
1. 打开 SDK
    * Vivado::File → Launch SDK
    * 启动后根据 `<prj>.hdf` 文件创建硬件平台 (`<prj>_hw_platform_0` 文件夹), 其中包含
        * `<prj>.bit`: bitstream
        * `ps7_init.html`: 根据用户配置的 ZYNQ7 的参数自动生成的文件, 包含
            * 参数设置: MIO 信息, DDR 信息, PS 时钟信息
            * 寄存器初始化数据:
                * PS PLL: `ps7_pll_init_data_3_0`
                * PS 时钟: `ps7_clock_init_data_3_0`
                * PS DDR: `ps7_ddr_init_data_3_0`
                * MIO: `ps7_mio_init_data_3_0`
                * 外设: `ps7_peripherals_init_data_3_0`
                    * SRAM, UART, QSPI, PL 电源, GPIO, ...
                * 后配置: `ps7_post_config_3_0`
                    * PS-PL 电平转换器开关, FCLKRESETN (解除复位), AXI_HP
                * 调试: `ps7_debug_3_0`
                    * CTI, ...
        * `ps7_init.{h,c}`: 根据用户配置的 ZYNQ7 的参数, 自动生成的初始化代码, 可用于 FSBL
            * `ps7_init_gpl.{h,c}` 和 `ps7_init.{h,c}` 内容一样
        * `ps7_init.tcl`: 根据用户配置的 ZYNQ7 的参数, 自动生成的初始化代码, 用于调试
1. 新建 BSP
    * SDK::File → New → Borad Support Package → 弹出对话框
    * 可以指定用于哪个处理器内核
    * 可以指定操作系统: Xilinx 定制的 FreeRTOS 10, 或无操作系统
    * 自动加入需要的软件库和驱动
    * 包含文件
        * *BSP Documentation*: 驱动的相关文档
        * Xilinx 提供的库: 在 `include` 目录中有对应的头文件
        * `system.mss`: BSP 的相关信息, 以及控制按钮: 修改 BSP, 重新生成 BSP
1. 修改 BSP 参数
    * 操作系统
        * 版本, 基本设置
    * 软件库: LibMetal, LwIP, OpenAMP, xilffs (FAT 文件系统), xilflash (并行 flash 库), xilisf (in-system 和 串行 flash 库), xilmfs (内存文件系统), xilpm (电源管理), xilrsa (加解密), xilskey (密钥)
    * 驱动
        * 处理器内核的编译器设置
1. 创建应用工程
    * SDK::File → New → Application Project → 弹出对话框
    * 选择合适的硬件平台和 BSP
        * 可以基于已有的 BSP, 也可以新建 BSP
    * 可以选择程序模板
1. 编写程序

## 1.3. 调试
* 启动调试:
    * SDK::Application Project 上点右键 → Run/Debug As → Launch on Hardware (GDB)
    * 可以在启动前, 修改 Run/Debug 配置
        * 需要文件: bitstream, ps7_init.tcl
        * 可选: 复位系统, 编程 FPGA, 运行 ps7 初始化, 运行 ps7 后初始化
* 从 **SDK::SDK Terminal** 观察打印输出
    * Xilinx 使用 Zynq 的 UART1 作为打印口
        * 115200, 8, 1, None
    * 调试开始前, 确保 SDK 中已设置好计算机的串口
* 可以通过设置断点, 对代码进行逐步调试
    * 断点可以被临时禁用, 也可以设置条件
    * 调试界面的 "Breakpoints" 选项卡
* 可以观察或修改程序运行中的变量
    * 调试界面的 "Variables" 选项卡

## 1.4. 下载
1. 准备好 `xx.bit` 文件和 `<prj>.elf` 文件
1. 创建 FSBL
    1. SDK::File → New → Application Project → 弹出对话框
    1. 选择合适的硬件平台, 并新建 BSP
    1. 点击 Next, 选择模板: Zynq FSBL
    1. 继续, SDK 会自动生成 `FSBL.elf` (最好选择生成 release 版)
1. 创建启动镜像
    1. SDK::Xilinx → Create Boot Image → 弹出对话框
    1. 设置 `BIF` 路径, 下载文件格式 (BIN, MCS 均可)
    1. 按顺序添加文件
        1. bootloader: `FSBL.elf`
        1. datafile: `xx.bit`
        1. datafile: `<prj>.elf`
    1. 创建镜像
1. 设置开发板的启动模式为 QSPI
1. 下载镜像
    1. SDK::Xilinx → Program Flash → 弹出对话框
    1. 选择镜像文件和 FSBL 文件, offset 留空 (实际是 0)
    1. 选择 Flash 类型
        * ZC706 是 QSPI-x8-dual_parallel
        * zedboard 是 QSPI-x4-single
    1. 下载

--------------------------------------------------------------------------------
# 2. 固件设计

## 2.1. Zynq 相关的基础 IP
* Processing System Reset
* AXI Interconnect

## 2.2. PS-PL 信号
类型         | 信号                 | 方向     | 说明
:----------: | :------------------- | :------: | -
EMIO         | IOP 的部分信号       | PS ↔ PL | 可以将 PS 外设的 I/O 引入 PL, 如 GPIO 的 Bank_{2,3}
总线         | AXI_ACP              | PS ↔ PL | 64-bit, ×1, PL 作主控
^            | AXI_HP               | ^        | 64-bit, ×4, PL 作主控
^            | M_AXI_GP             | ^        | 32-bit, ×2, PS 作主控
^            | S_AXI_GP             | ^        | 32-bit, ×2, PL 作主控
时钟和复位   | FCLK [3:0]           | PS → PL | -
^            | FCLKRESETN [3:0]     | ^        | 相对于时钟是异步信号
中断         | IRQF2P[19:0]         | PL → PS | -
^            | IRQP2F[27:0]         | PS → PL | -
事件信号     | EVENTEVENTI          | PL → PS | -
^            | EVENTEVENTO          | PS → PL | -
^            | EVENTSTANDBYWFE[1:0] | ^        | -
^            | EVENTSTANDBYWFI[1:0] | ^        | -
DDR Urgent   | DDRARB[3:0]          | PL → PS | -
DMA Req/Ack  | DMA[3:0]ACLK         | PL → PS | -
^            | DMA[3:0]DRREADY      | PS → PL | -
^            | DMA[3:0]DRVALID      | PL → PS | -
^            | DMA[3:0]DRTYPE[1:0]  | PL → PS | -
^            | DMA[3:0]DRLAST       | PL → PS | -
^            | DMA[3:0]DAREADY      | PL → PS | -
^            | DMA[3:0]DAVALID      | PS → PL | -
^            | DMA[3:0]DATYPE[1:0]  | PS → PL | -

## 2.3. PS-PL 间数据传输

### 2.3.1. PS 作为主设备
* PL 侧将从设备挂载在 M_AXI_GP 上
* 相关 IP:
    > 参见 *Zynq总结：设计流程_WangXH.md*
    * 总线桥: AXI vs AHB-Lite, AXI vs APB
    * **AXI-Stream FIFO**
    * AXI Data FIFO
    * AXI GPIO
    * AXI BRAM Controller
    * ...
* 数据传输
    * 方法1: PS 直接操作从设备
    * 方法2: 配合 DMA Req/Ack 接口, PL 请求 PS::DMA 开始传输数据

### 2.3.2. PL 作为主设备
* PL 侧使用 AXI 主设备控制 AXI_ACP, AXI_HP, S_AXI_GP
* 相关 IP:
    > 参见 *Zynq总结：设计流程_WangXH.md*
    * AXI Central DMA
    * AXI DMA
    * AXI Multi Channel DMA
    * AXI Video DMA
    * AXI DataMover
* 配合 DDR Urgent 信号, 可以调节数据传输的优先级
* 数据传输
    * 如果主设备的控制寄存器由 PS 管理
        * 方法1: 基于 SDK 提供的 API 初始化主设备, PL 触发主设备传输数据, PS 使用中断或状态轮询机制获取数据传输状态
    * 如果主设备的控制寄存器由 PL 管理
        * 方法1: PS 向 PL 传递目标地址
            1. PS 通过声明数组预先占用足够的内存空间 (注意对齐), 然后将数组的基地址传送给 PL
            1. PL 根据数组基地址传输数据, 传输完成后通知 PS, PS 访问数据以处理数据
        * 方法2: PS 和 PL 预先约定目标地址
            1. 通过修改链接脚本, 将一段内存空间隐藏, 专门留给 PL, PS 的程序不会使用该空间
                * 注意: SDK 中 OCM 默认对 PS 不可见
            1. PS 和 PL 从约定的地址开始读写数据
                * 注意要相互通知

## 2.4. PS-PL 间信号触发

### 2.4.1. PS 触发 PL
* PS 需要通知 PL 时, 可以控制 EMIO, AXI-GPIO 等接口翻转信号
* PS 中的外设完成任务后, 通过中断信号通知 PL
    * PL 以多种方式获取 PS 外设的数据
        * 使 PS::DMA 处于等待状态, PL 发送请求后 PS::DMA 转发数据, PL 中的 AXI 从设备接收数据
        * PL 中的 DMA 主动获取数据
* PS 需要停止工作以等待 PL 时, 运行 `WFE` 或 `WFI` 指令, PL 检查 `EVENTEVENTO` 或 `EVENTSTANDBYWFE`/`EVENTSTANDBYWFE`
    * ACP 传输可以基于此机制

### 2.4.2. PL 触发 PS
* PS 检查 EMIO, AXI-GPIO 等接口的信号
* PL 可以通过中断 (或事件信号 `EVENTEVENTI`) 通知 PS

--------------------------------------------------------------------------------
# 3. 软件设计

## 3.1. 库和 Macro

### 3.1.1. ARM EABI
* arm-none-eabi: 用于无操作系统的应用, 或基于 FreeRTOS 的应用
* arm-linux-gnueabihf: 用于基于 linux 系统的应用

> ABI (application binary interface), 应用程序二进制接口, 解释如何编译, 汇编, 链接, 以及其他相似的工具如何产生目标文件和可执行文件

> EABI: 嵌入式 ABI
> * EABI 的目的是使不同的编译器编译出来的二进制文件可以互相使用 (interoperability). 符合 EABI 标准编译器编译出的库可以相互链接, 这样软件开发人员就可以混合使用不同厂商提供的符合EABI标准的二进制库
> * EABI 指定了文件格式、数据类型、寄存器使用、堆栈组织优化和在一个嵌入式软件中的参数的标准约定

### 3.1.2. 标准 C/C++ 库
* SDK 提供了标准 C/C++ 库

### 3.1.3. ARM 相关库
* 伪汇编指令库 (`arm_acle.h`)
* FPU 和 NEON 的库 (`arm_neon.h`)
* ...

### 3.1.4. Xilinx BSP 库
类型     | 子类       | 相关文件
:------: | :--------: | :------:
编程基础 | BSP 配置   | `bspconfig.h`, `xenv.h`, `xenv_standalone.h`
^        | 数据类型   | `xil_types.h`, `xstatus.h`, `xil_assert.h`
^        | 标准输出   | `xil_printf.h`
CPU 相关 | 底层代码   | `asm_vectors.s` → `boot.S` → `translation_table.s` → `xil-crt0.S` → `cpu_init.s`
^        | 硬件抽象层 | `xil_hal.h`(引入了 "xil_cache.h", "xil_io.h", "xil_assert.h", "xil_exception.h", "xil_types.h")
^        | 内核相关   | `xcpu_cortexa9.h` (空文件), `xil_errata.h`
^        | 硬件信息   | `xparameters.h`, `xparameters_ps.h`, `xplatform_info.h`
^        | 伪指令     | `xreg_cortexa9.h`, `xpseudo_asm.h`, `xpseudo_asm_gcc.h`
^        | MMU        | `xil_mmu.h`
^        | Cache      | `xil_cache.h`, `xil_cache_l.h`, `xil_testcache.h`, `xl2cc.h`, `xl2cc_counter.h`
^        | 中断       | `vectors.h`, `xil_exception.h`, `xscugic.h`
^        | 内存       | `xddrps.h` (空文件), `xil_mem.h`, `xil_testmem.h`
^        | 总线访问   | `xil_io.h`, `xil_testio.h`
^        | 时间       | `sleep.h`, `xtime_l.h`
驱动     | 复位       | `xil_misc_psreset_api.h`
^        | SCU 外设   | `xscutimer.h`, `xscuwdt.h`
^        | IOP 外设   | ...

### 3.1.5. 预定义的 Macro
* SDK::Application Project 上点右键 → Properties → C/C++ General → Paths and Symbols → Symbols 选项卡
* 预定义 GNU C 的 Macro, 包括
    * 硬件平台相关
    * EABI 相关
    * 数据类型相关
    * GCC 编译器相关
    * ...

## 3.2. GCC 编译器

### 3.2.1. ARM 相关的编译器选项
> 参见 [GCC 官方文档](https://gcc.gnu.org/onlinedocs/gcc/ARM-Options.html)
* 浮点 ABI: `-mfloat-abi=<soft|softfp|hard>`
    * `soft`: 用库函数
    * `softfp`: 使用硬件浮点单元, 但使用软件函数调用标准, 使用通用寄存器传参, ISR 的负荷小, 但参数需要先转换为浮点后, 才进行计算
    * `hard`:  使用硬件浮点单元, 使用 FPU 特定的函数调用标准, 使用浮点寄存器 (s0-s31 / d0-d15 / q0-q15) 传参, 性能最好, 但 ISR 负荷大
* CPU 类型: `-mcpu=cortex-a9`
    * 可自动推导出 `–march=armv7-a -mtune=cortex-a9`
    * 与 mtune 相比, 可以 mcpu 可以加扩展, 如
        * `-mcpu=cortex-a9 +nofp` 不使用浮点指令
        * `-mcpu=cortex-a9 +nosimd` 不使用 SIMD 指令
* FPU: `-mfloat-abi=<softfp|hard> -mfpu=<vfpv3|vfpv3-d16|vfpv3-fp16|vfpv3-d16-fp16>`
* NEON: `-mfloat-abi=<softfp|hard> -mfpu=neon`
    * 在3个地方添加
        * 板级支持包
        * ARM gcc compiler --> Miscellaneous
        * ARM gcc linker --> Miscellaneous
    * 也可以开启编译器的向量化选项
* SDK 中默认的编译选项为 `-mcpu=cortex-a9 -mfpu=vfpv3 -mfloat-abi=hard -nostartfiles -g -Wall -Wextra`
    * `-nostartfiles`: 不适用 GCC 默认的启动代码, SDK 有自己的启动代码
    * `-g`: 支持 GDB
    * `-Wall -Wextra`: 运行各种警告
### 3.2.2. 修改编译选项
* SDK::BSP → `system.mss` → Modify BSP Settings → drivers → ps7_cortex_9_0 → extra_compiler_flags
* SDK::Application Project 上点右键 → Properties → C/C++ Build

## 3.3. APU 运行环境

### 3.3.1. 启动过程
> 详见 *[Zynq总结：器件结构_WangXH](.\Zynq总结：器件结构_WangXH.md)*
1. 上电初始化
    * 上电, 复位, 时钟 (初始化 PLL)
1. 运行 BootROM
    * 设置 APU, 运行自检, 清除 PL, 读取 BootROM Header, ...
    * 将 FSBL 加载到 OCM
1. 运行 FSBL
    * 初始化 PS 和 PL
    * 交接给用户程序
1. 运行用户程序
    * 运行底层代码, 初始化 APU
    * 运行 `main()`

### 3.3.2. FSBL 代码简析
* 内存使用:
    * OCM: 192 KB, 0x00000000 ~ 0x0002FFFF
        * 除了栈之外的所有代码和数据
    * OCM: 64 KB, 0xFFFF0000 ~ 0x0000FDFF
        * 栈
        > 最后 0.5 KB 留给了 CPU 1
* FSBL 工作流程
    1. 复位后从 `asm_vectors.s` 跳转到 `boot.S`, 配合 `translation_table.s`, 完成 APU 初始化
        1. 复位后从异常向量表跳转到 `_boot`; 检查 CPU ID, 如果是 CPU 0 则运行代码 (否则 `WFE`); 检查 eFuse, 如果正确则跳转到 `OKToRun` (否则启动 CPU 1); 更新 **CP15** 中的诊断寄存器信息, ==设置异常向量表地址==
        1. 禁用 SCU, Invalidate caches 和 TLB, 禁用 MMU
        1. 转到各种内核模式下, 设置通用寄存器 (R13) 和程序状态寄存器 **CPSR** (设置为小端), 最后的内核模式是 System
            * 内核模式的设置顺序: IRQ → SVC → Abort → FIQ → Undefined → System
        1. 使能 SCU, 初始化 CPU, MMU, L1 Cache 和 L2 Cache, 使能 VFP, 分支预测, 预取和异步中止异常
    1. 跳转到 `xil-crt0.S` 中的 `_start`, 配合 `cpu_init.s`, 完成 C 的启动
        1. 复位 **CP15** 中的一些寄存器, 复位并启动 PMU 的 Cycle Counter
        1. 清除一些 Section (sbss, bss), 设置栈指针, 根据 sleep 源的设置初始化 TTC 计时器或全局计时器 (默认), 初始化 Profiling, 初始化全局构造器
    1. 跳转到 `main(0, 0)` 函数
        1. 调用 `PS7_init()` 依次初始化
            1. MIO: 主要包括 PS_DDRIO, PS_MIO 等 IO, 相关寄存器 `sclr`
            1. PLL: 包括 ARM_PLL, DDR_PLL, IO_PLL, 相关寄存器 `sclr`
            1. Clock: 包括 CPU 和各种外设的时钟, 相关寄存器 `sclr`
            1. DDR 控制器: 相关寄存器 `ddrc`
            1. peripheral: 主要包括 PS_DDRIO, 各种外设, 相关寄存器 `sclr` 和各种外设的寄存器
            * Xilinx 根据 IP::Zynq 的设置自动生成代码, 具体数值可在 *SDK::xx_hw_platform_x/* 中查看
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

### 3.3.3. 用户程序的初始化
* 内存使用:
    * DDR: 0x100000 ~ +容量
    * OCM:
        * 192 KB, 0x00000000 ~ 0x0002FFFF
        * 64 KB, 0xFFFF0000 ~ 0x0000FDFF
            > 最后 0.5 KB 留给了 CPU 1
    * QSPI: 0xFC000000 ~ +容量
* 所有的 Section 都放在了 DDR 中
    * PL 可以使用所有的 OCM, 不用担心影响用户程序
* 程序进入 `main()` 之前, SDK 的底层代码进行了一系列操作, 与 FSBL 一样
    1. 复位后从 `asm_vectors.s` 跳转到 `boot.S`, 配合 `translation_table.s`, 完成 APU 初始化
    1. 跳转到 `xil-crt0.S` 中的 `_start`, 配合 `cpu_init.s`, 完成 C 的启动
    1. 跳转到 `main(0, 0)` 函数
* 因此, 运行用户代码前, CPU 已经设置好了: 异常向量表, CPU, FPU, L1, MMU, L2, SCU 等

## 3.4. APU API
* APU 信息:
    ```C
    #include "xplatform_info.h"
    u32 XGetPlatform_Info();
    ```
* 寄存器和伪汇编:
    ```C
    #include "xreg_cortexa9.h"
    #include "xpseudo_asm_gcc.h"
    ...
    ```
* FPU: 编译器自动调用相关指令集
* NEON:
    * 如果要启用, 需要设置编译器选项 `-mfloat-abi=softfp -mfpu=neon`
        * 在3个地方添加
            * 板级支持包
            * ARM gcc compiler --> Miscellaneous
            * ARM gcc linker --> Miscellaneous
    * EABI 和库提供 NEON 的 API
        ```C
        #include "arm_neon.h"
        ...
        ```
    * 也可以设置向量化相关的编译器属性, 使编译器调用 NEON 指令集
* MMU:
    * SDK 中的启动代码会开启和初始化
        * MMU 转换表 `translation_table.S`
    * SDK 中的 BSP 库提供 API
        ```C
        #include "xil_mmu.h"
        void Xil_SetTlbAttributes(INTPTR Addr, u32 attrib);
        void Xil_EnableMMU(void);
        void Xil_DisableMMU(void);
        ```
* L1 和 L2 Cache:
    * SDK 中的启动代码会开启和初始化
        * 如果需要用到 L2 的中断, 事件和 lockdown, 等需要自己设置
    * SDK 提供 API
        ```C
        #include "xil_hal.h"
        void Xil_DCacheEnable(void);
        void Xil_DCacheDisable(void);
        void Xil_DCacheInvalidate(void);
        void Xil_DCacheInvalidateRange(INTPTR adr, u32 len);
        void Xil_DCacheFlush(void);
        void Xil_DCacheFlushRange(INTPTR adr, u32 len);

        void Xil_ICacheEnable(void);
        void Xil_ICacheDisable(void);
        void Xil_ICacheInvalidate(void);
        void Xil_ICacheInvalidateRange(INTPTR adr, u32 len);
        ```
        * L1 和 L2 的底层函数:
            ```C
            #include "xil_cache_l.h"
            void Xil_DCacheInvalidateLine(u32 adr);
            void Xil_DCacheFlushLine(u32 adr);
            void Xil_DCacheStoreLine(u32 adr);
            void Xil_ICacheInvalidateLine(u32 adr);

            void Xil_L1DCacheEnable(void);
            void Xil_L1DCacheDisable(void);
            void Xil_L1DCacheInvalidate(void);
            void Xil_L1DCacheInvalidateLine(u32 adr);
            void Xil_L1DCacheInvalidateRange(u32 adr, u32 len);
            void Xil_L1DCacheFlush(void);
            void Xil_L1DCacheFlushLine(u32 adr);
            void Xil_L1DCacheFlushRange(u32 adr, u32 len);
            void Xil_L1DCacheStoreLine(u32 adr);

            void Xil_L1ICacheEnable(void);
            void Xil_L1ICacheDisable(void);
            void Xil_L1ICacheInvalidate(void);
            void Xil_L1ICacheInvalidateLine(u32 adr);
            void Xil_L1ICacheInvalidateRange(u32 adr, u32 len);

            void Xil_L2CacheEnable(void);
            void Xil_L2CacheDisable(void);
            void Xil_L2CacheInvalidate(void);
            void Xil_L2CacheInvalidateLine(u32 adr);
            void Xil_L2CacheInvalidateRange(u32 adr, u32 len);
            void Xil_L2CacheFlush(void);
            void Xil_L2CacheFlushLine(u32 adr);
            void Xil_L2CacheFlushRange(u32 adr, u32 len);
            void Xil_L2CacheStoreLine(u32 adr);
            ```
* SCU
    * SDK 中的启动代码会开启和初始化
* 异常与中断 (基础 API):
    * 机制:
        1. 异常向量入口 (`asm_vectors.S`) → 底层代码 (`asm_vectors.S`) 将通用寄存器和浮点寄存器压栈 → 通过函数指针定向到 ISR (`vectors.c`)
            * `vectors.c` 中有一个全局的 `XExc_VectorTable[]`, 它存储了各个异常的 ISR 函数指针, 默认提供了 Undefined, PrefetchAbort, DataAbort 三种异常的 ISR 实体, 其他的都是死循环, 所以在使用中断前, 需要先注册 ISR 函数指针
        1. 执行用户 ISR
        1. 从用户 ISR 返回, 底层代码 (`asm_vectors.S`) 将通用寄存器和浮点寄存器出栈 → 跳回异常发生前的代码
    * API
        ``` C
        #include "xil_hal.h"

        void Xil_ExceptionInit(void);   // 实际是空函数
        Xil_ExceptionEnableMask(Mask);
        Xil_ExceptionEnable();          // 只使能 IRQ
        Xil_ExceptionDisableMask(Mask);
        Xil_ExceptionDisable();

        Xil_EnableNestedInterrupts();
        Xil_DisableNestedInterrupts();

        void Xil_ExceptionRegisterHandler(u32 Exception_id, Xil_ExceptionHandler Handler, void *Data);
            // Exception_id 是指哪个异常 (IRQ, FIQ, ...)
        void Xil_ExceptionRemoveHandler(u32 Exception_id);
        void Xil_GetExceptionRegisterHandler(u32 Exception_id, Xil_ExceptionHandler *Handler, void **Data);

        void Xil_DataAbortHandler(void *CallBackRef);
        void Xil_PrefetchAbortHandler(void *CallBackRef);
        void Xil_UndefinedExceptionHandler(void *CallBackRef);
        ```
* 内存访问:
    ``` C
    #include "xil_mem.h"
    void Xil_MemCpy(void* dst, const void* src, u32 cnt);
    //
    #include "xil_hal.h"
    u16 Xil_EndianSwap16(u16 Data)`, `u16 Xil_Htons(u16 Data)`, `u16 Xil_Ntohs(u16 Data);
    u32 Xil_EndianSwap32(u32 Data)`, `u32 Xil_Htonl(u32 Data)`, `u32 Xil_Ntohl(u32 Data);

    static INLINE u8 Xil_In8 (UINTPTR Addr);
    static INLINE u16 Xil_In16 (UINTPTR Addr);
    static INLINE u32 Xil_In32 (UINTPTR Addr);
    static INLINE u64 Xil_In64 (UINTPTR Addr);
    static INLINE void Xil_Out8 (UINTPTR Addr, u8 Value);
    static INLINE void Xil_Out16 (UINTPTR Addr, u16 Value);
    static INLINE void Xil_Out32 (UINTPTR Addr, u32 Value);
    static INLINE void Xil_Out64 (UINTPTR Addr, u64 Value);

    static INLINE u16 Xil_In16LE (UINTPTR Addr);
    static INLINE u32 Xil_In32LE (UINTPTR Addr);
    static INLINE void Xil_Out16LE (UINTPTR Addr, u16 Value);
    static INLINE void Xil_Out32LE (UINTPTR Addr, u32 Value);
    static INLINE u16 Xil_In16BE (UINTPTR Addr);
    static INLINE u32 Xil_In32BE (UINTPTR Addr);
    static INLINE void Xil_Out16BE (UINTPTR Addr, u16 Value);
    static INLINE void Xil_Out32BE (UINTPTR Addr, u32 Value);
    ```
* 时间
    ``` C
    #include "sleep.h"
    void usleep(unsigned long useconds);
    void sleep(unsigned int seconds);

    #include "xtime_l.h"
    void XTime_SetTime (XTime Xtime_Global);
    void XTime_GetTime (XTime Xtime_Global);
    ```
* 标准字符 I/O:
    ``` C
    #include "xil_printf.h"
    void xil_printf( const char8 *ctrl1, ...);
    extern char8 inbyte(void);
    ```
* Assert:
    ``` C
    #include "xil_hal.h"
    #define Xil_AssertVoid(Expression)
    #define Xil_AssertNonvoid(Expression)
    #define Xil_AssertVoidAlways()
    #define Xil_AssertNonvoidAlways()
    ```
## 3.5. 外设 API

### 3.5.1. 复位外设
``` C
#include "xil_misc_psreset_api.h"`
void XDdr_ResetHw(void);                // 复位 DDR
void XOcm_Remap(void);                  // 复位 OCM 映射
void XSmc_ResetHw(u32 BaseAddress);     // 复位 SMC
void XSlcr_MioWriteResetValues(void);   // 复位 MIO
void XSlcr_PllWriteResetValues(void);   // 复位 PLL 的 FCLKRESETN
void XSlcr_DisableLevelShifters(void);  // 禁用 PS-PL 的电平转换器
void XSlcr_GpioPsReset(void);           // 复位 GPIO
void XSlcr_DmaPsReset(void);            // 复位 DMA
void XSlcr_SmcPsReset(void);            // 复位 SMC
void XSlcr_CanPsReset(void);            // 复位 CAN
void XSlcr_UartPsReset(void);           // 复位 UART
void XSlcr_I2cPsReset(void);            // 复位 I2C
void XSlcr_SpiPsReset(void);            // 复位 SPI
void XSlcr_QspiPsReset(void);           // 复位 QSPI
void XSlcr_UsbPsReset(void);            // 复位 USB
void XSlcr_EmacPsReset(void);           // 复位 EMAC
void XSlcr_OcmReset(void);              // 复位 OCM
```
### 3.5.2. 基本使用规则
* `#include "xparameters.h"` 中列举了所有外设的地址和 Macro
* 外设的使用一般为以下几步:
    1. 根据外设 ID 初始化外设信息结构体:
        ``` C
        xxConfig = xx_LookupConfig(DeviceId);
        ```
        * 外设 ID 可以从 "xparameters.h" 查找, `XPAR_xx_x_DEVICE_ID`, 如 I2C0 的 ID 为 0
        * 外设信息结构体主要内容: 外设 ID, 寄存器的基地址, ...
    1. 根据外设信息结构体初始化外设, 并返回外设句柄:
        ``` C
        Status = xx_CfgInitialize(&外设句柄, 外设结构体, 外设结构体->BaseAddr);
        ```
        * 外设句柄主要内容: 外设构体, 外设信息和状态, 回调函数, ...
    1. [可选] 初始化中断
        ``` C
        // 初始化 GIC 外设结构体
        IntcConfig = XScuGic_LookupConfig(INTC_DEVICE_ID);
        ...
        // 初始化 GIC 外设, 并得到 GIC 外设句柄
        Status = XScuGic_CfgInitialize(GicInstancePtr, IntcConfig, IntcConfig->CpuBaseAddress);
        ...
        // 将 GIC 中断入口注册到 IRQ
        Xil_ExceptionRegisterHandler(XIL_EXCEPTION_ID_INT,
                    (Xil_ExceptionHandler)XScuGic_InterruptHandler,
                    GicInstancePtr);
        // 连接外设的中断入口到 GIC 中断入口
        Status = XScuGic_Connect(GicInstancePtr, GpioIntrId,
                    (Xil_ExceptionHandler)XGpioPs_IntrHandler,
                    (void *)Gpio);
        // 外设中断的初始化, 使能
        ...
        // GIC 使能
        XScuGic_Enable(GicInstancePtr, GpioIntrId);
        // 异常入口使能
        Xil_ExceptionEnableMask(XIL_EXCEPTION_IRQ);
        ```
    1. 使用外设进行操作

### 3.5.3. GIC
* 机制
    1. 将用户 ISR 注册到外设 ISR 入口
    1. 将外设 ISR 入口注册到 GIC 的通用 ISR 入口
    1. 将 GIC 的通用 ISR 注册到底层代码的异常入口
* API
    ``` C
    #include "xscugic.h"
    // 初始化
    XScuGic_Config *XScuGic_LookupConfig(u16 DeviceId);
    s32 XScuGic_CfgInitialize(XScuGic *InstancePtr, XScuGic_Config *ConfigPtr, u32 EffectiveAddr);
        // ICD: 所有中断的优先级为 0xA0, 目标 CPU 为当前 CPU, 敏感电平为默认, 中断被关闭, ICD 使能
        // ICC: 优先级屏蔽值为 0xF0, 使用 FIQ 触发 MON 模式, ICC 使能

    // 标准 GIC 中断函数, 一般用它注册到 IRQ 或 FIQ
    void XScuGic_InterruptHandler(XScuGic *InstancePtr);
        // 1. 读中断确认寄存器 (ICCIAR), 提取中断 ID
        // 2. 根据中断 ID, 调用外设 ISR (继而调用用户 ISR)
        // 3. 读中断确认值到 (ICCEOIR) 以清除当前中断

    // 链接外设中断函数
    s32  XScuGic_Connect(XScuGic *InstancePtr, u32 Int_Id, Xil_InterruptHandler Handler, void *CallBackRef);
    void XScuGic_Disconnect(XScuGic *InstancePtr, u32 Int_Id);

    // 管理外设中断
    void XScuGic_Enable(XScuGic *InstancePtr, u32 Int_Id);
    void XScuGic_Disable(XScuGic *InstancePtr, u32 Int_Id);
    void XScuGic_Stop(XScuGic *InstancePtr);

    // SGI
    s32  XScuGic_SoftwareIntr(XScuGic *InstancePtr, u32 Int_Id, u32 Cpu_Id);

    // 外设中断参数: 优先级, 目标CPU, 信号敏感类型
    void XScuGic_SetPriorityTriggerType(XScuGic *InstancePtr, u32 Int_Id, u8 Priority, u8 Trigger);
    void XScuGic_GetPriorityTriggerType(XScuGic *InstancePtr, u32 Int_Id, u8 *Priority, u8 *Trigger);
    void XScuGic_InterruptMaptoCpu(XScuGic *InstancePtr, u8 Cpu_Id, u32 Int_Id);
    void XScuGic_InterruptUnmapFromCpu(XScuGic *InstancePtr, u8 Cpu_Id, u32 Int_Id);
    void XScuGic_UnmapAllInterruptsFromCpu(XScuGic *InstancePtr, u8 Cpu_Id);
    void XScuGic_SetCpuID(u32 CpuCoreId);
    u32  XScuGic_GetCpuID(void);
    ```

### 3.5.4. DMA

### 3.5.5. 定时器与看门狗
* 全局定时器:
    * 已被底层代码启动, 一般用于延时
    * API
        ``` C
        #include "xtime_l.h"

        void XTime_GetTime(XTime *Xtime_Global);
        void XTime_SetTime(XTime Xtime_Global);
        ```
    *
* 私有定时器



### 3.5.6. GPIO
* 基本流程
    1. 初始化
    1. 设置 GPIO 的方向和输出使能, 将 GPIO 配置为输入或输出
    1. 读写端口值
* API
    ``` C
    // 初始化
    XGpioPs_Config *XGpioPs_LookupConfig(u16 DeviceId);
    s32 XGpioPs_CfgInitialize(XGpioPs *InstancePtr, XGpioPs_Config *ConfigPtr, u32 EffectiveAddr);

    // GPIO 控制
    u32 XGpioPs_Read(XGpioPs *InstancePtr, u8 Bank);
    u32 XGpioPs_ReadPin(XGpioPs *InstancePtr, u32 Pin);
    void XGpioPs_Write(XGpioPs *InstancePtr, u8 Bank, u32 Data);
    void XGpioPs_WritePin(XGpioPs *InstancePtr, u32 Pin, u32 Data);
    void XGpioPs_SetDirection(XGpioPs *InstancePtr, u8 Bank, u32 Direction);
    void XGpioPs_SetDirectionPin(XGpioPs *InstancePtr, u32 Pin, u32 Direction);
    u32 XGpioPs_GetDirection(XGpioPs *InstancePtr, u8 Bank);
    u32 XGpioPs_GetDirectionPin(XGpioPs *InstancePtr, u32 Pin);
    void XGpioPs_SetOutputEnable(XGpioPs *InstancePtr, u8 Bank, u32 OpEnable);
    void XGpioPs_SetOutputEnablePin(XGpioPs *InstancePtr, u32 Pin, u32 OpEnable);
    u32 XGpioPs_GetOutputEnable(XGpioPs *InstancePtr, u8 Bank);
    u32 XGpioPs_GetOutputEnablePin(XGpioPs *InstancePtr, u32 Pin);

    void XGpioPs_GetBankPin(u8 PinNumber, u8 *BankNumber, u8 *PinNumberInBank);

    // 中断
    void XGpioPs_IntrEnable(XGpioPs *InstancePtr, u8 Bank, u32 Mask);
    void XGpioPs_IntrEnablePin(XGpioPs *InstancePtr, u32 Pin);
    void XGpioPs_IntrDisable(XGpioPs *InstancePtr, u8 Bank, u32 Mask);
    void XGpioPs_IntrDisablePin(XGpioPs *InstancePtr, u32 Pin);
    u32 XGpioPs_IntrGetEnabled(XGpioPs *InstancePtr, u8 Bank);
    u32 XGpioPs_IntrGetEnabledPin(XGpioPs *InstancePtr, u32 Pin);
    u32 XGpioPs_IntrGetStatus(XGpioPs *InstancePtr, u8 Bank);
    u32 XGpioPs_IntrGetStatusPin(XGpioPs *InstancePtr, u32 Pin);
    void XGpioPs_IntrClear(XGpioPs *InstancePtr, u8 Bank, u32 Mask);
    void XGpioPs_IntrClearPin(XGpioPs *InstancePtr, u32 Pin);
    void XGpioPs_SetIntrType(XGpioPs *InstancePtr, u8 Bank, u32 IntrType, u32 IntrPolarity, u32 IntrOnAny);
    void XGpioPs_SetIntrTypePin(XGpioPs *InstancePtr, u32 Pin, u8 IrqType);
    void XGpioPs_GetIntrType(XGpioPs *InstancePtr, u8 Bank, u32 *IntrType, u32 *IntrPolarity, u32 *IntrOnAny);
    u8 XGpioPs_GetIntrTypePin(XGpioPs *InstancePtr, u32 Pin);

    void XGpioPs_SetCallbackHandler(XGpioPs *InstancePtr, void *CallBackRef,XGpioPs_Handler FuncPointer);
    void XGpioPs_IntrHandler(XGpioPs *InstancePtr);
    ```

### 3.5.7. GigE

--------------------------------------------------------------------------------
==继续==

### 3.5.8. I2C
* 基本流程
    1. 初始化
    1. 设置时钟
    1. [可选] 设置选项
    1. 如果不使用中断
        1. 使用轮询发送或接收 API
    1. 如果使用中断
        1. 初始化 GIC, 注册异常
        1. 链接 IIC 中断标准入口
        1. 使能 GIC, 使能异常
        1. 链接用户 ISR 到 IIC 中断标准入口
        1. 使用普通发送或接收 API
* API
    ``` C
    // 初始化
    XIicPs_Config *XIicPs_LookupConfig(u16 DeviceId);
    s32 XIicPs_CfgInitialize(XIicPs *InstancePtr, XIicPs_Config * ConfigPtr, u32 EffectiveAddr);

    // 控制与状态
    void XIicPs_Reset(XIicPs *InstancePtr);
    void XIicPs_Abort(XIicPs *InstancePtr);
    s32 XIicPs_BusIsBusy(XIicPs *InstancePtr);
    s32 TransmitFifoFill(XIicPs *InstancePtr);

    s32 XIicPs_SetOptions(XIicPs *InstancePtr, u32 Options);
    s32 XIicPs_ClearOptions(XIicPs *InstancePtr, u32 Options);
    u32 XIicPs_GetOptions(XIicPs *InstancePtr);
    s32 XIicPs_SetSClk(XIicPs *InstancePtr, u32 FsclHz);
    u32 XIicPs_GetSClk(XIicPs *InstancePtr);

    // master
    void XIicPs_MasterSend(XIicPs *InstancePtr, u8 *MsgPtr, s32 ByteCount, u16 SlaveAddr);
    void XIicPs_MasterRecv(XIicPs *InstancePtr, u8 *MsgPtr, s32 ByteCount, u16 SlaveAddr);
    s32 XIicPs_MasterSendPolled(XIicPs *InstancePtr, u8 *MsgPtr, s32 ByteCount, u16 SlaveAddr);
    s32 XIicPs_MasterRecvPolled(XIicPs *InstancePtr, u8 *MsgPtr, s32 ByteCount, u16 SlaveAddr);
    void XIicPs_EnableSlaveMonitor(XIicPs *InstancePtr, u16 SlaveAddr);
    void XIicPs_DisableSlaveMonitor(XIicPs *InstancePtr);
    void XIicPs_MasterInterruptHandler(XIicPs *InstancePtr);    // 标准 IIC 中断 ISR

    // slave
    void XIicPs_SetupSlave(XIicPs *InstancePtr, u16 SlaveAddr);
    void XIicPs_SlaveSend(XIicPs *InstancePtr, u8 *MsgPtr, s32 ByteCount);
    void XIicPs_SlaveRecv(XIicPs *InstancePtr, u8 *MsgPtr, s32 ByteCount);
    s32 XIicPs_SlaveSendPolled(XIicPs *InstancePtr, u8 *MsgPtr, s32 ByteCount);
    s32 XIicPs_SlaveRecvPolled(XIicPs *InstancePtr, u8 *MsgPtr, s32 ByteCount);
    void XIicPs_SlaveInterruptHandler(XIicPs *InstancePtr);     // 标准 IIC 中断 ISR

    // 中断
    void XIicPs_SetStatusHandler(XIicPs *InstancePtr, void *CallBackRef, XIicPs_IntrHandler FunctionPtr);
    ```

## 3.6. OS

### 3.6.1. 嵌入式 - FreeRTOS

### 3.6.2. Linux - PetaLinux

