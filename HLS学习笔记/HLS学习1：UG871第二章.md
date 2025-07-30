---
deleted: true
title: Untitled
created: '2025-07-16T09:56:36.106Z'
modified: '2025-07-21T08:53:52.018Z'
---

<style>

/* 标题格式 */
h1, h2, h3, h4, h5, h6  
{
    text-align: justify;    /* 两端对齐 */
    font-weight: bold;      /* 加粗 */
    color: black;       /* 黑色 */
    font-family:        /* 中文为微软雅黑 英文为新罗马字体 */
                "微软雅黑" 
                "Times New Roman", 
                serif;  
}

/* 大标题居中 */
h1 
{
    text-align: center; 
}

/* 网页、段落、列表默认全局设置  */
body, p, li
{
    font-family:        /* 中文为微软雅黑 英文为新罗马字体 */
                "微软雅黑" 
                "Times New Roman", 
                serif;  
    color: black;   /* 黑色 */
    text-align: justify;    /* 两端对齐 */
}



pre, code 
{
    background-color: #f5f5f5; /* 设置代码块背景色 */
    padding: 10px; /* 添加内边距 */
    border: 1px solid #ccc; /* 设置边框 */
    border-radius: 5px; /* 可选：设置圆角 */
    white-space: pre-wrap; /* 自动换行 */
    word-wrap: break-word; /* 长单词换行 */
    page-break-inside: auto; /* 允许在代码块内分页 */
}

 figure 
{
    text-align: center; /* 全局定义 figure 的居中对齐 */
    margin: 20px 0; /* 添加上下间距 */
}

figcaption 
{
    font-family:        /* 中文为微软雅黑 英文为新罗马字体 */
                "微软雅黑" 
                "Times New Roman", 
                serif;
    color: black; /* 字体颜色 */
    font-size: 14px; /* 字体大小 */
    margin-top: 5px; /* 与图片的间距 */
}

@media print 
{
    body 
    {
        margin: 0 auto;
        width: 100%; /* 确保内容占满打印页面宽度 */
        text-align: center; /* 全局内容居中 */
        background-color: #fff;/* 背景色为白色 */
    }
    figure 
    {
        display: block;
        margin: 0 auto; /* 确保 <figure> 标签在打印时居中 */
        text-align: center;
    }
    img 
    {
        display: block;
        margin: 0 auto; /* 确保图片本身居中 */
    }
    figcaption 
    {
        text-align: center;
        font-size: 14px; /* 保持注释大小一致 */
    }
    pre, code 
    {
        background-color: #f5f5f5 !important; /* 使用 !important 强制覆盖 */
        color: #333; /* 设置文本颜色 */
        padding: 10px;
        border: 1px solid #ccc;
        border-radius: 5px;
        page-break-inside: auto !important; /* 代码块在打印时可以被拆分 */
        white-space: pre-wrap !important; /* 打印时自动换行 */
        word-wrap: break-word !important; /* 防止单行代码溢出 */
    }
}

</style>

<script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/3.2.0/es5/tex-mml-chtml.js"></script>
# HLS学习1：HLS简介与基本操作

## 什么是HLS？

* 赛灵思 Vivado® 高层次综合 (HLS) 工具将 C 语言规格转换为寄存器传输级 (RTL) 实现，供您综合到赛灵思现场可编程门阵列 (FPGA) 中。
您可使用 C、C++ 或 SystemC 来编写 C 语言规格，FPGA 可提供大规模并行处理架构，其性能、成本和功耗都比传统处理器更胜一筹。

* 借助HLS可以开展的工作
1.在 C 语言层次开发算法，在 C 语言层次执行验证以相比于传统硬件描述语言更快的速度验证设计的功能正确性。
2.通过最优化指令来控制 C 语言综合进程，创建特定的高性能硬件实现。
3.使用最优化指令从 C 语言源代码创建多种实现，浏览设计空间，提升找到最优化实现的可能性。
4.创建可读且可移植的 C 语言源代码，将 C 语言源代码目标调整为其它器件，并将 C 语言源代码整合到新工程中。


## 1.HLS流程实例：UG871 Introduction lab1

### 1.1下载Vivado2018.3安装链接：vivado2018.3
链接: https://pan.baidu.com/s/1ll350ozv0aH1VzBANt2ISw 提取码: pgsw 

### 1.2打开工程

#### HLS提供两种打开/创建工程的方式，使用图形GUI打开或使用命令行创建 Vivado HLS 工程，使用到的示例文件如下：
https://github.com/Zaoldyeckk/High-Level-Synthesis-Flow-on-Zynq-using-Vivado-HLS
该实例仓库包含四个简单的HLS lab工程

* GUI方法：安装Vivado开发套件后，直接在桌面可以找到Vivado HLS的图标，打开后如下图2所示，点击图标打开HLS

<div align=center><img src=".\\hls1\\1.png" width = 500 >
<div align=center><img src=".\\hls1\\2.png" width = 500 >

* TCL命令方法：
1.进入Vivado HLS 2018.2 Command Prompt（直接打开或cd到bin目录）
2.打开cmd，进入bin目录，输入vivado_hls_cmd.bat，启动命令模式
3.进入你自己的hls文件目录，输入vivado_hls –f run_hls.tcl ，创建项目
4.输入vivado_hls -p dct_prj，自动开启GUI，打开项目

### 1.4 验证C源代码
1.在source源文件中打开fir.c即可查看需要综合的C代码
```
#include "fir.h"

void fir (
  data_t *y,
  coef_t c[N],
  data_t x
  ) {

  static data_t shift_reg[N];
  acc_t acc;
  data_t data;
  int i;
  
  acc=0;
  Shift_Accum_Loop: for (i=N-1;i>=0;i--) {         //Shift_Accum_Loop:标签Loop
	if (i==0) {
			shift_reg[0]=x;
     	data = x;
    } else {
			shift_reg[i]=shift_reg[i-1];
			data = shift_reg[i];
    }
    acc+=data*c[i];;       
  }
  *y=acc;
}

```
2.在Testbench中打开fir_test.c，点击Project/C Simulation进行C的仿真，可以勾选debugger

<div align=center><img src=".\\hls1\\3.png" width = 1000 >

Vivado HLS 工具可以重用 C 测试平台来执行 RTL 的验证。
如果测试台具有前面描述的自检查特性，则在 RTL 验证期间将自动检查 RTL 结果。
Vivado HLS 在 RTL 验证期间重用测试工作台，并在测试工作台返回 0 值时确认 RTL 验证成功。 
如果 main() 返回任何其他值，包括无返回值，则表示 RTL 验证失败。无需创建 RTL 测试台。

### 1.5启用高阶合成并查看合成报告
单击 " C synthese "工具栏按钮或使用菜单 "解决方案 > 运行 C 合成
合成完成后，报告文件会自动打开。因为合成报告在 "信息 "窗格中打开时，辅助窗格中的 "大纲 "选项卡会自动更新以反映报告信息。



**报告卡中的信息详解如下：**

<div align=left><img src=".\\hls1\\4.png" width = 1200 >

* 在如上图所示的Performance Estimates窗格中，您可以看到时钟周期被设置为 10 ns。
Vivado  HLS 的目标时钟周期为时钟目标减去时钟不确定性(在本例中为 10.00-1.25 = 8.75  ns)。
则该实例的最大估计的时钟周期(最坏情况下的延迟)为5.772 ns，满足8.75 ns 的时序要求。


<div align=left><img src=".\\hls1\\5.png" width = 1000 >

* 总的Latency为34，代表34个时钟循环后才能进行下一组数据的读取。
* 展开Detail可以看到instance中没有子块，这表明所有的延时均由循环Shift_Accum_Loop.引起。该逻辑执行 11 次（跳闸次数）。
每次执行需要 3 个时钟周期（迭代延迟），总共 33 个时钟周期才能执行由该循环合成的逻辑的所有迭代（延迟）。
* 总循环为34的原因是要有一个时钟周期来进入和退出循环（在这种情况下，设计在循环结束时完成，因此没有退出周期）。

<div align=left><img src=".\\hls1\\6.png" width = 500 >

* 上图是该示例的利用率估计
该设计使用了作为 LUTRAM 实现的单个存储器（因为它包含的元素少于 1024 个）、3 个 DSP48 以及约 200 个触发器和 LUT。在此阶段，设备资源数量为估算值。
资源利用率数字是估计值，因为 RTL 合成可能会执行额外的优化，这些数字在 RTL 合成后可能会发生变化。


<div align=left><img src=".\\hls1\\7.png" width = 500 >


* 接口部分显示接口合成创建的端口和 I/O 协议：

设计有一个时钟和复位端口（ap_clk 和 ap_reset）。这两个端口与设计本身相关联。
如源对象 fir 所示，设计还关联了其他端口。合成自动添加了一些块级控制端口：ap_start、ap_done、ap_idle 和 ap_ready。
函数输出 y 现在是一个 32 位数据端口，并带有相关的输出有效信号指示器 y_ap_vld。
函数输入参数 c（一个数组）是作为一个块 RAM 接口实现的，它有一个 4 位输出地址端口、一个输出 CE 端口和一个 32 位输入数据端口。
最后，标量输入参数 x 是作为无 I/O 协议的数据端口 (ap_none) 来实现的。
UG871教程稍后将介绍如何优化端口 x 的 I/O 协议：实验 3：使用解决方案优化设计 将介绍如何优化端口 x 的 I/O 协议。


### 1.6RTL验证
点击“Solution/Run C/RTL Simulation”运行C和RTL联合仿真，RTL 协同仿真的默认选项是使用 Vivado 仿真器和 Verilog RTL 执行仿真。
要使用不同的仿真器或语言使用 C/RTL 协同模拟对话框中的选项。
<div align=left><img src=".\\hls1\\8.png" width = 500 >

C 测试台为 RTL 设计生成输入向量。
对 RTL 设计进行仿真。
RTL 的输出向量被应用回 C 测试台，而测试台的结果检查则验证结果是否正确。
如果测试台返回值为 0，则 Vivado HLS 表示仿真通过。
出现上图的信息则代表模拟成功。重要的是，只有当结果正确时，测试台才会返回 0 值。


### 1.7 IP输出
* HLS流程的最后一步是将设计打包为 IP 块，以便与 Vivado Design Suite 中的其他工具配合使用。
单击导出 RTL 工具栏按钮或使用菜单解决方案 > 导出 RTL。
展开 ip 文件夹，找到打包为 zip 文件的 IP，准备添加到 Vivado IP 目录

<div align=left><img src=".\\hls1\\9.png" width = 500 >
* 如果遇到如上图中的问题可以参考这个网址：https://adaptivesupport.amd.com/s/article/76960?language=en_US

## 2 Introduction lab2：使用TCL指令

### 2.1 TCL文件所在地：

* 在lab1中展开 solution1 中的 Constraints 文件夹，文件 script.tcl 包含 Tcl 命令，用于使用项目设置时指定的文件创建项目，并运行 HLS 流程的所有阶段。
* directives.tcl 文件包含应用于设计方案的任何优化。实验 1 中没有使用优化指令，因此该文件为空。


### 2.2 Vivado HLS 命令提示符

* 进入Vivado HLS 命令提示符，cd进lab2
<div align=left><img src=".\\hls1\\11.png" width = 500 >
* 使用任何文本编辑器，对 lab2 中的 run_hls.tcl 文件进行以下编辑

```
#Reset the project with the -rset option
open_project -reset fir_prj
set_top fir
add_files fir.c
add_files -tb fir_test.c
add_files -tb out.gold.dat

#Reset the project with the -rset option
open_solution -reset "solution1"
set_part {xcvu9p-flgb2104-1-e}
create_clock -period 10 -name default

#Configure default options
config_compile -no_signed_zeros=0 -unsafe_math_optimizations=0
config_schedule -effort medium -enable_dsp_full_reg=0 -relax_ii_for_timing=0
config_bind -effort medium
config_export -format ip_catalog -rtl verilog


#Comment out previous solutions directives
#source "./fir_prj/solution1/directives.tcl"

csim_design
csynth_design
cosim_design
export_design -format ip_catalog

#Exit vivado HLS
exit
```


1.在 open_project 命令中添加 -reset 选项。由于您通常会在同一项目上重复运行 Tcl 文件，因此最好覆盖任何现有的项目信息。
2.在 open_solution 命令中添加 -reset 选项。当在同一解决方案上重新运行 Tcl 文件时，该选项会删除任何现有的解决方案信息。
3.保留源代码指令的注释。如果前一个项目中有任何指令需要重复使用，可以直接将指令复制到此文件中。
4.在 Tcl 文件的最后一行添加退出命令。
5.保存并退出。

### 2.3查看结果：
Vivado HLS 会执行 lab1 中的所有步骤。完成后，结果将显示在项目目录 fir_prj 中。

* 综合报告可在 fir_prj\solution1\syn\report 中查阅。
* 模拟结果可在 fir_prj\solution\sim\report 中查阅。
* 输出软件包在 fir_prj\solution1\impl\ip 中提供。
* 最终输出的 RTL 可在 fir_prj\solution1\impl 和 Verilog 或VHDL.
**注意！从 Vivado HLS 项目复制 RTL 结果时，必须使用 impl 目录中的 RTL。Vivado HLS 会在 export_design 过程中执行额外处理，然后才能在其他设计工具中使用此 RTL。**


## 3 Introduction lab3：使用解决方案优化设计

3.1 进行优化的第一步是确保所有端口I/O协议正确，从solution1的报告中可以知道：

* 端口 C 必须有一个单端口 RAM 访问。
* 端口 X 必须有一个输入数据有效信号。 
* 端口 Y 必须有一个输出数据有效信号。

下面新建一个solution来为端口指定I/O接口

* 选择C参数 Insert Directive》RESOURCE》core》选择RAM_1P_BRAM
因为 I/O 协议不太可能更改，所以可以将这些优化指令作为实用程序添加到源代码中， 以确保在设计中嵌入了正确的 I/O 协议。
在指令编辑器(Directive Editor)的 DDDestination Destination 部分，选择源文件(Source File
* 选择X参数 Insert Directive》Interface》Destination”部分中选择“Source File，mode为ap_vld
* 选择Y参数，操作同X参数
     
3.2 Analysis视角与任意精度数据

* 第六章，DDDesign Analysis Design Analysis 教程提供了对分析视角的更全面的理解。
* 源代码使用int 数据类型。这是一个 32 位的数据类型，一个 DSP48 乘法器是 18 位的，需要多个DSP48实现乘法
任意精度类型教程展示了如何使用更适合硬件的数据类型创建设计。
使用任意精度类型可 以定义任意位大小的数据类型(超过标准的 C/ c++ 8-、16-、32-或 64 位类型)。


3.3 展开循环与矩阵分块

* 为循环shift_AAAccum Accum_loop插入指令Unroll
* 为数组shift_reg插入指令Array_Partition，指定类型为 complete。






* 可以对这种设计执行额外的优化。例如，您可以使用管道( 管道管道管道(((pipeling))))来进一步提高吞吐量
(throughput)并降低间隔(interval)。
在第七章“设计优化教程”中，详细介绍了如何使用管道来改进时间间隔。
如前所述，您可以修改代码本身以使用任意精度类型。例如，如果不要求数据类型为 32 
位 int 类型，则可以使用任意精确类型(例如，6 位、14 位或 22 位类型)，前提是它们满足
所 需的精度。有关使用任意精度类型的详细信息，请参阅第 5 章“任意精度类型教程”。






