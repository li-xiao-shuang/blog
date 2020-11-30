###  为什么选择Java11

- 容器环境支持，GC等领域的增强。
- 进行了瘦身，更轻量级，安装包体积小。
- JDK11 是一个长期支持版。

### 特性介绍

#### Jshell  @since 9

Jshell在Java9中就被提出来了，可以直接在终端写Java程序，回车就可以执行。Jshell默认会导入下面的一些包,所以在Jshell环境中这些包的内容都是可以使用的。

```
import java.lang.*;
import java.io.*;
import java.math.*;
import java.net.*;
import java.nio.file.*;
import java.util.*;
import java.util.concurrent.*;
import java.util.function.*;
import java.util.prefs.*;
import java.util.regex.*;
import java.util.stream.*;
```



##### 1.什么是Jshell?

Jshell是在 Java 9 中引入的。它提供了一个交互式 shell，用于快速原型、调试、学习 Java 及 Java API，所有这些都不需要 public static void main 方法，也不需要在执行之前编译代码。

##### 2.Jshell的使用

打开终端，键入jshell进入jshell环境，然后输入/help intro可以查看Jshell的介绍。

```
 lixiaoshuang@localhost  ~  jshell
|  欢迎使用 JShell -- 版本 11.0.2
|  要大致了解该版本, 请键入: /help intro

jshell> /help intro
|
|                                   intro
|                                   =====
|
|  使用 jshell 工具可以执行 Java 代码，从而立即获取结果。
|  您可以输入 Java 定义（变量、方法、类等等），例如：int x = 8
|  或 Java 表达式，例如：x + x
|  或 Java 语句或导入。
|  这些小块的 Java 代码称为“片段”。
|
|  这些 jshell 工具命令还可以让您了解和
|  控制您正在执行的操作，例如：/list
|
|  有关命令的列表，请执行：/help

jshell>
```

Jshell确实是一个好用的小工具，这里不做过多介绍，我就举一个例子，剩下的大家自己体会。比如我们现在就想随机生成一个UUID，以前需要这么做：

- 创建一个类。
- 创建一个main方法。
- 然后写一个生成UUID的逻辑，执行。

现在只需要,进入打开终端键入jshell,然后直接输入`var uuid = UUID.randomUUID()`回车。就可以看到uuid的回显，这样我们就得到了一个uuid。并不需要public static void main(String[] args);

```
 lixiaoshuang@localhost  ~  jshell
|  欢迎使用 JShell -- 版本 11.0.2
|  要大致了解该版本, 请键入: /help intro

jshell> var uuid = UUID.randomUUID();
 uuid ==> 9dac239e-c572-494f-b06d-84576212e012
jshell>
```

##### 3.怎么退出Jshell？

在Jshell环境中键入`/exit`就可以退出。

```
 lixiaoshuang@localhost  ~ 
 lixiaoshuang@localhost  ~  jshell
|  欢迎使用 JShell -- 版本 11.0.2
|  要大致了解该版本, 请键入: /help intro

jshell> var uuid = UUID.randomUUID();
uuid ==> 9dac239e-c572-494f-b06d-84576212e012

jshell> /exit
|  再见
 lixiaoshuang@localhost  ~ 
```



#### 模块化（Module）@since 9

##### 1.什么是模块化？

模块化就是增加了更高级别的聚合，是Package的封装体。Package是一些类路径名字的约定，而模块是一个或多个Package组成的封装体。

java9以前 ：package => class/interface。

java9以后 ：module => package => class/interface。

那么JDK被拆为了哪些模块呢？打开终端执行`java --list-modules`查看。

```
lixiaoshuang@localhost  ~ java --list-modules
java.base@11.0.2
java.compiler@11.0.2
java.datatransfer@11.0.2
java.desktop@11.0.2
java.instrument@11.0.2
java.logging@11.0.2
java.management@11.0.2
java.management.rmi@11.0.2
java.naming@11.0.2
java.net.http@11.0.2
java.prefs@11.0.2
java.rmi@11.0.2
java.scripting@11.0.2
java.se@11.0.2
java.security.jgss@11.0.2
java.security.sasl@11.0.2
java.smartcardio@11.0.2
java.sql@11.0.2
java.sql.rowset@11.0.2
java.transaction.xa@11.0.2
java.xml@11.0.2
java.xml.crypto@11.0.2
jdk.accessibility@11.0.2
jdk.aot@11.0.2
jdk.attach@11.0.2
jdk.charsets@11.0.2
jdk.compiler@11.0.2
jdk.crypto.cryptoki@11.0.2
jdk.crypto.ec@11.0.2
jdk.dynalink@11.0.2
jdk.editpad@11.0.2
jdk.hotspot.agent@11.0.2
jdk.httpserver@11.0.2
jdk.internal.ed@11.0.2
jdk.internal.jvmstat@11.0.2
jdk.internal.le@11.0.2
jdk.internal.opt@11.0.2
jdk.internal.vm.ci@11.0.2
jdk.internal.vm.compiler@11.0.2
jdk.internal.vm.compiler.management@11.0.2
jdk.jartool@11.0.2
jdk.javadoc@11.0.2
jdk.jcmd@11.0.2
jdk.jconsole@11.0.2
jdk.jdeps@11.0.2
jdk.jdi@11.0.2
jdk.jdwp.agent@11.0.2
jdk.jfr@11.0.2
jdk.jlink@11.0.2
jdk.jshell@11.0.2
jdk.jsobject@11.0.2
jdk.jstatd@11.0.2
jdk.localedata@11.0.2
jdk.management@11.0.2
jdk.management.agent@11.0.2
jdk.management.jfr@11.0.2
jdk.naming.dns@11.0.2
jdk.naming.rmi@11.0.2
jdk.net@11.0.2
jdk.pack@11.0.2
jdk.rmic@11.0.2
jdk.scripting.nashorn@11.0.2
jdk.scripting.nashorn.shell@11.0.2
jdk.sctp@11.0.2
jdk.security.auth@11.0.2
jdk.security.jgss@11.0.2
jdk.unsupported@11.0.2
jdk.unsupported.desktop@11.0.2
jdk.xml.dom@11.0.2
jdk.zipfs@11.0.2
```



##### 2.为什么这么做？

大家都知道JRE中有一个超级大的rt.jar(60多M)，tools.jar也有几十兆，以前运行一个hello world也需要上百兆的环境。

- 让Java SE程序更加容易轻量级部署。
- 强大的封装能力。
- 改进组件间的依赖管理，引入比jar粒度更大的Module。
- 改进性能和安全性。

##### 3.怎么定义模块?

模块的是通过module-info.java进行定义，编译后打包后，就成为一个模块的实体。下面来看下最简单的模块定义。

![](https://i.loli.net/2019/11/03/eaJ9cAKDjTWipo8.png)

![](https://i.loli.net/2019/11/03/FKVhUwTfy4AraqP.png)

##### 4.模块的关键字

- open

  用来指定开放模块,开放模块的所有包都是公开的,public的可以直接引用使用,其他类型可以通过反射得到。

  ```
  open module module.one {
      //导入日志包
     requires java.logging;
  
  }
  ```

- opens

  opens 用来指定开放的包,其中public类型是可以直接访问的,其他类型可以通过反射得到。

  ```
  module module.one {
  
      opens <package>;
  }
  ```

- exports

  exports用于指定模块下的哪些包可以被其他模块访问。

  ```
  module module.one {
      
      exports <package>;
      
      exports <package> to <module1>, <module2>...;
  }
  ```

- requires

  该关键字声明当前模块与另一个模块的依赖关系。

  ```
  module module.one {
  
      requires <package>;
  
  }
  ```

- uses、provides…with…

  uses语句使用服务接口的名字,当前模块就会发现它,使用java.util.ServiceLoader类进行加载,必须是本模块中的,不能是其他模块中的.其实现类可以由其他模块提供。

  ```
  module module.one {
  
      //对外提供的接口服务 ,下面指定的接口以及提供服务的impl，如果有多个实现类，用用逗号隔开
      uses <接口名>;
  
      provides <接口名> with <接口实现类>,<接口实现类>;
  
  }
  ```

  

#### var关键字   @since 10

##### 1.var是什么？

var是Java10中新增的局部类型变量推断。它会根据后面的值来推断变量的类型，所以var必须要初始化。

例：

```
var a;       ❌
var a = 1;   ✅
```

##### 2.var使用示例

- var定义局部变量

  ```
  var a = 1; 
  等于
  int a = 1;
  ```

- var接收方法返回时

  ```
  var result = this.getResult();
  等于
  String result = this.getResult();
  ```

- var循环中定义局部变量

  ```
  for (var i = 0; i < 5; i++) {
     System.out.println(i);
  }
  等于
  for (int i = 0; i < 5; i++) {
     System.out.println(i);
  }
  ```

- var结合泛型

  ```
  var list1 = new ArrayList<String>();  //在<>中指定了list类型为String
  等于
  List<String> list1 = new ArrayList<>();
  
  var list2 = new ArrayList<>();        //<>里默认会是Object
  ```

- var在Lambda中使用（java11才可以使用）

  ```
  Consumer<String> Consumer = (var i) -> System.out.println(i);
  等于
  Consumer<String> Consumer = (String i) -> System.out.println(i);
  ```

##### 3.var不能再哪里使用？

- 类成员变量类型。
- 方法返回值类型。
- Java10中Lambda不能使用var，Java11中可以使用。

#### 增强api

##### 1.字符串增强 @since 11

```
// 判断字符串是否为空白
" ".isBlank();                     // true

// 去除首尾空格
" Hello Java11 ".strip();          // "Hello Java11"

// 去除尾部空格 
" Hello Java11 ".stripTrailing();  // " Hello Java11"

// 去除首部空格 
" Hello Java11 ".stripLeading();   // "Hello Java11 "

// 复制字符串
"Java11".repeat(3);                // "Java11Java11Java11"

// 行数统计
"A\nB\nC".lines().count();         // 3
```



##### 2.集合增强  

从Java 9 开始，jdk里面就为集合（List、Set、Map）增加了of和copyOf方法。它们用来创建不可变集合。

- of()  @since 9
- copyOf()  @since 10

示例一：

```
        var list = List.of("Java", "Python", "C"); //不可变集合
        var copy = List.copyOf(list);         //copyOf判断是否是不可变集合类型，如果是直接返回
        System.out.println(list == copy);    // true

        var list = new ArrayList<String>();  // 这里返回正常的集合
        var copy = List.copyOf(list);        // 这里返回一个不可变集合
        System.out.println(list == copy);    // false
```

示例二：

```
        var set = Set.of("Java", "Python", "C");
        var copy = Set.copyOf(set);
        System.out.println(set == copy);     // true

        var set1 = new HashSet<String>();
        var copy1 = List.copyOf(set1);
        System.out.println(set1 == copy1);   // false
```

示例三：

```
        var map = Map.of("Java", 1, "Python", 2, "C", 3);
        var copy = Map.copyOf(map);
        System.out.println(map == copy);     // true

        var map1 = new HashMap<String, Integer>();
        var copy1 = Map.copyOf(map1);
        System.out.println(map1 == copy1);   // false
```

`注意：使用 of 和 copyOf 创建的集合为不可变集合，不能进行添加、删除、替换、排序等操作，不然会报java.lang.UnsupportedOperationException异常，使用Set.of()不能出现重复元素、Map.of()不能出现重复key，否则回报java.lang.IllegalArgumentException。`。

##### 3.Stream增强 @since 9

Stream是Java 8 中的特性，在Java 9 中为其新增了4个方法：

- ofNullable(T t)

  此方法可以接收null来创建一个空流

  ```
  以前
  Stream.of(null);  //报错
  现在
  Stream.ofNullable(null);
  ```

- takeWhile(Predicate<? super T> predicate) 

  此方法根据Predicate接口来判断如果为true就 `取出` 来生成一个新的流,只要碰到false就终止，不管后边的元素是否符合条件。

  ```
          Stream<Integer> integerStream = Stream.of(6, 10, 11, 15, 20);
          Stream<Integer> takeWhile = integerStream.takeWhile(t -> t % 2 == 0);
          takeWhile.forEach(System.out::println);   // 6,10
  ```

- dropWhile(Predicate<? super T> predicate)

  此方法根据Predicate接口来判断如果为true就 `丢弃` 来生成一个新的流,只要碰到false就终止，不管后边的元素是否符合条件。

  ```
          Stream<Integer> integerStream = Stream.of(6, 10, 11, 15, 20);
          Stream<Integer> takeWhile = integerStream.dropWhile(t -> t % 2 == 0);
          takeWhile.forEach(System.out::println);  //11,15,20
  ```

- iterate重载

  以前使用iterate方法生成无限流需要配合limit进行截断

  ```
          Stream<Integer> limit = Stream.iterate(1, i -> i + 1).limit(5);
          limit.forEach(System.out::println);   //1,2,3,4,5
  ```

  现在重载后这个方法增加了个判断参数

  ```
          Stream<Integer> iterate = Stream.iterate(1, i -> i <= 5, i -> i + 1);
          iterate.forEach(System.out::println);  //1,2,3,4,5
  ```

##### 4.Optional增强  @since 9

- stream()

  如果为空返回一个空流，如果不为空将Optional的值转成一个流。

  ```
          //返回Optional值的流
          Stream<String> stream = Optional.of("Java 11").stream();
          stream.forEach(System.out::println);    // Java 11
          
          //返回空流
          Stream<Object> stream = Optional.ofNullable(null).stream();
          stream.forEach(System.out::println);    // 
  ```

  

- ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction)

  个人感觉这个方法就是结合isPresent()对Else的增强，ifPresentOrElse 方法的用途是，如果一个 Optional 包含值，则对其包含的值调用函数 action，即 action.accept(value)，这与 ifPresent 一致；与 ifPresent 方法的区别在于，ifPresentOrElse 还有第二个参数 emptyAction —— 如果 Optional 不包含值，那么 ifPresentOrElse 便会调用 emptyAction，即 emptyAction.run()。

  ```
          Optional<Integer> optional = Optional.of(1);
          optional.ifPresentOrElse( x -> System.out.println("Value: " + x),() ->
                  System.out.println("Not Present."));    //Value: 1
          
          optional = Optional.empty();
          optional.ifPresentOrElse( x -> System.out.println("Value: " + x),() ->
                  System.out.println("Not Present."));    //Not Present.
  ```

- or(Supplier<? extends Optional<? extends T>> supplier)

  ```
          Optional<String> optional1 = Optional.of("Java");
          Supplier<Optional<String>> supplierString = () -> Optional.of("Not Present");
          optional1 = optional1.or( supplierString);
          optional1.ifPresent( x -> System.out.println("Value: " + x));  //Value: Java
  
          optional1 = Optional.empty();
          optional1 = optional1.or( supplierString);
          optional1.ifPresent( x -> System.out.println("Value: " + x)); //Value: Not Present
  ```

##### 5.InputStream增强  @since 9

```
        String lxs = "java";
        try (var inputStream = new ByteArrayInputStream(lxs.getBytes());
             var outputStream = new ByteArrayOutputStream()) {
            inputStream.transferTo(outputStream);
            System.out.println(outputStream);    //java
        }
```



#### HTTP Client API

改api支持同步和异步两种方式，下面是两种方式的示例：

```
        var request = HttpRequest.newBuilder()
                .uri(URI.create("https://www.baidu.com/"))
                .build();
        var client = HttpClient.newHttpClient();
        // 同步
        HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
        System.out.println(response.body());

        // 异步
        CompletableFuture<HttpResponse<String>> sendAsync = client.sendAsync(request, HttpResponse.BodyHandlers.ofString());
        //这里会阻塞
        HttpResponse<String> response1 = sendAsync.get();
        System.out.println(response1.body());
```



#### 直接运行java文件

我们都知道以前要运行一个.java文件，首先要javac编译成.class文件，然后在java执行：

```
//编译
javac Java11.java
//运行
java Java11
```

在java11中，只需要通过java一个命令就可以搞定

```
java Java11.java
```



### 移除内容

- com.sun.awt.AWTUtilities。
- sun.misc.Unsafe.defineClass   使用java.lang.invoke.MethodHandles.Lookup.defineClass来替代。
- Thread.destroy() 以及 Thread.stop(Throwable) 方法。
- sun.nio.ch.disableSystemWideOverlappingFileLockCheck 属性。
- sun.locale.formatasdefault 属性。
- jdk snmp 模块。
- javafx，openjdk 是从java10版本就移除了，oracle java10还尚未移除javafx ，而java11版本将javafx也移除了
- Java Mission Control，从JDK中移除之后，需要自己单独下载。
- Root Certificates ：Baltimore Cybertrust Code Signing CA，SECOM ，AOL and Swisscom。
- 在java11中将java9标记废弃的Java EE及CORBA模块移除掉。

### 完全支持Linux容器（包括docker）

许多运行在Java虚拟机中的应用程序（包括Apache Spark和Kafka等数据服务以及传统的企业应用程序）都可以在Docker容器中运行。但是在Docker容器中运行Java应用程序一直存在一个问题，那就是在容器中运行JVM程序在设置内存大小和CPU使用率后，会导致应用程序的性能下降。这是因为Java应用程序没有意识到它正在容器中运行。随着Java 10的发布，这个问题总算得以解决，JVM现在可以识别由容器控制组（cgroups）设置的约束。可以在容器中使用内存和CPU约束来直接管理Java应用程序，其中包括：

- 遵守容器中设置的内存限制
- 在容器中设置可用的CPU
- 在容器中设置CPU约束

`Java 10的这个改进在Docker for Mac、Docker for Windows以及Docker Enterprise Edition等环境均有效。`

### 总结

 ![Java版本特性.png](https://i.loli.net/2019/11/05/uesYIQnMUgOdJpN.png)

