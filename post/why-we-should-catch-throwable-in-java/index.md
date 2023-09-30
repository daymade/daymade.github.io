
<!--more-->

# 为什么不要 Catch Throwable

## 引言

在Java编程中，异常处理是编程中的重要组成部分。但是，是否知道我们在捕获异常时应避免捕获 `Throwable`?

## Throwable的定义及作用

[`Throwable`](https://docs.oracle.com/javase/8/docs/api/java/lang/Throwable.html) 是所有异常和错误的超类。虽然你可以在 catch 子句中使用它，但你绝对不应这样做！

## 为什么不应该捕获Throwable

如果你在 catch 子句中使用 `Throwable`，它不仅会捕获所有的异常，还会捕获所有的错误。

`Error` 是 `Throwable` 的一个子类，表示严重的问题，合理的应用程序不应试图捕获。大多数这类错误是异常情况。根据 [Java 规范](https://docs.oracle.com/javase/8/docs/api/java/lang/Throwable.html)，`Error` 类是 `Throwable` 的一个单独的子类，与类层次结构中的 `Exception` 不同，它允许程序使用 `catch (Exception e) {}` 的语法来捕获所有可能恢复的异常，而不捕获通常无法恢复的错误。

`Error` 是由 JVM 抛出的，表示应用程序不应该处理的严重问题。典型的例子包括 [`OutOfMemoryError`](https://docs.oracle.com/javase/8/docs/api/java/lang/OutOfMemoryError.html) 或 [`StackOverflowError`](https://docs.oracle.com/javase/8/docs/api/java/lang/StackOverflowError.html)。这两种情况都是应用程序无法控制和处理的。

因此，除非你绝对确定你处在一个需要处理错误的特殊情况，否则最好不要捕获 `Throwable`。

## 示例：错误的使用Throwable的情况

```java
public void doNotCatchThrowable() {
	try {
		// do something
	} catch (Throwable t) {
		// don't do this!
	}
}
```

## 测试代码：验证不应该捕获Throwable的观点

```java
public void testOom_terminateThread_doNotCrashEntireJvm() {
  new Thread(() -> {
    try {
      // -Xms10M -Xmx10M
      List<String> list = new ArrayList<>(1000 * 1000 * 1000);
      System.out.println("business logic");
    } catch (Exception e) {
      System.out.println("exception occurs, but is not OOM");
    } catch (Throwable e) {
      System.out.println("catch OOM");
      System.out.println(e);
      // 不要 catch Throwable, 会导致 OOM 无法被捕获, business logic 也执行不了
      throw e;
    }
  }).start();

  try {
    Thread.sleep(5000);
  } catch (InterruptedException e) {
    e.printStackTrace();
  }
  assertTrue(true);
}
```

## 结论
捕获 Throwable 可能会阻止 JVM 对严重问题的处理，从而导致更多的问题。因此，我们应该尽量避免在 catch 子句中使用 Throwable。


## 参考

- [Best practices for exception handling in Java](https://stackify.com/best-practices-exceptions-java/)
- [Java documentation on Throwable](https://docs.oracle.com/javase/8/docs/api/java/lang/Throwable.html)
- [Java documentation on Error](https://docs.oracle.com/javase/7/docs/api/java/lang/Error.html)
- [Java Language Specification on Catch](https://docs.oracle.com/javase/specs/jls/se8/html/jls-11.html#jls-11.2.3)
- [Why Catching Throwable or Error is bad?](https://www.baeldung.com/java-catch-throwable-bad-practice)