---
title: HLS学习5：优化FOR循环
created: '2025-07-15T16:42:36.944Z'
modified: '2025-07-30T08:46:46.807Z'
---

# HLS学习5：优化FOR循环

* 背景：为了更少的时延，我们需要增大吞吐量和流率，先从FOR循环的优化指令入手
* 目的：熟悉UG902文档中HLS关于FOR循环的优化指令。
* 注意：关于函数与FOR循环优化指令共同的地方会有介绍，如PIPELINE和DATAFLOW

## 性能指标
以下面这个代码示例来讲解for循环的衡量指标。
示例代码所做的就是将xin数组中的每个元素乘以a再加b加c后存入数组yo。
```
#include <ap_int.h>

#define N 3
#define XW 8
#define BW 16

typedef ap_int<XW> dx_t;
typedef ap_int<BW> db_t;
typedef ap_int<BW+1> d0_t;

void foo(dx_t xin[N],dx_t a,db_t b,db_t b. db_t c.do_t yo[Nz]);
```

```
#inculde "foo.h"
void foo(dx_t xin[N],dx_t a,db_t b,db_t b. db_t c.do_t yo[Nz])
{
  int i=0;
  loop:
  for(i = 0;i < N;i++)
  {
    yo[i] = a * xin[i] + b + c;
  }
}
```
<div align=center>
<img src=".\\hls5\\1.png" width = 500 >
示例代码状态图  <br>
<br><div align=left>

假设每一个状态消耗一个时钟周期。因为有3个c1、c2、c3的组合循环，所以Loop TripCount为3；在每个LoopTripCount要消耗3个时钟周期，所以LoopIterationLatency 为3；本次循环与下次循环的时间间隔在这里与LoopIterationLatency相等，所以Loop IterationInterva（lLoopII）也为3；整个for循环的latency为TripCount*IterationLatency，所以Looplatency为3*3=9；整个函数的Functionlatency为10，Functioninitialinterva（lII）为11。这些概念是衡量设计的重要指标，在优化时经常会提及。

## For循环优化

### 1.PIPELINE：可以用于函数和循环。
* 意思是一个操作并不需要完成所有的步骤，而下一个操作就会开始。
<div align=center>
<img src=".\\hls5\\2.png" width = 500 >
  pipeline 前和 pipeline 后执行结构对比<br>
<br><div align=left>


* PIPELINE指令的II（Initiation interval）默认值为1，II可以被确认。
* PIPELINE指令下面的层级的会被UNROLL，所有的子函数需要被单独的PIPELINE。
如果将PIPELINE运用于外层循环则内层循环会被自动展开。

### 2.Unroll展开

* 在默认情况下for循环是折叠的（rolled），可以理解为每次循环都使用同一套电路，每次循环分时复用。如果展开，则循环电路被复制若干份，具体数量可以通过参数设置。这些复制出的电路可以同时执行不同的数据。
<div align=center>
<img src=".\\hls5\\3.png" width = 800 >
  乘法器消耗了3个（UNROLL前为1），即循环被复制了3份<br>
<br><div align=left>

### 3.LOOP_MERGE循环合并

* 默认情况下VivadoHLS是不会进行循环合并的，除非两个loop之间没有依赖关系，是同时对某些数组进行操作。例如下面的两个循环,这里因为两个 loop 都是对a 和 b 两个数组进行计算，所以可以通过 LOOP_MERGE 指定进行合并优化。
* 

```
#include "for_merge.h"

void for_merge(data_t a[N],data_t b[N],data_t c[N],data_t d[N])
{
  int i=0;
  loop_region:
{
  add:
  for(i = 0;i < N;i++)
  {
    c[i] = a[i] + b[i];
  }
  sub:
  for(i = 0;i < N;i++)
  {
    d[i] = a[i] - b[i];
  }
}
}
```
**注意**
1. 如果两个循环的循环边界不同且都为常量，合并之后的循环边界取更大的作为新的循环边界。
2. 一个常量一个变量作为循环边界的两个循环是不可以合并的。如果两个都是变量作为循环边界的两个循环也是不可以合并的。
3. 对于两个变量作为循环边界，可以将边界较大的拆开，一部分等于较小的，剩下的作为
另一部分，这样就可以合并了。以上图的 J 和 K 为例，假设 J 大于 K，则可以把 J 拆成 0-K
和 K 到 J 两部分，然后就可以合并了，如下：

```
void for_merge(data_t a[N],data_t b[N],data_t c[N],data_t d[M],ctrl_t J,ctrl_t K)
{
  int i = 0;
  loop_region:
{
  add:
  for(i = 0;i < K;i++)
  {
    c[i] = a[i] + b[i];
  }
  sub:
  for(i = 0;i < K;i++)
  {
    d[i] = a[i] - b[i];
  }
}
  extar_loop:
  for(i = K;i < J;i++)
  {
    d[i] = a[i] + b[i];
  }
}
```


### 4.DATAFLOW

```
#inculde "foo.h"
void foo(data_t A[N],data_t B[N],data_t C[N])
{
  datax_t x[N];
  datax_t y[N];
  int i = 0;

  loop_A:
  for(i = 0;i < N;i++)
  {
    x[i] = A[i] + 4;
  }
   loop_B:
  for(i = 0;i < N;i++)
  {
    y[i] = x[i] * B[i];
  }
   loop_C:
  for(i = 0;i < N;i++)
  {
    C[i] = y[i] - 2;
  }
}

```
可以看出数组 A 通过 Task A 生成数组 x，数组 x 通过 Task B 生成数组 y，数组 y 通过Task C 生成数组 C,刚好可以用到 DATAFLOW 数据流优化方法。数据流优化就是在三个循环之间插入 Channel（可以是 Ping-pong RAM、FIFO 或 Register）。

<div align=center>
<img src=".\\hls5\\4.png" width = 500 >
  DATAFLOW优化原理<br>
<br><div align=left>

在没有使用 DATAFLOW 时，三个循环顺序执行，是没有交叠的。DATAFLOW 之后是的在执行 Loop B 的时候并不需要等到 Loop A 完全执行完之后再开始，只要 A 有输出，就可以利用这个输出去执行 Loop B。这样多个任务之间就可以有交叠，降低了 latency，提高了数据吞吐率。
<div align=center>
<img src=".\\hls5\\5.png" width = 500 >
   DATAFLOW前和DATAFLOW后执行结构对比<br>
<br><div align=left>

#### 对于ABC之间的通道的类型，UG902的123页
可以配置ping-pong RAM或FIFO。如果参数是个scalar（标量）、pointer（指针）或reference（引用）那么HLS会把它归类为FIFO。如果参数是个数组，通道的类型可能是FIFO（数据流是按顺序）也可能是RAM（这时就会把通道变成一个ping-pong RAM）。
<div align=center>
<img src=".\\hls5\\8.png" width = 400 >
<br><div align=left>
可以选择默认的配置方式，当然我们也可以通过工具来设定这个通道是ping-pong RAM还是FIFO（注意FIFO的深度）。

<br>

#### 在数据流内使用 ap_ctrl_none，UG902的125页

ap_ctrl_none 块级 I/O 协议会避免使用 ap_ctrl_hs 和 ap_ctrl_chain 协议所暗示的僵化的同步方案。这些协议要求区域内所有进程的执行次数完全相同以便与 C 语言行为更匹配。



#### 注意不能用DATAFLOW的情况，UG902文档119页
• 单一生产者使用者违例
• 绕过任务
• 任务间的反馈
• 任务的有条件执行
• 含多个退出条件的循环
违例和具体的优化方法请查看UG902文档119页。
这篇BLOG也作了介绍： https://blog.csdn.net/qq_35608277/article/details/104643645

例如简单介绍一下单一生产者使用者违例:

```
#include "foo.h"
void foo(din_t din[N], din_t scale, dout_t dout1[N],dout_t dout2[N])
{
  temp t temp1[N];
  loop1:for(int i =0;i < N; i++)
  {
    temp1[i]= din[i]* scale;
  }
  1oop2:for(int j=0;j< N; j++)
  {
    doutl[j]=temp1[j]* 2;
  }
  1oop3 :for(int k=0;k<N; k++)
  {
    dout2[k]=temp1[k]* 4;
  }
}

```
可以看到din通过Loop1生成temp1，然后temp1经过Loop2生成dout1，同时temp1也会到Loop3生成dout2。所以temp1被Loop2和Loop3都使用到了。这里Loop2和Loop3是可以做循环合并优化的，但是不能使用DATAFLOW优化，因为temp1 被Loop2和Loop3都使用了。

<div align=center>
<img src=".\\hls5\\6.png" width = 500 >
  Single-producer-consume<br>
<br><div align=left>

对于上述不能DATAFLOW的代码，可以稍作改变使之可以使用。
对于修改后的代码，主要就是增加了一个loop_copy，就是把temp1复制两份，分别赋给temp2和temp3。这时候数据流就改变了。din通过loop1生成temp1，然后temp1通过loop_copy生成temp2和temp3，temp2只给loop2用，temp3只给loop3用，分别生成dout1和dout2。这时候再用DATAFLOW就可以改善latency和utilization了。

```
#include "foo.h"
void foo(din_t din[N], din_t scale, dout_t dout1[N],dout_t dout2[N])
{
  temp t temp1[N];
  temp_t temp2[N];
  temp_t temp3[N];

  1oop1:
  for(inti=0;i<N; i++)
  {
    temp1[i]= din[i]* scale;
  }
  loop copy:
  for(int m=0;m<N; m++)
  {
    temp2[m]= temp1[m];
    temp3[m]= temp1[m];
  }
  1oop2:for(int j=0;j< N; j++)
  {
    dout1[j]= temp2[j]* 2;
  }
  1oop3 :for(int k=0;k<N;k++)
  {
    dout2[k]=temp3[k]* 4;
  }
}
```

## FOR循环优化的一些注意事项


### 如何处理嵌套的循环？
循环嵌套共有三种：


<div align=center>
<img src=".\\hls5\\9.png" width = 400 >
 三种循环嵌套类型<br>
<br><div align=left>

 **这里就注意一下**
  1. 对与Perfectloopnest，对外层做PIPELINE会把内层全部UNROLL，而其本身以及外层循环会被flatten。这样的latency肯定最少，不过注意消耗的资源。Vivado HLS 提供的set_directive_loop_flatten命令允许将已标记为完美和半完美的嵌套循环扁平化，这样就无需重新编码来提升硬件性能，并且还可减少执行循环中的运算所需的周期数。

 **unroll展开和flatten展开是不一样的概念。unroll是将循环体展开为并行操作，而flatten是将嵌套循环展开为单一循环。**

 2. 对于Imperfectloopnest可以通过代码优化手段将其转变为Semi-Perfectloopnest或者Perfectloopnest。看下面的处理办法。

 3. 对于Imperfectloopnest的话，貌似发现中间层优化效果的性价比最高

### for循环pipeline时的rewind选项

没有选择rewind时，在执行完一次for循环后有一个时钟周期的空挡然后才执行下一次循环。使用rewind之后，两次for循环之间是没有空挡的，这样就降低了整个函数的latency。

<div align=center>
<img src=".\\hls5\\9.png" width = 400 >
 勾上就对了<br>
<br><div align=left>


### for循环的循环边界是变量怎么办？

当循环边界是变量时会引发一些问题。首先VivadoHLS无法确定looplatency是多少，进而就无法确定函数的latency，此时相应的latency会用问号作为标记。这时可以：

1. 用Tripcount指令
2. 将循环边界的数据类型声明为ap_int<W>（也可以使用ap_uint<W>，但是当循环变量是i—的时候会出问题，当减到负数后仍然会被识别为正数）
3. 在C代码中可以使用assert宏



## FOR循环优化示例

1. UG871的69页第六章

2. https://github.com/Zaoldyeckk/High-Level-Synthesis-Flow-on-Zynq-using-Vivado-HLS/blob/master/Lab3.md





























