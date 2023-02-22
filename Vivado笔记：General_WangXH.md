
<h1 style="text-align:center">Vivado使用</h1>

--------------------------------------------------------------------------------
# 1. 设计流程

![UltraFast Design Methodology System-Level Design Flow](/assets/UltraFast%20Design%20Methodology%20System-Level%20Design%20Flow.png)

--------------------------------------------------------------------------------
# 2. 设计输入

## 2.1. 验证源代码
* 创建端口时可以使用 XPATH 函数：
    * `number max(node-set)`
    * `number min(node-set)`
    * `number sum(node-set)`
    * `number log(base number, number)`
    * `number pow(number, exp number)`
    * `number floor(number)`
    * `number ceiling(number)`
    * `number round(number)`
    * `number abs(number)`
    * `boolean not(boolean)`
    * `boolean true()`
    * `boolean false()`

## 2.2. Block Design
* 创建子图: "右键菜单 -> create hierarchy"
* 添加代码为 block: "右键菜单 -> add module"

## 2.3. 生成 NGC
* 在使用FPGA验证时，我们经常会需要复用一些以前的项目模块，或者不希望我们的源代码被别人抄袭。我们也可以这样操作。具体的产生过程如下：
    1. 在 `XST->Process property` 中设置不要插入 IOBUF 。否则最终文件将会带有IOBUF，无法再集成到其他模块中
    1. 按照正常程序综合并 translate
    1. 如果没有遇到错误，在ISE项目文件夹下，就会产生和顶层文件同名的.ngc文件。到此，ngc文件就产生完了。同样，在以后用到该模块时，我们可以将ngc文件复制到新项目的根文件夹下，并将一个只保留端口定义的HDL文件加入项目中。就可以在新的项目中使用原有的模块了

--------------------------------------------------------------------------------
# 3. 仿真

## 3.1. 仿真数据的保存和读取
* 相关文件说明
    1. 波形数据库文件 (`.WDB`), 其中包含所有的仿真数据
    1. 波形配置文件 (`.WCFG`), 其中包含于波形配置文件中的对象相关联的顺序和设置
        在保存 `.WCFG` 文件之前, 对波形配置的修改 (包括创建波形配置或添加 HDL 对象) 不是永久性的, 可以通过 `File -> Save Waveform Configuration As` 将波形配置保存下来
        波形数据库文件 (`.WDB`) 包含了波形配置文件中所有信号的仿真数据, 单个 `.WDB` 可以对应多个 `.WCFG` 文件, 可以通过打开 `.WDB` 文件查看上一次保存下来的仿真波形
* 保存
    1. 保存 `.WCFG` 波形配置文件到指定路径
    1. 保存 `.WDB` 仿真波形
        为了能将波形数据保存下来, 需要在测试代码中加上如下代码, 放在最末端即可。为了下一次仿真不影响已保存的仿真波形, 建议将 .WDB 文件拷贝并修改名字
        ``` verilog
        `define dump_level 10
        //module dump_task;
        initial begin #1;   //延迟1ns记录, 方便与其他仿真动作协调
            `ifdef VCD_DUMP //工业标准VCD格式存储
            $display("Start Recording Waveform in VCD format!");
            $dumpfile("dump.vcd");
            $dumpvars('dump_level);
            `endif
        end
        ```
    1. 运行仿真 (前后仿的 `.WDB` 会自动保存到 `.sim/sim_1/` 下的三个路径之一, 与仿真类型有关)
* 读取
    1. 点击 Vivado 的菜单栏中的 `Flow -> Open Static simulation`, 然后选中之前保存的 `.WDB` 文件即可
    1. 点击 Vivado 的菜单栏中的 `File -> Open Waveform Configuration`, 选择我们之前保存的 `.WCFG` 文件即可恢复上一次的仿真结果

--------------------------------------------------------------------------------
# 4. Debug


--------------------------------------------------------------------------------
# 5. Programming

## 5.1. 普通 FPGA
* Vivado 配置选项
    * 可通过 "Edit Device Properties" 对话框设置配置相关的选项, 将保存在 XDC 文件中
        ![Vivado-DeviceProperties.png](/assets/Vivado-DeviceProperties.png)
    * "Edit Device Properties" 对话框打开方式:
        1. 打开综合或布局布线设计 -> Menu::Tools -> Edit Device Properties
            ![Vivado-DeviceProperties-Edit-1.png](/assets/Vivado-DeviceProperties-Edit-1.png)
        1. 打开综合或布局布线设计 -> Setting -> Edit Device Properties
            ![Vivado-DeviceProperties-Edit-2-1.png](/assets/Vivado-DeviceProperties-Edit-2-1.png)
            ![Vivado-DeviceProperties-Edit-2-2.png](/assets/Vivado-DeviceProperties-Edit-2-2.png)
* 远程更新
    * 方案 1: 配置数据 $\xrightarrow {通信接口}$ FPGA $\xrightarrow {QSPI控制器 + STRATUP}$ 外部配置用的 QSPI Flash
        > ![XAPP1081_Fig7.png](/assets/XAPP1081_Fig7.png)
        * 需要使用 STRATUP 原语才能连接 QSPI Flash 的时钟
        * 最后启用 MultiBoot 和 Fallback 选项
            > 在 Vivado::Device Properties 中设置
        > 参见: **XAPP1081**, XAPP1257, UG570
        > 参见: \Xilinx\FPGA Configuration\ 下相关文件


## 5.2. Zynq




