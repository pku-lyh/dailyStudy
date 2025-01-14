# 类加载过程详解
## 类的生命周期
类从被加载到虚拟机内存中开始到卸载出内存为止，它的整个生命周期可以简单概括为 7 个阶段：加载（Loading）、验证（Verification）、准备（Preparation）、解析（Resolution）、初始化（Initialization）、使用（Using）和卸载（Unloading）。其中，验证、准备和解析这三个阶段可以统称为连接（Linking）。

这 7 个阶段的顺序如下图所示：

![](https://cdn.nlark.com/yuque/0/2025/png/39185937/1736690804087-70a81de0-fd6d-491a-94c9-5aa1bae8553e.png)

## 类加载过程
**Class 文件需要加载到虚拟机中之后才能运行和使用，那么虚拟机是如何加载这些 Class 文件呢？**

系统加载 Class 类型的文件主要三步：**加载->连接->初始化**。连接过程又可分为三步：**验证->准备->解析**。

![](https://cdn.nlark.com/yuque/0/2025/png/39185937/1736690907889-20bc1cac-edae-4875-b2ce-62911b6ccf5d.png)



### 加载
1. <font style="color:rgb(60, 60, 67);">通过全类名获取定义此类的二进制字节流。</font>
2. <font style="color:rgb(60, 60, 67);">将字节流所代表的静态存储结构转换为方法区的运行时数据结构。</font>
3. <font style="color:rgb(60, 60, 67);">在内存中生成一个代表该类的 </font>`<font style="color:rgb(60, 60, 67);">Class</font>`<font style="color:rgb(60, 60, 67);"> 对象，作为方法区这些数据的访问入口。</font>

### <font style="color:rgb(60, 60, 67);">验证</font>
**<font style="color:rgb(60, 60, 67);">验证是连接阶段的第一步，这一阶段的目的是确保 Class 文件的字节流中包含的信息符合《Java 虚拟机规范》的全部约束要求，保证这些信息被当作代码运行后不会危害虚拟机自身的安全。</font>**

<font style="color:rgb(60, 60, 67);">验证阶段主要由</font>**<font style="color:rgb(60, 60, 67);">四个检验阶段</font>**<font style="color:rgb(60, 60, 67);">组成：</font>

1. <font style="color:rgb(60, 60, 67);">文件格式验证（Class 文件格式检查）</font>
2. <font style="color:rgb(60, 60, 67);">元数据验证（字节码语义检查）</font>
3. <font style="color:rgb(60, 60, 67);">字节码验证（程序语义检查）</font>
4. <font style="color:rgb(60, 60, 67);">符号引用验证（类的正确性检查）</font>

![](https://cdn.nlark.com/yuque/0/2025/png/39185937/1736691071031-ad6335cd-cd3a-4016-bb6b-b91e2d624cc4.png)

### 准备
**准备阶段是正式为类变量分配内存并设置类变量初始值的阶段**，这些内存都将在方法区中分配。对于该阶段有以下几点需要注意：

1. 这时候进行内存分配的仅包括类变量（ Class Variables ，即静态变量，被 `static` 关键字修饰的变量，只与类相关，因此被称为类变量），而不包括实例变量。实例变量会在对象实例化时随着对象一块分配在 Java 堆中。
2. 从概念上讲，类变量所使用的内存都应当在 **方法区** 中进行分配。不过有一点需要注意的是：JDK 7 之前，HotSpot 使用永久代来实现方法区的时候，实现是完全符合这种逻辑概念的。 而在 JDK 7 及之后，HotSpot 已经把原本放在永久代的字符串常量池、静态变量等移动到堆中，这个时候类变量则会随着 Class 对象一起存放在 Java 堆中。
3. 这里所设置的初始值"通常情况"下是数据类型默认的零值（如 0、0L、null、false 等），比如我们定义了`public static int value=111` ，那么 value 变量在准备阶段的初始值就是 0 而不是 111（初始化阶段才会赋值）。特殊情况：比如给 value 变量加上了 final 关键字`public static final int value=111` ，那么准备阶段 value 的值就被赋值为 111。

### <font style="color:rgb(60, 60, 67);">解析</font>
**<font style="color:rgb(60, 60, 67);">解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程。</font>**<font style="color:rgb(60, 60, 67);"> 解析动作主要针对类或接口、字段、类方法、接口方法、方法类型、方法句柄和调用限定符 7 类符号引用进行。</font>

### 初始化
**初始化阶段是执行初始化方法 **`**<clinit> ()**`**方法的过程，是类加载的最后一步，这一步 JVM 才开始真正执行类中定义的 Java 程序代码(字节码)。**

<font style="color:rgb(60, 60, 67);">说明：</font>`<font style="color:rgb(60, 60, 67);"><clinit> ()</font>`<font style="color:rgb(60, 60, 67);">方法是编译之后自动生成的。</font>

<font style="color:rgb(60, 60, 67);">对于</font>`<font style="color:rgb(60, 60, 67);"><clinit> ()</font>`<font style="color:rgb(60, 60, 67);"> 方法的调用，虚拟机会自己确保其在多线程环境中的安全性。因为 </font>`<font style="color:rgb(60, 60, 67);"><clinit> ()</font>`<font style="color:rgb(60, 60, 67);"> 方法是带锁线程安全，所以在多线程环境下进行类初始化的话可能会引起多个线程阻塞，并且这种阻塞很难被发现。</font>

<font style="color:rgb(60, 60, 67);">对于初始化阶段，虚拟机严格规范了有且只有 6 种情况下，必须对类进行初始化(只有主动去使用类才会初始化类)：</font>

1. 当遇到 `new`、 `getstatic`、`putstatic` 或 `invokestatic` 这 4 条字节码指令时，比如 `new` 一个类，读取一个静态字段(未被 final 修饰)、或调用一个类的静态方法时。 
    - 当 jvm 执行 `new` 指令时会初始化类。即当程序创建一个类的实例对象。
    - 当 jvm 执行 `getstatic` 指令时会初始化类。即程序访问类的静态变量(不是静态常量，常量会被加载到运行时常量池)。
    - 当 jvm 执行 `putstatic` 指令时会初始化类。即程序给类的静态变量赋值。
    - 当 jvm 执行 `invokestatic` 指令时会初始化类。即程序调用类的静态方法。
2. 使用 `java.lang.reflect` 包的方法对类进行反射调用时如 `Class.forName("...")`, `newInstance()` 等等。如果类没初始化，需要触发其初始化。
3. 初始化一个类，如果其父类还未初始化，则先触发该父类的初始化。
4. 当虚拟机启动时，用户需要定义一个要执行的主类 (包含 `main` 方法的那个类)，虚拟机会先初始化这个类。
5. `MethodHandle` 和 `VarHandle` 可以看作是轻量级的反射调用机制，而要想使用这 2 个调用，  
就必须先使用 `findStaticVarHandle` 来初始化要调用的类。
6. 当一个接口中定义了 JDK8 新加入的默认方法（被 default 关键字修饰的接口方法）时，如果有这个接口的实现类发生了初始化，那该接口要在其之前被初始化。

## 类卸载
**卸载类即该类的 Class 对象被 GC。**

卸载类需要满足 3 个要求:

1. 该类的所有的实例对象都已被 GC，也就是说堆不存在该类的实例对象。
2. 该类没有在其他任何地方被引用
3. 该类的类加载器的实例已被 GC

所以，在 JVM 生命周期内，由 jvm 自带的类加载器加载的类是不会被卸载的。但是由我们自定义的类加载器加载的类是可能被卸载的。

只要想通一点就好了，JDK 自带的 `BootstrapClassLoader`, `ExtClassLoader`, `AppClassLoader` 负责加载 JDK 提供的类，所以它们(类加载器的实例)肯定不会被回收。而我们自定义的类加载器的实例是可以被回收的，所以使用我们自定义加载器加载的类是可以被卸载掉的。

# 类加载器详解
## 类加载器介绍
类加载器是一个负责加载类的对象。`ClassLoader` 是一个抽象类。给定类的二进制名称，类加载器应尝试定位或生成构成类定义的数据。典型的策略是将名称转换为文件名，然后从文件系统中读取该名称的“类文件”。

每个 Java 类都有一个引用指向加载它的 `ClassLoader`。不过，数组类不是通过 `ClassLoader` 创建的，而是 JVM 在需要的时候自动创建的，数组类通过`getClassLoader()`方法获取 `ClassLoader` 的时候和该数组的元素类型的 `ClassLoader` 是一致的。

## 类加载器加载规则
JVM 启动的时候，并不会一次性加载所有的类，而是根据需要去动态加载。也就是说，大部分类在具体用到的时候才会去加载，这样对内存更加友好。

对于已经加载的类会被放在 `ClassLoader` 中。在类加载的时候，系统会首先判断当前类是否被加载过。已经被加载的类会直接返回，否则才会尝试加载。也就是说，对于一个类加载器来说，相同二进制名称的类只会被加载一次。

## 类加载器总结
JVM 中内置了三个重要的 `ClassLoader`：

1. `**BootstrapClassLoader**`**(启动类加载器)**：最顶层的加载类，由 C++实现，通常表示为 null，并且没有父级，主要用来加载 JDK 内部的核心类库（ `%JAVA_HOME%/lib`目录下的 `rt.jar`、`resources.jar`、`charsets.jar`等 jar 包和类）以及被 `-Xbootclasspath`参数指定的路径下的所有类。
2. `**ExtensionClassLoader**`**(扩展类加载器)**：主要负责加载 `%JRE_HOME%/lib/ext` 目录下的 jar 包和类以及被 `java.ext.dirs` 系统变量所指定的路径下的所有类。
3. `**AppClassLoader**`**(应用程序类加载器)**：面向我们用户的加载器，负责加载当前应用 classpath 下的所有 jar 包和类。

除了这三种类加载器之外，用户还可以加入自定义的类加载器来进行拓展，以满足自己的特殊需求。就比如说，我们可以对 Java 类的字节码（ `.class` 文件）进行加密，加载时再利用自定义的类加载器对其解密。

![](https://cdn.nlark.com/yuque/0/2025/png/39185937/1736845891790-fe32493b-14d5-4a67-8138-0aed372fb1ca.png)

除了 `BootstrapClassLoader` 是 JVM 自身的一部分之外，其他所有的类加载器都是在 JVM 外部实现的，并且全都继承自 `ClassLoader`抽象类。这样做的好处是用户可以自定义类加载器，以便让应用程序自己决定如何去获取所需的类。

每个 `ClassLoader` 可以通过`getParent()`获取其父 `ClassLoader`，如果获取到 `ClassLoader` 为`null`的话，那么该类是通过 `BootstrapClassLoader` 加载的。

**为什么 获取到 **`**ClassLoader**`** 为**`**null**`**就是 **`**BootstrapClassLoader**`** 加载的呢？** 这是因为`BootstrapClassLoader` 由 C++ 实现，由于这个 C++ 实现的类加载器在 Java 中是没有与之对应的类的，所以拿到的结果是 null。

# <font style="color:rgb(60, 60, 67);">双亲委派模型</font>
## <font style="color:rgb(60, 60, 67);">双亲委派模型介绍</font>
`ClassLoader` 类使用委托模型来搜索类和资源。每个 `ClassLoader` 实例都有一个相关的父类加载器。需要查找类或资源时，`ClassLoader` 实例会在试图亲自查找类或资源之前，将搜索类或资源的任务委托给其父类加载器。  
虚拟机中被称为 "bootstrap class loader"的内置类加载器本身没有父类加载器，但是可以作为 `ClassLoader` 实例的父类加载器。

## 双亲委派模型的好处
双亲委派模型保证了 Java 程序的稳定运行，可以避免类的重复加载（JVM 区分不同类的方式不仅仅根据类名，相同的类文件被不同的类加载器加载产生的是两个不同的类），也保证了 Java 的核心 API 不被篡改。

如果没有使用双亲委派模型，而是每个类加载器加载自己的话就会出现一些问题，比如我们编写一个称为 `java.lang.Object` 类的话，那么程序运行的时候，系统就会出现两个不同的 `Object` 类。双亲委派模型可以保证加载的是 JRE 里的那个 `Object` 类，而不是你写的 `Object` 类。这是因为 `AppClassLoader` 在加载你的 `Object` 类时，会委托给 `ExtClassLoader` 去加载，而 `ExtClassLoader` 又会委托给 `BootstrapClassLoader`，`BootstrapClassLoader` 发现自己已经加载过了 `Object` 类，会直接返回，不会去加载你写的 `Object` 类。

