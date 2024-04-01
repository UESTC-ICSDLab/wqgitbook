# LAB3: "灯, 等灯, 等灯" - 流水灯的几种点法

<!-- -->
> #### hint::关于LAB3
> + 本实验对应实验指导书的第 6 章 "数据存储器与流水灯外设"
> + 本实验将在 LAB2 的基础上, 给 SoC 系统添加数据存储器, 并介绍如何给SoC添加外设.
> + 本实验将通过比对流水灯的的两种实现方式, 理解软硬件的结合和控制字的概念.

<!-- -->
> #### info::通过这个实验能学到什么
> 本实验会用两种不同的方式实现流水灯外设, 两种不同的方式实现流水灯就体现了我们常说的软硬件结合. 软件的执行效率是没有专用硬件高的, 但是软件的灵活性更强, 在对于效率要求高的场景, 可以设计硬件替代软件的部分工作, 实现更高的效率.