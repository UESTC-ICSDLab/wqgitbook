# 自己动手, 理解函数调用
汇编程序之间原则上不需要通过栈进行相互调用, 而 c 程序之间基于栈的相互调用在编译器的处理下是对编程者屏蔽的, 所以, 要动手理解 ARM 程序的调用方式, 最好的选择是进行 c 程序与汇编程序的相互调用.

## c 程序调用汇编程序
在 c 程序的 main 函数中调用一段汇编程序, 实现字符串的复制.

首先, 在之前创建的 Keil 工程的基础上, 移除 Target 中的所有源文件.

<center><img src="/img/lab1/02-27.png" alt="02-27" style="zoom:75%;" /></center><center style="color:#0";>移除源文件的方法</center> 

打开文件夹 "./Task1/call", 发现这里已经为你准备好了本实验需要的启动文件, 直接将其添加至工程 Target 中即可. 打开启动文件, <font color='red'>你会发现这里多了几行用于栈初始化的代码 (记住这句话, 待会要考).</font>

然后在 "./Task1/call" 中新建文件 "main.c" 和 "strcopy.s" 并添加至工程 Target 中. 

在 "main.c" 中添加以下代码:

```c
#include <string.h>
#include <stdint.h>
#include <stdio.h>

extern void strcopy(char *d, const char *s);

int main()
{
    char srcstr[] = "First string - soure";         /* 定义源字符串数组并初始化 */
    char dststr[] = "Second string - destination";  /* 定义目的字符串数组并初始化 */
    strcopy(dststr,srcstr);                         /* 调用字符串复制汇编函数 */
    return 0;
}
```

在 "strcopy.s" 中添加以下代码:

```armasm
			AREA SCopy, CODE, READONLY
			EXPORT  strcopy
strcopy
			LDRB 	r2,     [r1]    ; r1对应源字符串首地址，利用寄存器间接寻址读取字符
			ADDS 	r1,     #1
			STRB 	r2,     [r0]    ; r0对应目的字符串首地址，利用寄存器间接寻址保存字符
			ADDS 	r0,     #1
			CMP  	r2,     #0      ; 判断字符串是否结束
			BNE  	strcopy         ; 循环执行字符复制，直到字符串结束
			BX   	lr              ; 汇编子程序返回
			END
```

保存文件后, 进行编译 (Build).

<!-- -->
> #### important::What? 我编译失败了
> 1. 根据错误提示检查是否存在语法错误.
> 2. 尝试在 Optiins for Target -> Target 右侧菜单的 ARM Compiler 处选择 V5 版本的编译器. 
> 3. 尝试在 Optiins for Target -> Linker 里取消勾选 Don't Search Standard Libraries.

编译成功后, 运行仿真调试.

别慌, 一步一步来. 请一边单步运行, 一遍观察源代码窗口中程序的运行情况, 当程序进入 strcopy 程序后, 观察寄存器窗口中 r0 和 r1 的值, 此时的 r0 便是字符串拷贝的目的地址, r1 是字符串拷贝的源地址. 

<center><img src="/img/lab1/02-20.png" alt="02-20" style="zoom:80%;" /></center><center style="color:#0";>寄存器窗口</center> 

为了在接下来更直观地观察字符串拷贝的过程, 将当前 r0 的值拷贝到页面底部的 Memory 窗口中, 这时便可以看到该地址周围的所有数据.

<center><img src="/img/lab1/02-21.png" alt="02-21" style="zoom:100%;" /></center><center style="color:#0";>Memory 窗口</center> 

<!-- -->
> #### question::内存映射, Debug 的利器
> Memory 窗口的内存映射功能允许我们去查看任意地址处存储的数据, 这里的内存是抽象的内存, 因为 ARM 架构采用统一编址方式, 所有的存储设备以及外设空间共用同一个地址, 这意味着通过内存映射, Flash, ROM, 外设寄存器等所有的一切都能够尽收眼底. 它的作用不仅限于在线仿真, 在基于调试器的实际调试中同样能够发挥作用.
>
> 如果 Memory 窗口不小心被关闭, 可以从菜单栏 -> View -> Memory Windows 打开.

字符串是采用 ASCII 标准编码的, 所以需要右键 Memory 区域选择 Ascii, 这样数据就以字符串的形式呈现出来了.

<center><img src="/img/lab1/02-22.png" alt="02-22" style="zoom:80%;" /></center><center style="color:#0";>以字符串解码 Memory 窗口内容</center> 

可以看到, 我们在 "main.c" 中定义的 dststr 出现在了字符串的目的地址处, 这时, 你可以单步运行, 观察 dststr 一步步被 srcstr 拷贝覆盖的过程, 也可以点击<img src="/img/lab1/02-23.png" alt="02-23" style="zoom:100%;" />直接运行, 观察字符串拷贝的最终结果.

<center><img src="/img/lab1/02-24.png" alt="02-24" style="zoom:80%;" /></center><center style="color:#0";>最终运行结果</center> 

<!-- -->
> #### important::思考
> 启动文件中是怎样进行栈初始化的? 我们知道 RAM 的起始地址是 0x20000000, 在启动文件里我们分配了 0x400 个字节的栈空间, 然后将栈顶地址 __initial_sp 传给中断向量表的第一个表项, 也就是将初始的 SP 值存储在 0x00000000 地址处, 以便程序启动时将其读入 R13 中.
>
> ```armasm
> Stack_Size      EQU     0x00000400
>                 AREA    STACK, NOINIT, READWRITE, ALIGN=4
> Stack_Mem       SPACE   Stack_Size
> __initial_sp
> 
>                 AREA    RESET, DATA, READONLY
>                 EXPORT  __Vectors
> __Vectors       DCD     __initial_sp              ; Top of Stack
> ```
>
> 在本实验中, main 函数中定义的字符串作为局部变量被存放在栈里. 另外, 注意我们在 main 函数中 strcopy 函数的声明方式:
>
> ```c
> extern void strcopy(char *d, const char *s);
> ```
>
> <font color="blue">*d 和 *s 是两个 32 位地址参数, 所以根据 ATPCS 标准, 在调用该函数前, 程序会把事先把准备好的实参放在 r0 和 r1 中, 这是程序成功实现传参的关键. 注意 strcopy 程序中的最后一条指令 BX lr, 它能够指导该程序返回 main 函数, 这是因为在 main 函数中调用 strcopy 时, 会执行一条 BL 跳转指令, 跳转到 strcopy 的同时会把返回地址保存在 lr 中.</font>

## 汇编程序调用c程序
接下来的实验中, 我们将进行一个 "c调用汇编, 汇编再调用c" 的两层嵌套, 实现一个简单的数学运算功能: i+2i+3i+4i.
将 "main.c" 的内容更改为:

```c
#include <string.h>
#include <stdint.h>
#include <stdio.h>
extern void f(void);
int main()
{
	f();
    return 0;
}

/*a+b+c+d+e*/
int g(int a, int b, int c, int d)
{
	return a + b + c + d;
}
```

在 "./Task1/call" 中创建汇编文件 "f.s" 并添加以下内容:

```armasm
        EXPORT f
        PRESERVE8
        AREA f,CODE,READONLY
        ENTRY					    
        IMPORT g                 ; 声明g为外部引用符号
        PUSH   {lr}
        MOVS   r0, #2            ; i=2
        ADDS   r1, r0, r0        ; (R1)=i*2
        ADDS   r2, r1, r0        ; (R2)=i*3
        ADDS   r3, r1, r1        ; (R3)=i*4
        BL     g                 ; 调用C函数g()，返回值在R0中
        POP    {pc}  
```

保持启动文件不变, 确保更改后的 "main.c" 和 "f.s" 已经添加至 Target, 编译并仿真调试.
仍然单步运行, 观察程序一步步从 main 函数进入 f 程序, 在 f 程序中, <font color="blue">根据 ATPCS 标准, 我们要将 g 函数需要的四个参数一一准备好, 分别放在 r0-r3 中.</font> 单步运行程序, 直到程序进入 g 函数之前, 我们会看到参数 2, 4, 6, 8 已经被存放到了 r0-r3 中:

<center><img src="/img/lab1/02-25.png" alt="02-25" style="zoom:100%;" /></center><center style="color:#0";>在进入 g 函数之前, r0-r3 中已经备好了四个参数</center> 

继续单步运行, 直到运行到 g 函数返回之前, 可以看到计算结果 2+4+6+8=20=14H 被存放到了 r0 中, <font color="blue">这是因为根据 ATPCS 标准, 函数返回的第一个数据应该存放到 r0 中.</font>

<center><img src="/img/lab1/02-26.png" alt="02-26" style="zoom:100%;" /></center><center style="color:#0";>在 g 函数返回之前, 其运算结果已经被存放到了 r0 中</center> 

继续单步运行, 观察程序一步步从 g 函数返回至 f 程序, 最后返回至 main 函数中.

<!-- -->
> #### important::思考
> 1. 请思考, "f.s" 中的 PUSH 和 POP 指令有什么作用呢? 如果去掉这两条指令, 程序还能顺利返回到 main 函数中吗?
> 2. 其实, ARM 架构中的保护现场不仅保护了通用寄存器, 有时还需要保护 lr. 那么请思考, 究竟什么时候需要保护 lr, 什么时候不需要呢? 