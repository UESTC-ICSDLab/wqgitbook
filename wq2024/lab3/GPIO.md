# GPIO外设实现流水灯

<div align ="center"><img src="/img/lab3/09.png" alt="01" style="zoom:100%;" />
<center style="color:#000000;font-size:10pt;">本章实现的SoC</center>

<!-- -->
> #### flag::GPIO外设实现流水灯
> 接下来, 将介绍如何将 GPIO 集成到 SoC 系统中, 并且使用软件编程控制它实现流水灯的功能.

## GPIO 介绍

通用输入/输出接口(General-purpose input/output, GPIO), 是一种常用于处理器系统的外设接口, 其特点是具有输出/输入逻辑电平的功能.

通俗地说, 就是一些引脚, 可以通过它们输出高低电平或者通过它们读入引脚的状态是高电平或是低电平. GPIO 是个比较重要的概念, 用户可以通过 GPIO 口和硬件进行数据交互 (如UART), 控制硬件工作 (如LED、蜂鸣器等), 读取硬件的工作状态信号 (如中断信号) 等. GPIO 口的使用非常广泛. 掌握了 GPIO, 差不多相当于掌握了操作硬件的能力.

设计 GPIO 外设的目的就是为了提高 SoC 系统的通用性. 以单片机这一 SoC 系统为例, 单片机作为工业设备的控制核心, 往往需要驱动不同的芯片, 这些芯片的通信协议各不相同, SoC 设计人员不可能把每一种芯片的通信协议都制作成硬件的形式集成在 SoC 中. 因此, 设计 GPIO 外设, 让编程人员通过处理器控制 GPIO 接口模拟各个驱动芯片的通信时序以达到驱动各个外部芯片的功能. GPIO 接口能够避免没有特殊芯片的通信时序的尴尬.

如果熟悉单片机的同学, 就会知道, 几乎所有的单片机都具有 GPIO 接口. 在一些单片机中, GPIO 接口还可以配置连接到处理器的 IRQ 中断上, 允许 SoC 系统外的电平信号触发中断.在市场上商用的单片机中, GPIO 往往具有几种不同输入/输出模式, 以适应不同的应用场景. 这几不同的模式, 大致可以分为: 浮空输入、上拉输入、下拉输入、模拟输入、开漏输出、推挽输出和一些复用功能的输入/输出模式. 下面分别介绍一下它们的用途:

- 浮空输入: 可以理解为 GPIO 端口既没有上拉电阻也没有下拉电阻, 在无信号输入的情况输入状态的电平是不确定的.
- 上拉输入: 可以理解为 GPIO 端口拥有一个上拉电阻接到 VDD , 在无信号输入的情况下输入状态是高电平.
- 下拉输入: 可以理解为 GPIO 端口拥有一个下拉电阻接到 GND , 在无信号输入的情况下输入状态是低电平.
- 模拟输入: 就是信号输入的过程中会经过放大器、缓冲器等模拟电路, 通常用于内部集成的 ADC 输入端口.
- 开漏输出: 可以理解为三极管集极输出电路, 要输出高电平需要外接上拉电阻.
- 推挽输出: 输出端两个互补 MOS 管构成推挽结构输出, 其特点是既可以输入电流也可以输出电流.

要实现这些不同状态的切换, 需要在设计芯片的时候设计了不同的模拟电路进行配合. 在做数字设计时, 我们不能实现上述的那些模拟电路, 能够实现的是一些选择不同输出模式开关控制信号和如下图所示的 GPIO 模块.

<div align ="center"><img src="/img/lab3/10.png" alt="01" style="zoom:100%;" />
<center style="color:#000000;font-size:10pt;">GPIO模块</center>

本次实验实现的 GPIO 外设就如上图所示. 说到这里, 我们先介绍处理器核心是如何让这些外设按照想要的方式工作. 这里就涉及到了**控制字**的概念, 在之前的内容中, 我们介绍了处理器通过总线给固定的地址写入一个字长数据, 这个用于控制总线外部硬件电路的数据就被称为控制字. 实验三设计的 GPIO 模块就包含了 3 个控制字, 一个是 oData 用于控制 GPIO 的引脚输出的电平, 另一个是 iData 用于表明 GPIO 引脚输入的电平, 最后一个是 outEn 用于控制 GPIO 的输入输出状态.

## 硬件代码

文件 "CortexM0_SoC/Task3/GPIO/rtl/GPIO.v" 就是上图所示的 8 个 GPIO 接口, 无需自行设计.

同理, 向 SoC 中添加 GPIO , 参考实验二添加 RAMCODE 步骤. 在顶层模块中已经添加好 GPIO 总线接口以及对应的硬件代码, 需要完成将流水灯总线接口接入总线扩展模块中预留的 P4 接口.

第一步, 在 "AHBlite_Decoder.v" 中 GPIO(Port 4) 端口参数将其使能.

```verilog
/*GPIO enable parameter*/
parameter Port4_en = 0,
/************************/
```

改为

```verilog
/* GPIO enable parameter*/
parameter Port4_en = 1,
/************************/
```

第二步, 本次实验给 GPIO 的三个控制字分配地址空间如下: 输出数据寄存器 oData 的地址为0x40000020, 输入数据寄存器 iData 的地址为 0x40000024 , 输出使能寄存器 outEn 的地址为 0x40000028 . 因为对于一次总线操作, 只要地址总线的高 28 位为 0x4000002 , 则 Decoder 认为这是一次对 GPIO 的操作, 进而生成 GPIO 总线选择信号. 在译码部分插入 GPIO 的译码器代码.

```verilog
//0x40000028 OUT  ENABLE
//0X40000024 IN DATA
//0X40000020 OUT DATA
/*Insert GPIO decoder code there*/
assign P4_HSEL = 1’b0;
/***********************************/
```

改为

```verilog
//0x40000028 OUT  ENABLE
//0X40000024 IN DATA
//0X40000020 OUT DATA
/*Insert GPIO decoder code there*/
assign P4_HSEL = (HADDR[31:4] == 28'h4000002) ? Port4_en : 1'd0;
/***********************************/
```

接下来, 需要在顶层模块中将 GPIO 总线接口与数据存储器总线接口分别与总线扩展模块的 P1、P4 端口连接, GPIO 外设和数据存储器以及它们的总线接口都已经在顶层模块 "CortexM0_SoC.v" 中例化完成, 只需要将其总线接口的连线部分补充完毕, 实现正确的端口连接.

## 汇编代码

按照实验一所述, 新建 keil 工程, 编写汇编程序, 让流水灯流起来.

在 "/CortexM0_SoC/Task3/GPIO/keil/startup_CMSDK_CM0.s" 文件中, 程序进入 GPIO 段后, R2 存储 GPIO 输出寄存器的地址, R3 存储 GPIO 输入寄存器的地址, R4 存储输入/输出模式控制寄存器的地址, R1 存储计数器时间.

如何用我们设计的 8 个 GPIO 端口实现流水灯功能, 其程序流程如下:

1. 配置 GPIO 为输出模式;
2. 往 oData 控制寄存器写入 0x01 点亮右边第一个灯并且延迟一定时间;
3. 将 0x01 左移一位(即0x02)写入 oData 寄存器中, 并且延迟一定时间;
4. 重复第 2 、第 3 步, 第 8 个灯点亮之后, 再次跳转到第 2 步.

需要注意的是, 为了方便仿真观察, 我们将延迟的间隔时间设置得非常小, 在接下来的上板调试时, 需要重新修改 R0 的值, 使流水灯模式转换时间保持在 1s 左右.

在汇编文件中补充代码段, 实现往左流的流水灯功能, 并且利用 R1 计数至 16 后返回实现delay功能.

```
; Finish function code 

;;;;;;;;;;;;;;;;;;;;;;

```

改为

```ARM
; Finish function code 
GPIO            LDR R6, =0x00         ;GPIO INPUT ENABLE VALUE
                STR R6, [R4]          ;Set input ENABLE
                LDR R5, [R3]          ;read GPIO value

                LDR R6, =0x01         ;GPIO OUTPUT ENABLE VALUE
                STR R6, [R4]          ;Set OUTPUT ENABLE
                
                LDR R6, =0X01         ;GPIO_0 Set value
                STR R6, [R2]          ;store
                MOVS R1, #1           ;Interval cnt initial
                BL delay
                LDR R6, =0X02         ;GPIO_1 Set value
                STR R6, [R2]          ;store
                MOVS R1, #1           ;Interval cnt initial
                BL delay
                LDR R6, =0X04         ;GPIO_2 Set value
                STR R6, [R2]           ;store
                MOVS R1, #1            ;Interval cnt initial
                BL delay
                LDR R6, =0X08          ;GPIO_3 Set value
                STR R6, [R2]           ;store
                MOVS R1, #1            ;Interval cnt initial
                BL delay
                LDR R6, =0X10          ;GPIO_4 Set value
                STR R6, [R2]           ;store
                MOVS R1, #1            ;Interval cnt initial
                BL delay
                LDR R6, =0X20          ;GPIO_5 Set value
                STR R6, [R2]           ;store
                MOVS R1, #1            ;Interval cnt initial
                BL delay
                LDR R6, =0X40          ;GPIO_6 Set value
                STR R6, [R2]           ;store
                MOVS R1, #1            ;Interval cnt initial
                BL delay
                LDR R6, =0X80          ;GPIO_7 Set value
                STR R6, [R2]           ;store
                MOVS R1, #1            ;Interval cnt initial
                BL delay
                B GPIO

delay           ADDS R1,R1,#1
                LDR R0,=0X10
                CMP R0,R1
                BNE delay
                BX  LR
;;;;;;;;;;;;;;;;;;;;;;
```

从上面的示例代码中, 我们在代码的一开始还将 GPIO 配置成输入模式, 以检验 GPIO 的输入功能, 这一功能可以在 FPGA开发板中验证中得到检验.

## 系统仿真和调试

### ModelSim 仿真

按照实验一所述, 新建 modelsim 工程, 将之前编写的 verilog 文件添加进工程, 编译并开始仿真, 观察波形, 可以看到每个一段时间, GPIO 的输出都会左移一位, 将这些接口接到开发板的 LED 灯上, 就构成了左向流水灯.

### 硬件调试

由于 Keil 调试是进行的单步调试, 因此暂时不必对 R1 的值进行调整, 当前 R1 的值决定了每 30 步左右流水灯模式改变一次.

将新编写的 verilog 文件添加进 vivado 工程中, 将 GPIO[0:6] 分配到开发的 LED0-LED6 上, 同时为了验证输入功能将GPIO7 约束到开关 SW1 上, 综合布局布线后生成比特流文件并下载进 FPGA 中.

按照实验二讲述的方法连接调试器与 PC , 打开 Keil 进行调试, 调试结果如下图所示. 在进行调试之前, 将开发板的拨码开关 SW1 往上拨, 使其连接到电源 VDD 上. 在单步调试到下图所示的 38 行代码, 读取 GPIO 输入的值时, 我们可以看到读取到的值存储到 R5 寄存器中, 其值是 0x00000080, 表示 GPIO7 输入的值为 1 , 符合实验现象. 接下来继续单步调试就能看到 LED0 至 LED6 依次点亮.

<div align ="center"><img src="/img/lab3/15.png" alt="01" style="zoom:100%;" />