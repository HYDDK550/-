---
title: HLS学习6：优化数组
created: '2025-07-23T09:49:18.575Z'
modified: '2025-07-30T08:56:02.175Z'
---

# HLS学习6：优化数组

## 为什么要优化数组？
当对函数进行流水线优化时常常遇到以下的问题：
```
INFO: [SCHED 204-61] Pipelining loop 'SUM_LOOP'.
WARNING: [SCHED 204-69] Unable to schedule 'load' operation ('mem_load_2', 
bottleneck.c:62) on array 'mem' due to limited memory ports.
WARNING: [SCHED 204-69] The resource limit of core:RAM:mem:p0 is 1, current 
assignments: 
WARNING: [SCHED 204-69] 'load' operation ('mem_load', bottleneck.c:62) 
on array 
'mem',
WARNING: [SCHED 204-69] The resource limit of core:RAM:mem:p1 is 1, current 
assignments: 
WARNING: [SCHED 204-69] 'load' operation ('mem_load_1', 
bottleneck.c:62) on array 
'mem',
INFO: [SCHED 204-61] Pipelining result: Target II: 1, Final II: 2, Depth: 3.
```
此问题通常是由数组所导致的。数组作为最多只含有 2 个数据端口的块 RAM 来实现。这可能限制读写（或加载/存储）密集型算法的吞吐量。通过将该数组（单一块 RAM 资源）拆分为多个更小的数组（多个块 RAM）从而有效增加端口数量，即可改善带宽。

* 对于数组可以通过RESOURCE指令来告诉VivadoHLS当前的数组采用什么类型的memory来实现（如分布式的register、LUT或BRAM）以及RAM是采用单端口还是双端口。如果没有使用RESOURCE，VivadoHLS会自动决定是使用双端口还是单端口，默认情况是用单端口，如果双端口能降低initiationinterval或latency的话就使用双端口。
* 如果这个数组作为顶层函数的形参，那么最终就会以相应的memory接口形式呈现，包括相应的读写地址和数据以及读写使能。如果数组在设计内部，最终就会映射为真正的blockRAM，LUTRAM，UltraRAM 或者register，具体类型取决于在设计中使用的优化方法。
**用memory或memory ports实现的数组往往是设计中的性能瓶颈。**


## 数组的优化指令介绍


### 1.数组分割ARRAY_PARTITION  

数组分割会生成包含多个小型存储器或多个寄存器（而不是一个大型存储器）的 RTL。能够有效增加存储器读写端口数量，可能改善设计吞吐量。但需要更多存储器实例或寄存器。

#### 1.1 ARRAY_PARTITION的类型 
Vivado HLS 可提供 3 种类型的数组分区，如下图所示。
• block：原始数组分割为原始数组的连续元素块（大小相同）。
• cyclic：原始数组分割多个大小相同的块，这些块交织成原始数组的元素。
• complete：默认操作是将数组按其独立元素进行拆分。这对应于将内存解析为寄存器。
<div align=center>
<img src=".\\hls6\\1.png" width = 500 >
数组分区方式图  <br>
<br><div align=left>
* 对于block和cyclic分区，factor选项可指定要创建的数组数量。并不是分割的块数越多越好的，分块的个数要取决于真实的数据流，更多的分块只是意味着更多的访问端口。如图 20-5 示例代码所示，在每次运算中要读取 3 个数组元素，每块 RAM 能提供最多 2 个端口，所以一共两个分块就可以达到最优的访问速率了。如果将数组分为 3 块，其结果和分为两块的情况在 latency 和 interval上的表现是一样的，在资源消耗上也并没有显著改善。

```
#include "array_mem.h"
void array_mem(di_t mem[N]),do_t sum[N-2])
{
  int i;
  sum_loop:
  for(i = 2;i < N;++i)
  {
    sum[i-2] = mem[i] + mem[i-1] +mem[i-2];
  }
}
```

#### 1.2 数组分区维度
对多维数组进行分区时，dimension选项可用于指定对哪个维度进行分区。对于多维数组的分割要注意分割的数组维度，如数组 array[10][6][4],这里 10、6、4 所
对应的维度数分别是 1、2、3。 

#### 自动数组分区
config_array_partition配置可根据元素数量判定数组的自动分区方式。可通过菜单访问。

**“Soluton” →“SolutonSetngs” → “General” → “Add” → “confg_array_partton”**


通过throughput_driven选项可对分区阈值进行调整，并且可完全实现自动分区。选中throughput_driven选项时，Vivado HLS 会自动对数组进行分区以实现指定的吞吐量。

#### 1.3 数组分割的示例与原语

* 详细内容可以在UG902的107页
* 详细原语解析：https://www.ppmy.cn/ops/15646.html
* 优化示例：UG871的92页第七章

### 2.数组映射ARRAY_MAP

当 C 语言代码中存在大量小型数组时，将其映射到单一大型数组通常可减少所需的块 RAM 数量。
受器件支持的前提下，每个数组都映射到 1 个块 RAM 或 UltraRAM。任一 FPGA 中提供的基本块 RAM 单元为 18K。如有大量小型数组且占用资源不足 18K，那么为了更有效地利用块 RAM 资源，可将大量小型数组映射到单一大型数组。如果块 RAM 大于 18K，则会自动将其映射到多个 18K 单元。在综合报告中，请复查“Utlizaton Report” →“Details” → “Memory”，以便详细了解设计中块 RAM 的使用情况。

#### 2.1 水平的ARRAY_MAP


```
void foo (...) {
 int8  array1[M];
 int12 array2[N];
#pragma HLS ARRAY_MAP variable=array1 instance=array3 horizontal 
#pragma HLS ARRAY_MAP variable=array2 instance=array3 horizontal  
...   
loop_1: for(i=0;i<M;i++) {
 array1[i] = ...;
 array2[i] = ...;
 ...
}
...
}
```
上面的示例代码展示了水平映射的方式，本来会生成的array1与array2被映射到单一数组array3中。映射从较大的数组中的位置 0 开始，并遵循指定命令的顺序执行映射。在 Vivado HLS GUI 中，此映射操作基于使用菜单命令指定数组的顺序来执行。在 Tcl 环境中，此操作基于发出命令的顺序来执行。
<div align=center>
<img src=".\\hls6\\2.png" width = 500 >
水平数组映射  <br>
<br><div align=left>

* 带偏移的数组映射用offset实现，在UG902的131页有介绍

#### 2.2 垂直的ARRAY_MAP
```
void foo (...) {
 int8  array1[M];
 int12 array2[N];
#pragma HLS ARRAY_MAP variable=array1 instance=array3 vertical 
#pragma HLS ARRAY_MAP variable=array2 instance=array3 vertical   
...   
loop_1: for(i=0;i<M;i++) {
 array1[i] = ...;
 array2[i] = ...;
 ...
}
...
}
```
在垂直映射中，通过并置多个数组来生成位宽更高的单个数组。垂直映射通过使用vertcal选项应用于 INLINE 指令。下图显示了应用垂直映射模式时，前述示例所发生的变化。

<div align=center>
<img src=".\\hls6\\2.png" width = 500 >
垂直数组映射  <br>
<br><div align=left>

#### 特殊注意事项

* 数组变换的对象必须先包含在源代码中，随后才能应用任何其它指令。
* 可对全局数组进行映射。但生成的数组实例为全局实例，映射到该数组实例的任何局部数组都会变为全局数组。当不同函数的局部数组映射到同一目标数组时，目标数组实例就会变为全局实例。
* 仅当数组函数实参属于同一函数的实参时，才能对其进行映射。

### 3.ARRAY_RESHAPE

ARRAY_RESHAPE 指令将 ARRAY_PARTITIONING 与 ARRAY_MAP 的垂直模式相结合，用于减少块 RAM 数量，同时仍支持分区的有利特性：并行访问数据。

**3.1与array_partition区别？**
array_partition指令更加灵活。
array_reshape指令更加节约资源，并且在时序较差的情况下有奇效。

3.2 ARRAY_RESHAPE的类型
```
void foo (...) {
int  array1[N];
int  array2[N];
int  array3[N];
#pragma HLS ARRAY_RESHAPE variable=array1 block factor=2 dim=1 
#pragma HLS ARRAY_RESHAPE variable=array2 cycle factor=2 dim=1 
#pragma HLS ARRAY_RESHAPE variable=array3 complete dim=1
...   
}
```
ARRAY_RESHAPE 指令可将数组转换为下图所示形式。
<div align=center>
<img src=".\\hls6\\4.png" width = 500 >
  ARRAY_RESHAPE<br>
<br><div align=left>

3.3 提高吞吐量
ARRAY_RESHAPE 指令支持在单一时钟周期内访问更多数据。只要能在单一时钟周期内访问更多数据，Vivado HLS 即可自动展开使用此数据的所有循环，前提是这样有助于提升吞吐量。循环可全部或部分展开以创建足够的硬件以便在单一时钟周期内使用更多数据。此功能可使用config_unroll命令和tripcount_threshold选项来加以控制。在以下示例中，行程计数小于 16 的任何循环都将自动展开（前提是可提高吞吐量）。

```
config_unroll -tripcount_threshold 16
```


## 数组优化的示例以及总结













