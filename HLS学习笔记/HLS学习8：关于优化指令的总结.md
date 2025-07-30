---
title: HLS学习8：关于优化指令的总结
created: '2025-07-30T13:42:31.734Z'
modified: '2025-07-30T13:52:31.465Z'
---

# HLS学习8：关于优化指令的总结

## 改善吞吐率的指令
常用于改善吞吐率的 directive 有 PIPELINE、ARRAY_PARTITION、UNROLL、DATAFLOW
等。

<div align=center>
<img src=".\\hls8\\1.png" width = 500 >
  改善吞吐率的Directives<br>
<br><div align=left>

* PIPELINE 可以作用于函数和循环，当作用于循环是有个 option，叫做 rewind，该选项
可以进一步改善吞吐率。
* 当 PIPELINE 作用于函数的时候是连续的，从 IO 的角度来看也是连续的。当 PIPELINE 作用于循环的时候，在两次循环之间是有一个空挡的，从 IO 的角度来看有一个 Bubble。这是因为需要一个bubble来进入循环
* 对于数组可以使用 ARRAY_PARTITION 来把数组分割成不同的部分，有 3 种分割方式。
* 对于循环还可以采用 UNROLL 来优化，这时有个选项 factor 用于控制循环被复制成几
份来并行执行。
* DATAFLOW 可以作用于函数和循环，是一种 ping-pong 操作的方式。

## 改善时延的指令
常用于改善时延的directives有LATENCY、LOOP_MERGE、LOOP_FLATTEN等。LATENCY既可以作用于函数也可作用于循环。

<div align=center>
<img src=".\\hls8\\2.png" width = 500 >
  改善时延的Directives<br>
<br><div align=left>

* LOOP_MERGE用于将顺序的Loop合并在一起。
* LOOP_FLATTEN用于将嵌套的循环展开为一个大的循环。


## 改善资源（Area）的指令
常用于改善资源的方法是设置更精确的数据类型和位宽，还有一些directives，如INLINE、ALLOCATE、ARRAY_MAP、ARRAY_RESHAPE、FUNCTION_INSTANTIATE等。

<div align=center>
<img src=".\\hls8\\3.png" width = 500 >
  改善资源的Directives<br>
<br><div align=left>


