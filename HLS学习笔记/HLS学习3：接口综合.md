---
title: HLS学习3：接口综合
created: '2025-07-21T08:31:05.716Z'
modified: '2025-07-23T06:03:33.455Z'
---

# HLS学习3：接口综合

* 本笔记致力于解释Vivado HLS中关于端口的类型与定义
* 参考示例为Xilinix UG902 与 UG871 文档

## 1.端口类型
当C函数被综合的时候，接口产生被综合成端口；对顶层函数进行综合时，函数的实参（或参数）将综合到 RTL 端口中。
此流程称为“接口综合 (interface synthesis)”。

```
#include "sum_io.h"
dout_t sum_io(din_t in1, din_t in2, dio_t *sum) {
 dout_t temp;
 *sum = in1 + in2 + *sum;
 temp = in1 + in2;
 return temp;
}
```

对以上的C代码进行综合后的硬件模块如下图所示
<div align=left><img src=".\\hls3\\1.png" width = 500 >

### 1.1生成的模块含有以下三种接口：（UG902的66页有详细的解释）
* 1.时钟和复位端口：ap_clk 和 ap_rst。
* 2.块级接口协议。在前图中已显示并展开这些端口：ap_start、ap_done、ap_ready 和 ap_idle。
* 3.端口级接口协议。这些端口是针对顶层函数和函数返回（如果函数返回值）中的每个实参创建的。在此示例中，这
些端口包括：in1、in2、sum_i、sum_o、sum_o_ap_vld 和 ap_return。


### 1.2可指定接口协议

下图是每一种 C 语言实参类型上指定的接口协议模式。此图使用以下首字母缩略词：
• D：每一种类型的默认接口模式。如果指定非法接口，Vivado HLS 会发出 1 条消息，并实现默认接口模式。
• I：输入实参（只读）。
• O：输出实参（只写）。
• I/O：输入/输出实参（可读写）。

<div align=left><img src=".\\hls3\\2.png" width = 500 >


### 1.3模块级接口协议（详见UG902的383页）
块级接口协议包括 ap_ctrl_none、ap_ctrl_hs 和 ap_ctrl_chain。这些协议在且只能在函数或函数返回时指定。
在 GUI 中指定该指令时，会将这些协议应用于函数返回。即使函数不使用返回值，也可在函数返回时指定块级协议。

* ap_ctrl_hs主要用于模块级的握手信号，即需要开始操作，结束操作并返回一个值；ap_ctrl_hs 模式是默认协议。
* ap_ctrl_chain 协议类似于 ap_ctrl_hs，但具有额外的输入端口 ap_continue 以提供来自使用此块数据的其它块的回压。
如果函数完成时 ap_continue 端口为逻辑 0，此块将停止操作，并且不会继续执行下一项传输事务。
仅当 ap_continue 断言为逻辑 1 时，才会继续执行下一项传输事务。
* ap_ctrl_none 模式用于实现不含任何块级 I/O 协议的设计。

### 1.4端口级接口协议（详见UG902的386页）

* **AXI4 接口**
Vivado HLS 支持的 AXI4 接口包括 AXI4-Stream 接口 (axis)、AXI4-Lite 接口 (s_axilite) 和 AXI4 主接口 (m_axi)，
这些接口可按以下方式指定：
• AXI4-Stream 接口：仅在输入实参或输出实参上指定，而不在输入/输出实参上指定。
• AXI4-Lite 接口，在任何类型的实参上指定，但是流传输除外。您可以将多个实参分组到同一 AXI4-Lite 接口中。
• AXI4 主接口：仅在数组和指针（以及 C++ 中的引用）上指定。您可以将多个实参分组到同一 AXI4 接口中。

* **无 I/O 协议**
ap_none 和 ap_stable 模式可指定不向端口添加任何 I/O 协议。指定这些模式时，实参作为不含任何其它关联信号
的数据端口来实现。ap_none 模式是标量输入的默认模式。ap_stable 模式用于仅当器件处于复位模式时才可更改
的配置输入。

* **有线握手**
接口模式 ap_hs 包含与数据端口的双向握手信号。此握手属于业界标准的有效和确认握手。ap_vld 模式同样如此，
但仅含有效端口，ap_ack 仅含确认端口。
ap_ovld 模式用于输入输出实参。将输入输出拆分为独立输入端口和输出端口时，ap_none 模式适用于输入端口，
ap_vld 适用于输出端口。这是支持读写的指针实参的默认行为。
ap_hs 模式可应用于按顺序读写的数组。如果 Vivado HLS 可判定读访问或写访问为无序访问，它将停止综合并报错。
如果无法判定访问顺序，Vivado HLS 将发出警告。

* **内存接口**
默认情况下，数组实参作为 ap_memory 接口来实现。这是含数据、地址、芯片使能和写使能端口的标准块 RAM 接
口。
ap_memory 接口可作为单端口接口或双端口接口来实现。如果 Vivado HLS 可判定使用双端口接口将缩短启动时间间
隔，那么它将自动实现双端口接口。RESOURCE 指令用于指定内存资源，如果在含单端口块 RAM 的数组上指定该指
令，那么将实现单端口接口。相反，如果使用 RESOURCE 指令指定双端口接口，并且 Vivado HLS 判定此接口并无益
处，那么它将自动实现单端口接口。
bram 接口模式的运作方式与 ap_memory 接口相同。唯一差异是在 Vivado IP integrator 中使用设计时，端口的实现方
式。
• ap_memory 接口显示为多个独立端口。
• bram 接口显示为单个组合端口，可使用单一点对点连接来连接到赛灵思块 RAM。
如果按顺序访问数组，可使用 ap_fifo 接口。就像 ap_hs 接口一样，如果 Vivado HLS 判定未按顺序进行数据访问，
那么它将停止；如果无法判定是否采用顺序访问，则将发出警告；如果判定已采用顺序方式访问，则不发出任何消息。
ap_fifo 接口只能用于读取或写入，不能用于同时读写。
ap_bus 接口可与总线网桥进行通信。此接口不遵循任何特定总线标准，但鉴于其泛用性，可配合总线网桥一起使用，
从而与系统总线进行仲裁。总线网桥必须能够将所有突发写操作存入高速缓存。




## 2.块级端口指定示例

### 2.1打开示例工程

* 从Vivado HLS 2018.3 Command Prompt 进入UG871示例的接口综合lab1文件夹
* 输入vivado_hls -f run_hls.tcl 执行 Tcl 脚本来设置 Vivado HLS 项目
* 输入vivado_hls -p adders_prj 在 Vivado HLS GUI 中打开项目。

```
#include "adders.h"

int adders(int in1, int in2, int in3) {
#pragma HLS INTERFACE ap_ctrl_none port=return

// Prevent IO protocols on all input ports
#pragma HLS INTERFACE ap_none port=in3
#pragma HLS INTERFACE ap_none port=in2
#pragma HLS INTERFACE ap_none port=in1      


	int sum;
	
	sum = in1 + in2 + in3;
	
	return sum;

}
```
观察C源代码，函数有返回值且阻止了端口协议的合成

### 2.2 检查端口

* 进行综合后请滚动到综合报告末尾的 Interface Summary。

<div align=left><img src=".\\hls3\\3.png" width = 500 >

可以看到默认的块级端口协议是ap_ctrl_none

### 2.3 修改块级端口协议

* 新建一个solution，在Directive里选择adder
* Insert Directive > INTERFACE > mode > ap_ctrl_hs(区块级握手协议)
* 有关四个块级接口协议的详细情况可以查看UG902的383页
<div align=center><img src=".\\hls3\\4.png" width = 300 >
修改块级端口协议

<div align=center><img src=".\\hls3\\5.png" width = 500 >
修改后的端口综合报告

<div align=left>

## 3.端口IO协议

###　3.1打开项目

* cd到UG871的接口综合lab2
* vivado_hls -f run_hls.tcl创建项目
* vivado_hls -p adders_io_prj打开项目
* **观察项目的C源代码,没有函数返回,而是通过指针参数传递函数输出**

```
#include "adders_io.h"

void adders_io(int in1, int in2, int *in_out1) {

	*in_out1 = in1 + in2 + *in_out1;
	
}
```



### 3.2指定端口的IO协议
* 指定IN1的INTERFACE > mode为ap_vld
* IN2为ap_ack
* In_out1为ap_hs

<div align=center><img src=".\\hls3\\6.png" width = 500 >
指定端口后的接口综合报告
<div align=left>

### 3.3解读报告
* IN1被综合成两个端口，一个数据端口和一个使能端。只有当in1_ap_vld为高才能被读出
* IN2被综合成两个端口，一个数据端口和一个应答端。当IN2被读取时in2_ap_ack会为高
* INOUT1被zoo被综合为两个部分。

## 4.将数组实现为RTL接口

### 4.1打开项目

* cd到UG871的接口综合lab3
* vivado_hls -f run_hls.tcl创建项目
* vivado_hls -p array_io_prj打开项目
* 查看C语言源代码
* 进行C综合
```
#include "array_io.h"
void array_io (dout_t d_o[N], din_t d_i[N]) {
	int i, rem;
	
	// Store accumulated data
	static dacc_t acc[CHANNELS];
	dacc_t temp;

	// Accumulate each channel
	For_Loop: for (i=0;i<N;i++) {
		rem=i%CHANNELS;
		temp = acc[rem] + d_i[i];
		acc[rem] = temp;
		d_o[i] = acc[rem];
	}
}
```
 <div align=center>
C语言代码
<img src=".\\hls3\\7.png" width = 500 >
接口综合报告
<div align=left>


* d_o数组已经被总和为RAM端口（ap_memory）,它包括：
1.一个数据端口(d_o_d0)。
2.一个地址端口(d_o_address0)。
3.控制芯片启用端口(d_o_ce0)和写启用端口(do_we0)。

* d_i已经综合到类似的RAM接口，但是具有输入数据端口（d_i_q0），并且没有写使能端口，因为该接口仅读取数据。

### 4.2 使用双端口的RAM和FIFO

* 新建solution2，展开For_Loop(UNROLL)
* 在Directive选项卡中选择portd_i，在RESOURCE中指定core为RAM_2P_BRAM
* 在Directive选项卡中选择端口d_o，Interface》Mode》ap_fifo
 <div align=center>
<img src=".\\hls3\\8.png" width = 500 >
选项卡中的指令
<img src=".\\hls3\\9.png" width = 500 >
综合后的端口报告
<div align=left>
通过使用双端口RAM该设计可以以以前设计两倍的速率接受输入数据。因为for循环已展开，所以循环中的逻辑能够以此速率使用数据。默认情况下，每个循环迭代依次执行。此实现代码将逻辑限制为在每次迭代中对d_i进行一次读取。展开循环可以执行更多读取（但会创建N个逻辑副本）但是，在输出上使用单端口FIFO接口输出速率与之前相同。

### 4.3分区RAM与FIFO接口

* 新建solution3
* 在Directive选项卡中，选择d_o，Insert Directive
* 并选择 ARRAY_PARTITION，设置类型为block，factor为4
* 选择d_i并重复前面的步骤，但设置factor为2
* 进行综合并查看报告

<div align=center>
<img src=".\\hls3\\10.png" width = 500 >
综合后的端口报告
<div align=left>

### 4.4 完全分区数组接口

* 新建solution4
* 在 Directive 选项卡中，选择现有d_o的 partition Directive并选择Modify指令。
* 将type选择为complete，删除factor为4的指令
* 重复前面的步骤来完全划分数组d_i，删除RESOURCE Directive
* 查看综合报告

### 4.5 解决方法对比

<div align=center>
<img src=".\\hls3\\11.png" width = 500 >
<img src=".\\hls3\\12.png" width = 500 >
<div align=left>

* 具有更多I/O 端口的解决方案(解决方案2、3 和4)允许更多的并行处理，但也会使用更多的资源。


## 5.AXI4总线接口
















































