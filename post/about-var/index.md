
<!--more-->

## 0x00 TL;DR

`var` 支持了局部变量的隐式推导, `var` 的引入支持了匿名类型, 支持了 `LINQ`.

在可以不写 `var` 的使用场景下, 写 `var` 和不写 `var` 的编译后代码是相同的.

`var` 简化了代码的书写方式, 让你繁忙搬砖的生活变得更加美好.

## 0x01 引子

在知乎上看到有题主问 

> C# 作为一种静态类型语言，为什么会引入 `var`？

我想你应该有自己的答案, 

在本篇文章中, 我将努力把自己的看法表达出来, 

然后我们可以就文中谈到的一些概念进行更深♂入的讨论.

## 0x02 解释

### 首先什么是 `var`

`var` 是在 C# 3.0 时引入的一个关键字, 其作用是根据初始值推断变量的类型, 从而简化局部变量的声明. 

我们知道, C# 3.0 相对于 C# 2.0, 最重要的就是引入了LINQ, 从2007年 C# 3.0 发布到目前为止, LINQ在各个方向大放光明, 并且在不断创新, 比如 `PLINQ` (Parallel LINQ)和 `Rx`(Reactive Extensions).

那么LINQ 是怎么样被创造出来的, 底层使用了哪些技术, 这些问题并不难解答, 以下是部分黑科技列表: 

- 自动属性(Auto Property)
- 隐式类型的局部变量(implicit typed local variable)
- 对象和集合初始化程序(Object/Collection initializer)
- 隐式类型数组(implicit typed array)
- 匿名类型(anonymous type)

可以看出几乎 C# 3.0 的每一个特性都是为了实现LINQ而引入的. 其中, `隐式类型推导` 和 `匿名类型` 特性相互搞基, 两者缺一不可.

### 再来说说静态

我们知道 C# 1.0 的类型系统是静态、显式和安全的。

引入了泛型之后, C# 2.0 仍然是静态、显式和安全的。

在 C# 3.0 中, "静态"和"安全"仍然是成立的, 现在你可以要求编译器帮你推断局部变量的类型, 所以, 在大多数情况下, 它是隐式类型的, 当然, 有些情况你还是需要帮助编译器显式指定类型.

理解"静态"这一点很重要, 有些开发者对`var`感到恐惧, 认为`var`使 C# 变成了动态类型或弱类型的语言, 我们知道,

>  ![恐惧来源于未知](data:image/jpeg;base64,/9j/4AAQSkZJRgABAQAAAQABAAD/2wCEAAkGBxMTERUSEhMWFRUVFRcTFxcVFR0XFxoVFRMYFhgZFRYYHiggGBolHRUXIjEjJSkrLi4uGB8zODMtNygtLisBCgoKDg0OFxAQFS4dHx0rLSstLS0tLS0tLS0rLS0tKy0tLS0tLS0tLTcrKzMtKy03LSstLS0tLSsyKzctLS0rK//AABEIAJoBRwMBIgACEQEDEQH/xAAcAAEAAgMBAQEAAAAAAAAAAAAABQYCBAcDAQj/xABHEAABAwICBQgFCgUDAwUAAAABAAIDBBESIQYTFjFRBUFSVJOU0tMHFCJhtCMkMjRxc3SBocRCkbHB8GLR4TOy8RVDRIKS/8QAGAEBAQEBAQAAAAAAAAAAAAAAAAECAwT/xAAeEQEBAQEAAgIDAAAAAAAAAAAAARECITEDQRJRYf/aAAwDAQACEQMRAD8AqGleklRTVGog1DI2QUpANJTvN30cL3EufEXElznHM86iNt63pQdypfJT0g/XnfcUfwFOq9hQWHbet6UHcqXyU23relB3Kl8lV9rQswwIJ3bet6UHcqXyU23relB3Kl8lQ2qavRsDeCCV23relB3Kl8lNt63pQdypfJUUIG8F8dA3ggltt63pQdypfJTbet6UHcqXyVEtpwnq7f8AZBLbb1vSg7lS+Sm29b0oO5UvkqDdG1eCCx7b1vSg7lS+Sm29b0oO5UvkquIgse29b0oO5UvkptvW9KDuVL5KriILHtvW9KDuVL5Kbb1vSg7lS+Sq4iCx7b1vSg7lS+Sm29b0oO5UvkquIgse29b0oO5UvkptvW9KDuVL5KriILHtvW9KDuVL5Kbb1vSg7lS+Sq4iCx7b1vSg7lS+Sm29b0oO5UvkquIgse29b0oO5UvkptvW9KDuVL5KriILHtvW9KDuVL5Kbb1vSg7lS+Sq4iCx7b1vSg7lS+Sm29b0oO5UvkquIgse29b0oO5UvkptvW9KDuVL5KriILHtvW9KDuVL5Kbb1vSg7lS+Sq4iCx7b1vSg7lS+Sm29b0oO5UvkquIgse29b0oO5UvkptvW9KDuVL5KriIL9o1y5LVtq46gQva2mEjbUsEZDxWUzLh0cbT9F7ha9s18Ud6Pt9Z+D/fUiINX0hH5877ij+Ap1XiclYvSD9ed9xR/AU6rwao1BqzA3LONi2YoidwJtfIIPBjSfeeCkaDk5z73vlzAXt9oK+t5Nkb7TmObfO+Hm+0qTjkdbJr7XuXC9v6ZBWRLXi7k1jRfL382/nsd323UVUOZezbixO/3L1q53Y7h5NuIy+wgjMfktR5F72AB5hzH3e5LRmP+Vg5fGEDf/nvXq5mamq1n2utZbb2rUSJRERVBERAREQEREBERAREQEREBERAREQEREBERAREQEREBERBbPR9vrPwf76kRPR9vrPwf76kRBr6f/XnfcUXwECgWBT2n/wBed9xR/AU6hYGXzUanps08asuhdPerY0i4s5xvwFrfrZQtPGrxoBFHDNJPO5rWNhFjcby43/QfqE68RJ7dYo6aNzLPYCCLZhYO0epALCBhHAj+60+StJKeX2YniQgEgDfl/wCVrcrad0kb9W95uMjZpsD/AKuC8X4974dbYjuWtCaGQOOqwEc7SR/Vcm0m0f8AVn+y7Ew7r712Co0ipZmkQyi/AlUbSmlMrHYd7eZer4p1nljr+OcAC+fuv9i2nc452ktP/wBTZa88duY33FesDr4v5/z3rcSvCVq01vyBaCqURERBERAREQEREBERAREQEREBERAREQEREBERAREQEREBERBbPR9vrPwf76kRPR9vrPwf76kRB4aej5878PR/AU6jKVildOh8+d9xRfAU6j6ViL9JOBis+jnJ5na+NrsLsN7jeBiFyPyuLqu07clZdD5yypbna4IP9f7K9evBPa56EaPxU75qlxc9zI8AxbruN93HID+aq9dooJZnzOc6znF+FozB/wBvertyjpHTw05DiW4nfwtOdsjY7lCch6QCSUyQxyGNos8vN7Z5Efle/wBq4c3rb1jdkVubRYvfaMvFhmXW5uNluPcIGBjziI3+9WPl/lloadUBnzAWuVzarq3vc4uO7L/9ZZD3Bdubb5rFyekZy5Rgue+PcLE2PMXAf3WieTnxNDni2MHCDvtcb1YuRNQMRmfbC7JmEk2G42GR/NR3L9eZpC+1mjJreDRu/NXEQj3WUct2QrSUBERARF8duQfUXQNINC2moMFNEyJkUYlfNrX1D3NLYRZ8EWN7HY5RYYRcXO4FRUOgc7g8NkiL46ptG9lpQGyOqPVwdaY8DhjINmkkNINuZBVEUkzkoGqFMJ2O9rDrWslczEG3IDBHrHG4LcmZn3ZqwcoaFOjge27NfDLVue67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)
>  恐惧来源于未知

下面两段代码

```c#
MyType a = new MyType();
```

和

```c#
var a = new MyType();
```

的最终编译结果是一样的.

用 `ILspy` 来看下生成的 IL 代码:

```c#
//假装这里有 IL 代码, 你们配合一下
```

可以看到变量仍然是静态类型的, 你只是没把他的类型**显式**写出来而已.

如果你在下面搞事情, 写出了这样的代码

```c#
var a = new MyType();
...
a = 0;  //x 这里是编译不通过的
```

这就是静态类型带来的好处,

说到这里我们应该知道, 一个语言是不是静态类型的, 究竟应该怎么来判断? 

> 区别就是他的类型是在编译时检查的, 还是在运行时检查的.

所以在 C# 4.0 引入了 `dynamic` 后, 我们不能再说 C# 完全是静态类型语言. 



### 对比

#### Javascript var

//todo 

#### C++ auto

我不会 C++, 
请[哔]大的说下 C++ 最新加入的 `auto` 关键字和 `var` 有什么异同?

### 那么哪里才可以买得到呢?

之所以使用隐式类型, 主要原因是它减少了代码量, 这就意味着可读性增强. 举个栗子你感受一下 

栗子1:

```c#
Dictionary<string,List<Person>> groupedPerson = new Dictionary<string,List<Person>>();
```

栗子2:

```c#
var groupedPerson = new Dictionary<string,List<Person>>();
```

使用 `var` 改变了这行代码的重心, 你的同事可以将注意力放到确切的类型上.

你也可以 `var` 一个 `lambda` 表达式, 你的同事可以不用知道这个 `lambda` 表达式的类型是什么, 他只需要关心函数的逻辑. 



> 只要还能嘎嘎叫, 我是鸭子我骄傲



并且, 最重要的, `var` 提供了对 `匿名类型` 的支持.



那么问题来了, "你把 `var` 说的天花乱坠, 是不是我在所有地方都应该弃用显式类型, 改用 `var` 进行声明呢? "

从小学做判断题我们都知道, 凡是太绝对的用词都打x, 比如所有. 比如凡是.

什么时候适合使用隐式类型, 也是可以引起圣战的一个问题. 

> 为了使用 C# 3.0 的另一个特性, 匿名类型, 你必须要使用 `var` 来进行声明, 这是毋庸讨论的. 

那么是在任何地方都用呢, 还是在任何地方都不用呢(我不使用匿名类型, 再问自杀!)? 



可读性是一个重要的影响因素, `var` 可以提升可读性, `var` 也可以降低可读性, 这听起来有些打自己的脸, 但是事实如此. 



一个降低可读性的场景:

假如不显示地指出要声明的变量是什么类型, 只通过读代码的方式, 有时候可能不好确定它的类型.



注意这里我们使用了有时候可能不好确定这一串车轱辘话, 如果你能说出这一段话脸不红心不跳, 

说明你有成为架构师的潜力.

因为合格的架构师, 经常挂在嘴边的话就是: 

> It depends

### Jon Skeet 一些关于 `var` 的建议

- 如果让读代码的人一眼就能看出变量的类型是很重要的，就使用显式类型。
- 如果变量直接用一个构造函数初始化，而且类型名称很长（用泛型时经常会这样） ，就考
  虑使用隐式类型。
- 如果变量的确切类型不重要，而且它的本质在当前上下文中已很清楚，就用隐式类型，
  从而不去强调代码具体是如何达到其目标的，而是关注它想要达到什么目标这个更高级
  别的主题。
- 在开发一个新项目的时候，与团队成员就这件事情进行商议。
- 如有疑虑，一行代码用两种方式都写一写，根据直觉选一个最“顺眼”的。

## 0x03 脑洞

`RPC` 静态调用分析

### A story

> 作为服务消费者, 我希望在写完代码就可以知道自己的 `RPC` 调用是否存在问题, 如果服务不存在, 我希望立刻知道结果, 而不是部署到测试服务器或者生产环境中才知道调用失败. 
>
> 或者作为服务提供方. 我的某个版本的服务要下线了, 我希望能够看到是否有人在调用这个版本, 具体都有哪些人在调用, 这样我可以预留时间并告知他们及时升级, 从而能够做到安全下线.

### 解决方案

- 服务的消费者在编译时可以知道自己调用的服务是否存在
- 服务提供方在下线某些服务前可以知道自己的服务是否有客户端在调用. 

## 0x04 Reference

- [深入理解C#（第3版）](https://book.douban.com/subject/25843328/)
- [弱类型、强类型、动态类型、静态类型语言的区别是什么？](https://www.zhihu.com/question/19918532)


- 另外感谢谭浩强老师提供上面代码片段里几个变量的命名指导