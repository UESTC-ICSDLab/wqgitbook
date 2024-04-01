# LAB2: "点石成金" - 实现你的首个 SoC

<!-- -->
> #### important::README
> 1. 在本实验中, 你将动手实现一个基于 CortexM0 的 SoC 基础软硬件系统, 然后通过 Modelsim 对系统功能进行仿真, 仿真通过后, 使用 Vivado 搭建系统并将其下载配置到 FPGA 板卡中, 然后使用 Keil 进行实际上板调试.
> 2. 本实验旨在加深大家对基于 CortexM0 的 SoC 的理解, 熟悉各个软件的基本使用方法, 所以本实验的软件部分和硬件部分都非常简单. 
> 3. 在进行本实验之前, 你应该已经理解了硬件设计的涵义, 掌握了四种 verilog 关键字语法: always, assign, if, case, 而且掌握了模块例化的基本方式. 另外, 你还应当对 AHB 总线协议有一定的了解.

<!-- -->
> #### fread::LAB2 文件结构说明
>|  文件夹 |  说明 |
| ---- | ---- |
| keil | 软件程序文件,包括启动文件 |
| modelsim | 与 modelsim 仿真有关的文件 |
| rtl  | 硬件设计源代码文件 |
| TD | TD 工程文件,如约束文件 |

<!-- -->
> #### question::为什么硬件设计文件要取名为 rtl?
> RTL 原指寄存器传输级, 但一般使用 RTL 表示基于 HDL 编写的硬件代码, 使之区分于 c 语言这样的软件代码.

<!-- -->
> #### hint::VSCode 插件推荐
> + verilog 插件推荐 SystemVerilog - Language Support
> + 另外推荐插件 Rainbow Highlighter, 变量高亮功能方便信号查找
