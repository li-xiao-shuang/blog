#### 什么是Lambda？

Lambda是一个匿名函数，我们可以把Lambda表达式理解为是一段可以传递的代码(将代码像参数一样进行传递，称为行为参数化)。Lambda允许把函数作为一个方法的参数（函数作为参数传递进方法中），要做到这一点就需要了解，什么是函数式接口，这里先不做介绍，等下一篇在讲解。

首先先看一下lambda长什么样？ 正常写法：

```
new Thread(new Runnable() {
    @Override
    public void run() {
       System.out.println("hello lambda");
    }
}).start();
```

lambda写法：

```
new Thread(
    () -> System.out.println("hello lambda")
).start();
```

怎么样？是不是感觉很简洁，没错，这就是lambda的魅力，他可以让你写出来的代码更简单、更灵活。

#### Lambda怎么写？

![](https://user-gold-cdn.xitu.io/2019/7/16/16bf81c30ea9b895?w=798&h=434&f=png&s=59257) 大家看一些上面的这个图，这就是lambda的语法，一个lambda分为三部分：参数列表、操作符、lambda体。以下是lambda表达式的重要特征：

- `可选类型声明：` 不需要声明参数类型，编译器可以统一识别参数值。也就说(s) -> System.out.println(s)和 (String s) -> System.out.println(s)是一样的编译器会进行类型推断所以不需要添加参数类型。

- `可选的参数圆括号：` 一个参数无需定义圆括号，但多个参数需要定义圆括号。例如：

  - s -> System.out.println(s) 一个参数不需要添加圆括号。
  - (x, y) -> Integer.compare(y, x) 两个参数添加了圆括号，否则编译器报错。

- `可选的大括号：` 如果主体包含了一个语句，就不需要使用大括号。

  - s -> System.out.println(s) , 不需要大括号.
  - (s) -> { if (s.equals("s")){ System.out.println(s); } }; 需要大括号

- `可选的返回关键字：` 如果主体只有一个表达式返回值则编译器会自动返回值，大括号需要指定明表达式返回了一个数值。

  Lambda体不加{ }就不用写return:

  ```
  Comparator<Integer> com = (x, y) -> Integer.compare(y, x);
  ```

  Lambda体加上{ }就需要添加return:

  ```
    Comparator<Integer> com = (x, y) -> {
              int compare = Integer.compare(y, x);
              return compare;
          }; 
  ```

#### 类型推断

上面我们看到了一个lambda表达式应该怎么写，但lambda中有一个重要特征是 `可选参数类型声明`，就是说不用写参数的类型，那么为什么不用写呢？它是怎么知道的参数类型呢？这就涉及到类型推断了。

**java8的泛型类型推断改进：**

- 支持通过方法上下文推断泛型目标类型
- 支持在方法调用链路中，泛型类型推断传递到最后一个方法

```
List<Person> ps = ...
Stream<String> names = ps.stream().map(p -> p.getName());
```

在上面的代码中，ps的类型是List ``，所以ps.stream()的返回类型是Stream ``。map()方法接收一个类型为Function的函数式接口，这里T的类型即是Stream元素的类型，也就是Person，而R的类型未知。由于在重载解析之后lambda表达式的目标类型仍然未知，我们就需要推导R的类型：通过对lambda表达式lambda进行类型检查，我们发现lambda体返回String，因此R的类型是String，因而map()返回Stream ``。绝大多数情况下编译器都能解析出正确的类型，但如果碰到无法解析的情况，我们则需要：

- 使用显式lambda表达式（为参数p提供显式类型）以提供额外的类型信息
- 把lambda表达式转型为Function
- 为泛型参数R提供一个实际类型。（ <String>map(p -> p.getName())）

#### 方法引用

方法引用是用来直接访问类或者实例已经存在的方法或构造方法，提供了一种引用而不执行方法的方式。是一种更简洁更易懂的Lambda表达式，当Lambda表达式中只是执行一个方法调用时，直接使用方法引用的形式可读性更高一些。 方法引用使用 “ :: ” 操作符来表示，左边是类名或实例名，右边是方法名。 `（注意：方法引用::右边的方法名是不需要加（）的，例：User::getName）`

**方法引用的几种形式：**

- 类 :: 静态方法
- 类 :: 实例方法
- 对象 :: 实例方法

```
例如：
    Consumer<String> consumer = (s) -> System.out.println(s);
等同于：
    Consumer<String> consumer = System.out::println;

例如：
    Function<String, Integer> stringToInteger = (String s) -> Integer.parseInt(s);
等同于：
    Function<String, Integer> stringToInteger = Integer::parseInt;

例如：
    BiPredicate<List<String>, String> contains = (list, element) -> list.contains(element);
等同于：
    BiPredicate<List<String>, String> contains = List::contains;
```

**`注意:`**

- Lambda体中调用方法的参数列表与返回值类型，要与函数式接口中抽象方法的函数列表和返回值类型保存一致
- 若Lambda参数列表中的第一个参数是实例方法的调用者，而第二个参数是实例方法的参数时，可以使用ClassName::method

#### 构造方法引用

语法格式：类名::new

```
例如：
    Supplier<User> supplier = ()->new User();

等同于：
    Supplier<User> supplier = User::new;
```

**`注意:`** 需要调用的构造器方法与函数式接口中抽象方法的参数列表保持一致。

#### Lambda是怎么实现的？

研究了半天Lambda怎么写，可是它的原理是什么？我们简单看个例子，看看真相到底是什么：

```
public class StreamTest {

    public static void main(String[] args) {
        printString("hello lambda", (String s) -> System.out.println(s));

    }

    public static void printString(String s, Print<String> print) {
        print.print(s);
    }
}

@FunctionalInterface
interface Print<T> {
    public void print(T t);
}
```

上面的代码自定义了一个函数式接口，定义一个静态方法然后用这个函数式接口来接收参数。编写完这个类以后，我们到终端界面javac进行编译，然后用javap（javap是jdk自带的反解析工具。它的作用就是根据class字节码文件，反解析出当前类对应的code区（汇编指令）、本地变量表、异常表和代码行偏移量映射表、常量池等等信息。）进行解析，如下图：

- 执行javap -p 命令 ( -p -private 显示所有类和成员)

  看上图发现在编译Lambda表达式生成了一个 `lambda$main$0`静态方法，这个静态方法实现了Lambda表达式的逻辑，现在我们知道原来Lambda表达式被编译成了一个静态方法，那么这个静态方式是怎么调用的呢？我们继续进行

- 执行javap -v -p 命令 ( -v -verbose 输出附加信息)

  ```
   public com.lxs.stream.StreamTest();
      descriptor: ()V
      flags: ACC_PUBLIC
      Code:
        stack=1, locals=1, args_size=1
           0: aload_0
           1: invokespecial #1                  // Method java/lang/Object."<init>":()V
           4: return
        LineNumberTable:
          line 7: 0
  
    public static void main(java.lang.String[]);
      descriptor: ([Ljava/lang/String;)V
      flags: ACC_PUBLIC, ACC_STATIC
      Code:
        stack=2, locals=1, args_size=1
           0: ldc           #2                  // String hello lambda
           2: invokedynamic #3,  0              // InvokeDynamic #0:print:()Lcom/lxs/stream/Print;
           7: invokestatic  #4                  // Method printString:(Ljava/lang/String;Lcom/lxs/stream/Print;)V
          10: return
        LineNumberTable:
          line 10: 0
          line 12: 10
  
    public static void printString(java.lang.String, com.lxs.stream.Print<java.lang.String>);
      descriptor: (Ljava/lang/String;Lcom/lxs/stream/Print;)V
      flags: ACC_PUBLIC, ACC_STATIC
      Code:
        stack=2, locals=2, args_size=2
           0: aload_1
           1: aload_0
           2: invokeinterface #5,  2            // InterfaceMethod com/lxs/stream/Print.print:(Ljava/lang/Object;)V
           7: return
        LineNumberTable:
          line 15: 0
          line 16: 7
      Signature: #19                          // (Ljava/lang/String;Lcom/lxs/stream/Print<Ljava/lang/String;>;)V
  
    private static void lambda$main$0(java.lang.String);
      descriptor: (Ljava/lang/String;)V
      flags: ACC_PRIVATE, ACC_STATIC, ACC_SYNTHETIC
      Code:
        stack=2, locals=1, args_size=1
           0: getstatic     #6                  // Field java/lang/System.out:Ljava/io/PrintStream;
           3: aload_0
           4: invokevirtual #7                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
           7: return
        LineNumberTable:
          line 10: 0
  }
  SourceFile: "StreamTest.java"
  InnerClasses:
       public static final #58= #57 of #61; //Lookup=class java/lang/invoke/MethodHandles$Lookup of class java/lang/invoke/MethodHandles
  BootstrapMethods:
    0: #27 invokestatic java/lang/invoke/LambdaMetafactory.metafactory:(Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodHandle;Ljava/lang/invoke/MethodType;)Ljava/lang/invoke/CallSite;
      Method arguments:
        #28 (Ljava/lang/Object;)V
        #29 invokestatic com/lxs/stream/StreamTest.lambda$main$0:(Ljava/lang/String;)V
        #30 (Ljava/lang/String;)V
  ```

这里只贴出了一部分的字节码结构，由于常量池定义太长了，就没有粘贴。

```
InnerClasses:
     public static final #58= #57 of #61; //Lookup=class java/lang/invoke/MethodHandles$Lookup of class java/lang/invoke/MethodHandles
BootstrapMethods:
  0: #27 invokestatic java/lang/invoke/LambdaMetafactory.metafactory:(Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodHandle;Ljava/lang/invoke/MethodType;)Ljava/lang/invoke/CallSite;
    Method arguments:
      #28 (Ljava/lang/Object;)V
      #29 invokestatic com/lxs/stream/StreamTest.lambda$main$0:(Ljava/lang/String;)V
      #30 (Ljava/lang/String;)V
```

通过这段字节码结构发现是要生成一个内部类，使用invokestatic调用了一个LambdaMetafactory.metafactory方法，并把 `lambda$main$0`作为参数传了进去，我们来看metafactory 的方法里的实现代码：

```
    public static CallSite metafactory(MethodHandles.Lookup caller,
                                       String invokedName,
                                       MethodType invokedType,
                                       MethodType samMethodType,
                                       MethodHandle implMethod,
                                       MethodType instantiatedMethodType)
            throws LambdaConversionException {
        AbstractValidatingLambdaMetafactory mf;
        mf = new InnerClassLambdaMetafactory(caller, invokedType,
                                             invokedName, samMethodType,
                                             implMethod, instantiatedMethodType,
                                             false, EMPTY_CLASS_ARRAY, EMPTY_MT_ARRAY);
        mf.validateMetafactoryArgs();
        return mf.buildCallSite();
    }
```

在buildCallSite的函数中,是函数spinInnerClass 构建了这个内部类。也就是生成了一个StreamTest$$Lambda$1.class这样的内部类,这个类是在运行的时候构建的，并不会保存在磁盘中。

```
    @Override
    CallSite buildCallSite() throws LambdaConversionException {
        final Class<?> innerClass = spinInnerClass();
        以下省略。。。
    }
```

如果想看到这个构建的类，可以通过设置环境参数 System.setProperty("jdk.internal.lambda.dumpProxyClasses", " . "); 会在你指定的路径 . 当前运行路径上生成这个内部类。我们看下一下生成的类长什么样 ![在这里插入图片描述](https://user-gold-cdn.xitu.io/2019/7/16/16bf81c31b9be099?w=1772&h=286&f=png&s=297718) 从图中可以看出动态生成的内部类实现了我自定义的函数式接口，并且重写了函数式接口中的方法。

我们在javap -v -p StreamTest\$\$Lambda\$1.class看下：

```
{
  private com.lxs.stream.StreamTest$$Lambda$1();
    descriptor: ()V
    flags: ACC_PRIVATE
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #10                 // Method java/lang/Object."<init>":()V
         4: return

  public void print(java.lang.Object);
    descriptor: (Ljava/lang/Object;)V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=2, args_size=2
         0: aload_1
         1: checkcast     #15                 // class java/lang/String
         4: invokestatic  #21                 // Method com/lxs/stream/StreamTest.lambda$main$0:(Ljava/lang/String;)V
         7: return
    RuntimeVisibleAnnotations:
      0: #13()
}
```

发现在重写的parint方法中使用invokestatic指令调用了lambda$main$0方法。

**总结：** 这样实现了Lambda表达式，使用invokedynamic指令，运行时调用LambdaMetafactory.metafactory动态的生成内部类，实现了函数式接口，并在重写函数式接口中的方法，在方法内调用 `lambda$main$0`，内部类里的调用方法块并不是动态生成的，只是在原class里已经编译生成了一个静态的方法，内部类只需要调用该静态方法。