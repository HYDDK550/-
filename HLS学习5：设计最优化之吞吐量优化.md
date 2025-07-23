---
title: HLS学习5：设计最优化之吞吐量优化
created: '2025-07-15T16:42:36.944Z'
modified: '2025-07-23T09:50:25.124Z'
---

# HLS学习5：设计最优化之吞吐量优化
背景：为了更少的时延，我们需要增大吞吐量和流率，因此需要用到下面的优化指令。

目的：熟悉UG902文档中HLS关于增大吞吐量和流率的优化指令。

## 常用加大吞吐量的指令

### 1.PIPELINE：可以用于函数和循环。

意思是一个操作并不需要完成所有的步骤，而下一个操作就会开始。
如果将PIPELINE运用于外层循环则内层循环会被自动展开。

函数的Initiation Interval（II）为3，latency为2。加了pipeline之后，II=1
PIPELINE指令的II（Initiation interval）默认值为1，II可以被确认。Pipeline指令下面的层级的会被UNROLL，所有的子函数需要被单独的PIPELINE。

### 2.ARRAY_PARTITION and array_reshape ：将阵列分区为更小的阵列或者独立元素

生成包含多个小型存储器或多个寄存器（而不是一个大型存储器）的 RTL。
有效增加存储器读写端口数量。
可能改善设计吞吐量。
需要更多存储器实例或寄存器。

2.1区别？
array_partition指令更加灵活。
array_reshape指令更加节约资源，并且在时序较差的情况下有奇效。

详细原语解析：https://www.ppmy.cn/ops/15646.html

### 3.DATAFFLOW：用于顶层，函数与函数之间的数据优化
注意不是所有任务都能被优化：
https://blog.csdn.net/qq_35608277/article/details/104643645

###　

