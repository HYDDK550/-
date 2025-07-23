---
title: HLS学习2：高效数据类型与任意精度类型
created: '2025-07-21T16:22:25.056Z'
modified: '2025-07-23T16:52:06.514Z'
---

# # HLS学习2：高效数据类型与任意精度类型

* 本笔记致力于解释Vivado HLS中的数据类型
* 参考示例为Xilinix UG902 与 UG871 文档

## C语言中的数据类型对HLS综合有影响吗？

**C语言的数据类型固定为8位边界：**
char(8 位)
短(16 位)
int(32 位)
long long(64 位)
浮动(32 位)
双(64 位）
精确宽度整数类型，如 int16_t(16 位)和 int32_t(32 位)

**但在创建硬件时，通常需要更精确的位宽。** 例如，考虑这样一种情况：过滤器的输入是 12 位的，而结果的累积只需要 27 位的最大范围。在硬件设计中使用标准 C 数据类型会导致不必要的硬件成本。操作可以使用比所需精度更多的 LUTs 和寄存器，延迟甚至可能超过时钟周期，需要更多的周期来计利结果。Vivado 高级综合(HLS)提供了许多位精度或任意精度的数据类型，允许您使用任意宽度对变量进行建模。

**使用任意精度数据类型可以以更少的资源以更快的速度计算并且仍然保持相同的精度。**

* UG902第60页提供了一个很好的硬件友好数据类型的例子

## 任意精度整数数据类型

Vivado® HLS 为 C 和 C++ 提供了整数和定点任意精度数据类型，支持 System C 的任意精度数据类型参考 ug902 的表 1-7。

<div align=center>
<img src=".\\hls2\\1.png" width = 800 >
表：任意精度数据类型<br>
<div align=left>

前面有 u 表示无符号数(C 语言示例如：uint<3>)，[]表示为可选（没有 u 则为有符号数，
C++示例如 ap_int<3>），<W>表示位宽，后面括号里的数字表示 W 的取值范围。C++还引
入了定点数 ap_[u]fixed<W,I,Q,O,N>

* 以下示例显示了如何添加头文件并实现 2 个变量来使用 9 位整数和 10 位无符号的整数类型: 
```
#include "ap_int.h"         //将头文件 ap_cint.h 添加到源代码
void foo_top () {
ap_int<9> var1;             // 9-bit,C语言是intN 或 uintN，其中 N 介于 1 到 1024 
ap_uint<10> var2;           // 10-bit unsigned
```
如果N大于1024，参考UG902的63页

* 通常声明任意精度数据类型时应当在头文件中用typedef定义，便于调试和修改，示例如下：

```
#include "ap_int.h"

#define W 18
#define __NO_SYNTH__
#ifdef __NO_SYNTH__
typedef int dawta_t;
typedef int prod_t;
#else
typedef ap_int<W> data_t;
typedef ap_int<2*W> prod_t;
#endif

prod_t ScalarMult(data_t A,data_t B);
```

## 任意精度定点数据类型
定点数据类型将数据作为整数位和小数位形式进行建模。

```
#include <ap_fixed.h>
...
ap_fixed<18,6,AP_RND > my_type;
...
```
上面的例子中Vivado HLS ap_fixed 类型用于定义 18 位变量，
其中 6 位用于表示二进制小数点前的数值，12 位用于表示小数点后的值。
AP_RND是类型标识，UG902的表8中提供了 ap_fixed 类型的标识的汇总。

## sizeof()函数的使用

sizeof()函数在 Vivado HLS 仍然可用。其部分示例及输出如下图所示：
<div align=center>
<img src=".\\hls2\\1.png" width = 800 >
sizeof()对任意精度数据类型的输出<br>
<div align=left>

由结果可以看出，对于任意精度数据类型，sizeof()输出的是向上取整的字节数，并且向
最接近的 2 的整数次幂取值。

## 设置Visual Studio支持任意精度数据类型

另外在 visual studio 中也可以通过设置设 MSVC 编译器支持任意精度数据类型。具体步
骤为：右击项目名称，打开属性，在 C/C++的 General 设置项目中将 Additional include
Directories 选择为..\Vivado_HLS\2018.2\include 即可。
























































































