---
title: JIT即时编译分享
date: 2020-4-29 20:46:02
tags: 生活
---

# 一、即使编译介绍

## 1.1 java代码是如何执行的
我们大家都知道，通常通过 javac 将程序源代码编译，转换成 java 字节码（class文件），JVM 通过解释字节码将其翻译成对应的机器指令（二进制字节码），逐条读入，逐条解释翻译。

读取-翻译-执行

## 1.2 为什么会有即时编译
首先，如果一段代码本身在将来只会被执行一次，那么从本质上看，编译就是在浪费精力。因为将代码翻译成 java 字节码相对于编译这段代码并执行代码来说，要快很多。

Java程序最初是通过解释器进行解释执行的，当虚拟机发现某个方法或代码块运行的特别频繁时，会把这些代码认定为“热点代码”（Hot Spot Code）。为了提高热点代码的执行效率，在运行时，虚拟机会把这些代码编译成本地平台相关的机器码，并进行各种层次的优化，完成这个任务的编译器称为即时编译器（JIT编译器，不是Java虚拟机内必须的部分）

## 1.3 如何开启即时编译的功能
通过参数禁用JIT					
-Djava.compiler=NONE 
-XX:CompileThreshold=2000000000

此外，在程序中也可以即时地禁用和开启JIT。
java.lang.Compiler.disable(); 
java.lang.Compiler.enable(); 

及时编译器：
HotSpot 虚拟机包含多个即时编译器 C1、C2 和 Graal（实验阶段）。
Graal 是一个实验性质的即时编译器，（默认不开启）可以通过参数 
-XX:+UnlockExperimentalVMOptions
-XX:+UseJVMCICompiler 启用，并且替换 C2

# 二、即时编译原理

## 2.1 即时编译器
C1及C2（或称为Client及Server）。两者的区别在于，前者没有应用激进的优化技术，因为这些优化往往伴随着耗时较长的代码分析。因此，C1的编译速度较快，而C2所编译的方法运行速度较快。而Graal采用更加激进的优化方式，因此当程序达到稳定状态后，其执行效率（峰值性能）将更有优势。
举例来说，针对偏好高启动性能的GUI用户端程序则使用C1，针对偏好高峰值性能的服务器端程序则使用C2。

## 2.2 分层编译等级
Java 7引入了tiered compilation的概念，综合了C1的高启动性能及C2的高峰值性能。这两个JIT compiler以及interpreter将HotSpot的执行方式划分为五个级别：

level 0：interpreter解释执行（也会profiling）
level 1：C1编译，无profiling「1」
level 2：C1编译，仅方法及循环及循环回边执行次数的profiling
level 3：C1编译，所有 profiling 
level 4：C2编译

通常情况下，C2 代码的执行效率要比 C1 代码的高出 30% 以上。然而，对于 C1 代码的三种状态，按执行效率从高至低则是 1 层 > 2 层 > 3 层。

「1」profiling
这里解释一下，profiling 是指在程序执行过程中，收集能够反映程序执行状态的数据。这里所收集的数据我们称之为程序的 profile。

方法及循环back-edge执行次数的profiling
branch（针对分支跳转字节码）的profiling
receiver type（针对成员方法调用或类检测，如checkcast，instnaceof，aastore字节码）的profiling

https://www.ibm.com/developerworks/cn/java/j-lo-profiling/index.html

## 2.3 分层编译划分

![图片](http://static.suroot.win/image/jit-1.png)


上图列举了4种编译模式（非全部）。通常情况下，一个方法先被解释执行（level 0），然后被C1编译（level 3），再然后被得到profile数据的C2编译（level 4）。如果编译对象非常简单，虚拟机认为通过C1编译或通过C2编译并无区别，便会直接由C1编译且不插入profiling代码（level 1）。在C1忙碌的情况下，interpreter会触发profiling，而后方法会直接被C2编译；在C2忙碌的情况下，方法则会先由C1编译并保持较少的profiling（level 2），以获取较高的执行效率（与3级相比高30%）。

## 2.4 即时编译的触发
Java 虚拟机是根据方法的调用次数以及循环回边的执行次数来触发即时编译的。前面提到，Java 虚拟机在 0 层、2 层和 3 层执行状态时进行 profiling，其中就包含方法的调用次数和循环回边的执行次数。

这里的循环回边是一个控制流图中的概念。在字节码中，我们可以简单理解为往回跳转的指令。
``` java
public static void foo(Object obj) {
  int sum = 0;
  for (int i = 0; i < 200; i++) {
    sum += i;
  }
}
```
我们用javap 反编译上面的java代码的字节码文件
``` java
public static void foo(java.lang.Object);
  Code:
     0: iconst_0
     1: istore_1
     2: iconst_0
     3: istore_2
     4: goto 14
     7: iload_1
     8: iload_2
     9: iadd
    10: istore_1
    11: iinc 2, 1
    14: iload_2
    15: sipush 200
    18: if_icmplt 7
    21: return
```
其中，偏移量为 18 的字节码将往回跳至偏移量为 7 的字节码中。在解释执行时，每当运行一次该指令，Java 虚拟机便会将该方法的循环回边计数器加 1。
在即时编译过程中，我们会识别循环的头部和尾部。在上面这段字节码中，循环的头部是偏移量为 14 的字节码，尾部为偏移量为 11 的字节码。

实际上，Java 虚拟机并不会对这些计数器进行同步操作，因此收集而来的执行次数也并非精确值。不管如何，即时编译的触发并不需要非常精确的数值。只要该数值足够大，就能说明对应的方法包含热点代码。

具体来说，在不启用分层编译的情况下，当方法的调用次数和循环回边的次数的和，超过由参数 -XX:CompileThreshold 指定的阈值时（使用 C1 时，该值为 1500；使用 C2 时，该值为 10000），便会触发即时编译。

当启用分层编译时，Java 虚拟机将不再采用由参数 -XX:CompileThreshold 指定的阈值（该参数失效），而是使用另一套阈值系统。在这套系统中，阈值的大小是动态调整的。

所谓的动态调整其实并不复杂：在比较阈值时，Java 虚拟机会将阈值与某个系数 s 相乘。该系数与当前待编译的方法数目成正相关，与编译线程的数目成负相关。

系数的计算方法为：
`s = queue_size_X / (TierXLoadFeedback * compiler_count_X) + 1`

其中 X 是执行层次，可取 3 或者 4；
queue_size_X 是执行层次为 X 的待编译方法的数目；
TierXLoadFeedback 是预设好的参数，其中 Tier3LoadFeedback 为 5，Tier4LoadFeedback 为 3；
compiler_count_X 是层次 X 的编译线程数目。

在 64 位 Java 虚拟机中，默认情况下编译线程的总数目是根据处理器数量来调整的（对应参数 -XX:+CICompilerCountPerCPU，默认为 true；当通过参数 -XX:+CICompilerCount=N 强制设定总编译线程数目时，CICompilerCountPerCPU 将被设置为 false）。

Java 虚拟机会将这些编译线程按照 1:2 的比例分配给 C1 和 C2（至少各为 1 个）。举个例子，对于一个四核机器来说，总的编译线程数目为 3，其中包含一个 C1 编译线程和两个 C2 编译线程。

对于四核及以上的机器，总的编译线程的数目为：
`n = log2(N) * log2(log2(N)) * 3 / 2`
其中 N 为 CPU 核心数目。

当启用分层编译时，即时编译具体的触发条件如下：
当方法调用次数大于由参数 -XX:TierXInvocationThreshold 指定的阈值乘以系数，
或者当方法调用次数大于由参数 -XX:TierXMINInvocationThreshold 指定的阈值乘以系数，并且方法调用次数和循环回边次数之和大于由参数 -XX:TierXCompileThreshold 指定的阈值乘以系数时，便会触发 X 层即时编译。

触发条件为：
`i > TierXInvocationThreshold * s || (i > TierXMinInvocationThreshold * s  && i + b > TierXCompileThreshold * s)`

其中 i 为调用次数，b 为循环回边次数。

# 三、 OSR编译

可以看到，决定一个方法是否为热点代码的因素有两个：方法的调用次数、循环回边的执行次数。即时编译便是根据这两个计数器的和来触发的。为什么 Java 虚拟机需要维护两个不同的计数器呢？

实际上，除了以方法为单位的即时编译之外，Java 虚拟机还存在着另一种以循环为单位的即时编译，叫做 On-Stack-Replacement（OSR）编译。循环回边计数器便是用来触发这种类型的编译的。

OSR 实际上是一种技术，它指的是在程序执行过程中，动态地替换掉 Java 方法栈桢，从而使得程序能够在非方法入口处进行解释执行和编译后的代码之间的切换。事实上，去优化（deoptimization）采用的技术也可以称之为 OSR。

在不启用分层编译的情况下，C1 的 OSR 编译的阈值为 13500，而 C2 的为 10700。

在启用分层编译的情况下，触发 OSR 编译的阈值则是由参数 -XX:TierXBackEdgeThreshold 指定的阈值乘以系数。

# 四、 基于分支 profile 的优化

``` java
// java代码
public static int foo(boolean f, int in) {
  int v;
  if (f) {
    v = in;
  } else {
    v = (int) Math.sin(in);
  }
 
  if (v == in) {
    return 0;
  } else {
    return (int) Math.cos(v);
  }
}

```
上面的代码便后后的字节码如下

``` java
// 编译而成的字节码：
public static int foo(boolean, int);
  Code:
     0: iload_0
     1: ifeq          9
     4: iload_1
     5: istore_2
     6: goto          16
     9: iload_1
    10: i2d
    11: invokestatic  java/lang/Math.sin:(D)D
    14: d2i
    15: istore_2
    16: iload_2
    17: iload_1
    18: if_icmpne     23
    21: iconst_0
    22: ireturn
    23: iload_2
    24: i2d
    25: invokestatic java/lang/Math.cos:(D)D
    28: d2i
    29: ireturn
    
```
举个例子，下面这段代码中包含两个条件判断。第一个条件判断将测试所输入的 boolean 值。

如果为 true，则将局部变量 v 设置为所输入的 int 值。如果为 false，则将所输入的 int 值经过一番运算之后，再存入局部变量 v 之中。

第二个条件判断则测试局部变量 v 是否和所输入的 int 值相等。如果相等，则返回 0。如果不等，则将局部变量 v 经过一番运算之后，再将之返回。显然，当所输入的 boolean 值为 true 的情况下，这段代码将返回 0。

![图片](http://static.suroot.win/image/jit-2.png)

假设应用程序调用该方法时，所传入的 boolean 值皆为 true。那么，偏移量为 1 以及偏移量为 18 的条件跳转指令所对应的分支 profile 中，跳转的次数都为 0。

![图片](http://static.suroot.win/image/jit-3.png)

C2 可以根据这两个分支 profile 作出假设，在接下来的执行过程中，这两个条件跳转指令仍旧不会发生跳转。基于这个假设，C2 便不再编译这两个条件跳转语句所对应的 false 分支了。

![图片](http://static.suroot.win/image/jit-4.png)

我们暂且不管当假设错误的时候会发生什么，先来看一看剩下来的代码。经过“剪枝”之后，在第二个条件跳转处，v 的值只有可能为所输入的 int 值。因此，该条件跳转可以进一步被优化掉。最终的结果是，在第一个条件跳转之后，C2 代码将直接返回 0。

总结一下，根据条件跳转指令的分支 profile，即时编译器可以将从未执行过的分支剪掉，以避免编译这些很有可能不会用到的代码，从而节省编译时间以及部署代码所要消耗的内存空间。此外，“剪枝”将精简程序的数据流，从而触发更多的优化。

在现实中，分支 profile 出现仅跳转或者仅不跳转的情况并不多见。当然，即时编译器对分支 profile 的利用也不仅限于“剪枝”。它还会根据分支 profile，计算每一条程序执行路径的概率，以便某些编译器优化优先处理概率较高的路径。

``` java
public static int hash(Object in) {
  if (in instanceof Exception) {
    return System.identityHashCode(in);
  } else {
    return in.hashCode();
  }
}
```
``` java
// 编译而成的字节码：
public static int hash(java.lang.Object);
  Code:
     0: aload_0
     1: instanceof java/lang/Exception
     4: ifeq          12
     7: aload_0
     8: invokestatic java/lang/System.identityHashCode:(Ljava/lang/Object;)I
    11: ireturn
    12: aload_0
    13: invokevirtual java/lang/Object.hashCode:()I
```
另外一个例子则是关于 instanceof 以及方法调用的类型 profile。下面这段代码将测试所传入的对象是否为 Exception 的实例，如果是，则返回它的系统哈希值；如果不是，则返回它的哈希值。

![图片](http://static.suroot.win/image/jit-5.png)

假设应用程序调用该方法时，所传入的 Object 皆为 Integer 实例。那么，偏移量为 1 的 instanceof 指令的类型 profile 仅包含 Integer，偏移量为 4 的分支跳转语句的分支 profile 中不跳转的次数为 0，偏移量为 13 的方法调用指令的类型 profile 仅包含 Integer。

生成的代码将测试所输入的对象的动态类型是否为 Integer。如果是的话，则继续执行接下来的代码。（该优化源自 Graal，采用 C2 可能无法复现。）
然后，即时编译器会采用和第一个例子中一致的针对分支 profile 的优化，以及对方法调用的内联。

``` java
public final class Integer ... {
    @Override
    public int hashCode() {
        return Integer.hashCode(value);
    }
    public static int hashCode(int value) {
        return value;
    }
    ...
}
```
![图片](http://static.suroot.win/image/jit-6.png)

![图片](http://static.suroot.win/image/jit-7.png)

根据数据流分析，上述代码可以最终优化为极其简单的形式。
和基于分支 profile 的优化一样，基于类型 profile 的优化同样也是作出假设，从而精简控制流以及数据流。这两者的核心都是假设。
对于分支 profile，即时编译器假设的是仅执行某一分支；对于类型 profile，即时编译器假设的是对象的动态类型仅为类型 profile 中的那几个。

那么，当假设失败的情况下，程序将何去何从？
Java 虚拟机给出的解决方案便是去优化，即从执行即时编译生成的机器码切换回解释执行。

在生成的机器码中，即时编译器将在假设失败的位置上插入一个陷阱（trap）。该陷阱实际上是一条 call 指令，调用至 Java 虚拟机里专门负责去优化的方法。与普通的 call 指令不一样的是，去优化方法将更改栈上的返回地址，并不再返回即时编译器生成的机器码中。

在上面的程序控制流图中，我画了很多红色方框的问号。这些问号便代表着一个个的陷阱。一旦踏入这些陷阱，便将发生去优化，并切换至解释执行。

最后奉上大纲图
![图片](http://static.suroot.win/image/jit-8.png)
