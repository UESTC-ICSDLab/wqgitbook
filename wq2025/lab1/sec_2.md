# 搭建 Keil 环境

Keil 是一款用于 ARM 架构处理器开发的 SDK 软件, 支持灵活丰富的配置和调试等功能. 在本课程中, 我们主要使用 Keil 完成对软件工程的基础编译, 仿真和调试.

## 创建 Keil 工程

打开 Keil 软件, 点击左上角 Project 菜单, 选择 new uVision project.

<img src="/img/lab1/02-01.png" alt="02-01" style="zoom:100%;" /><center style="color:#0";>Keil 主界面</center>  

选择在工程目录下 "/Task1/keil/" 文件夹下新建一个名为 code 的工程 (你也可以给工程取你喜欢的名字, 但是为了方便比对, 建议按照本书的要求创建和命名), 之后会弹出一个让你选择器件型号的窗口, 因为本课程的实验都是基于 Cortex-M0 这款处理器进行的, 所以这里选择 CMSDK_CM0.

<img src="/img/lab1/02-02.png" alt="02-02" style="zoom:100%;" /><center style="color:#0";>器件型号选择界面</center> 

<!-- -->
> #### hint::如何获取 Keil 的器件包?
> Keil 的器件包都可以从官方网站上直接下载.
> 如果你的弹框处没有 CMSDK_CM0 这个器件选项, 可以点击[这里](https://www.keil.com/dd2/arm/cmsdk_cm0/)进入官网下载, 下载后的文件直接打开即可选择对应目录安装.

## 添加文件

为新建的工程添加源代码文件, 右键 Source Group 1, 选择 "Add Existing Files to Group 'Source Group 1'", 将 "/Task1/keil/" 文件夹下的汇编文件 "startup_CMSDK_CM0.s" 添加进来.

<img src="/img/lab1/02-03.png" alt="02-03" style="zoom:100%;" /><center style="color:#0";>添加源文件</center> 

<!-- -->
> #### hint::Target 是什么? Source Group 又是什么?
> +  一个 Keil 工程可以包含多个 Target, 多个 Target 共享相同的源文件, 可以单独进行某些文件的屏蔽等操作. 设置多个 Target 有利于同一个工程中的代码版本分离. 同一时刻只能有一个 Target 处于激活状态, 这个 Target 就代表了当前的整个工程.
> + 将源文件分到多个 Source Group 中, 方便开发者根据需要对文件进行分组管理.
> + Target 和 Source Group 都可由开发者任意创建和命名.

## 配置存储器参数

右键 Target 1, 选择 "Options for Target 'Target 1'", 进入选项设置界面, 也可以通过点击 Keil 界面最后一行工具栏中的 "魔术棒" 工具进入该界面. 
首先配置 Target 栏. 设置一块起始地址为 0x00000000, 大小为 0x10000 字节的片上 ROM 地址空间, 并设置一块片上起始地址为 0x20000000, 大小为 0x10000 字节的 RAM 地址空间.

<center><img src="/img/lab1/02-04.png" alt="02-04" style="zoom:70%;" /></center><center style="color:#0";>Target 设置</center> 

<!-- -->
> #### question::为什么要设置 ROM 和 RAM?
> 这一步设置是为了匹配你的 SoC 硬件, 我们已经学过了 Cortex-M0 的存储映射方式, 知道 ARM 的存储器地址空间分为只读和可读可写两部分, 比如, 在 ARM 的经典产品 STM 系列 MCU 中, 前者以片上 NAND FLASH 的形式存在, 后者以片上 SRAM 的形式存在. 在我们将要实现的 SoC 中, 对这两部分的实现方式其实不用过于区分, 但是其功能本质确是需要严格划分的: 
> + 前者用来存放代码和常量, 对应于 Target 设置中的 片上 ROM, 在 Cortex-M0 存储体系中被映射至 0x00000000 - 0x1FFFFFFF. 
> + 后者用来存放程序变量, 对应于 Target 设置中的 片上 RAM, 在 Cortex-M0 存储体系中被映射至 0x20000000 - 0x3FFFFFFFF.  
>
> 这里的 Size 其实可以任意分配, 只要满足程序运行的需要.
> 这样设置以后, 当你在程序中定义一个只读数据块时, 其存储地址就会从 0x00000000 向上生长.

## 修改 Output 路径

每个 Keil 工程都自动包含一个名为 Objects 的文件夹, 默认用来存放输出文件, 包括每个源文件编译或汇编得到的可重定位目标文件和最终链接生成的可执行文件等. 这里为了方便后续的一些操作, 我们将输出文件目录修改为工程根目录.  
同样在 Options 界面中, 切换到 Output 栏的设置, 点击 "Select Folder for Objects", 将其修改为工程根目录.

<center><img src="/img/lab1/02-05.png" alt="02-05" style="zoom:96%;" /></center><center style="color:#0";>Output 设置</center> 

<center><img src="/img/lab1/02-06.png" alt="02-06" style="zoom:85%;" /></center><center style="color:#0";>将输出目录修改为工程根目录</center> 

## 可执行文件处理

一个 Target 经过编译链接最终会生成一个唯一的可执行文件, 在 Keil 的 ADS 编译器下, 这个可执行文件的格式为 .axf, 这个文件中就蕴含着所谓的程序的机器码信息.   
由于我们之后需要实现软硬件的联合仿真, 也就是必须要把这些机器码导入到硬件工程中的内存里, 在之后的实验中我们将会知道, 硬件工程中的内存其实是一个 Verilog 编写的 RAM, 当使用 Modelsim 对硬件工程进行仿真时, 将机器码作为初始化文件对该 RAM 进行初始化, 就相当于我们在仿真层面上把编写的汇编程序 "下载" 进了硬件内存里. 由于 Modelsim 要求使用 hex 文件进行 RAM 的初始化, 所以这里我们需要把 .axf 文件的机器码信息输出到一个 .hex 文件中.   
另外, 为了方便阅读分析 .axf 文件的内容, 我们再将其内容转换输出到一个新的 .txt 文件中.
通过在 Options 界面的 User 栏中添加两条 Keil 命令来完成上述操作:

```keil
fromelf -cvf .\code.axf --vhx --32x1 -o code.hex
fromelf -cvf .\code.axf -o code.txt
```

注意, 这里 .axf 文件是在工程根目录下的, 是因为我们上一步将输出目录修改到了根目录.

<center><img src="/img/lab1/02-07.png" alt="02-07" style="zoom:92%;" /></center><center style="color:#0";>在 User 栏添加命令</center> 

<!-- -->
> #### fread::关于这一步如果你还想了解更多
> 
> + .axf 文件格式非常复杂, 类似于 Linux 中的 ELF 文件格式, 包含了代码, 数据, 符号表和调试信息等等, 这里我们只对代码和数据部分感兴趣, 采用 keil 的命令 "fromelf" 可以实现对 .axf 文件的转换, 第一条命令中的参数 --vhx 指明将其转换到只包含代码(机器码)和数据(变量存储)的 hex 文件中.
> + Options 界面的 User 栏根据执行时间将预设置命令分为三类: 编译前, 生成 可执行文件 前和生成 可执行文件 后, 由于我们要对 可执行文件 进行处理, 所以只能将命令设置在生成 可执行文件 后执行.

## 链接设置

在 Options 界面中的 Linker 栏中勾选 "Use Memory Layout from Target Dialog" 以及 "Don’t Search Standard Librarie" 两个选项.

<center><img src="/img/lab1/02-08.png" alt="02-08" style="zoom:80%;" /></center><center style="color:#0";>Linker 设置</center> 


## 仿真/调试设置

打开 Options 界面中的 Debug 栏, 其中分为两大块设置: 仿真和调试, 调试工作需要在物理硬件和调试器的配合下进行, 考虑到我们接下来将要编写的程序没有涉及对专用外设的操作, 只涉及一些基础的内存访问和计算, 所以这里勾选 "Use Simulator", 指明通过软件仿真进行程序的测试.

<center><img src="/img/lab1/02-09.png" alt="02-09" style="zoom:70%;" /></center><center style="color:#0";>设置运行在软件仿真模式</center> 

## 其他设置

在 Options 界面的 Utilities 栏中取消勾选 "Update Target before Debugging".

<center><img src="/img/lab1/02-10.png" alt="02-10" style="zoom:100%;" /></center><center style="color:#0";>Utilities 设置</center> 


至此, 一个新的 Keil 环境就搭建好啦.