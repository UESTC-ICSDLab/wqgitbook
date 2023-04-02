# 将硬件固化到开发板

<center><img src="/img/lab2/39.png" alt="39" style="zoom:80%;" /></center><center style="color:#0";>开发板上 FPGA 与 FLASH 的关系</center> 

聪明的你可能已经发现, 如果按下了 FPGA 开发板上的复位键, 或者断电重启, 刚刚使用比特流配置的硬件逻辑就丢失了. 如果你想将硬件逻辑固化到开发板上, 实现复位/断电不丢失, 可以使用 Vivado 的 Add Configuration Memory Device 功能, 具体操作方法如下:

在 Vivado 的导航栏中点击 Settings 进入设置界面, 点击设置界面左侧的 Bitstream 进行比特流设置, 勾选上 -bin_file, 指明在生成比特流的同时生成相应的 bin 文件. 勾选好记得点击 OK.

<center><img src="/img/lab2/40.png" alt="40" style="zoom:100%;" /></center><center style="color:#0";>勾选生成 bin 文件</center> 

**然后重新综合, 实现, 生成比特流.** 比特流成功生成后, 你就可以在上节提到的 impl_1 文件夹中找到刚刚生成的 bin 文件了. 

按照上节所述的方式打开 Hardware Manager 界面并连接开发板. 这时右键器件, 在下拉栏中点击 Add Configuration Memory Device, 或者也可以在左侧导航栏的下方直接点击 Add Configuration Memory Device.

<center><img src="/img/lab2/41.png" alt="41" style="zoom:100%;" /></center><center style="color:#0";>Add Configuration Memory Device</center> 

选择 FLASH 型号, 这个要和开发板上的 FLASH 芯片型号对应起来.

<center><img src="/img/lab2/43.png" alt="43" style="zoom:100%;" /></center><center style="color:#0";>选择 FLASH 型号</center> 

<!-- -->
> #### important::关于 FLASH 型号
>
> 经过我们的不完全调查, 装有黄色排母的开发板上的 FALSH 是 64 位的 (n25q64-3.3v), 而装有黑色排母的开发板上的 FLASH 是 32 位的 (n25q32-3.3v), 请根据自己的情况选择正确的 FLASH 型号.

点击 OK 后进入选择 bin 文件的窗口, 这里要选择刚刚生成的 bin 文件, 选择时检查好文件路径是否正确.

<center><img src="/img/lab2/44.png" alt="44" style="zoom:70%;" /></center><center style="color:#0";>选择 bin 文件</center> 

选择好 bin 文件后, 点击 OK 开始 FLASH 的配置, 这里需要等待的时间稍微有点长. 当进度条加载完毕后, 硬件逻辑就被成功配置到 FLASH 里了. <font color=red>如果没有见效, 不要着急, 可以尝试按下 FPGA 开发板上的复位键并等待 Done 指示灯亮起.</font>

当开发板开始正常工作后, 本次的固化工作就完满结束了.

<!-- -->
> #### question::硬件是怎样实现固化的?
> 如果你只想完成实验, 而对硬件固化的原理不感兴趣, 可以跳过此部分.
>
> 硬件的固化是依靠一个叫做 FLASH 的芯片实现的, FLASH 的中文名叫做 "闪存", 是一种掉电数据不丢失的存储器件. 而 bin 文件正是一种 FLASH 的标准配置文件格式.
>
> Add Configuration Memory Device 操作会将包含有硬件电路信息的 bin 文件下载到 FLASH 中, 而 Program Device 操作只是将比特流文件下载到了 FPGA 中, 每次重启或者复位后, FPGA 都会首先从开发板上的 FLASH 中读取数据, 并进行硬件逻辑的配置. 从 FLASH 中读进来的数据会冲走原先配置的比特流信息, 这就是为什么 Program Device 的结果会在重启或者复位后丢失.
>
> 所以, 所谓的硬件固化并不是将数据固化在 FPGA 中, 而是固化在了它的 "好邻居" FLASH 中.
>
> 由于 FLASH 每次被写入前都需要进行漫长的擦除操作, 而写入操作本身也是一个比较缓慢的过程, 所以相比 Program Device, Add Configuration Memory Device 一般需要花费更多的时间.

<!-- -->
> #### hint::我的 Add Configuration Memory Device 变成灰色了, 点不动
> 硬件固化过一次后就会出现这样的情况, 重新固化之前, 需要右键相应的 FLASH 器件点击 Remove Configuration Memory Device, 移除掉之前的配置.
> <center><img src="/img/lab2/42.png" alt="42" style="zoom:70%;" /></center><center style="color:#0";>Remove Configuration Memory Device</center> 