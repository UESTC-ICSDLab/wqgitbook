# 搭建 Keil 工程

注意, 本实验没有实现数据存储器, 所以我们暂时不能使用 Load/Store 指令. 在本实验中, 我们仍然使用汇编语言实现一个简单的循环计数器.

首先, 按照在 LAB1 中的方式, 在 "/Task2/keil" 文件夹中创建名为 code 的 Keil 工程, <font color="red">由于涉及到实际上板调试, 这里的 Target 设置方式与之前有些不同:</font>

<center><img src="/img/lab2/02.png" alt="02" style="zoom:75%;" /></center><center style="color:#0";>仿真/调试设置</center> 

其中 "code.ini" 文件位于 "Task2/keil/code.ini", 直接将其添加到图中位置即可.

接着, 我们点击进入 Debugger 的 settings 窗口, 选择上方的 Flash Download. 设置方式如下:

<center><img src="/img/lab2/03.png" alt="03" style="zoom:80%;" /></center><center style="color:#0";>Flash 下载设置</center> 

点击 "OK" 后, 进入 Utilities 设置界面, 取消勾选 Update Target before Debugging.

<center><img src="/img/lab2/04.png" alt="04" style="zoom:83%;" /></center><center style="color:#0";>Utilities 设置</center> 

<!-- -->
> #### important::注意
> 1. 上述几步设置与上板调试有关, 与接下来的 modelsim 仿真无关, 我们之后再回来讨论这几步的作用.
> 2. 除上述对 Debug 和 Utilities 的设置, 不要忘了其他部分 (Device, Target, Output 等) 的设置哦, 这些部分的设置方法与 LAB1 "搭建 Keil 环境" 一节中相同.


将 "/Task2/keil/startup_CMSDK_CM0.s" 文件添加至工程中, 进行编译. 这里的启动文件中的程序与 LAB1 相同, 仍然是一个循环计数器.

注意得到的 "/Task2/keil/code.hex" 文件, 它代表了程序的可执行文件, 是系统的软件组成部分, 为了将它与我们的硬件结合起来, 实现整个系统的 modelsim 仿真, 我们需要修改 "/Task2/rtl/Block_RAM.v" 文件, 将 RAM 初始化文件路径设为上述 "code.hex" 的路径. <font color="red">注意这里必须使用绝对路径:</font>

```verilog
initial begin
    $readmemh("Your/absolute/path/to/code.hex",mem);
end
```
这样, 我们就在仿真的层面上, 将程序的软件代码 "下载" 到了硬件的 "内存" 中.


