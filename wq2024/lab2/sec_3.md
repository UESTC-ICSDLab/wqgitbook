# Modelsim 仿真

打开 Modelsim 软件, 点击左上角 File 菜单, 再点击 New -> Project 创建一个 Modelsim 工程, 命名为 "code", 工程地址选为 "/Task2/modelsim/" 文件夹.

<center><img src="/img/lab2/05.png" alt="05" style="zoom:75%;" /></center><center style="color:#0";>新建 Modelsim 工程</center> 

点击 OK 后, 选择 Add Existing File, 添加源文件.

<center><img src="/img/lab2/16.png" alt="16" style="zoom:75%;" /></center><center style="color:#0";>添加源文件</center> 

由于我们的软件程序已经被 "下载" 到了 RAM 中, 这里只需要把 "/Task2/rtl/" 下的所有的 RTL 文件添加进来, 另外, 还需要把 Testbench 文件 "/Task2/modelsim/CortexM0_SoC_vlg_tst.v" 中的添加进来.

<!-- -->
> #### important::什么是 Testbench ?
> Testbench 是数字系统仿真的测试用例, 以 verilog module 的形式存在, Testbench 用于测试激励的产生和响应信号的收集处理, 虽然不属于设计的组成部分, 但必须是仿真时的顶层文件.

<!-- -->
> #### fread::关于本实验中的 Testbench
> 打开 "/Task2/modelsim/CortexM0_SoC_vlg_tst.v" 可以看到:
> ```verilog
> `timescale 1 ps/ 1 ps
> module CortexM0_SoC_vlg_tst();
> 
> reg clk;
> reg RSTn;
> reg TXD;
>                         
> CortexM0_SoC i1 (
>     .clk(clk),
>     .RSTn(RSTn)
> );
> 
> initial begin                                                  
>     clk = 0; RSTn=0;
>     #100 RSTn=1;
> end  
>     
> always begin #10 clk = ~clk; end       
> 
> endmodule
> ```
> 可见, Testbench 中只是对整个 SoC 做了简单的例化, 并提供了时钟信号和初始复位.

文件添加完毕后, 点击编译按钮, 对所有源文件进行编译查错. 通过编译的文件后的 "?" 都变成了 "√":

<center><img src="/img/lab2/07.png" alt="07" style="zoom:65%;" /></center><center style="color:#0";>编译之前</center> 

<center><img src="/img/lab2/08.png" alt="08" style="zoom:69%;" /></center><center style="color:#0";>编译通过</center> 

正如之前所说, Testbench 是仿真时的顶层文件, 所以仿真的对象应该是 Testbench 文件. 我们选择左侧导航栏下方的 Library, 展开 work, 右键单击 CortexM0_SoC_vlg_tst, 选择 Simulate without Optimization, 开始仿真.

<center><img src="/img/lab2/06.png" alt="06" style="zoom:75%;" /></center><center style="color:#0";>选择 Library -> work -> CortexM0_SoC_vlg_tst</center> 

<center><img src="/img/lab2/09.png" alt="09" style="zoom:75%;" /></center><center style="color:#0";>右键 CortexM0_SoC_vlg_tst 选择 Simulate without Optimization</center> 

<!-- -->
> #### important::Instance, Object 和 Wave
> 仿真时三个非常重要的窗口:
> + Instance: 以 Testbench 为顶层模块, 层次化地展示所有被例化的模块. 注意, 展示的是实例名而不是模块名.
> + Object: 可以理解为展示了某个模块中的所有信号.
> + Wave: 波形窗口, 展示被添加的信号及其波形.
>
> 在 Modelsim 中, 如果不小心关闭了这些窗口, 可以在 View 菜单栏中重新打开.

下一步, 我们要把待观察的信号添加到 Wave 窗口. 那么我们应该添加哪些信号呢? 为了观察程序的运行情况, 我们需要观察处理器中寄存器的值, 而 Cortex-M0 中的调试接口恰好有捕捉寄存器值的功能, 所以我们只需要把 Cortex-M0 中的对应信号添加进来即可.

注意, 在 CortexM0_SoC_vlg_tst.v 中, SoC 的实例名为 i1, 而在 SoC 中, Cortex-M0 核的实例名为 u_logic. 我们在 Instance 窗口中展开 CortexM0_SoC_vlg_tst, 依次选择 i1 -> u_logic. 

双击 u_logic, 该模块中的所有信号就呈现在了 Object 窗口中. 在 Object 窗口中选中调试信号 vis_r0_o, vis_r1_o, vis_pc_o; 另外, 为了观察 Cortex-M0 数据接口的工作过程, 我们继续选中 HADDR, HSIZE, HTRANS, HRDATA 这几个 AHB 信号, 右键点击 Add Wave, 这些信号就被添加到了 Wave 窗口中.

<center><img src="/img/lab2/11.png" alt="11" style="zoom:80%;" /></center><center style="color:#0";>选择待观察的信号</center> 

在 Wave 窗口中选中所有信号, 右键点击 Radix -> Hexadecimal, 将其改为 16 进制显示, 以便观察.

<center><img src="/img/lab2/12.png" alt="12" style="zoom:100%;" /></center><center style="color:#0";>Wave 窗口中的信号</center> 

<!-- -->
> #### hint::保存波形格式
> 在 Modelsim 中, 所有对波形格式 (如显示格式, 颜色, 信号分组) 的修改可以被保存为一个 do 脚本文件, 以便下次进入仿真后加载执行. 
>
> 保存方式为: 点击 Wave 窗口左上角 File 菜单, 选择 Save Format:
> <center><img src="/img/lab2/13.png" alt="13" style="zoom:70%;" /></center><center style="color:#0";>选择波形格式文件保存路径</center> 
> 加载方式为: 点击 Wave 窗口左上角 File 菜单, 选择 Load -> Macro File.

在运行仿真前, 首先要设置仿真时间, 这里设置仿真时间为 100ns, 然后点击旁边按钮正式运行仿真.

<center><img src="/img/lab2/14.png" alt="14" style="zoom:70%;" /></center><center style="color:#0";>设置仿真时间并运行仿真</center> 

仿真运行停止后, 点击 Wave 窗口菜单栏中的<img src="/img/lab2/17.png" alt="17" style="zoom:100%;" />将波形缩放到满屏, 然后通过滚动条和 Ctrl + 鼠标滚轮将波形缩放到合适比例.

<center><img src="/img/lab2/15.png" alt="15" style="zoom:100%;" /></center><center style="color:#0";>仿真波形</center> 

下面根据程序功能, 分析仿真波形:

+ 寄存器

vis_r1_o 在 0-4 范围内循环计数, 与程序功能相符.

vis_pc_o 在 0x25-0x29 之间循环, 为什么? 这时我们就需要进入之前生成的 "/Task2/keil/code.txt" 文件一探究竟了, 在 "code.txt" 的 .text 中有这样一段:

```
    start
        0x00000048:    2104        .!      MOVS     r1,#4
        0x0000004a:    2000        .       MOVS     r0,#0
        0x0000004c:    1c40        @.      ADDS     r0,r0,#1
        0x0000004e:    4288        .B      CMP      r0,r1
        0x00000050:    d0fb        ..      BEQ      0x4a ; start + 2
        0x00000052:    d1fb        ..      BNE      0x4c ; start + 4
```

这表明, 程序运行时, 取指地址在 0x0000004a-0x00000052 之间循环, 不难发现, vis_pc_o 恰好是取指地址去除 LSB 后的结果.

+ AHB 信号

Cortex-M0 采用的是冯诺依曼架构, 指令和数据都要通过同一个 AHB 接口访问. 而本实验中未涉及对数据的读写, 所以这里的 AHB 接口仅用于指令的读取. 另外, 由于 Cortex-M0 内部未设有 Cache 来缓存指令和数据, 所以对于循环程序而言, 也需要不断去主存中读取同一段指令. 所以我们会看到 HADDR 不断在 0x0000004a-0x00000052 之间循环, 对应的 HTRANS 在传输时变为 1, HSIZE 在传输时变为 2, 表示 32 位数据传输, 相应的指令数据在下一拍表现在 HRDATA 总线上.

若要退出仿真, 点击主窗口菜单中的 Simulate -> End Simulation.

<center><img src="/img/lab2/18.png" alt="18" style="zoom:80%;" /></center><center style="color:#0";>退出仿真</center> 

<!-- -->
> #### hint::命令行操作 - 从小白到工程师
> 在工程中, 使用命令行或脚本操作软件可以为我们节省大量的操作时间, 提升我们的工作效率. 熟练的命令行技能是一个优秀工程师必备的专业素质.
>
> 如果你厌倦了繁琐的图形界面操作, 请跟我一起学习使用命令行操作 Modelsim 吧.
> 
> Modelsim 采用 TCL 命令, 你可以在命令行窗口 (Transcript) 中执行单条命令, 也可以使用 do 脚本来执行多条命令. 对于本仿真实验, 这里给出了一种等效的 do 脚本的编写方法, 有兴趣的同学可以自行尝试<font color="red">(注意你的文件路径)</font>:
>
> 编写 "run.do" 脚本文件:
> ```tcl
> vlib work 
> vlog "./CortexM0_SoC_vlg_tst.v" 
> vlog "./rtl/*"
> vsim -voptargs=+acc -c work.CortexM0_SoC_vlg_tst
> add wave "CortexM0_SoC_vlg_tst/i1/u_logic/*"
> run 100ns
> ```
>
> 在 Modelsim Transcript 窗口中执行命令:
> ```tcl
> do run.do
> ```
