> 点击 [blog](https://github.com/xiaoshuanglee/blog)即可查看原文和更多的文章，欢迎star。

## 什么是Java字节码

![img](https://tva1.sinaimg.cn/large/0081Kckwly1gl7g063uc6j30oy0dpdgf.jpg)

Java字节码是由(.Java)文件编译成(.class)的文件。之所以叫字节码是因为(.class)文件是由十六进制组成的。而JVM以两个十六进制值为一组，即以字节为单位进行读取。java之所以能够做到一次编译、到处运行，就是因为不同的平台都会编译成相同的(.class)文件，所以才能在不同的平台执行。这种跨平台执行的实现，极大的提高了开发和维护的成本。

## 怎么查看字节码

查看字节码有很多种方法，网上也有一些插件可以查看。我们这里直说一种就是通过javap命令来查看。

先通过javap -help来查看下这个命令怎么使用：

```java
用法: javap <options> <classes>
其中, 可能的选项包括:
  -? -h --help -help               输出此帮助消息
  -version                         版本信息
  -v  -verbose                     输出附加信息
  -l                               输出行号和本地变量表
  -public                          仅显示公共类和成员
  -protected                       显示受保护的/公共类和成员
  -package                         显示程序包/受保护的/公共类
                                   和成员 (默认)
  -p  -private                     显示所有类和成员
  -c                               对代码进行反汇编
  -s                               输出内部类型签名
  -sysinfo                         显示正在处理的类的
                                   系统信息 (路径, 大小, 日期, MD5 散列)
  -constants                       显示最终常量
  --module <模块>, -m <模块>       指定包含要反汇编的类的模块
  --module-path <路径>             指定查找应用程序模块的位置
  --system <jdk>                   指定查找系统模块的位置
  --class-path <路径>              指定查找用户类文件的位置
  -classpath <路径>                指定查找用户类文件的位置
  -cp <路径>                       指定查找用户类文件的位置
  -bootclasspath <路径>            覆盖引导类文件的位置

```

接下来我们定义一个简单的类

```java
/**
 * @author: lixiaoshuang
 * @create: 2020-11-30 19:57
 **/
public class HelloByteCode {
    public static void main(String[] args) {
        int a = 1;
        int b = 2;
        int c = a + b;
        System.out.println(c);
    }
}
```

![image-20201130215033695](https://tva1.sinaimg.cn/large/0081Kckwly1gl7jfcifq4j30l608m0vp.jpg)

然后执行 **javac HelloByteCode.java** ，这样就的到了HelloByteCode.class文件，它就是我们所说的字节码文件。编译完成后我们先用文本工具打开(.class)文件看下：

![image-20201130220049549](https://tva1.sinaimg.cn/large/0081Kckwly1gl7jq31pybj30oe0ri43u.jpg)

### 魔数

打开后是一堆十六进制数，可以看到上图中用蓝色框标记起来的cafe babe就是魔数，所有的字节码文件都是以这个为开头的，魔数的固定值为：0xCAFEBABE，魔数放在文件的开头是为了让jvm识别这个文件是不是一个.class文件，如果不是就不会进行下一步的操作。

### 版本号

同样还是上边的字节码图，黄色框圈起来的是版本号，0000 0037，0000为次版本号，0037位主版本，次版本号转化为十进制为0，主版本号转化为十进制为55，通过Oracle官网查询可知，55对应的版本号是jdk 11。



### 查看反编译

接下来使用 **javap -v -l -c HelloByteCode** 命令对(.class)文件进行反编译。具体每一块是干什么我在图中详细的标出来了，大家可以仔细看下面的图片。就不一一介绍了，主要包括版本号、访问标志、接口信息、常量池、方法描述、操作指令、行号表、本地变量表（图中没有体现出来，大家可以用命令将本地变量表输出出来自己看下）

![](https://tva1.sinaimg.cn/large/0081Kckwly1gl7lhcf1ugj30u015jteh.jpg)

![](https://tva1.sinaimg.cn/large/0081Kckwly1gl870t2lelj31bi0hc76t.jpg)

