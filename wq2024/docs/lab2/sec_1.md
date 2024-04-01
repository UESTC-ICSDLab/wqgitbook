# 硬件部分说明

<center><img src="/img/lab2/01.png" alt="02-26" style="zoom:80%;" /></center><center style="color:#0";>本实验中实现的硬件结构</center> 

<!-- -->
> #### important::BRAM ?
> 本实验中搭建的 BRAM 用于存放程序代码, 根据 Cortex-M0 体系推荐, 其地址范围设为 0x00000000-0x0000ffff.

<!-- -->
> #### fread::我需要做什么 ?
> 在小节中, 我们已经为你准备好了大部分的硬件设计代码, 你只需要做一些修改完善:
> 
> 1. 修改总线扩展模块 "/Task2/rtl/AHBlite_Decoder.v" 中的译码模块代码, 根据 BRAM 地址范围 0x00000000-0x0000ffff 完善对应的译码逻辑. 
> 2. 修改顶层文件 "/Task2/rtl/CortexM0_SoC.v" 代码, 连接 BRAM 接口 (AHBlite_Block_RAM) 与总线互连模块 (AHBlite_Interconnect). 
> 
> 如果你没有思路, 不要慌, 可以看看接下来的讲解:

| rtl 模块 | 说明 |
| ---- | ---- |
| cortexm0ds_logic | Cortex-M0 处理器核 |
| Block_RAM | RTL 实现的 RAM 单元 |
| AHBlite_Interconnect | 总线互连模块, 用于所有主设备和从设备的互连 |
| AHBlite_Block_RAM | RAM 接口 |
| AHBlite_Decoder | 地址译码器, 是总线互连模块的重要组成部分 |
| AHBlite_SlaveMUX | MUX 单元, 用于从设备返回数据多路选择, 是总线互连模块的重要组成部分 |
| CortexM0_SoC     | 系统的顶层文件 |

##### cortexm0ds_logic
由 ARM 官方提供的评估版 Cortex-M0, 反向结果, 不要尝试去读懂它.

##### Block_RAM 
+ 单端口 RAM, 输入输出数据位宽固定为 32, 存储深度可调节.
+ 使用 Vivado 提供的 IP 也可以实现 RAM, 但是不方便跨平台仿真, 考虑到本实验中需要的 RAM 功能非常简单, 适合直接通过 RTL 实现.

##### AHBlite_Interconnect
+ 设有 1 个主设备接口和 3 个从设备接口, 主设备接口用于连接 Cortex-M0, 从设备接口 Port0 用于连接 BRAM, <font color="red">Port1 和 Port2 在本实验中没有被使用到.</font>
+ 所有设备接口均采用 AHB 协议标准.

##### AHBlite_Block_RAM
由于总线互连接口都采用 AHB 协议标准, 不能直接与 Block_RAM 连接, 因此需要本模块作为 RAM 接口, 起到桥接的作用.

##### AHBlite_Decoder 
+ 地址译码是总线互连模块实现从设备选择的关键, 总线根据为每个从设备划分的地址区域, 以及当前接收到的来自于 CM0 的地址 (HADDR), 使能对应的从设备. 其本质是对 HADDR 高位的译码.
+ <font color="red">敲黑板: 本实验中的地址译码模块应当采用组合逻辑实现, 在编写设备使能有关的代码时, 应当注意使用 Port0_en 至 Port1_en 这几个 parameter, 以保证参数化设计, 这就要求应当根据从设备接口的使用情况合理地为 Port0_en 至 Port1_en 赋值. 

##### AHBlite_SlaveMUX
+ 三个从设备返回的信号 (例如 HRDATA, HREADYOUT, HRESP) 进行多路选择后返回至 CM0.
+ 在本实验中已经编写完善, 可以不用管.

##### CortexM0_SoC
+ 顶层文件. 用于每个系统模块的互连.
+ <font color="red">敲黑板: 在对 AHBlite_Block_RAM 和 AHBlite_Interconnect 的 AHB 协议接口进行连接时, 只需将同名信号连接, 并注意信号的来源和方向.

为了不影响后面的实验, 请参考标准答案:
<!-- -->
> #### hint::代码修改方式
> 将 "/Task2/rtl/AHBlite_Decoder.v" 中的:
> ```verilog
> /*RAMCode enable parameter*/
> parameter Port0_en = 0,
> /************************/
> ```
> 改为:
> ```verilog
> /*RAMCode enable parameter*/
> parameter Port0_en = 1,
> /************************/
> ```
> 然后将:
> ```verilog
> //0x00000000-0x0000ffff
> /*Insert RAMCODE decoder code there*/
> assign P0_HSEL = 1'b0;
> /***********************************/
> ```
> 改为:
> ```verilog
> //0x00000000-0x0000ffff
> /*Insert RAMCODE decoder code there*/
> assign P0_HSEL = (HADDR[31:16] == 16'h0000) ? Port0_en : 1'b0; 
> /***********************************/
> ```
> 将 "/Task2/rtl/CortexM0_SoC.v" 中的:
> ```verilog
> /* Connect to Interconnect Port 0 */
> .HCLK              (clk),
> .HRESETn           (cpuresetn),
> .HSEL              (/*Port 0*/),
> .HADDR             (/*Port 0*/),
> .HPROT             (/*Port 0*/),
> .HSIZE             (/*Port 0*/),
> .HTRANS            (/*Port 0*/),
> .HWDATA            (/*Port 0*/),
> .HWRITE            (/*Port 0*/),
> .HRDATA            (/*Port 0*/),
> .HREADY            (/*Port 0*/),
> .HREADYOUT         (/*Port 0*/),
> .HRESP             (/*Port 0*/),
> .BRAM_WRADDR       (RAMCODE_WADDR),
> .BRAM_RDADDR       (RAMCODE_RADDR),
> .BRAM_RDATA        (RAMCODE_RDATA),
> .BRAM_WDATA        (RAMCODE_WDATA),
> .BRAM_WRITE        (RAMCODE_WRITE)
> /**********************************/ 
> ```
> 改为:
> ```verilog
> /* Connect to Interconnect Port 0 */
> .HCLK              (clk),
> .HRESETn           (cpuresetn),
> .HSEL              (HSEL_P0),
> .HADDR             (HADDR_P0),
> .HPROT             (HPROT_P0),
> .HSIZE             (HSIZE_P0),
> .HTRANS            (HTRANS_P0),
> .HWDATA            (HWDATA_P0),
> .HWRITE            (HWRITE_P0),
> .HRDATA            (HRDATA_P0),
> .HREADY            (HREADY_P0),
> .HREADYOUT         (HREADYOUT_P0),
> .HRESP             (HRESP_P0),
> .BRAM_WRADDR       (RAMCODE_WADDR),
> .BRAM_RDADDR       (RAMCODE_RADDR),
> .BRAM_RDATA        (RAMCODE_RDATA),
> .BRAM_WDATA        (RAMCODE_WDATA),
> .BRAM_WRITE        (RAMCODE_WRITE)
> /**********************************/
> ```