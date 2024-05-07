
<!--more-->

## 0x00 什么是高可用

大家都知道, 在分布式系统中, 故障是不可避免的. 时序故障, 随机故障, 遗漏故障无时无刻不在发生.

虽然故障不可避免, 但我们可以通过设计, 屏蔽, 降级, 补偿掉某些故障.

迄今为止, DSF 生产环境基本没有出现过全面故障. 只发生过一次机房断电, 服务和服务之间网络不通导致的局部失败. 整个集群还是正常运行的. 

DSF 是怎么做到这一点的? 下面简单介绍几点设计理念.

#### SLA

首先定义下可用性( [SLA](https://en.wikipedia.org/wiki/Service-level_agreement) )是什么:

> A service-level agreement is an agreement between two or more parties, where one is the customer and the others are service providers.

简单来讲, 可用性是一个承诺, 代表了你可以保证系统有多少时间是可以正常提供服务的.

通常衡量可用性的单位是 `宕机时间` 和 `正常运行时间` 的百分比, 也就是大家常说的几个 9 的可用性.

### 怎么测量可用性

The software industry generally measures availability by using two types of metrics (measurements):

* The mean time to recover
* The mean time between failures

通过一系列的高可用设计手段, 可以提高可用性.

高可用的目标:

* 一个服务的依赖的失败, 不能影响到服务本身.
* 当依赖失败时候, 服务应该能够自动采取正确的操作.
* 服务应该能够显示现在的运行状态, 和历史的运行状态, 发生过什么事情.

## 0x01 高可用的常用手段

### 复制(冗余)

When failures occur, the [**failover**](https://docs.oracle.com/cd/A91202_01/901_doc/rac.901/a89867/glossary.htm#432337) process moves processing performed by the failed component to the backup component.

The more transparent that failover is to users, the higher the availability of the system.

案例:

* DSF 控制中心有复制. 每个控制中心节点都是无状态的. 其中保存了一份 ZK 集群的完整副本. 

  这个复制提供了控制中心在核心数据库 ZK 集群挂掉的情况下继续运行的能力.

### 缓存

cache 和复制一脉相承, 是在 C 端实现的复制.

案例:

DSF 客户端缓存所有的寻址请求. 如果控制中心失败, 或者网络发生分区, 客户端拿不到最新的终结点信息, 但是会一直沿用之前成功的结果.

这个复制提供了客户端在控制中心集群挂掉的情况下继续运行的能力.

### 限流

如果对访问的次数不加控制，很可能会造成 API 被滥用，甚至被 [DDos 攻击](https://en.wikipedia.org/wiki/Denial-of-service_attack)。根据使用者不同的身份对其进行限流，可以防止这些情况，减少服务器的压力。

> 人啊, 认清你自己
>
> --尼采

设计容量等于承诺. 有很多系统在提供给外部使用时, 并不知道自己能承受多大的访问量, 也就没有承诺.

承诺带来了契约, 高于你对外公布的容量之外的请求都会被拒绝, 从而保护后端不被额外的压力冲垮.

限流有一个额外的作用, 就是防范来自队友的不合理请求.

你不能保证队友写的代码没有 BUG, 如果下游某个服务突然狂暴的将流量打到你这边, 你的服务应该能够自动合理的进行拒绝.

案例:

控制中心对所有服务容器( Kernel ) 的请求进行了限流. 每个 Kernel 单位时间能够发起的请求是有限的.

这个设计曾经无数次成功防御了来自 Kernel 的额外流量.

### 柔性设计

#### 强一致性 -> 最终一致性

分布式系统中有 CAP 的限制, 三者只能取其二, 既然要提高可用性 A, 那么我们一般选择放弃 C. 保留 P

DSF 控制中心是一个 AP 系统. 通过最终一致性的设计, 能够提供任一台节点挂掉. 不影响集群可用性的能力.

#### 同步 -> 异步

对 DB 的操作改为异步.

案例:

线下环境发生过一次数据库被误操作删除 (数据库: 怎么又是我) 的情况,

这时 DSF 集群的服务完全不受影响. 只是后台无法查看运行状态数据了. 将整体失败降级为局部失败.

## Reference

* [High Availability Concepts and Best Practices](https://docs.oracle.com/cd/A91202_01/901_doc/rac.901/a89867/pshavdtl.htm#6062)
* [Making the Netflix API More Resilient](https://medium.com/netflix-techblog/making-the-netflix-api-more-resilient-a8ec62159c2d)
* [关于高可用的系统](http://coolshell.cn/articles/17459.html)