# 微处理器系统结构与嵌入式系统设计 课程实验 2025秋

## 实验前阅读

> #### important::必读通知
>
> - 2025/10/26: 本指导书正式发布.
> 
> - 2025/10/26: 更新 <font color="red">LAB0</font> 和 <font color="red">LAB1</font>, 其余实验将在后续发布.
> - 2025/10/27: 更新 <font color="red">LAB2</font>.

<!-- -->

> #### fread::关于本指导书的配套代码
> 点击[此处](https://gitee.com/zhang-xiaomou/CortexM0_SoC)获取本指导书的配套代码, 文件夹名称为 CortexM0_SoC, <font color="red">请务必将该文件夹放在英文路径下</font>.
> 本指导书中实验与代码文件的对应关系:

> | 实验名称 | 对应代码文件夹 |
> | ---- | ---- |
> | LAB0 - 工欲善其事 必先利其器 | Task0 |
> | LAB1 - "施法"让 CPU 动起来 | Task1 |
> | LAB2 - "点石成金" - 实现你的首个 SoC | Task2 |


<!-- -->
> #### question::如何提交实验报告
>
> 请大家按每一次实验的要求, 在规定时间内, 将符合规范的实验报告提交至自己所在班级助教的邮箱.
>
> | 授课教师 | 助教 | 实验报告提交邮箱 |
> |  :-:  | :-:  | :-:  |
> | 黄乐天  | 陈晨 | 1647242532@qq.com |
> | 杨成韬  | 陈飞扬 | 457063678@qq.com |
> | 万里冰  | 陈雨阳 | 212913904@qq.com |
> | 廖永波  | 曹欣雨 | 3143216985@qq.com |
> | 张驰  | 郝禹 | 3627923058@qq.com |

<!-- -->
> #### hint::如果你遇到问题
>
> - 请先阅读[常见问题](faq/introduction.md).
> - 阅读官方文档.
> - 尝试使用搜索引擎(只推荐 [Google](https://google.com))寻找解决方案.
> - 如果你做了以上尝试, 却依然无法解决问题, 请向助教或老师提问. 在此之前请确保你了解[提问的智慧](https://github.com/ryanhanwu/How-To-Ask-Questions-The-Smart-Way/blob/main/README-zh_CN.md).

<!-- -->
> #### fread::外部资源
> 
> - [B 站录播](https://www.bilibili.com/video/BV1Wf4y1W7gd?spm_id_from=333.999.0.0)
> - [知乎专栏](https://www.zhihu.com/column/conquest-on-chip)(如果不熟悉 Verilog 和 FPGA)
>

