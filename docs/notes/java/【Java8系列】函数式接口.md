#### 函数式接口是什么？

有且只有一个抽象方法的接口被称为函数式接口，函数式接口适用于函数式编程的场景，Lambda就是Java中函数式编程的体现，可以使用Lambda表达式创建一个函数式接口的对象，一定要确保接口中有且只有一个抽象方法，这样Lambda才能顺利的进行推导。

#### @FunctionalInterface注解

与@Override 注解的作用类似，Java 8中专门为函数式接口引入了一个新的注解：@FunctionalInterface 。该注解可用于一个接口的定义上,一旦使用该注解来定义接口，编译器将会强制检查该接口是否确实有且仅有一个抽象方法，否则将会报错。但是这个注解不是必须的，只要符合函数式接口的定义，那么这个接口就是函数式接口。

#### static方法和default方法

实在不知道该在哪介绍这两个方法了，所以就穿插在这里了。

##### static方法：

java8中为接口新增了一项功能，定义一个或者多个静态方法。用法和普通的static方法一样,例如：

```java
public interface Interface {
    /**
     * 静态方法
     */
    static void staticMethod() {
        System.out.println("static method");
    }
}
```

`注意:实现接口的类或者子接口不会继承接口中的静态方法。`

##### default方法：

java8在接口中新增default方法，是为了在现有的类库中中新增功能而不影响他们的实现类，试想一下，如果不增加默认实现的话，接口的所有实现类都要实现一遍这个方法，这会出现兼容性问题，如果定义了默认实现的话，那么实现类直接调用就可以了，并不需要实现这个方法。default方法怎么定义？

```java
public interface Interface {
    /**
     * default方法
     */
    default void print() {
        System.out.println("hello default");
    }
}
```

`注意：如果接口中的默认方法不能满足某个实现类需要，那么实现类可以覆盖默认方法。不用加default关键字，`例如：

```java
public class InterfaceImpl implements Interface {
    @Override
    public  void print() {
        System.out.println("hello default 2");
    }
}
```

在函数式接口的定义中是只允许有一个抽象方法，但是可以有多个static方法和default方法。


#### 自定义函数式接口

按照下面的格式定义，你也能写出函数式接口：

```java
 @FunctionalInterface
 修饰符 interface 接口名称 {
    返回值类型 方法名称(可选参数信息);
    // 其他非抽象方法内容
 }
```

虽然@FunctionalInterface注解不是必须的，但是自定义函数式接口最好还是都加上，一是养成良好的编程习惯，二是防止他人修改，一看到这个注解就知道是函数式接口，避免他人往接口内添加抽象方法造成不必要的麻烦。

```java
@FunctionalInterface
public interface MyFunction {
    void print(String s);
}
```

看上图是我自定义的一个函数式接口，那么这个接口的作用是什么呢？就是输出一串字符串，属于消费型接口，是模仿Consumer接口写的，只不过这个没有使用泛型，而是将参数具体类型化了，不知道Consumer没关系，下面会介绍到，其实java8中提供了很多常用的函数式接口，Consumer就是其中之一，一般情况下都不需要自己定义，直接使用就好了。那么怎么使用这个自定义的函数式接口呢？我们可以用函数式接口作为参数，调用时传递Lambda表达式。如果一个方法的参数是Lambda，那么这个参数的类型一定是函数式接口。例如：

```java
public class MyFunctionTest {
    public static void main(String[] args) {
        String text = "试试自定义函数好使不";
        printString(text, System.out::print);
    }

    private static void printString(String text, MyFunction myFunction) {
        myFunction.print(text);
    }
}
```

执行以后就会输出“试试自定义函数好使不”这句话，如果某天需求变了，我不想输出这句话了，想输出别的，那么直接替换text就好了。函数式编程是没有副作用的，最大的好处就是函数的内部是无状态的，既输入确定输出就确定。函数式编程还有更多好玩的套路，这就需要靠大家自己探索了。😝

#### 常用函数式接口

##### Consumer`<T>`：消费型接口

**抽象方法：**  void accept(T t)，接收一个参数进行消费，但无需返回结果。

**使用方式：**

```java
  Consumer consumer = System.out::println;
  consumer.accept("hello function");
```

**默认方法：** andThen(Consumer<? super T> after)，先消费然后在消费，先执行调用andThen接口的accept方法，然后在执行andThen方法参数after中的accept方法。

**使用方式：**		

```java
  Consumer<String> consumer1 = s -> System.out.print("车名："+s.split(",")[0]);
  Consumer<String> consumer2 = s -> System.out.println("-->颜色："+s.split(",")[1]);

  String[] strings = {"保时捷,白色", "法拉利,红色"};
  for (String string : strings) {
     consumer1.andThen(consumer2).accept(string);
  }
```

**输出：**
车名：保时捷-->颜色：白色
车名：法拉利-->颜色：红色

##### Supplier`<T>`: 供给型接口 

**抽象方法**：T get()，无参数，有返回值。

**使用方式：**

```java
 Supplier<String> supplier = () -> "我要变的很有钱";
 System.out.println(supplier.get());
```

最后输出就是“我要变得很有钱”，这类接口适合提供数据的场景。

#####  Function`<T,R>`: 函数型接口

**抽象方法：** R apply(T t)，传入一个参数，返回想要的结果。

**使用方式：**

```java
 Function<Integer, Integer> function1 = e -> e * 6;
 System.out.println(function1.apply(2));
```

很简单的一个乘法例子，显然最后输出是12。

**默认方法：**

- compose(Function<? super V, ? extends T> before)，先执行compose方法参数before中的apply方法，然后将执行结果传递给调用compose函数中的apply方法在执行。

**使用方式：**

```java
 Function<Integer, Integer> function1 = e -> e * 2;
 Function<Integer, Integer> function2 = e -> e * e;

 Integer apply2 = function1.compose(function2).apply(3);
 System.out.println(apply2);
```

还是举一个乘法的例子，compose方法执行流程是先执行function2的表达式也就是3*3=9，然后在将执行结果传给function1的表达式也就是9*2=18，所以最终的结果是18。

- andThen(Function<? super R, ? extends V> after)，先执行调用andThen函数的apply方法，然后在将执行结果传递给andThen方法after参数中的apply方法在执行。它和compose方法整好是相反的执行顺序。

**使用方式：**

```java
 Function<Integer, Integer> function1 = e -> e * 2;
 Function<Integer, Integer> function2 = e -> e * e;

 Integer apply3 = function1.andThen(function2).apply(3);
 System.out.println(apply3);
```

这里我们和compose方法使用一个例子，所以是一模一样的例子，由于方法的不同，执行顺序也就不相同，那么结果是大大不同的。andThen方法是先执行function1表达式，也就是3*2=6，然后在执行function2表达式也就是6*6=36。结果就是36。
**静态方法：**identity()，获取一个输入参数和返回结果相同的Function实例。

**使用方式：** 

```java
 Function<Integer, Integer> identity = Function.identity();
 Integer apply = identity.apply(3);
 System.out.println(apply);
```

平常没有遇到过使用这个方法的场景，总之这个方法的作用就是输入什么返回结果就是什么。

##### Predicate`<T>` ： 断言型接口

**抽象方法：** boolean test(T t),传入一个参数，返回一个布尔值。

**使用方式：**

```java
 Predicate<Integer> predicate = t -> t > 0;
 boolean test = predicate.test(1);
 System.out.println(test);
```

当predicate函数调用test方法的时候，就会执行拿test方法的参数进行t -> t > 0的条件判断，1肯定是大于0的，最终结果为true。

**默认方法：**

- and(Predicate<? super T> other)，相当于逻辑运算符中的&&，当两个Predicate函数的返回结果都为true时才返回true。

**使用方式：**

```java
 Predicate<String> predicate1 = s -> s.length() > 0;
 Predicate<String> predicate2 = Objects::nonNull;
 boolean test = predicate1.and(predicate2).test("&&测试");
 System.out.println(test);
```

- or(Predicate<? super T> other) ,相当于逻辑运算符中的||，当两个Predicate函数的返回结果有一个为true则返回true，否则返回false。

**使用方式：**

```java
 Predicate<String> predicate1 = s -> false;
 Predicate<String> predicate2 = Objects::nonNull;
 boolean test = predicate1.and(predicate2).test("||测试");
 System.out.println(test);
```

- negate()，这个方法的意思就是取反。

**使用方式：**

```java
 Predicate<String> predicate = s -> s.length() > 0;
 boolean result = predicate.negate().test("取反");
 System.out.println(result);
```

很明显正常执行test方法的话应该为true，但是调用negate方法后就返回为false了。
**静态方法：**isEqual(Object targetRef)，对当前操作进行"="操作,即取等操作,可以理解为 A == B。

**使用方式:**

```java
 boolean test1 = Predicate.isEqual("test").test("test");
 boolean test2 = Predicate.isEqual("test").test("equal");
 System.out.println(test1);   //true
 System.out.println(test2);   //false
```

#### 其他函数式接口

##### Bi类型接口

BiConsumer、BiFunction、BiPrediate 是 Consumer、Function、Predicate 的扩展，可以传入多个参数，没有 BiSupplier 是因为 Supplier 没有入参。

##### 操作基本数据类型的接口

IntConsumer、IntFunction、IntPredicate、IntSupplier、LongConsumer、LongFunction、LongPredicate、LongSupplier、DoubleConsumer、DoubleFunction、DoublePredicate、DoubleSupplier。
其实常用的函数式接口就那四大接口Consumer、Function、Prediate、Supplier，其他的函数式接口就不一一列举了，有兴趣的可以去java.util.function这个包下详细的看。