# Keil

Keil MDK 是用于一系列基于 Arm Cortex-M 的微控制器设备的一个完整的软件开发环境. Keil 软件的主要作用是将 C 语言 / 汇编语言写的程序编译成机器码, 在进入调试模式时, 通过调试器下载机器码到 RAM 中, CPU 启动后, 开始一条条地取指并执行. 

<div align ="center"><img src="/img/lab0/08.png" alt="keil界面" style="zoom:120%;" /><div align ="center">


<center style="color:#000000;font-size:10pt;">keil 界面</center>

Keil 的界面如上图所示, 与 TD Vivado 软件类似, 主界面也是由工具栏、工程目录、代码编辑、调试信息组成. Keil 窗口中包含菜单栏、快捷键、寄存器界面、内存界面等. 菜单栏中依次为 File、Edit、View、Project、Debug、Peripherals、Tools、SVCS、Window 和 Help. 通过点击菜单栏中的选项可以实现代码调试所需的全部功能. 在快捷键一栏则对应有新建文件、打开文件、复制和粘贴、调试功能、编译代码等一系列常用菜单的快捷键按钮, 方便开发人员使用. 在工程目录一栏可以看到当前工程名及工程中包含的源文件、头文件等各种程序文件. 双击文件名可以在右侧打开窗口查看并编辑该文件的内容. 

<div align ="center"><img src="/img/lab0/09.png" alt="keil调试界面" style="zoom:120%;" /><div align ="center">


<center style="color:#000000;font-size:10pt;">keil 调试界面</center>

调试界面可以看到的信息更多, 如上图. 从主窗口中可以查看当前正在调试的源代码, 寄存器界面可以查看执行当前指令时 ARM 处理器中各个寄存器的值, 而右下角的存储器界面则可以查看当前存储器中不同地址的存储单元的值. 通过观察寄存器和存储单元中的值, 软件开发人员能够判断当前代码的执行过程是否正确, 从而通过修改代码完成对软件的调试与纠错. 



> #### note::有关keil的具体使用
>
> 在 LAB1 的[搭建 Keil 环境](../lab1/sec_2.md)中, 有关于 keil 在本实验中的具体用法.
>



