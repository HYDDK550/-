---
title: HLS学习7：函数级优化与全局优化
created: '2025-07-28T16:01:50.623Z'
modified: '2025-07-30T13:40:02.828Z'
---

# # HLS学习7：函数级优化与全局优化

## 函数优化的指令

### 1.Inline

对函数的 inline 就是去除了函数的层次化，可以通过 INLINE 这个 directive 来实现。对
函数inlining的好处是可以改善资源消耗，因为inline之后就不再需要调用函数的相关逻辑。
对于一些小函数 Vivado HLS 会自动进行 inlining 处理，可以改善相应的 QoR（quality of
results）。如果不希望对某个函数自动 inline 的话可以通过 INLINE 这个 directive 中的-off 选项实现。自动 inline 时在综合时的信息输出窗口中可以看到相应的输出信息。 Inlinin 之后
在综合后的 RTL 代码中相应的函数结构就没有了。

### Allocation

Allocation 实际上是定义了函数和相应的 RTL module 之间的关系，也就是定义了在 RTL
代码中实现某个具体函数或者操作的实例数量。这个功能可以通过 ALLOCATION 这个
directive 来实现。

#### Allocation限制运算符数量

```
dout_t array_arith (dio_t d[317]) {
 static int acc;
 int i;
#pragma HLS ALLOCATION instances=mul limit=256 operation  for (i=0;i<317;i++) {
#pragma HLS UNROLL
 acc += acc * d[i];
 }
 rerun acc;
}
```
* 在上面的代码中包含 317 次乘法，但FPGA 仅有 256 项乘法器资源 (DSP48),ALLOCATION指令可指示 Vivado HLS 创建含最多 256 个乘法 (mul)运算符的设计。
* 因为Vivado HLS 的默认操作是首先最大限度提升性能。限制设计中的运算符数量是一项减小面积的实用技巧：它通过强制共享运算来减小面积。
**关于Vivado HLS 的所有运算符可以在UG902的135页的表14中查阅**

* 你可以使用指令全局最大限度减少运算符

#### Allocation限制硬件核

执行综合时，Vivado HLS 会使用由时钟指定的时序约束、由目标器件指定的延迟以及由您指定的任意指令来判定使用哪个核来实现运算符。例如，要实现乘法运算，Vivado HLS 可使用组合乘法器核或使用流水线乘法器核。
综合期间映射到运算符的核可采用与运算符相同的方式来加以限制。您无需限制乘法运算总数，而可改为选择限制组合乘法器核的数量以强制使用流水线化乘法器来执行所有剩余乘法（或反之亦然）。这是通过将 ALLOCATION 指令type选项指定为core来实现的。

```
int foo (int a, int b) {
 int c, d;
#pragma HLS RESOURCE variable=c latency=2  c = a*b;
 d = a*c;
 return d;
}
```
RESOURCE 指令用于显式指定要用于特定操作的核。在上面的示例中指定使用 2 阶流水线化乘法器以实现变量的乘法运算。命令会告知 Vivado HLS 针对变量c使用 2 阶流水线化乘法器。由 Vivado HLS 判定用于变量d的核。

```
void apint_arith(dinA_t  inA, dinB_t  inB,
          dout1_t *out1
  ) {
 dout2_t temp;
#pragma HLS RESOURCE variable=temp core=AddSub_DSP  temp = inB + inA;
 *out1 = temp;
}
```
上面的代码展示了RESOURCE 指令指定变量 temp 的加法运算，并使用AddSub_DSP核来实现。这样可确保在最终设计中使用 DSP48 原语来实现此运算 - 默认情况下加法运算是使用 LUT 来实现的。

* list_core命令用于获取有关库中可用的核的详细信息。list_core只能在 Tcl 命令界面中使用，并且必须使用set_part命令指定器件。如果未选中器件，此命令将无效。
* list_core命令的-operation选项列出了库中可通过指定运算实现的所有核。
**UG902的137页到140页列出了所有实现逻辑运算，浮点，存储的RTL核**


### FUNCTION_INSTANTIATE

函数例化是一种最优化技巧，不仅具有维持函数层级的面积优势，还可提供另一个强大的选项：在函数的特定实例上执行针对性局部最优化。这样可以简化围绕函数调用的控制逻辑，也可能改进时延和吞吐量。鉴于调用函数时部分函数输入可能是常量，FUNCTION_INSTANTIATE 指令可藉此简化周围控制结构，并生成进一步优化的、更小的函数块。观察下面的代码：

```
void foo_sub(bool mode){
#pragma HLS FUNCTION_INSTANTIATE variable=mode
if (mode) {
     // code segment 1 
  } else {
     // code segment 2
  }
}
void foo(){  
#pragma HLS FUNCTION_INSTANTIATE variable=select foo_sub(true);
foo_sub(false);
}
```
FUNCTION_INSTANTIATE 最优化允许对每个实例进行独立最优化，从而减少功能和面积。完成FUNCTION_INSTANTIATE 最优化后，以上代码可有效转换为 2 个独立函数，每个函数都针对模式的不同可能值来完成最优化，如下所示：

```
void foo_sub1() {
  // code segment 1 }
void foo_sub1() {
  // code segment 2 }
void A(){
  B1();
  B2();
}
```

### -latency控制运算符流水线化

Vivado HLS 会自动判定用于内部运算的流水线化级别。您可将 RESOURCE 指令与-latency选项配合使用，以显式指定流水线阶段的数量，并覆盖由 Vivado HLS 判定的数量。
RTL 综合可使用多个额外流水线寄存器来帮助改善布局布线后可能导致的时序问题。在运算输出中添加寄存器通常有助于改善输出数据路径中的时序。在运算输入中添加寄存器通常有助于改善输入数据路径和来自 FSM 的控制逻辑中的时序。

可使用config_core配置对设计中特定核的具有相同流水线深度的所有实例进行流水线化。要设置此配置，请执行以下操作：
1.选择“Solutons” → “SolutonSetngs”。
2.在“解决方案设置(SolutonSetngs)”对话框中，选择“General”类别，然后单击“Add”。
3.在“添加命令 (Add Command)”对话框中，选择config_core命令，并指定参数。

```
//指定 DSP48 核实现的运算均流水线化，时延设置为 3，这是该核允许的最大时延
config_core DSP48 -latency 3

//RAM_1P_BRAM 核实现的所有块 RAM 均采用流水线化，且时延设置为 3
config_core RAM_1P_BRAM -latency 3

```




