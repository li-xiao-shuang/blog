#### Optional类入门

Optional`<T>` 类(java.util.Optional) 是一个容器类，代表一个值存在或不存在，原来用 null 表示一个值不存在，现在Optional可以更好的表达这个概念。并且可以避免空指针异常。你可以把Optional对象看成一种特殊的集合数据，它至多包含一个元素。

**常用方法**:

- Optional.of(T t) : 将指定值用 Optional 封装之后返回，如果该值为 null，则抛出一个 NullPointerException 异常。
- Optional.empty() : 创建一个空的 Optional 实例。
- Optional.ofNullable(T t) : 将指定值用 Optional 封装之后返回，如果该值为 null，则返回一个空的 Optional 对象。
- get() : 如果该值存在，将该值用 Optional 封装返回，否则抛出一个 NoSuchElementException 异常。
- orElse(T t) : 如果调用对象包含值，返回该值，否则返回t。
- orElseGet(Supplier s) : 如果调用对象包含值，返回该值，否则返回 s 获取的值。
- orElseThrow() ：它会在对象为空的时候抛出异常。
- map(Function f) : 如果值存在，就对该值执行提供的 mapping 函数调用。
- flatMap(Function mapper) : 如果值存在，就对该值执行提供的mapping 函数调用，返回一个 Optional 类型的值，否则就返回一个空的 Optional 对象。

`注意：Optional类的设计初衷仅仅是要支持能返回Optional对象的语法，并未考虑作为类的字段使用，也没有实现序列化接口，在领域模型中使用Optional，有可能引发程序故障。`

#### 使用Optional实战

用Optional封装可能为null的值，我们在项目中很多时候都会遇到，掉一个方法然后返回一个null，最后需要不断的判空。比如获取Map中的不含指定键的值，它的get方法返回的就是一个null。

```java
 //例如：
 Object value = map.get("key");
 
 //使用Optional封装结果后可以这么写：
 Optional<Object> value = Optional.ofNullable(map.get("key"));
 
 /**
 * 如果想在获取为null以后给个默认值，可以这么写：
 * orElse和orElseGet的区别是当Optional的值是空值时，无论orElse还是orElseGet都会执行；而当返回的             Optional有值时，orElse会执行，而orElseGet不会执行。
 */
 Object value = Optional.ofNullable(map.get("key")).orElse("value");
 Object value1 = Optional.ofNullable(map.get("key")).orElseGet(()->"value");
```

 由于某种原因，函数无法返回某个值，这时除了返回null，Java API比较常见的替代做法是抛出一个异常。这种情况比较典型的例子是使用静态方法Integer.parseInt(String)，将String转换为int。在这个例子中，如果String无法解析到对应的整型，该方法就抛出一个NumberFormatException。最后的效果是，发生String无法转换为int时，代码发出一个遭遇非法参数的信号，唯一的不同是，这次你需要使用try/catch 语句，而不是使用if条件判断来控制一个变量的值是否非空。

你也可以用空的Optional对象，对遭遇无法转换的String时返回的非法值进行建模，这时你期望parseInt的返回值是一个optional。我们无法修改最初的Java方法，但是这无碍我们进 行需要的改进，你可以实现一个工具方法，将这部分逻辑封装于其中，最终返回一个我们希望的 Optional对象，代码如下所示。

```java
public static Optional<Integer> stringToInt(String s) {
try { 
        //如果String能转换为对应的Integer，将其封装在Optioal对象中返回
        return Optional.of(Integer.parseInt(s));
    } catch (NumberFormatException e) {
        //否则返回一个空的Optional对象
        return Optional.empty();
    }
}
```

Optional就是讲到这里，这个实在没什么好说的了，大家自己实践吧。