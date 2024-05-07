
<!--more-->

# 什么时候不该使用 Optional?

Java的Optional类是一个可以为非空值包装一个值的容器，使开发人员可以更为明确地表达某个值可能为null的情况。然而，它的使用并非没有争议。在本文中，我将阐述什么情况下适合使用Optional，以及开发过程中可能遇到的常见误区。

## 几种常见误区

首先，Optional不应被视为一个可以替代null检查的全能工具。一些开发人员可能认为，使用Optional的ifPresent方法可以消除程序中的null检查，但这种观念并不准确。在你可以预见到一个方法或值可能返回null的情况下，老实实使用一个if判断仍然是最简洁、易于理解的方案。

用 `ifPresent` 代替 `null` 是没有意义的.

   ```java
   // Bad
   Optional<String> optional = ...;
   if(optional.isPresent()) {
     String str = optional.get();
   }
   
   // Good
   String str = ...;
   if(str != null) {
     // do something with str
   }
   ```

其次，使用Optional的ofNullable方法替代空值检查（?:）也同样是一种误解。Optional设计的初衷是为了更好地表达可能存在的null值，而并非为了替代空值检查。过度使用Optional会导致代码难以理解和维护，而我们的目标应该是写出清晰、易读的代码。

用 `nullable` 代替 `?:` 是更不符合阅读习惯的.

   ```java
   // Bad
   String value = Optional.ofNullable(someString).orElse("wasNull");
   
   // Good
   String value = someString == null ? "wasNull" : someString;
   ```

此外，一些开发人员可能误解Optional与函数式编程的关系。虽然Optional确实提供了一些函数式编程的功能，例如map、flatMap等，但这并不意味着使用Optional就等于使用函数式编程。事实上，如果函数式编程的代码使得程序的可读性下降，那么老实实写if判断往往是更好的选择。

函数式编程😅, 可读性远远不如老老实实写 if.

   ```java
   // Bad
   Optional<BigDecimal> first = ...
   Optional<BigDecimal> second = ...
   Optional<BigDecimal> result = Stream.of(first, second)
                                       .filter(Optional::isPresent)
                                       .map(Optional::get)
                                       .reduce(BigDecimal::add);
   
   // Good
   BigDecimal first = ...
   BigDecimal second = ...
   BigDecimal result;
   if (first == null || second == null) {
       result = BigDecimal.ZERO;
   } else {
       result = first.add(second));
   }
   ```

函数式 if 可以高级到让人看不懂.

   ```java
   // Bad
   String result = Optional.ofNullable(name)
             .filter(Constants.ALICE::equalsIgnoreCase)
             .map(i -> "Alice")
             .orElse("Bob");
   
   // Good
   String result = "";
   if(Constants.ALICE.equalsIgnoreCase(name)) {
     result = "Alice";
   } else {
     result = "Bob";
   }
   ```

更不应该将 Optional 当做一个容器类型使用, 比如在 Optional 里藏一个 List 或者 Set.

1. 一个 class 内部使用 Optional 的 field 或者一个方法接受 Option 的参数

   ```java
   public class Person {
       private Optional<String> name = Optional.empty();
     
       public void setName(String name) {
           this.name = Optional.of(name);
       }
   }
   ```

2. Optional 里藏一个 List 或者 Set

   ```java
   private Optional<Set<String>> DONOT_DO_THIS;  
   ```
   

## 常见场景

那么，何时该使用Optional呢？实际上，适用的场景比较有限。Optional在链式调用时的使用可以避免空指针异常。比如在C#中，我们可以使用?.操作符来链式调用深层的对象字段。类似的，Java中的Optional也可以实现这样的功能，避免因为null值而导致的异常。

1. 主要应用在 Library Method

   ```java
   @Test
   public void whenEmptyStream_thenReturnDefaultOptional() {
       List<User> users = new ArrayList<>();
       User user = users.stream().findFirst().orElse(new User("default", "1234"));
       
       assertEquals(user.getEmail(), "default");
   }
   ```

   这个例子说明了 Optional 主要应用在 Library Method 场景(你甚至在业务代码中看不到 Optional)

   ![image-20210317152047051.png](/blog_img/when-and-how-to-use-optional-in-java/1.png)

2.  类似 C# 的 `?.` 来链式调用取一个对象深层的字段
    例如，假设我们有一个User对象，它可能有一个Address对象，Address对象又可能有一个City对象。如果我们想获取用户的城市名，可能会这样写：

    ```java
    if (user != null) {
        Address address = user.getAddress();
        if (address != null) {
            City city = address.getCity();
            if (city != null) {
                return city.getName();
            }
        }
    }
    return "Unknown";
    ```

    而使用Optional，我们可以更优雅地实现这一功能：
    ```java
    return Optional.ofNullable(user)
                .map(User::getAddress)
                .map(Address::getCity)
                .map(City::getName)
                .orElse("Unknown");

    ```
   
总结，尽管Optional有其应用场景，但我们必须警惕不要滥用。其应用应当遵循编程的最佳实践：代码应当易于阅读和理解。最重要的是，我们应当理解Optional的真正价值并在适当的地方使用它，而不是盲目地将其应用在所有可能出现null的地方。

## 参考文档

1. https://stackoverflow.com/questions/26327957/should-java-8-getters-return-optional-type/26328555#26328555
2. https://blog.indrek.io/articles/misusing-java-optional/
3. https://softwareengineering.stackexchange.com/questions/364211/why-use-optional-in-java-8-instead-of-traditional-null-pointer-checks
4. https://www.oracle.com/technical-resources/articles/java/java8-optional.html
5. http://dolszewski.com/java/java-8-optional-use-cases/
6. https://medium.com/@yassinhajaj/optionals-are-bad-practices-still-bad-practices-if-everyone-practices-them-6ec5a66c30aa
7. https://www.dariawan.com/tutorials/java/using-java-optional-correctly/








