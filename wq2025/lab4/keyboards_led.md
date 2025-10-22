## 使用矩阵键盘控制数码管显示

<!-- -->
> #### flag::使用矩阵键盘控制数码管显示
> 在这次实验中, 我们需要把矩阵键盘的全部 16 个按键连接到 IRQ0 端口上, 当任何一个按键被按下时, CPU 进入中断处理程序, 读取按键信息, 根据按下的按键编号在数码管上显示对应的数值.

### 外设简介

数码管显示的具体代码见工程文件夹下 "Segdisp.v、seg_led_decoder.v、seg_sel_decoder.v" 文件.

由于所有按键连接到同一个中断端口, 因此键盘外设除了基本的消抖检测模块以外, 还需要使用一个寄存器组将被按下的键盘的序号记录下来. 本节用到了矩阵键盘的全部按键, 因此不能像上一个实验中为了使用某一行的 4 个按键, 将该行的行信号线拉低. 为了实现对矩阵键盘全部 16 个按键的检测, 需要进行扫描. 

由于 FPGA 板载时钟频率为 50MHz, 所以还需要先对时钟信号分频得到扫描时钟信号, 按扫描时钟信号周期性地改变行信号线的值. 当第一行行信号线为低电平时, 此时检测列信号的值便可判断出是第一行的某个按键被按下. 以此类推, 使用这种扫描方法可以检测矩阵键盘 16 个按键中是哪一个按键被按下. 除了扫描模块以外, 键盘模块还包括消抖模块和记录按键信息的寄存器组. 

最后实现的键盘外设模块中, 把每个按键的脉冲信号 key_pulse 通过或运算以后, 得到键盘中断信号key_interrupt. 只要有一个按键被按下, 中断信号都将产生一个脉冲, 使CPU进入中断处理程序. 

<!-- -->
> #### hint::提示
> 可以看出, 本节介绍的方法中中断信号和上一节中的有很大不同, 应当认真体会!

### SoC硬件部分

上一个实验中按键模块的中断信号为 4 位 key_interrupt 信号, 本实验中该信号为 1 位, 因此在 CortexM0_SoC.v 文件做如下修改.

```verilog
wire [31:0] IRQ;
wire [3:0] key_interrupt;
/*Connect the IRQ with keyboard*/
assign IRQ = {28'b0,key_interrupt};
/***************************/
```

改为:

```verilog
wire [31:0] IRQ;
wire key_interrupt;
/*Connect the IRQ with keyboard*/
assign IRQ = {31'd0,key_interrupt};
/***************************/
```

然后在上一个实验的 TD 约束文件的基础上添加对数码管的约束.

### 启动代码与 C 编程

由于中断信号的变化, 我们需要对启动代码进行修改. 上一个实验中已经介绍过 _main 函数和 __user_initial_stackheap 函数, 在本实验中我们只需要修改中断向量表与中断服务函数即可.

修改__Vector中断向量表如下：

```cpp
__Vectors     DCD     __initial_sp              ; Top of Stack
                DCD     Reset_Handler             ; Reset Handler
                DCD     0			               ; NMI Handler
                DCD     0				          ; Hard Fault Handler
                DCD     0                          ; Reserved
                DCD     0                          ; Reserved
                DCD     0                          ; Reserved
                DCD     0                          ; Reserved
                DCD     0                          ; Reserved
                DCD     0                          ; Reserved
                DCD     0                          ; Reserved
                DCD     0			               ; SVCall Handler
                DCD     0                          ; Reserved
                DCD     0                          ; Reserved
                DCD     0            			; PendSV Handler
                DCD     0          				; SysTick Handler
                DCD     Keyboard_Handler        ; IRQ0 Handler
```

现在只有一个中断服务函数, 因此还需要修改中断服务函数的入口:

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

改为:

```
Keyboard_Handler    PROC
                    EXPORT Keyboard_Handler            [WEAK]
			       IMPORT KEY_ISR
			       PUSH	{R0,R1,R2,LR}
                    BL		KEY_ISR
			       POP	{R0,R1,R2,PC}
                    ENDP
```

然后, 我们需要定义外设的地址, 以及自己实现的函数, 参考 CMSIS 编写自己头文件. 具体代码见 "/Task4/key_led/keil/code_def.h".

```cpp
#include <stdint.h>

//INTERRUPT DEF
#define NVIC_CTRL_ADDR (*(volatile unsigned *)0xe000e100)

//KEYBOARD DEF
#define Keyboard_keydata_clear (*(volatile unsigned *)0x40000000)

//SEGDISP DEF
#define Segdisp_data (*(volatile unsigned *)0x40000010)

void KEY_ISR(void);
```

其中 Keyboard_keydata_clear 连到键盘模块中的 clear 端口, 用于按键信息寄存器组的清零. Segdisp_data 为数码管上需要显示的数据. 中断处理函数 KEY_ISR 将变量 key_flag 的值改为 1, 用于 C 语言程序中判断是否有按键被按下产生了中断, 具体见 "/Task4/key_led/keil/keyboard.c".

```c
void KEY_ISR(void)
{
	key_flag = 1;
}
```

最后, 编写主函数文件, 具体见 "/Task4/key_led/keil/main.c", 需要注意的是, 在中断启动之前, 我们需要使能所用到的中断.

```cpp
#include "code_def.h"
#include <string.h>
#include <stdint.h>

extern uint32_t key_flag;

int main()
{   
	NVIC_CTRL_ADDR	= 1;
	
    while(1){
		  while(!key_flag);
		    uint32_t din;
		    din = Keyboard_keydata_clear;
		    int i = 0;
		    int ans = 0;
		    for (i = 0; i < 16; i++) {
			    if ((din >> i) & 1) {
			     	ans = i;
				    Segdisp_data = 16 + ans; //enable
				    break;
				}
			}
		key_flag = 0;
		Keyboard_keydata_clear = 1;
	}
}
```

当任一按键被按下后, 键盘模块会产生中断信号, CPU 进入中断处理程序, key_flag 的值变为 1. 此时, main 函数中内层 while 循环被跳过, 开始读取键盘的值. 键盘寄存器组的值被读到变量 din 中, 接着用一个 for 循环找出哪一位为 1, 记录在变量 ans 中, 同时向数码管外设写值 16+ans, 跳出循环. 因为数码管模块中 5 位输入数据中最高位是使能信号 (表示数据输入是否有效), 因此在写入低四位的数据基础上, 还需要加上 16, 让使能信号有效. 最后就是在每次循环处理完成之后将 key_flag 等变量以及键盘外设的寄存器组清零.

### 调试与运行结果

将相关的 Verilog 文件添加到 TD 工程中, 将 TD 生成的比特流 (bitstream) 文件下载到 FPGA 开发板上, 使用 Keil 调试. 在按下按键时, 可以看到数码管上显示了按键编号 (最左边的数码管), 之前的显示结果右移一位.