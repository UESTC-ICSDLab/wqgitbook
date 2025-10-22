# 下载比特流到 FPGA

上一小节中, 我们通过仿真验证了 SoC 的功能, 接下来我们将系统实现在 FPGA 上, 进行上板调试.

打开TD软件, 点击 Project--New Project, 创建一个新的工程，给工程命名为 "CortexM0_SoC", 工程文建立在 "/Task2/TD/" 文件夹下. 记得检查一下下方显示的工程路径是否合理,注意不能出现中文字体。选择 FPGA 芯片型号, 在Device Family 栏处选择 EG4, 然后在下方列表中选择 EG4S20BG256.

<center><img src="/img/lab2/pics/1.png" alt="20" style="zoom:60%;" /></center><center style="color:#0";>创建文件与型号选择</center> 

<center><img src="/img/lab2/pics/2.png" alt="20" style="zoom:60%;" /></center><center style="color:#0";>创建文件与型号选择</center> 

<center><img src="/img/lab2/pics/3.png" alt="20" style="zoom:60%;" /></center><center style="color:#0";>创建文件与型号选择</center> 

进入源文件添加界面, 因为我们的设计源文件都在 "/Task2/rtl" 下, 所以这里右键点击 Hierarchy, 选择ADD Sources 直接将 "/Task2/rtl" 目录添加进来. 点击OK即可添加成功.

<center><img src="/img/lab2/pics/4.png" alt="20" style="zoom:60%;" /></center><center style="color:#0";>添加源文件</center> 

<center><img src="/img/lab2/pics/5.png" alt="20" style="zoom:60%;" /></center><center style="color:#0";>添加源文件</center> 

右键constraint_1（active），选择 Add ADC File进入约束文件添加界面, 我们已经在 "/Task2/TD/" 中为你准备好了约束文件 "pin.xdc", 将这个文件添加进来即可.

<center><img src="/img/lab2/pics/6.png" alt="20" style="zoom:60%;" /></center><center style="color:#0";>添加约束文件</center> 

<center><img src="/img/lab2/pics/7.png" alt="20" style="zoom:60%;" /></center><center style="color:#0";>添加约束文件</center> 

<!-- -->

> #### question::什么是约束文件?
> + 本实验中的约束文件用于将设计的外部端口连接到 FPGA 的物理引脚上.
> + 除了引脚约束之外, 约束文件还可以用于时钟约束, 常用于静态时序分析 (STA).
> + 不同的开发工具采用的约束文件格式是不同的, TD 中的约束文件以 .adc 为后缀.

<!-- -->
> #### hint::本实验中的约束文件
>
> 打开 "/Task2/TD/pin.adc":
> ```
> set_pin_assignment	{ RSTn }	{ LOCATION = A9; IOSTANDARD = LVCMOS25; PULLTYPE = PULLUP; }
> set_pin_assignment	{ SWCLK }	{ LOCATION = K3; IOSTANDARD = LVCMOS25; PULLTYPE = PULLUP; }
> set_pin_assignment	{ SWDIO }	{ LOCATION = K6; IOSTANDARD = LVCMOS25; DRIVESTRENGTH = 8; PULLTYPE = PULLUP; }
> set_pin_assignment	{ clk }	{ LOCATION = R7; IOSTANDARD = LVCMOS25; PULLTYPE = PULLUP; }
> ```
> 上述代码中, clk 管脚约束到 FPGA 开发板上的 50MHZ 时钟; RSTn 管脚约束到 SW0 开关, <font color="red">开关向上拨时, 连接到电源 VDD 上, 此时 SoC 才可以工作</font>，



完成 TD 工程的创建. 在进入 TD 主界面后, 可以在 Source 窗口的 Hierarchy 标签页中看到我们的设计文件, 仿真文件和约束文件. 若要对某个文件临时进行编辑, 以约束文件为例, 我们可以双击 "pin.adc", 该文件的内容会被显示到右侧的编辑窗口中.



<!-- -->
> #### fread::Hierarchy
> + Hierarchy 标签页是对设计的层次化展示, 便于我们直观理解多个模块之间的调用关系. TD 会自动选择设计的顶层文件, 并标上<img src="/img/lab2/pics/8.png" alt="30" style="zoom:100%;" />图标. 你也可以通过右键文件选择 Set as Top 将任意文件设为顶层文件. <font color="red">注意, RTL 分析, 综合和实现的对象都是顶层文件.</font>

<center><img src="/img/lab2/pics/9.png" alt="25" style="zoom:100%;" /></center><center style="color:#0";>TD 工程主界面</center>

然后点击上侧导航栏的Run, 这时 TD 将会依次执行 RTL 分析 (RTL Analysis), 综合 (Synthesis), 实现 (Implementation) 和 生成比特流文件 (Generate Bitstream) 四个步骤. 最终生成的比特流文件用于 FPGA 的配置.

等待 TD 运行一段时间后, 可以看到弹出的比特流下载成功提示窗口, 关闭该窗口. 

<center><img src="/img/lab2/pics/10.png" alt="31" style="zoom:73%;" /></center><center style="color:#0";>比特流下载成功提示窗口</center>

点击上侧导航栏中Tools下的 Schematic Viewer--Read Design Schematic 选项, 可以看到综合和布局布线得到的原理图. 

<center><img src="/img/lab2/pics/11.png" alt="32" style="zoom:73%;" /></center><center style="color:#0";>进入 Schematic 界面</center>

<center><img src="/img/lab2/pics/12.png" alt="32" style="zoom:73%;" /></center><center style="color:#0";>进入 Schematic 界面</center>

点击Chip Viewer 标签, 可以看到设计对 FPGA 内部资源的占用情况.

<center><img src="/img/lab2/pics/13.png" alt="27" style="zoom:100%;" /></center><center style="color:#0";>Device 界面</center>

接下来进行板卡的连接. 使用两条数据线将 FPGA 板卡上的两个 Micro-USB 接口与 PC 上的两个 USB 接口连接起来.

<center><img src="/img/lab2/pics/14.png" alt="28" style="zoom:63%;" /></center><center style="color:#0";>板卡连接</center>

<!-- -->

> #### important::为什么是两个接口?
+ 标有 JTAG & UART 的接口是 FPGA 的下载配置接口, 用于比特流文件或其他配置文件的下载.
+ 标有 DAP-Link 的接口是 CMSIS 调试接口, 用于 FPGA 中实现的 CortexM0 的调试. 该接口与板卡上的 CMSIS-DAP 调试器连接, 该调试器的另一端连接在 FPGA 的引脚上, 这些引脚通过约束文件与 CortexM0 的 SWDIO 和 SWCLK 这两个调试信号连接. 

确保连线无误后, 点击左侧栏中的 Run, 这里一般系统会默认指定刚生成的比特流文件路径. 但这里仍要格外注意, 因为当你在多个工程间来回穿梭时, TD 可能会突然抽风, 给你默认指定一个其他工程中的比特流文件, 如果你没有注意而将它下载了进去, 那么接下来的调试工作可能会让你心态爆炸. 如若没有文件选择右侧导航栏中的Download -> ADD+，选择你刚刚创建的文件夹里面的***_RUNS文件，选择phy_1文件夹，选择刚刚生成的bit文件

<center><img src="/img/lab2/pics/15.png" alt="29" style="zoom:70%;" /></center><center style="color:#0";>Bit文件选择</center>

<center><img src="/img/lab2/pics/16.png" alt="29" style="zoom:70%;" /></center><center style="color:#0";>Bit文件选择</center>

<center><img src="/img/lab2/pics/17.png" alt="29" style="zoom:70%;" /></center><center style="color:#0";>Bit文件选择</center>

<center><img src="/img/lab2/pics/18.png" alt="36" style="zoom:70%;" /></center><center style="color:#0";>每次下载前记得检查比特流文件路径是否有误</center>

<!-- -->
> #### hint::TD 的工程目录结构
> 为了更好地帮助你找到工程中的文件(如日志文件,比特流文件等), 这里有必要简单介绍一下 TD 默认的工程目录结构.
>
> + ***_Runs: 存放综合实现的中间文件和结果文件
> + + .logs:日志文件
> + + syn_1: 与整个设计有关的综合 (Synthesis) 文件
> + + phy_1: 与整个设计有关的实现 (Implementation) 文件, 包括比特流文件和 bin 文件等

选择好比特流文件的路径之后, 点击 Program 开始下载比特流, 等待进度条加载完毕, 便可开始下一步的调试工作了.

<!-- -->

> #### question::思考
> 我们知道, 下载比特流文件是将硬件电路配置到 FPGA 中, 那么我们的软件程序是如何被下载到 FPGA 中的呢? 
>
> 还记得我们使用 keil 生成的 "code.hex" 文件对 RAM 进行了初始化. 所以在生成比特流文件时, RAM 的初始数据, 即程序的指令代码也就成为了比特流文件的一部分, 被下载到了 FPGA 中.

<!-- -->

