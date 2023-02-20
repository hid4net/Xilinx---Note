
Xilinx Ultrascale+系列FPGA总结：电路设计
---

==继续编写==


----
==以下作废==

相关文档:
分类      | 文档名称                                                       | 文档编号
:-------- | :------------------------------------------------------------- | :-------
基本结构  | Configuration User Guide                                       | UG470
^         | SelectIO Resources User Guide                                  | UG471
^         | Clocking Resources User Guide                                  | UG472
^         | Memory Resources User Guide                                    | UG473
^         | Configurable Logic Block User Guide                            | UG474
^         | DSP48E1 Slice User Guide                                       | UG479
收发器    | GTX/GTH Transceivers User Guide                                | UG476
^         | GTP Transceivers User Guide                                    | UG482
ADC       | XADC Dual 12-Bit 1 MSPS Analog-to-Digital Converter User Guide | UG480
封装和PCB | **Packaging and Pinout Product Specification**               | UG475
^         | **PCB Design Guide**                                          | UG483

--------------------------------------------------------------------------------

# 1. 硬件开发
Xilinx建议硬件开发时将时钟设计作为首要因素之一, 应在管脚分配之前

## 1.1. 电源
分类 | 电源      | 注意
:--: | :-------: | :-------
核心 | VCCINT    | 0.9V / 1.0V
^    | VCCBRAM   | 1.0V
^    | VCCAUX    | 1.8V
^    | VCCBATT_0 | 密钥的备用电池, 不用时连 Vcc 或 Gnd
BANK | VCCO_#    | HP Bank 最大支持 1.8V, HR 最大支持 3.3V
^    | VREF      | 信号的参考电压

### 1.1.1. 电源启动顺序
* 上电
    * 推荐的顺序: V~CCINT~ -> V~CCBRAM~ -> V~CCAUX~ -> V~CCAUX_IO~ -> V~CCO~
    * 如果 V~CCINT~ 和 V~CCBRAM~ 电压相同, 可以同时启动
    * 如果 V~CCAUX~, V~CCAUX_IO~ 和 V~CCO~ 电压相同, 可以同时启动
* 断电
    * 与上电顺序相反
    * 推荐的顺序: V~CCO~ -> V~CCAUX_IO~ -> V~CCAUX~  -> V~CCBRAM~ V~CCINT~
* GTX 收发器
    * 推荐的上电顺序: V~CCINT~ -> V~MGTAVCC~ -> V~MGTAVTT~ 或 V~MGTAVCC~ -> V~CCINT~ -> V~MGTAVTT~
    * V~CCINT~ 和 V~MGTAVCC~ 可以同时启动
    * V~MGTVCCAUX~ 没有要求
    * 断电顺序与上电顺序相反

## 1.2. 特殊信号
* 专用信号
    分类  | Pin         | I/O    | 注意
    :---: | :---------: | :----: | :-------
    JATG  | TCK_0       | I      | 官方使用 2mm 间距的插针
    ^     | TDI_0       | I      | ^
    ^     | TDO_0       | O      | ^
    ^     | TMS_0       | I      | ^
    CFG   | PROGRAM_B_0 | I      | 复位配置, 低有效, 最好用 RC POR 复位电路(或复位芯片), 上拉(4.7k)
    ^     | INIT_B_0    | IO(OD) | 表明配置存储器开始初始化, 需要上拉 (4.7k)
    ^     | DONE_0      | IO     | 是否配置成功, 高有效, 最好通过 NMOS 驱动一个 LED, 上拉(330)
    ^     | M[2:0]_0    | I      | 配置模式, Master_SPI 模式为 001, 串联电阻需 ≤ 1 kΩ
    ^     | CFGBVS_0    | I      | 如果与配置相关的 BANK 0, 14, 15 是 2.5/3.3 V供电, 需要拉到 2.5/3.3 V; 如果是 1.8V, 拉到 Gnd
    Flash | CCLK_0      | IO     | 配置时钟
* 多功能信号
    分类  | Pin          | I/O    | 注意
    :---: | :----------: | :----: | :-------
    Flash | D00 - D4     | IO     | 配置 Flash 的数据接口, 排序最好与 QSPI 保持一致
    ^     | FCS          | O      | 配置 Flash 的片选信号, 上拉 ≤ 4.7 kΩ
    Init  | PUDC_B       | I      | 0 = 使能内部上拉电阻, 1 = 禁用; 不能 float
* 内部 ADC
    Pin             | 注意
    :-------------: | :-------
    VCCADC_0        | 1.8V
    GNDADC_0        | Gnd
    VP_0/VN_0       | 专用模拟输入
    VREFP_0/VREFN_0 | 1.25V参考电压, 如果使用内部基准, 全接地
    AD[0:15]_P/N    | 辅助的模拟输入
* 温度传感器
    Pin          | 注意
    :----------: | :-------
    DXP_0, DXN_0 | 内部温度传感器, 不用时, 接地

## 1.3. 时钟输入
1. 供多个 Region 使用的时钟需要连接在 `Multi Region Clock Capable (MRCC)` 管脚上
1. 仅供单个 Region 使用的时钟需要连接在 `Single Region Clock Capable (SRCC)` 管脚上
1. 单端时钟必须该连到 `_P` 管脚

## 1.4. 用户IO
1. DCI 使用注意事项
    - 只有 HP Bank 支持, 最大电压 1.8V
    - VRN 上拉到 V~CCO~, VRP 下拉到 GND,
    - 阻值与特征阻抗相同, 或2倍于特征阻抗 (与电平标准相关), 内部DCI会校准到外部电阻值
    - 可以级联
1. 可以在逻辑设计前, 仅先分配信号的方向, 以便用于 SSN 分析（输入、输出对SSN的影响很不同）
1. Clock 的选择对管脚分配影响重大
1. Interface 信号最好分配在一个 Bank 内, 如果不行, 尽量分配在相邻的 Bank
1. 控制信号（时钟、复位、使能、选通等）最好在中间
1. 大扇出、系统级控制信号最好分配在器件中间
1. 配置管脚要特别注意
1. 高要求的接口需要通过硬件验证：
    * 为每个硬件接口创建一个专门的设计, 并用所需的最大带宽进行验证
    * A separate design should be created per hardware interface, and should be used to exercise the hardware at full bandwidth at the desired speed. FPGA internal loop-back or simple checkers can be used to verify that the data transmittal is successful at the desired speed.

## 1.5. 高速收发器
