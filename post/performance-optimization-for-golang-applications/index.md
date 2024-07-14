
<!--more-->

## 0x00 简介

本篇文章主要讲如何测量一个 Golang 应用的性能.

>
> if you can't measure it , you can't improve it 
>
> --*Peter Drucker*

包括以下内容:

- 性能优化方法论
- pprof 的基本使用方法
- 使用 runtime/trace

## 0x01 性能优化基础

### 时机

什么时候该严肃地考虑性能问题? 当你觉得性能不够用的时候. 也许就是你该优化程序的性能了.

性能优化在软件开发的三个步骤里排在最后.

> 先跑起来, 再跑正确, 再跑快.


### 方法论

#### 如何衡量和优化服务的性能？

要衡量和优化服务的性能，我们可以从四个步骤入手：

1. 测量（measure）
2. 分析（analyze）
3. 剖析（profile）
4. 改进（improve）



### 性能标准

```
Latency Comparison Numbers
--------------------------
L1 cache reference                           0.5 ns
Branch mispredict                           5   ns
L2 cache reference                           7   ns                     14x L1 cache
Mutex lock/unlock                           25   ns
Main memory reference                     100   ns                     20x L2 cache, 200x L1 cache
Compress 1K bytes with Zippy             3,000   ns       3 us
Send 1K bytes over 1 Gbps network       10,000   ns       10 us
Read 4K randomly from SSD*             150,000   ns     150 us         ~1GB/sec SSD
Read 1 MB sequentially from memory     250,000   ns     250 us
Round trip within same datacenter     500,000   ns     500 us
Read 1 MB sequentially from SSD*     1,000,000   ns   1,000 us   1 ms ~1GB/sec SSD, 4X memory
Disk seek                           10,000,000   ns   10,000 us   10 ms 20x datacenter roundtrip
Read 1 MB sequentially from disk   20,000,000   ns   20,000 us   20 ms 80x memory, 20X SSD
Send packet CA->Netherlands->CA   150,000,000   ns 150,000 us 150 ms
 
Notes
-----
1 ns = 10^-9 seconds
1 us = 10^-6 seconds = 1,000 ns
1 ms = 10^-3 seconds = 1,000 us = 1,000,000 ns
 
Credit
------
By Jeff Dean:               http://research.google.com/people/jeff/
Originally by Peter Norvig: http://norvig.com/21-days.html#answers
 
Contributions
-------------
Some updates from:       https://gist.github.com/2843375
'Humanized' comparison: https://gist.github.com/2843375
Visual comparison chart: http://i.imgur.com/k0t1e.png
Animated presentation:   http://prezi.com/pdkvgys-r0y6/latency-numbers-for-programmers-web-development/latency.txt
```

## 0x01 工具

### 简介

在 .Net 平台上, 有 dotProfiler, vs profiler 可供使用,

虽然 golang 并没有一个 真正意义上 Full-featured 的 Profiling 工具, 但基于 golang 的 (`简单即美`) 哲学, 有许多小工具可以协作完成性能测量

* import "runtime/pprof"

  Package pprof writes runtime profiling data in the format expected by the pprof visualization tool.


### 更新

在 go 1.12 以后, 官方的 pprof 已经自带了 FlameGraph, 所以之前的 `uber/go-torch` 等工具已经不是必须安装的了.
下面介绍如何使用内置的 pprof 来查看火焰图

1. 首先在程序内部先启动 pprof

```go
import _ "net/http/pprof"

func main() {
	go func() {
		log.Println(http.ListenAndServe("localhost:6060", nil))
    }()
}
```

2. pprof 可以直接使用下面的命令调用:

```shell
go tool pprof -http localhost:6061 -seconds 60 localhost:6060

# 6060 端口是程序运行 pprof 的端口
# 6061 是 pprof 运行结束后, 在浏览器里打开的端口
```

3. 在 `http://localhost:6061/ui/flamegraph` 即可以看到火焰图

火焰图长这样:

![img](http://www.brendangregg.com/FlameGraphs/example-dtrace.svg)

## Reference

* [https://github.com/google/pprof/blob/master/doc/pprof.md](https://github.com/google/pprof/blob/master/doc/pprof.md)
* [https://github.com/uber/go-torch](https://github.com/uber/go-torch)
* [Service performance 101](https://mp.weixin.qq.com/s/MZv8DMowKKOWXf-8YE15_g)