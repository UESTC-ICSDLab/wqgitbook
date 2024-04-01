## 使用矩阵键盘4个按键控制流水灯模式

<!-- -->
> #### flag::使用矩阵键盘4个按键控制流水灯模式
> + 在这次实验中, 我们需要把 4 个按键分别分配到 Cortex-M0 的 IRQ0-IRQ3 四个中断上, 按键的电平变化就会触发 Cortex-M0 对应的中断. 然后处理器在中断服务程序中, 控制硬件流水灯以不同模式运行. 
> + 本实验实现如下图所示的 SoC.

<div align ="center"><img src="/img/lab4/03.png" alt="01" style="zoom:100%;" />

<center style="color:#000000;font-size:10pt;">本次实验的SoC</center>

### 中断的相关知识

中断是指计算机运行过程中, 出现某些意外情况需要主机干预时, 机器能自动停止正在运行的程序并转入处理新情况的程序, 处理完毕后又返回原被暂停的程序继续运行.

根据 ARMv6-M 架构参考手册以及 Cortex-M0 用户手册, CPU 中断处理过程如下:

- CPU 接收到中断信号(IRQ、NMI、Systick等等);
- 将 R0,R1,R2,R3,R12,LR,PC,xPSR 寄存器入栈, 如下图所示;
- 根据中断信号查找中断向量表(对应汇编启动代码中的__Vector段), 跳转至中断处理函数;
- 中断处理函数执行完成后, 利用链接寄存器返回, 寄存器出栈, PC 跳转.

<div align ="center"><img src="/img/lab4/01.png" alt="01" style="zoom:100%;" />

<center style="color:#000000;font-size:10pt;">寄存器入栈</center>

异常中断向量表如下图所示.

<div align ="center"><img src="/img/lab4/02.png" alt="01" style="zoom:100%;" />

<center style="color:#000000;font-size:10pt;">异常中断向量表</center>

<!-- -->
> #### note::换个角度看中断
> 中断的本质是处理器对外开放的实时受控接口. 可以设想一个没有中断的计算机体系: 得知某个时刻 CPU 和内存的全部数据状态, 就可以推衍出未来的全部过程. 这样的计算机无法交互, 只是个加速器. 添加中断后, 计算机指定了会兼容哪些外部命令, 并设定服务程序, 这种服务可能打断当前任务. 这使得 CPU "正在执行的程序" 与 "随时可能发生的服务" 二者形成了异步关系, 外界输入的引入使得计算机程序可以实现与外界的交互. 由人实时控制的中断输入, 是无法预测的. 再将中断响应规范化, 进而推广开, 普通人群也就可以控制计算机, 发挥每个人的创造力.

### 硬件模块代码

第一步, 将按键模块输出的信号接到 Cortex-M0 系统的 IRQ 信号上, 因此在 CortexM0_SoC.v 文件做如下修改.

```verilog
/*Connect the IRQ with keyboard*/
assign IRQ = 32'b0;
/***************************/
```

改为:

```verilog
/*Connect the IRQ with keyboard*/
assign IRQ = {28'b0,key_interrupt};
/***************************/
```

接下来，我们在 WaterLight 实验的 TD 约束文件的基础上添加按键端口的约束.

### 启动代码与 C 编程

我们需要根据 CMSIS 提供的启动代码重新完成自己的启动代码, 具体代码见 "/Task4/key_waterlight/keil/startup_CMSDK_CM0.s".

与之前的汇编代码不同的是, 我们在复位处理函数内调用了_main函数, 此函数的作用是将堆栈初始化后跳转至 C 语言中的 main 函数, 而最后一段   __user_initial_stackheap 则是初始化堆栈过程的一部分. 初始化堆栈的具体过程由编译器提供, 无需人为添加. 

在中断处理的地方可以看到, 当按键中断发生后, CPU 会根据 __Vector中的中断地址跳转到按键中断处理函数, 在这个函数里面, 首先人为地将寄存器入栈, 然后跳转至 C 语言中的 key 函数, 执行完成后寄存器出栈并返回.

先修改 __Vector 中断向量表如下:

```c
__Vectors       DCD     __initial_sp              ; Top of Stack
                DCD     Reset_Handler             ; Reset Handler
                DCD     0			              ; NMI Handler
                DCD     0				          ; Hard Fault Handler
                DCD     0                         ; Reserved
                DCD     0                         ; Reserved
                DCD     0                         ; Reserved
                DCD     0                         ; Reserved
                DCD     0                         ; Reserved
                DCD     0                         ; Reserved
                DCD     0                         ; Reserved
                DCD     0			              ; SVCall Handler
                DCD     0                         ; Reserved
                DCD     0                         ; Reserved
                DCD     0            			  ; PendSV Handler
                DCD     0          				  ; SysTick Handler
                DCD     KEY0_Handler              ; IRQ0 Handler
                DCD     KEY1_Handler              ; IRQ1 Handler
                DCD     KEY2_Handler              ; IRQ2 Handler
                DCD     KEY3_Handler              ; IRQ3 Handler
```

添加中断服务函数的入口, 如下:

```c
; add IRQ Handler function here

 ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
```

改为:

```c
KEY0_Handler    PROC
                  EXPORT KEY0_Handler            [WEAK]
			    IMPORT KEY0
				PUSH	{R0,R1,R2,LR}
                   BL		KEY0
				POP		{R0,R1,R2,PC}
                   ENDP

KEY1_Handler    PROC
                  EXPORT KEY1_Handler            [WEAK]
			    IMPORT KEY1
				PUSH	{R0,R1,R2,LR}
                   BL		KEY1
				POP		{R0,R1,R2,PC}
                   ENDP

KEY2_Handler    PROC
                   EXPORT KEY2_Handler            [WEAK]
				IMPORT KEY2
				PUSH	{R0,R1,R2,LR}
                   BL		KEY2
				POP		{R0,R1,R2,PC}
                   ENDP

KEY3_Handler    PROC
                   EXPORT KEY3_Handler            [WEAK]
				IMPORT KEY3
				PUSH	{R0,R1,R2,LR}
                   BL		KEY3
				POP		{R0,R1,R2,PC}
                   ENDP
```

然后, 我们需要定义外设的地址, 以及自己实现的函数, 参考 CMSIS 编写自己头文件. 具体代码见 “/Task4/key_waterlight//keil/code_def.h”.

```cpp
#include <stdint.h>
//INTERRUPT DEF
#define NVIC_CTRL_ADDR (*(volatile unsigned *)0xe000e100)
//WATERLIGHT DEF
typedef struct{
    volatile uint32_t WaterLight_MODE;
    volatile uint32_t WaterLight_SPEED; 
}WaterLightType;
#define WaterLight_BASE 0x40000000
#define WaterLight ((WaterLightType *)WaterLight_BASE)

void SetWaterLightMode(int mode);
void SetWaterLightSpeed(int speed);
```

第一行 <stdint.h> 头文件提供了结构体以及结构体运算符 "->" 的支持, 高效地利用结构体定义外设地址, 能够有效地减少代码量, 节约存储空间.

下面以 WaterLight 为例讲解结构体与基地址的使用. 首先我们根据之前 WaterLight 硬件部分设计, WaterLight 在地址空间能有两个寄存器, 分别为 Waterlight_MODE、Waterlight_SPEED, 它们的地址分别为 0x40000000、0x40000004. 两个寄存器在内存空间中是连续的两个字 (word), 因此在结构体中定义两个寄存器时需要按照它们地址的顺序依次定义, 并且类型为 32bit 的 uint32_t. 之后再定义 WaterLight 的基地址为 0x40000000. 这样一来, 当我们使用结构体中第一个元素时, 它的地址则为基地址 +0; 第二个地址为基地址 +4; 第三个地址为基地址 +8 依次类推. 完全符合我们在硬件时定义的地址.

然后, 我们需要完成函数的实现, 具体见 "/Task4/key_waterlight/keil/code_def.c".

```c
#include "code_def.h"
#include <string.h>

void SetWaterLightMode(int mode)
{
	WaterLight -> Waterlight_MODE = mode;
}

void SetWaterLightSpeed(int speed)
{
	WaterLight -> Waterlight_SPEED = speed;
}
```

在有了这些函数以后, 我们根据 startup_CMSDK_CM0.s 启动文件中所写的按键中断服务函数的名称, 在 keyboard.c 中补充完整对应的中断服务函数. 按键中断服务函数中, 每个按键都对应一种流水灯的模式.

```cpp
#include <stdint.h>
#include "code_def.h"

void KEY0(void)
{
}

void KEY1(void)
{
}

void KEY2(void)
{
}

void KEY3(void)
{
}
```

改为:

```cpp
#include <stdint.h>
#include "code_def.h"
void KEY0(void)
{
	SetWaterLightMode(0);
}

void KEY1(void)
{
	SetWaterLightMode(1);
}

void KEY2(void)
{
	SetWaterLightMode(2);
}

void KEY3(void)
{
	SetWaterLightMode(3);
}
```

最后, 编写主文件, 具体在 "/Task4/key_waterlight/keil/main.c". 需要注意的是, 在中断开启之前我们需要使能所用到的中断.

```c
#include "code_def.h"
#include <string.h>
#include <stdint.h>

#define WaterLight_SPEED_VALUE 0x00c9d2ff

int main()
{ 
	//interrupt initial
	NVIC_CTRL_ADDR = 0xf;
	
	//WATERLIGHT
	SetWaterLightSpeed(WaterLight_SPEED_VALUE);
	while(1){	
	}
}
```

<!-- -->
> #### hint::C语言中, 头文件  和源文件的关系
> + 在实际的项目中, 可能有成千上万个源文件,并且会根据不同的功能模块划分不同的源文件, 那么随之而来的问题是, 不同文件之间如何共享函数和全局变量? 这里就有了头文件和源文件的划分了.
> + 我们将需要共享的函数、外部变量和一些宏定义声明在头文件中, 头文件一般以h结尾. 将定义 (其实就是实现) 写在源文件中, 源文件一般以c结尾.
> + 当不同源文件之间需要共享函数或者变量, 可以通过 #include 包含指令包含头文件即可 (当然, 头文件中也可以包含其他的头文件, 即头文件的嵌套).

<!-- -->
> #### fread::编译器的工作过程
> + (1) 预处理阶段
> + (2) 词法与语法分析阶段
> + (3) 编译阶段, 首先编译成纯汇编语句, 再将之汇编成跟CPU相关的二进制码, 生成各个目标文件    (.obj文件)
> + (4) 连接阶段, 将各个目标文件中的各段代码进行绝对地址定位, 生成跟特定平台相关的可执行文件, 当然, 最后还可以用objcopy生成纯二进制码, 也就是去掉了文件格式信息.
> + 编译器在编译时是以 C 文件为单位进行的, 也就是说如果你的项目中一个 C 文件都没有, 那么你的项目将无法编译, 连接器是以目标文件为单位, 它将一个或多个目标文件进行函数与变量的重定位, 生成最终的可执行文件. 而这些 C 文件中又需要一个 main 函数作为可执行程序的入口.

### 调试与运行

打开 Keil 工程将编写好的文件添加至工程中, 并在如下图所示的设置中取消勾选 "Don’t Search Standard Libraries", 然后编译.

<div align ="center"><img src="/img/lab4/06.png" alt="01" style="zoom:100%;" />

<center style="color:#000000;font-size:10pt;">取消勾选</center>

在软件编译通过之后, 我们使用 modelsim 进行仿真, 我们在 testbench 文件中添加了一个按键信号用于触发处理器的中断, 在触发中断后, 我们能够观察处理器内部的变化. 在 object 界面, 我们选择添加 clk,col 按键输入端口, 以及处理器内核的 IPSR 中断程序状态寄存器和 PC 寄存器 vis_ipsr_o. 可以看到在按键输入保持一段时候有效后, IPSR 的值就会变为 16, 根据第三章所述, IPSR 为记录异常标号的寄存器. 此时 IPSR 为 16, 查找中断向量表可知, 此时处理器接收到了 IRQ0 中断.

在 modelsim 仿真通过之后, 我们将相关的文件添加到 TD 工程中, 最后将 TD 生成的 bit 流文件下载到 FPGA 开发板上. 我们就能够通过按键控制硬件流水灯的模式.

至此, 我们使用矩阵键盘中的 4 个按键, 通过中断响应的方法实现了对流水灯外设的控制. 然而, 对于一些复杂的应用场景, 可能需要用到更多按键, 如果仍然采用这种一个按键对应一个中断的方式将带来很大的开销 (比如占用过多 CPU 中断端口导致添加新功能时可用中断端口不足). 因此, 本实验介绍另外一种方法, 将矩阵键盘全部按键对应 CPU 上一个中断端口, CPU 在中断响应时通过总线读取按键信息, 进行相应的操作.