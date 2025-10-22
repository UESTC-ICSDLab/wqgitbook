# 数据存储器

<!-- -->
> #### flag::构建数据存储器
> 在实验二中, 我们构建了有处理器内核和指令存储器的 SoC, 它能执行指令和利用内部的寄存器存储数据. 使用寄存器存储数据显然是不现实的, 因此, 在一个能够执行复杂程序的系统中, 必须拥有数据存储器.

<!-- -->
> #### question::数据存储器在SoC中扮演什么角色?
> 数据存储器不仅用于数据的运算, 还在程序跳转、异常处理的过程中发挥重要作用. 
> 数据存储器特定的一段地址空间会被编译器分配为栈存储空间, 在程序跳转和异常处理的过程中, 会通过堆栈和出栈的方式保存和恢复跳转前的执行状况. 这里介绍Cortex-M0处理器对异常的处理过程:
>
> + 异常发生后, 寄存器组中的 8 个寄存器, 即R0-R3, R12, 链接寄存器 (R14/LR), 返回地址寄存器 (R15/PC) 以及程序状态寄存器 (xPSR) 会被自动压入当前的栈空间里面, 以确保服务程序执行完成后还能继续返回当前程序.
> + 链接寄存器 (LR/R14) 会被自动更新为异常返回时的特殊值 (EXC_RETURN, 该值的使用可以保证返回地址的选择性, 如选择 MSP 还是 PSP), 然后 PC 寄存器的值更新为对应的异常向量地址, 然后开始执行异常服务程序.
> + 在异常服务程序执行结束以后, 会利用 LR 寄存器中的特殊值, 触发异常的返回机制. 处理器还会查看当前是否有其他异常需要处理, 如果没有, 处理器就会恢复之前存储在栈空间的寄存器, 继续执行异常之前的程序.

<!-- -->
> #### important::添加数据存储器的理论根据
> + 我们这里添加的数据存储器相当于是SoC的外设, 添加外设就要涉及到Memory Map与总线扩展的相关知识了.

## Memory Map 与总线扩展

本实验所搭建的 SoC 的 memory map 如下图所示.

<div align ="center"><img src="/img/lab3/05.png" alt="01" style="zoom:100%;" />
<center style="color:#000000;font-size:10pt;">memory map</center>

处理器核通过地址编码访问外设, 所有的外设在处理器核看来都是 memory map 上的一块连续区域, 访问这块区域就是访问对应的外设. 例如上图中的 RAMCODE , 其地址编码为 0x00000000-0x0000ffff , 那么处理器核通过 AHB 总线发出的任何一次总线操作, 只要地址总线上的值在 0x00000000-0x0000ffff 之间, 都认为是处理器核在向 RAMCODE 提出总线操作.

那么在 SoC 具有多个外设, 但是处理器核只有一个 Master 总线接口的情况下, 就需要用到总线扩展模块, 使处理器核能够访问多个外设, 在 *AMBA® 3 AHB-Lite Protocol* 文档中提供了 Single Master AHB Interconnect 结构, 如下图.

<div align ="center"><img src="/img/lab3/06.png" alt="01" style="zoom:100%;" />
<center style="color:#000000;font-size:10pt;">总线扩展</center>

总线扩展模块主要由两部分组成: Decoder 与 Slave MUX .

Decoder 的作用是对地址总线进行译码, 生成对应的外设的选择信号, 同样以 RAMCODE 外设为例, 由于其地址编码为0x00000000-0x0000ffff , 那么只要地址总线 HADDR 的高 16bit 为 0x0000 时, 地址总线的值必定处于 RAMCODE 的地址编码区域中, 则判定为处理器核对 RAMCODE 提起的一次总线操作, 因此 RAMCODE 对应的选择信号 HSEL 将会被置位有效; 若 HADDR 的高 16bit 不为 0x0000 时(例如0x4000), 地址总线的值不处于 RAMCODE 的地址编码区域, 则判定为不是对 RAMCODE 的总线操作, 因此对应的选择信号被置位无效. 如下图所示, 每一个外设在 Decoder 中都需要一个比较器用于产生相应的选择信号.

<div align ="center"><img src="/img/lab3/07.png" alt="01" style="zoom:100%;" />
<center style="color:#000000;font-size:10pt;">Decoder 内部结构</center>

需要注意的是, 对于每个外设, Decoder 利用地址总线生成选择信号 HSEL 所需要的宽度是不同的. 例如 RAMCODE 的地址编码为 0x00000000-0x0000ffff , 其有效长度为 0x0000-0xffff , 那么 RAMCODE 对于的选择信号(HSEL_RAMCODE)需要对 HADDR 的高 16bit 进行译码; 而 WaterLight 的地址编码为 0x40000000-0x4000000f , 其有效长度为 0x0-0xf , 因此WaterLight 对应的选择信号(HSEL_WaterLight)需要对 HADDR 的高 28bit 进行译码.

Slave MUX 的作用则是通过每个外设的选择信号, 对所有外设返回的读取数据(HRDATA) 、响应信号(HRESP)以及反馈信号(HREADYOUT)进行选择, 保证返回给 Master 端口的数据来自于当前总线操作的目标外设. 例如当前总线操作是读取RAMCODE 的数据, 那么所有外设的选择信号中只有 RAMCODE 对应的选择信号为 '1' , 其他所有选择信号为 '0' , 那么Slave MUX 则根据这些选择信号选中 RAMCODE 返回的数据, 保证处理器核能够正确读取.

根据以上分析, 本实验已经编写好如下图所示的总线扩展模块.

<div align ="center"><img src="/img/lab3/08.png" alt="01" style="zoom:100%;" />

<center style="color:#000000;font-size:10pt;">总线扩展模块结构图</center>

在接下来的讲解中, 每增加一个外设, 只需要将外设 Slave 接口与 Peripheral Side AHBlite Master 接口相连接, 再在 Dcoder 模块中添加对应的译码模块生成选择信号即可.

## 修改硬件代码添加数据存储器

参考实验二添加 RAMCODE 步骤. 在顶层模块中已经添加好 RAMDATA 总线接口以及对应的 Block RAM , 需要将RAMDATA 总线接口接入总线扩展模块中预留的 P1 接口.

第一步, 在 "AHBlite_Decoder.v" 中的 RAMDATA (Port 1) 端口参数将其使能.

```verilog
/*RAMDATA enable parameter*/
parameter Port1_en = 0,
/************************/
```

改为

```verilog
/*RAMDATA enable parameter*/
parameter Port1_en = 1,
/************************/
```

第二步, 根据存储系统章节所述的 memory map 推荐地址分配, 我们在地址 0x20000000 分配了一个 512KB 的RAMDATA.所以 RAMDATA 的总线编码为 0x20000000-0x2000ffff , 因为对于一次总线操作, 只要地址总线的高 16 位为 0x2000 , 则Decoder 认为这是一次对数据存储器的操作, 进而生成数据存储器总线选择信号. 在译码部分插入 RAMDATA 的译码器代码.

```verilog
//0x20000000-0x2000ffff
/*Insert RAMDATA decoder code there*/
assign P1_HSEL = 1’b0;
/***********************************/
```

改为

```verilog
//0x20000000-0x2000ffff
/*Insert RAMDATA decoder code there*/
assign P1_HSEL = (HADDR[31:16] == 16'h2000) ? Port1_en : 1'b0;
/***********************************/
```

第三步, 我们借助于 readmemh 函数, 将keil编译生成的机器码, 初始化到程序RAM中. 即在 "Block_RAM.v" 文件中修改初始化函数中的文件路径, 使得编译器能够找到 code.hex 文件.

至此, 我们就完成了 RAMDATA 部分的修改.
