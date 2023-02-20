Zynq总结: 设计流程
---

--------------------------------------------------------------------------------
# 1. AXI DMA
> 参见官方文档: PG201

![AXI DMA Block Diagram](/assets/PG201_Fig1-1.png)
* 两种模式:
    * 直接传输模式 (简单模式): 设置目标地址和传输长度
    * Scatter-Gather 模式: 使用Buffer Descriptor (BD) 和内存空间的方法
* 支持多通道: 最多 16 通道
    > 多通道 DMA 用 IP 核: "AXI Multichannel Direct Memory Access"
* 支持字节对齐
* 支持用户应用数据
* 支持 Micro 模式
    * 性能较低, 但结构简单

## 1.1. 寄存器
> 参见官方文档: PG201

![AXI DMA Register](/assets/PG201_Tab2-5.png)

![AXI DMA Register](/assets/PG201_Tab2-6-1.png)
![AXI DMA Register](/assets/PG201_Tab2-6-2.png)

## 1.2. 简单模式
### 1.2.1. 编程顺序
* MM2S 通道
    1. 启动通道
    1. 【可选】配置中断
    1. 设置 MM2S_SA 寄存器 (待发送数据的地址)
    1. 设置 MM2S_LENGTH 寄存器 (发送多少字节数据)
* S2MM 通道
    1. 启动通道
    1. 【可选】配置中断
    1. 设置 MM2S_DA 寄存器 (接收的数据存在哪里)
    1. 如果 DMA 没有配置为数据重对齐模式, 那么 MM2S_DA 必须是对齐的, 否则出现错误
    1. 设置 S2MM_LENGTH 寄存器 (接收多少字节数据)

## 1.3. Scatter / Gather 模式
### 1.3.1. Scatter Gather Descriptor
> 参见官方文档: PG201

![Scatter Gather Descriptor](/assets/PG201_Tab2-26.png)
* 控制: 最大接收数据 buffer 的长度
* 状态: SOF (Start of Frame), EOF (End of Frame), 接收到的数据 buffer 的长度, 错误状态, 该 BD 是否接收完成
* 可以配置为环形模式
    * 最后一个 BD 的指针指向第一个 BD

### 1.3.2. 编程顺序
* MM2S 通道
    1. 将 BD 链的起始地址写入寄存器
    1. 启动通道
    1. 【可选】配置中断
    1. 将 BD 链的终止地址写入寄存器
    1. 开始传输
* S2MM 通道
    1. 将 BD 链的起始地址写入寄存器
    1. 启动通道
    1. 【可选】配置中断
    1. 将 BD 链的终止地址写入寄存器
        > 如果 BD 链是环形模式, 终止地址可以设置成一个无效地址
    1. 开始传输

## 1.4. API

### 1.4.1. 初始化
1. `XAxiDma_Config *XAxiDma_LookupConfig(u32 DeviceId)`
1. `int XAxiDma_CfgInitialize(XAxiDma * InstancePtr, XAxiDma_Config *Config)`
1. 【可选】查询 AXI DMA 的工作模式: `bool XAxiDma_HasSg(InstancePtr)`

### 1.4.2. 简单模式
1. 【可选】设置中断, 相关 API
    ``` c
    void XAxiDma_IntrEnable(InstancePtr, Mask, Direction)
    u32 XAxiDma_IntrGetEnabled(InstancePtr, Direction)
    void XAxiDma_IntrDisable(InstancePtr, Mask, Direction)
    ```
1. 发送或接收
    * `u32 XAxiDma_SimpleTransfer(XAxiDma *InstancePtr, UINTPTR BuffAddr, u32 Length, int Direction)`
1. 【可选】ISR
    1. 获取中断状态, 并确认中断
        ``` c
        u32 XAxiDma_IntrGetIrq(InstancePtr, Direction)
        void XAxiDma_IntrAckIrq(InstancePtr, Mask, Direction)
        ```
    1. 根据中断状态, 进行后续处理

### 1.4.3. SG 模式

#### 1.4.3.1. 发送
* 初始化
    1. 获取 BD ring 指针: `XAxiDma_BdRing * XAxiDma_GetTxRing(XAxiDma * InstancePtr)`
    1. 禁用中断: `void XAxiDma_BdRingIntDisable(XAxiDma_BdRing* RingPtr,*u32 Mask)`
        > 与 `void XAxiDma_IntrDisable(InstancePtr, Mask, Direction)` 一样
    1. 创建 BD ring, 相关 API
        ``` c
        u32 XAxiDma_BdRingCreate(XAxiDma_BdRing * RingPtr, UINTPTR PhysAddr, UINTPTR VirtAddr, u32 Alignment, int BdCount);
        int XAxiDma_BdRingClone(XAxiDma_BdRing * RingPtr, XAxiDma_Bd * SrcBdPtr);
        ```
        > 可用 `XAxiDma_BdClear(&BdTemplate)` 清除一个 BD 作为模板
    1. 设置 IRQDelay 和 IRQThreshold: `int XAxiDma_BdRingSetCoalesce(XAxiDma_BdRing * RingPtr, u32 Counter, u32 Timer)`
        > IRQDelay: 每隔多长时间产生一次中断
        > IRQThreshold: 发送完成多少次之后产生一个中断
    1. 使能中断: `void XAxiDma_BdRingIntEnable(XAxiDma_BdRing* RingPtr, u32 Mask)`
    1. 启动发送通道: `int XAxiDma_BdRingStart(XAxiDma_BdRing * RingPtr)`
* 发送数据
    1. 初始化数据, 必要时冲刷 Cache
        * 冲刷 Cache: `Xil_DCacheFlushRange`
    1. 分配发送 buffer 到 BD ring
        1. 申请使用一些 BD: `int XAxiDma_BdRingAlloc(XAxiDma_BdRing * RingPtr, int NumBd, XAxiDma_Bd ** BdSetPtr)`
            > 可用 `int XAxiDma_BdRingGetFreeCnt(XAxiDma_BdRing* RingPtr)` 查询当前有多少个 BD 可供使用
            > `BdSetPtr` 是当前 BD 的指针, 可作为锚点以访问后续的 BD
        1. 依次初始化申请的 BD
            1. 设置 buffer 地址: `u32 XAxiDma_BdSetBufAddr(XAxiDma_Bd* BdPtr, UINTPTR Addr)`
            1. 设置接收长度: `int XAxiDma_BdSetLength(XAxiDma_Bd* BdPtr, u32 LenBytes, u32 LengthMask)`
            1. 设置控制寄存器 (TXSOF、TXEOF): `void XAxiDma_BdSetCtrl(XAxiDma_Bd *BdPtr, u32 Data)`
            > 可用 `XAxiDma_Bd *XAxiDma_BdRingNext(XAxiDma_BdRing* RingPtr, XAxiDma_Bd *BdPtr)` 跳转到下一个 BD
    1. 【可选】启用循环模式:
        ``` c
        void XAxiDma_BdRingEnableCyclicDMA(XAxiDma_BdRing* RingPtr)
        int XAxiDma_SelectCyclicMode(XAxiDma *InstancePtr, int Direction, int Select)
        ```
    1. 将设置好的 BD 链接到硬件: `int XAxiDma_BdRingToHw(XAxiDma_BdRing * RingPtr, int NumBd, XAxiDma_Bd * BdSetPtr)`
* ISR
    1. 获取中断状态, 并确认中断
        ``` c
        u32 XAxiDma_BdRingGetIrq(XAxiDma_BdRing* RingPtr)
        void XAxiDma_BdRingAckIrq(XAxiDma_BdRing* RingPtr)
        ```
    1. 根据中断状态进行后续处理
        * 如果无中断: 退出
        * 如果有错误:
            * 可以用 `void XAxiDma_BdRingDumpRegs(XAxiDma_BdRing *RingPtr)` 输出错误信息以便诊断
            * 复位: `void XAxiDma_Reset(XAxiDma * InstancePtr)`
        * 如果有数据: 处理数据
    1. 处理数据
        1. 获取已处理的 BD: `int XAxiDma_BdRingFromHw(XAxiDma_BdRing * RingPtr, int BdLimit, XAxiDma_Bd ** BdSetPtr)`
        1. 检查每个 BD 的发送状态: `u32 XAxiDma_BdGetSts(XAxiDma_Bd* BdPtr)`
            > 可用 `XAxiDma_Bd *XAxiDma_BdRingNext(XAxiDma_BdRing* RingPtr, XAxiDma_Bd *BdPtr)` 跳转到下一个 BD
    1. 【非 Cyclic DMA Mode】为下次发送准备好 BD
        1. 释放已经处理过的 BD: `int XAxiDma_BdRingFree(XAxiDma_BdRing * RingPtr, int NumBd, XAxiDma_Bd * BdSetPtr)`
        1. 分配发送 buffer 到 BD ring
            1. 申请使用一些 BD: `int XAxiDma_BdRingAlloc(XAxiDma_BdRing * RingPtr, int NumBd, XAxiDma_Bd ** BdSetPtr)`
                > 可用 `int XAxiDma_BdRingGetFreeCnt(XAxiDma_BdRing* RingPtr)` 查询当前有多少个 BD 可供使用
                > `BdSetPtr` 是当前 BD 的指针, 可作为锚点以访问后续的 BD
            1. 依次初始化申请的 BD
                1. 设置 buffer 地址: `u32 XAxiDma_BdSetBufAddr(XAxiDma_Bd* BdPtr, UINTPTR Addr)`
                1. 设置接收长度: `int XAxiDma_BdSetLength(XAxiDma_Bd* BdPtr, u32 LenBytes, u32 LengthMask)`
                1. 设置控制寄存器 (TXSOF、TXEOF): `void XAxiDma_BdSetCtrl(XAxiDma_Bd *BdPtr, u32 Data)`
                > 可用 `XAxiDma_Bd *XAxiDma_BdRingNext(XAxiDma_BdRing* RingPtr, XAxiDma_Bd *BdPtr)` 跳转到下一个 BD
        1. 将设置好的 BD 链接到硬件: `int XAxiDma_BdRingToHw(XAxiDma_BdRing * RingPtr, int NumBd, XAxiDma_Bd * BdSetPtr)`
#### 1.4.3.2. 接收
* 初始化
    1. 获取 BD ring 指针: `XAxiDma_BdRing * XAxiDma_GetRxRing(XAxiDma * InstancePtr)`
    1. 禁用中断: `void XAxiDma_BdRingIntDisable(XAxiDma_BdRing* RingPtr,*u32 Mask)`
        > 与 `void XAxiDma_IntrDisable(InstancePtr, Mask, Direction)` 一样
    1. 创建 BD ring, 相关 API
        ``` c
        u32 XAxiDma_BdRingCreate(XAxiDma_BdRing * RingPtr, UINTPTR PhysAddr, UINTPTR VirtAddr, u32 Alignment, int BdCount);
        int XAxiDma_BdRingClone(XAxiDma_BdRing * RingPtr, XAxiDma_Bd * SrcBdPtr);
        ```
        > 可用 `XAxiDma_BdClear(&BdTemplate)` 清除一个 BD 作为模板
    1. 分配接收 buffer 到 BD ring
        1. 申请使用一些 BD: `int XAxiDma_BdRingAlloc(XAxiDma_BdRing * RingPtr, int NumBd, XAxiDma_Bd ** BdSetPtr)`
            > 可用 `int XAxiDma_BdRingGetFreeCnt(XAxiDma_BdRing* RingPtr)` 查询当前有多少个 BD 可供使用
            > `BdSetPtr` 是当前 BD 的指针, 可作为锚点以访问后续的 BD
        1. 依次初始化申请的 BD
            1. 设置 buffer 地址: `u32 XAxiDma_BdSetBufAddr(XAxiDma_Bd* BdPtr, UINTPTR Addr)`
            1. 设置接收长度: `int XAxiDma_BdSetLength(XAxiDma_Bd* BdPtr, u32 LenBytes, u32 LengthMask)`
            1. 设置控制寄存器 (RXSOF、RXEOF): `void XAxiDma_BdSetCtrl(XAxiDma_Bd *BdPtr, u32 Data)`
            > 可用 `XAxiDma_Bd *XAxiDma_BdRingNext(XAxiDma_BdRing* RingPtr, XAxiDma_Bd *BdPtr)` 跳转到下一个 BD
    1. 设置 IRQDelay 和 IRQThreshold: `int XAxiDma_BdRingSetCoalesce(XAxiDma_BdRing * RingPtr, u32 Counter, u32 Timer)`
        > IRQDelay: 每隔多长时间产生一次中断
        > IRQThreshold: 发送完成多少次之后产生一个中断
    1. 将设置好的 BD 链接到硬件: `int XAxiDma_BdRingToHw(XAxiDma_BdRing * RingPtr, int NumBd, XAxiDma_Bd * BdSetPtr)`
    1. 使能中断: `void XAxiDma_BdRingIntEnable(XAxiDma_BdRing* RingPtr, u32 Mask)`
    1. 【可选】启用循环模式:
        ``` c
        void XAxiDma_BdRingEnableCyclicDMA(XAxiDma_BdRing* RingPtr)
        int XAxiDma_SelectCyclicMode(XAxiDma *InstancePtr, int Direction, int Select)
        ```
    1. 启动接收通道: `int XAxiDma_BdRingStart(XAxiDma_BdRing * RingPtr)`
* ISR
    1. 获取中断状态, 并确认中断
        ``` c
        u32 XAxiDma_BdRingGetIrq(XAxiDma_BdRing* RingPtr)
        void XAxiDma_BdRingAckIrq(XAxiDma_BdRing* RingPtr)
        ```
    1. 根据中断状态进行后续处理
        * 如果无中断: 退出
        * 如果有错误:
            * 可以用 `void XAxiDma_BdRingDumpRegs(XAxiDma_BdRing *RingPtr)` 输出错误信息以便诊断
            * 复位: `void XAxiDma_Reset(XAxiDma * InstancePtr)`
        * 如果有数据: 处理数据
    1. 处理数据
        1. 获取已处理的 BD: `int XAxiDma_BdRingFromHw(XAxiDma_BdRing * RingPtr, int BdLimit, XAxiDma_Bd ** BdSetPtr)`
        1. 检查每个 BD 的发送状态: `u32 XAxiDma_BdGetSts(XAxiDma_Bd* BdPtr)`
            > 可用 `XAxiDma_Bd *XAxiDma_BdRingNext(XAxiDma_BdRing* RingPtr, XAxiDma_Bd *BdPtr)` 跳转到下一个 BD
        1. 处理数据:
    1. 【非 Cyclic DMA Mode】为下次发送准备好 BD
        1. 释放已经处理过的 BD: `int XAxiDma_BdRingFree(XAxiDma_BdRing * RingPtr, int NumBd, XAxiDma_Bd * BdSetPtr)`
        1. 分配接收 buffer 到 BD ring
            1. 申请使用一些 BD: `int XAxiDma_BdRingAlloc(XAxiDma_BdRing * RingPtr, int NumBd, XAxiDma_Bd ** BdSetPtr)`
                > 可用 `int XAxiDma_BdRingGetFreeCnt(XAxiDma_BdRing* RingPtr)` 查询当前有多少个 BD 可供使用
                > `BdSetPtr` 是当前 BD 的指针, 可作为锚点以访问后续的 BD
            1. 依次初始化申请的 BD
                1. 设置 buffer 地址: `u32 XAxiDma_BdSetBufAddr(XAxiDma_Bd* BdPtr, UINTPTR Addr)`
                1. 设置接收长度: `int XAxiDma_BdSetLength(XAxiDma_Bd* BdPtr, u32 LenBytes, u32 LengthMask)`
                1. 设置控制寄存器 (RXSOF、RXEOF): `void XAxiDma_BdSetCtrl(XAxiDma_Bd *BdPtr, u32 Data)`
                > 可用 `XAxiDma_Bd *XAxiDma_BdRingNext(XAxiDma_BdRing* RingPtr, XAxiDma_Bd *BdPtr)` 跳转到下一个 BD
        1. 将设置好的 BD 链接到硬件: `int XAxiDma_BdRingToHw(XAxiDma_BdRing * RingPtr, int NumBd, XAxiDma_Bd * BdSetPtr)`

#### 1.4.3.3. 设置中断
1. 初始化 GIC
1. 初始化 AXI DMA 相关的 GIC 设置: `XScuGic_SetPriorityTriggerType(GIC 结构体指针, AXI DMA 中断号, 0xA0, 0x3)`
1. 链接 AXI DMA 的 ISR: `XScuGic_Connect(GIC 结构体指针, AXI DMA 中断号,
    ISR, BD ring 指针)`
    > 获取 BD ring 指针的 API:
    > * `XAxiDma_BdRing * XAxiDma_GetRxRing(XAxiDma * InstancePtr)`
    > * `XAxiDma_BdRing * XAxiDma_GetTxRing(XAxiDma * InstancePtr)`
