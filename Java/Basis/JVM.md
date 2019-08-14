---
title: JVM
date: 2019/05/01 11:39
categories:
- Java
tags:
- Java
- JVM
---

## 简介

### JVM 组成架构

![UTOOLS1556684648062.png](https://i.loli.net/2019/05/01/5cc91f6a9de2f.png)

1. 类加载器

   类加载器主要用于定位类定义的二进制信息, 然后将这些信息解析并加载至虚拟机. 类加载器在方法区构造具有这个类的信息的数据结构后, 会在堆上创建一个 Class 对象作为访问这个数据结构的接口.  同时, 类加载器还会初始化类的静态数据, 也就是调用类的 `<clinit>()` 方法.

2. 运行时数据区

   1. 堆: 线程共享. 存放对象及数组实例, 也就是运行期间 `new` 出来的对象. 
   2. 方法区: 线程共享. 存放*类型信息*和*运行时常量池*. 
   3. 程序计数器PC(Program Counter): 线程私有. 生命周期与线程相同, 是对 CPU 中 PC 的一种模拟, 如果正在执行 Java 方法, 存放下一条*字节码指令*的地址; 执行 Native 方法, 存放的值为空.
   4. Java 栈: 线程私有. 生命周期与线程相同, 栈中存放*栈帧*(用于进行方法调用和返回), *局部变量*以及计算的中间结果.
   5. 本地方法栈: 用于支持**本地方法**调用.

3. 执行引擎

   以*指令*为单位读取 Java 字节码. 像 CPU 一样, 一条一条地执行*机器指令*. 每个字节码指令都由一个 **1字节** 的*操作码*和附加的*操作数*组成. 执行引擎取得一个操作码, 然后根据*操作数*来执行任务, 完成后就继续执行下一条**操作码**.

4. 垃圾回收器

   用于管理*运行时数据区*的分配和释放. 

## Java 类的加载机制

类的加载机制指的是将类的 .class 文件中的二进制数据读取到内存中, 将其存放在*运行时数据区*的*方法区*内, 然后在*堆*创建一个 `java.lang.Class` 对象, 用来封装类在*方法区*类的数据结构. 类的加载的最终结果是位于*堆*中的 `Class` 对象, 其封装了类在*方法区*内的数据结构, 并且对外提供了访问*方法区*类的数据结构的接口.

![UTOOLS1556691074767.png](https://i.loli.net/2019/05/01/5cc9388438f90.png)

类加载器并不需要等到某个类被**首次主动使用**时再加载它, JVM 规范允许类加载器在预料某个类将要被使用时就预先加载它, 如果在预先加载的过程中遇到了 .class 文件缺失或存在错误, 类加载器必须在程序首次主动使用该类时才报告错误. 如果这个类一直没有被程序主动使用, 那么类加载器就不会报错.

### 类的生命周期

![UTOOLS1556691416326.png](https://i.loli.net/2019/05/01/5cc939d845bf8.png)

#### 类的加载过程

类的加载过程包括了: 加载, 验证, 准备, 解析, 初始化五个阶段. 在这五个阶段中, 加载, 验证, 准备, 初始化和卸载这五个阶段发生的顺序是确定的. 解析阶段可以在初始化之后再开始, 这是为了支持 Java 的*运行时绑定*(也称为 *动态绑定*或*晚期绑定*).

##### 加载

将 Java 字节码数据从不同的数据源读取到 JVM 中, 并映射为 JVM 认可的数据结构(Class 对象). 在加载阶段, 虚拟机需要完成以下三件事:

1. 通过一个类的全限定类名来获取此类的二进制字节流.
2. 将这个字节流所代表的静态存储结构转化为*方法区*的运行时数据结构.
3. 在 Java 堆中生成一个代表这个类的 `java.lang.Class` 对象, 作为对*方法区*这些 数据的访问的入口.

加载阶段是可控性最强的阶段, 开发人员可以使用自定义自己的类加载器来完成类的加载.

加载阶段完成后, 虚拟机外部的二进制字节流就按照虚拟机所需的格式存储在*方法区*之中, 而且在*堆*中也创建一个 `java.lang.Class` 类的对象, 这样便可以通过该对象访问*方法区*中的这些数据.

##### 连接

连接就是将已读入到内存的类的二进制数据合并到虚拟机的*运行时环境*中, 即把原始的类定义信息平滑地转化入 JVM 运行中. 该阶段可分为三个步骤: 

1. 验证: 虚拟机安全的重要保障, JVm 需要核验字节信息是符合 Java 虚拟机规范的, 否则就被认为是 VerifyError, 确保被加载类的正确性. 验证阶段有可能触发更多 class 的加载.
2. 准备: 为类的静态变量分配内存, 并将其初始化为默认值.
3. 解析: 将常量池中的符号引用替换为直接引用.

##### 验证

验证是连接阶段的第一步, 这一阶段的目的是为了确保 Class 文件的字节流中包含的信息符合当前虚拟机的要求, 并且不回危害到虚拟机自身的安全. 验证阶段大致会完成 4 个阶段的检验动作.

1. **文件格式验证**: 验证字节流是否符合 Class 文件格式的规范. 例如: 是否以 `0XCAFEBABE` 开头, 主次版本号是否在当前虚拟机的处理范围之内, 常量池中的常量是否有不被支持的类型.
2. **元数据验证**: 对字节码描述的信息进行语义分析, 确定程序语义是合法的, 符合逻辑的.
3. **字节码验证**: 通过数据流和控制流分析, 确定程序语义是合法的, 符合逻辑的.
4. **符号引用验证**: 确保解析动作能正确执行.

验证阶段是非常重要的, 但不是必须的, 它对程序运行期没有影响, 如果所引用的类经过反复验证, 那么可以考虑采用 `-Xverifynone` 参数来关闭大部分的类验证措施, 以缩短虚拟机类加载的时间.

##### 准备

准备阶段是正式为类变量分配内存并设置类变量初始值的阶段, 这些内存都将在*方法区*中分配. 对于该阶段有以下几点需要注意:

1. 这时进行内存分配的仅包括类变量(static), 而不包括实例变量, 实例变量会在对象实例化时随着对象一块分配在*堆*中.

2. 这里所设置的初始值通常情况下是数据类型默认的*零值*(如 int->0, long->0L, Object->null, boolean->false 等), 而不是在 Java 代码中被显式地赋予的值.

   假设一个类变量的定义为: `publib static int val = 3;`

   那么变量 val 在准备阶段过后的初始值为 0, 而不是 3. 因为这时候尚未开始执行任何 Java 方法, 而把 val 赋值为 3 的 `public static` 指令是在程序编译后, 存放于类构造器 `<clinit>()` 方法中的, 所以把 val 赋值为 3 的动作将在初始化阶段才会执行.

   > 注意: 
   >
   > - 对于*基本数据类型*来说, 对于*类变量*(static)和*全局变量*, 如果不显式地对其赋值而直接使用, 系统会为其赋予默认的零值, 而对于*局部变量*来说, 在使用前必须显式地为其赋值, 否则编译时不通过.
   > - 对于*引用数据类型*来说, 如果没有对其进行显式地赋值而直接使用, 系统都会为其赋予默认的零值.
   > - 如果在数组初始化时没有对数组中的个元素赋值, 那么其中的元素将根据对应的数据类型而被赋予默认的零值.

3. 如果类字段的字段属性表中存在 `ConstantValue` 属性, 即同时被 *final* 和 *static* 修饰, 那么在准备阶段变量就会被初始化为 ConstantValue 属性所指定的值.

   假设一个类变量被定义为: `public static final int val = 3;`
   
   编译时 Javac 将会为 val 生成 `ConstanValue` 属性, 在准备阶段虚拟机就会根据 `ConstantValue` 的设置将 val 赋值为 3. 可以理解为 *static final* 常量在编译器就将其结果放入了调用它的类的常量池中.

##### 解析

解析阶段是虚拟机将*常量池*内的*符号引用*替换为*直接引用*的过程. 解析动作主要针对*类*或*接口*, *字段*, *类方法*, *接口方法*, *方法类型*, *方法句柄*和*调用点限定符* 7 类*符号引用*进行. 

*符号引用*就是一组符号来描述目标, 可以使任何字面量.

*直接引用*就是直接指向目标的指针, 相对偏移量或一个间接定位到目标的句柄.

##### 初始化

初始化为类的静态变量赋予正确的初始值. JVM 负责对类进行初始化, 主要对类变量进行初始化, 父类型的初始化逻辑优先于当前类型的逻辑. 在 Java 中对类变量进行初始值设定有两种方式:

1. 在声明类变量时为其指定初始值.
2. 使用静态代码块为类变量指定初始值.

两者的顺序取决于在代码中的先后顺序. 如:

```java
private static String name = "Anny";
static{
    name = "Bob";
}
// 结果
// name = "Bob";
// ----------------------------------
static {
    name = "Bob";
}
private static String name = "Anny";
// 结果
// name = "Anny";
```

**JVM 初始化步骤**:

1. 如果这个类还没有被加载和连接, 程序先加载并连接该类.
2. 如果该类的直接父类还没有被初始化, 则先初始化其直接父类.
3. 如果类中有初始化语句, 则系统依次执行这些初始化语句.

**类初始化时机**: 只有当对类的主动使用的时候才会导致类的初始化, 类的主动使用包括以下 5 中情况:

1. 遇到 `new`, `getstatic`, `putstatic` 或 `invokestatic` 这 4 条字节码指令时没初始化则触发初始化. 使用场景: 使用 `new` 关键字实例化对象, 读取一个类的静态字段(被 *static final* 修饰, 已在编译器把结果放入常量池的静态字段除外), 调用一个类的静态方法.
2. 使用 `java.lang.reflect` 包的方法对类进行反射调用的时候.
3. 当初始化一个类的时候, 如果发现其父类还没有进行初始化, 则需先触发其父类的初始化.
4. 当虚拟机启动时, 用户需指定一个要加载的*主类*(包含 `main()` 方法的那个类), 虚拟机会先初始化那个类.
5. 当使用 JDK1.7 的动态语言支持时, 如果一个 `java.lang.invoke.MethodHandle` 实例最后的解析结果 `REF_getStatic`, `REF_putStatic`, `REF_invokeStatic` 的方法句柄, 并且这个方法句柄所对应的的类没有进行过初始化, 则需先触发其初始化.

除此以外, 所有引用类的方法都不会触发初始化, 叫做被动引用. 如:

```java
public class SuperClass {
    static {
        System.out.println("SuperClass init!");
    }
    public static int value = 1127;
}

public class SubClass extends SuperClass {
    static {
        System.out.println("SubClass init!");
    }
}

public class ConstClass {
    static {
        System.out.println("ConstClass init!");
    }
    public static final String HELLOWORLD = "hello world!"
}

public class NotInitialization {
    public static void main(String[] args) {
        System.out.println(SubClass.value);
        /**
         *  output : SuperClass init!
         * 
         * 通过子类引用父类的静态对象不会导致子类的初始化
         * 只有直接定义这个字段的类才会被初始化
         */

        SuperClass[] sca = new SuperClass[10];
        /**
         *  output : 
         * 
         * 通过数组定义来引用类不会触发此类的初始化
         * 虚拟机在运行时动态创建了一个数组类
         */

        System.out.println(ConstClass.HELLOWORLD);
        /**
         *  output : 
         * 
         * 常量在编译阶段会存入调用类的常量池当中，本质上并没有直接引用到定义类常量的类，
         * 因此不会触发定义常量的类的初始化。
         * “hello world” 在编译期常量传播优化时已经存储到 NotInitialization 常量池中了。
         */
    }
}
```

在准备阶段, 变量都被赋予了初始值, 但是到了初始化阶段, 所有变量还要按照用户编写的代码进一步初始化. 也可以说, 初始化阶段是执行类构造器 `<clinit>()` 方法的过程.

`<clinit>()` 方法是由编译器自动收集类中定义的所有*类变量*的**赋值动作**和*静态语句块*(static 语句块)中的语句**合并生成的**. 编译器**收集的顺序**是由语句在源文件中**出现的顺序**决定的. *静态语句块*中只能访问到定义在*静态语句块*之前的变量; 定义在它之后的变量, 在前面的*静态语句块*中可以赋值, 但是不能访问.

`<clinit>()` 方法与构造函数 `<init>()` 方法不同, 它不需要显式地调用父类构造器, 虚拟机会保证在子类的 `<clinit>()` 方法执行之前, 父类的 `<clinit>()` 已经执行完毕. 因此, 在虚拟机中第一个被执行的 `<clinit>()` 方法一定是 `java.lang.Object` 的.

如果一个类中既没有*静态语句块*, 也没有*静态变量*赋值动作, 那么编译器都不会为类生成 `<clinit>()` 方法.

虚拟机会保证一个类的 `<clinit>()` 方法在**多线程环境中能正确的加锁**, 同步. 如果多个线程初始化一个类, 那么只有一个线程回去执行 `<clinit>()` 方法, 其他线程都需要等待.

##### 结束生命周期:

在以下几种情况下, Java 虚拟机将结束生命周期:

1. 执行了 `System.exit()`方法.
2. 程序正常执行结束.
3. 程序在执行过程中遇到了异常或错误而异常终止.
4. 由于操作系统出现错误而导致 Java 虚拟机进程终止.

### 类加载器

虚拟机设计团队把类加载阶段中的"通过一个类的全限定类名来获取此类的二进制字节流"这个步骤放到 Java 虚拟机外部去实现, 以便让应用程序自己决定如何去获取所需要的类. 实现这个动作的代码模块称为*类加载器*.

> 从 Java 虚拟机的角度来说, 只存在两种类加载器: 一种是启动类加载器(C++ 实现, 是虚拟机的一部分); 另一种就是其他所有的类加载器(Java 语言实现, 独立于虚拟机外部, 且全部继承自抽象类 `java.lang.ClassLoader`).

从开发人员的角度来看, 类加载器可以划分为以下 3 种:

![UTOOLS1556698530365.png](https://i.loli.net/2019/05/01/5cc955a256673.png)

> 注意: 这里的父类加载器并不是通过*继承*来实现的, 而是采用组合实现的.

1. 启动类加载器(Bootstrap ClassLoader): 负责加载存放在 `JAVA_HOME/JRE/lib` 目录中的, 或被 `-Xbootclasspath` 参数指定的路径中的, 并且能被虚拟机识别的类库(如 rt.jar, 所有的 `java.` 开头的类均被 Bootstrap ClassLoader 加载). 启动类加载器时无法被 Java 程序直接引用的. 通过 `ClassLoader.getParent()` 方法获取会得到 `null`.
2. 扩展类加载器(Extension ClassLoader): 这个加载器由 `sun.misc.Launcher$ExtClassLoader` 实现, 它负责加载 `Java_HOME/JRE/lib/ext` 目录中的, 或者被 `java.ext.dirs` 系统变量所指定的路径中的所有类库(如 `javax.` 开头的类). 开发者可以直接使用扩展类加载器.
3. 应用程序类加载器(Application ClassLoader): 该类加载器由 `sun.misc.Launcher$AppClassLoader` 来实现, 它负责加载用户类路径(classpath)所指定的类. 开发者可以直接使用该类加载器, 如果应用程序中没有自定义过自己的类加载器, 一般情况下这个就是程序中默认的类加载器.

应用程序都是由这三种类加载器相互配合进行加载的, 如果有必要, 还可以加入自定义的类加载器. 因为 JVM 自带的 ClassLoader 只会从本地文件系统中加载标准的 Java Class 文件, 因此如果编写了自定义的 ClassLoader, 可以实现以下几点: 

1. 在执行非置信代码之前, 自动验证数字签名.
2. 动态地创建符合用户特定需要的定制化构建类.
3. 从特定的场所取得 Java Class, 如数据库和网络等.



#### JVM 类加载机制

- **全盘负责**: 当一个类加载器负责加载某个 Class 时, 该 Class 所依赖的和引用的其他 Class 也将由该类加载器负责载入, 除非显示使用另外一个类加载器来载入.
- **父类委托**: 先让父类加载器试图加载该类, 只有在父类加载器无法加载该类时才尝试从自己的类路径中加载该类.
- **缓存机制**: 缓存机制将会保证所有加载过的 Class 都会被缓存, 当程序中需要使用到某个 Class 时, 类加载器会先从缓存区寻找该 Class, 只有缓存区不存在, 系统才会读取该类对应的二进制数据, 并将其转换成 Class 对象, 存入缓存区. 这也就解释了为什么修改 Class 后必须重启 JVM, 程序的修改才会生效.

#### 类的加载

类加载有三种方式:

1. 命令行启动应用时由 JVM 初始化加载.

2. 通过 `Class.forName()` 方法动态加载.

   ```java
   Class.forName("com.mysql.jdbc.Driver");
   ```

3. 通过 `ClassLoader.loadClass()` 方法动态加载.

   ```java
   ClassLoader loader = TestClass.class.getClassLoader();
   loader.loadClass("TestClass");
   // -----------------------
   public class TestClass{
   	static{
   		System.out.println("TestClass init...")
   	}
   }
   ```

**Class.forName() 和 ClassLoader.loadClass() 区别**

`Class.forName()` 方法:

```java
   public static Class<?> forName(String name, boolean initialize,
                                   ClassLoader loader)
```

参数的含义是:

- name: 要加载的 Class 的全限定类名.
- initialize: 是否初始化.
- loader: 指定的 ClassLoader.

`ClassLoader.loadClass()` 方法:

```java
protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
```

- 参数的含义是:
- name: 要加载的 Class 的全限定类名.
- resolve: 是否要进行*连接*.

两个方法的比较: 

- `Class.forName(className)`: 调用的是 `Class.forName(className,true,classLoader)`, 意思是在加**载类之后必须初始化**. 也就是说, 在执行过此方法后, 目标对象的*静态代码块*已经被执行, *静态变量*也已经被初始化.
- `ClassLoader.loadClass(className)`: 调用的是 `ClassLoader.loadClass(className,false)`, 意思是**目标对象被装载后不进行连接**, 也就意味着不会去执行该类*静态代码块*, *静态变量*还是默认值. 在使用 `newInstance()` 创建实例时才连接.

#### 双亲委派模型

双亲委派模型的工作流程是: 如果一个类加载器收到了类加载的请求, 它首先不会去尝试加载这个类, 而是把请求委托给父加载器去完成. 依次向上. 因此, **所有的类加载请求**最终都应该被传递到顶层的*启动类加载器*中, 只有当父加载器在它的搜索范围中没有找到所需的类时, 即无法完成该加载, 只加载器才会尝试自己去加载该类. 

双亲委派机制:

1. 当 `AppClassLoader` 加载一个 Class 时, 它首先不会自己尝试加载这个类, 而是把类加载请求委派给 `ExtClassLoader` 去完成.
2. 当 `ExtClassLoader` 加载一个 Class 时, 它首先不会自己尝试加载这个类, 而是把类加载请求委派给 `BootstrapClassLoader` 去完成.
3. 如果 `BootstrapClassLoader` 加载失败(例如在 `$JAVA_HOME/jre/lib` 中未查找到该 Class), 会让 `ExtClassLoader` 来尝试加载.
4. 如果 `ExtClassLoader` 加载失败, 则会使用 `AppClassLoader` 来加载, 如果 `AppClassLoader` 也加载失败, 则会报异常 `ClassNotFoundException`.

![双亲委派模型](https://i.loli.net/2019/05/13/5cd8c01c17d4063119.png)

## JVM 内存结构

Java 虚拟机在执行 Java 程序的过程中会把它管理的内存区域划分为若干个不同的数据区域. 根据*《Java虚拟机规范》*的规定, Java 虚拟机所管理的内存包括的运行时数据区域分为: *方法区*, *堆*, *栈*, *程序计数器*, *本地方法栈*. 其中*堆*是 JVM 中最大的一块, 可以进一步分为*新生代*和*老年代*, *新生代*又可进一步分为*Eden(伊甸区)*和2个*Survivor(幸存区)*[^1]. 可参考下图进行理解:

![UTOOLS1556800707127.png](https://i.loli.net/2019/05/02/5ccae4c3c50ca.png)

### 栈(Java Stack)

*栈*是线程私有的内存区域, 它的生命周期与线程相同. 

*栈*描述的是 Java 方法执行的内存模型, 每个方法执行的同时都会创建一个*栈帧(Stack Frame)*用于存储*局部变量表*, *操作数栈*, *动态链接*和*方法出口*等信息. 每一个方法被调用直至执行完成的过程, 就对应着一个*栈帧*在*栈*中从入栈到出栈的过程.

*局部变量表*存放了编译期可知的各种*基本数据类型*(boolean, byte, char, short, int, float, long, double), *对象引用*(reference 类型, 它不等同于对象本身, 可能是一个指向对象起始地址的指针, 也可能是指向一个代表对象的句柄或其他与此对象相关的位置, 见下图)和 returnAddress 类型(指向一条字节码指令的地址). 其中 64 位长度的 long 和 double 类型的数据会占用 2 个局部变量空间(Slot), 其余的数据类型只占用 1 个. *局部变量表*所需的内存在编译期间完成分配, 当进入一个方法时, 这个方法需要在*栈帧*中分配多大的局部变量空间是完全确定的, 在方法运行期间不会改变局部变量表的大小.

![UTOOLS1556808495138.png](https://i.loli.net/2019/05/02/5ccb032e4ad6d.png)

![UTOOLS1556808417330.png](https://i.loli.net/2019/05/02/5ccb02e0a67fb.png)

在 Java 虚拟机规范中, 对这个区域规定了两种异常情况: 

1. 如果线程请求的栈深度大于虚拟机所允许的深度, 将抛出 `StackOverflowError` 异常.
2. 如果*栈*可以动态扩展(当前大部分的 Java 虚拟机都可动态扩展, 只不过 Java 虚拟机规范中也允许固定长度的*栈*), 当扩展时无法申请到足够的内存时会抛出 `OutOfMemoryError` 异常.

### 本地方法栈(Native Method Stacks)

*本地方法栈*与*栈*所发挥的作用是非常相似的, 它们之间的区别不过是: *栈*为虚拟机执行 Java 方法(也就是字节码)服务, 而*本地方法栈*则是为虚拟机使用到的*Native方法*服务. 虚拟机规范中对*本地方法栈*的方法使用的语言, 使用方式与数据结构没有强制规定. 因此具体的虚拟机可以自由实现它, 甚至有的虚拟机(譬如 Sun HotSpot)直接就把*本地方法栈*和*栈*合二为一. 与*栈*一样, *本地方法栈*也会抛出相同的异常.

### 程序计数器(Program Counter Register)

*程序计数器*是一块较小的内存空间, 是线程私有的. 它的作用可以看作是当前线程所执行的字节码的行号指示器. 在虚拟机的概念模型中, 字节码解释器工作时就是通过改变这个计数器的值来选取下一条需要执行指令的字节码指令. 分支, 循环, 跳转, 异常处理, 线程恢复等基础功能都需要依赖这个计数器来完成.

由于 Java 虚拟机的多线程是通过线程轮流切换并分配处理器换行执行时间的方式来实现的, 在任何一个确定的时刻, 一个处理器(对于多核处理器来说是一个核)只会执行一条线程中的指令. 因此, 为了线程切换后能恢复到正确的执行位置, 每条线程都需要一个独立的程序计数器, 各条线程之间的计数器互不影响, 独立存储.

如果线程正在执行的是一个 Java 方法, 这个计数器记录的是正在执行的虚拟机字节码指令的地址; 如果正在执行的是 Native 方法, 这个计数器的值为空(Undefined).

**此内存区域是唯一一个在 Java 虚拟机规范中没有规定任何 `OutOfMemoryError` 情况的区域**.

### 堆(Heap)

对于大多数应用来说, *堆*是 Java 虚拟机所管理的内存中最大的一块. *堆*是被所有线程共享的一块内存区域, 在虚拟机启动时创建. 此内存区域的唯一目的就是存放对象实例, 几乎所有的对象实例都在这里分配内存.

*堆*是*GC(垃圾收集器)*管理的主要区域, 如果从内存回收的角度来看, 由于现在收集器都是采用的分代收集算法, 所以*堆*中还可以细分为: *新生代*和*老年代*, 再进一步细分有: *EdenSpace(伊甸区)*, *FromSpace(幸存区1)*和*ToSpace(幸存区2)*.

根据 Java 虚拟机规范的规定, *堆*可以处于**物理上不连续**的内存空间中, 只要**逻辑上连续**即可. 在实现时, 既可以实现成固定大小的, 也可以是可扩展的, 不过当前主流的虚拟机都是按照可扩展来实现的(通过 `-Xmx` 和 `-Xms` 控制).

如果在*堆*中没有内存完成实例分配, 并且*堆*也无法再扩展时, 将会抛出 `utOfMemoryError: Java heap space` 异常.

![UTOOLS1556804619931.png](https://i.loli.net/2019/05/02/5ccaf40c711e8.png)

### 方法区(Method Area)

*方法区*与*堆*一样, 是各个线程共享的内存区域. 它用于存储已被虚拟机加载的*类信息*, *常量*, *静态变量*, *即时编译器*编译后的代码等数据, 也就是用来存储类的描述信息-*元数据(Metadata)*. 虽然 Java 虚拟机规范把*方法区*描述为*堆*的一个逻辑部分, 但它却有一个别名叫做 *Non-Heap(非堆)*, 目的应该是与*堆*区分. 相对而言, *GC* 对于这个区域的收集是很少出现的. 当方法区无法满足内存分配需求时, 将抛出 `OutOfMemoryError` 异常.

**注意**: 如果使用的是 Sun HotSpot 虚拟机, 在 **JDK7** 及之前版本, 我们也习惯称它为*永久代(Permanent Generation)*, 更确切地来说, 应该是"HotSpot 使用*永久代*实现了*方法区*". 而从 **JDK8** 开始, *永久代*已经被彻底移除, 取而代之的是*元空间(Metaspace)*, 它使用**本地内存**来存储类元数据信息. 也就是说, 只要本地内存足够, 它不会出现像*永久代*中 `java.lang.OutOfMemoryError: PermGen space` 这种错误. 同样的, 对*永久代*的设置参数 `Permsize` 和 `MaxPermSize` 也会失效. 默认情况下, *Metaspace*的大小可以动态调整, 或者使用新参数 `MaxMetaspaceSize` 来限制本地内存分配给类元数据的大小.

![Metaspace](https://i.loli.net/2019/05/02/5ccafa829033b.png)

### 运行时常量池(Runtime Constant Pool)

*运行时常量池*是*方法区*的一部分. Class 文件中除了有类的版本, 字段, 方法, 接口等描述信息外, 还有一项信息是*常量池(Constant pool table)*, 用于**存放编译期生成的各种字面量和符号引用**, 这部分内容将在类加载后进入*运行时常量池*中存放. *运行时常量池*相比较于 Class 文件中的*常量池*的另外一个特性是具备**动态性**, Java 语言并不要求常量一定只有编译器才产生, 也就是并非预置到 Class 文件中的常量池的内容才能进入*方法区*的*运行时常量池*, 运行期间也可能将新的常量放入池中.

> `String.intern()` 就是扩展常量池的一个方法. 它的作用是: 查找当前常量池中是否已有相同的字符串常量, 如果有就返回其引用, 如果没有就在常量池中添加对应的字符串, 并返回对应字符串常量的引用.
>
> 使用 `new String("xxx")` 是在堆中创建一个 `String` 对象实例, 所以两个 `new String("hello")` 使用 `==` 时返回 `false`. 而使用 `new String("hello").intern()` 则为 `true`.

### 直接内存(Direct Memory)

*直接内存*并不是虚拟机*运行时数据区*的一部分, 也不是 Java 虚拟机规范中定义的内存区域. 但这部分内存也被频繁地使用, 而且也可能导致 `OutOfMemoryError` 异常出现. 在 JDK 中最直观的表现就是 **NIO**, 基于*Channel(通道)*与*Buffer(缓冲区)*的 I/O 方式, 它可以使用 Native 函数库**直接分配堆外内存**, 然后通过一个存储在*堆*中的 `DirectByteBuffer` 对象作为这块内存的引用进行操作. 这样能在一些场景中显著提高性能, 因为避免了在*堆*和*Native堆*中来回复制数据.

## 垃圾回收(GC)

> *垃圾回收(Garbage Collection)*简称*GC*. 在 JVM 中, *程序计数器*, *栈*, *本地方法栈*都是随线程而生随线程而灭, *栈帧*随着方法的进入和退出做入栈和出栈操作, 实现了自动的内存清理. 而*堆*和*方法区*则不同, 一个接口中的多个实现类需要的内存可能不一样, 一个方法中的多个分支需要的内存也可能不一样, 我们只有在程序处于运行期才知道哪些对象会创建, 这部分的内存的分配和回收都是动态的. *垃圾回收*所关注的就是这部分内存.

### 对象存活判断

判断对象是否存活一般有两种方式:

1. **引用计数**: 每个对象有一个引用计数属性, 新增一个引用时计数加 1; 引用释放时计数减 1. 计数为 0 时可以回收. 此方法比较简单, 但无法解决对象相互循环引用的问题.

   ![UTOOLS1556852622302.png](https://i.loli.net/2019/05/03/5ccbaf8ca0346.png)

   从图中可以看出, 如果直接把 Obj1-reference 和 Obj2-reference 置为 `null`, 在*堆*当中的两块内存依然保持着互相引用无法回收.

2. **可达性分析(Reachability Analysis)**: 从 GC Roots 开始向下搜索, 搜索所走过的路径称为*引用链*. 当一个对象到 GC Roots 没有任何*引用链*相连时, 则证明此对象是不可用的, 即不可达对象.

   在 Java 中, GC Roots 包括: 

   - *栈*(*栈帧*中的本地变量表)中引用的对象.
   - *方法区*中类静态属性引用的对象.
   - *方法区*中常量引用的对象.
   - *本地方法栈*中 JNI(即 Native 方法) 引用的对象.

   ![UTOOLS1556852653375.png](https://i.loli.net/2019/05/03/5ccbafabb0ec2.png)

### 引用

从 JDK1.2 开始, 对象的引用被划分为 4 个级别, 从而使程序能够更加灵活地控制**对象的生命周期**. 这 4 种级别**由高到低**依次为: *强引用*, *软引用*, *弱引用*和*虚引用*.

![UTOOLS1556853027254.png](https://i.loli.net/2019/05/03/5ccbb121808a7.png)

#### 强引用(Strong Reference)

*强引用*是最简单, 使用最普遍的引用. 如果一个对象具有强引用, 那么垃圾回收器就绝不会回收它. 当**内存空间不足**时, Java 虚拟机宁愿抛出 `OutOfMemoryError` 异常, 也不会通过回收具有*强引用*的对象来解决内存不足的问题. 如果*强引用*对象不使用时, 需要弱化从而使 *GC* 能够回收, 如下:

```java
// 创建强引用对象
Object strongReference = new Object();
// 弱化方便 GC 回收
strongReference = null;
```

如果在一个*局部变量*是*强引用*, 这个引用保存在*栈*中, 而真正的引用内容保存在*堆*中. 当*局部变量*所在的方法执行完成后, 就会退出方法栈, 则引用对象的引用数为 0, 这个对象会被回收.

### 软引用(SoftReference)

如果一个对象只具有*软引用*, 则**内存空间充足**时, 垃圾回收器**不会**回收它. 如果**内存空间不足**时, 就会**回收**这些对象.

> *软引用*可以用来实现内存敏感的高速缓存.

*软引用*可以和一个*引用队列(ReferenceQueue)*结合使用, 如果*软引用*所引用对象被垃圾回收, Java 虚拟机就会把这个*软引用*加入到与之关联的*引用队列*中.

```java
// 创建引用队列
ReferenceQueue<String> refQueue = new ReferenceQueue<>();
// 创建软引用对象
String string = new String("soft");
SoftReference<String> softReference = new SoftReference(string,refQueue);

// 当软引用被垃圾回收后, 调用 poll() 方法可以获得该对象
Reference< ? extends String> ref = refQueue.poll();
```

### 弱引用(WeakReference)

*弱引用*与*软引用*的区别在于: 只具有*弱引用*的对象拥有更短暂的生命周期. 一旦进行垃圾回收, 无论当前内存空间足够与否, *弱引用*对象都会被回收. 

```java
// 创建弱引用对象
String string = new String("weak");
WeakReference<String> weakReference = new WeakReference<>(string);
// 获取弱引用所引用的对象
String res = weakReference.get();
```

*弱引用*也可以和*引用队列*结合使用.

**注意**: 这里是用 `new` 来创建的字符串对象, 引用的对象在*堆*中, 所以会被 GC 回收. 但如果弱引用所引用的对象位于*字符串常量池*中时(如 `String str = "str"`), 因为常量池中的字符串不会被 GC 回收(有一个表来维护字符串), 这种情况下, `get()` 方法在 GC 后仍会获得字符串常量池中的对象.

### 虚引用(PhantomReference)

*虚引用*与其他引用不同, *虚引用*不会决定对象的声明周期. *虚引用*在任何时候都可能被垃圾回收.

**注意**: *虚引用*必须和*引用队列*结合使用, 当垃圾回收器准备回收一个对象时, 如果发现它还有*虚引用*, 就会在回收对象的内存之前, 把这个*虚引用*加入到与之关联的*引用队列*中.

```java
// 创建引用队列
ReferenceQueue<String> refQueue = new ReferenceQueue<>();
// 创建虚引用
String str = new String("PhantomReference");
PhantomReference<String>  phantomReference(str,refQueue);

System.out.println(phantomReference.get()); // 获取的结果为 null
// 触发 GC
System.gc();
// 此时虚引用对象被加入到引用队列
System.out.println(refQueue.poll().get()); // 获取到 "PhantomReference"
```

程序可以通过判断*引用队列*中是否已经加入了*虚引用*来了解被引用的对象是否将要进行垃圾回收. 如果程序发现某个*虚引用*已经被加入到*引用队列*, 那么就可以在所引用的对象的被回收之前采取最后的收尾工作. 比如**资源清理和释放**, 它比 finalize 更灵活, 可以基于*虚引用*做更安全可靠的对象关联的资源回收.

### 垃圾收集算法

#### 标记-清除算法(Mark-Sweep)

*标记-清除算法*分为**标记**和**清除**两个阶段: 首先**标记**出所有需要回收的对象, 在标记完成后统一回**清除**回收掉所有被**标记**的对象. 之所以说它是最基础的收集算法, 是因为后续的收集算法都是基于这种思路并对其缺点进行改进而得到的.

主要的缺点有两个: 一个是**效率问题**, 标记和清除过程的效率都不高; 另外一个是**空间问题**, 标记清除之后会产生大量的内存碎片, 内存碎片太多可能会导致当程序在以后的运行过程中需要分配较大对象时无法找到足够的连续内存而不得不提前触发另一次垃圾回收.

![UTOOLS1556871419286.png](https://i.loli.net/2019/05/03/5ccbf8fa17f40.png)



#### 复制收集算法(Copying)

*复制收集算法*将可用内存按容量划分为大小相等的两块, 每次只使用其中的一块. 当这一个块的内存用完时, 就将还存活着的对象复制到另一块内存上, 然后再把已使用的内存空间一次清理掉.

这样使得每次都是对其中的一块内存进行回收, 内存分配时也就不用考虑内存碎片等复杂情况, 只要移动堆顶指针, 按顺序分配内存即可, 实现简单, 运行高效. 这种算法的代价是将可用内存缩小为原来的一半, 持续复制长生存期的对象则导致效率降低.

![UTOOLS1556870907145.png](https://i.loli.net/2019/05/03/5ccbf6faca0cb.png)

#### 标记-压缩算法(Mark-Compact)

*复制收集算法*在对象存活率较高时就要频繁执行复制操作, 效率将会变低. 更关键的是, 如果不想浪费 50% 的空间, 就需要有额外的空间进行分配担保, 以应对被使用的内存中所有对象都 100% 存活的极端情况, 所以在老年代一般不能直接选用这种算法.

根据老年代存活期长的特点, 有人提出了另外一种*标记-整理算法*, 标记过程仍然与*标记-清除算法*相同, 但后续步骤不是直接对可回收对象进行整理, 而是让所有存活的对象都向一端移动, 然后直接清理掉端边界以外的内存.

![UTOOLS1556871873581.png](https://i.loli.net/2019/05/03/5ccbfac15a819.png)

#### 分代收集算法(Generation Collection)

*分代回收算法*的基本思想是: 根据垃圾回收对象的特性, 对不同阶段使用最合适的算法用于本阶段的垃圾回收. 它将内存区间根据对象的特点分成几块, 根据每块内存区间的特点, 使用不同的回收算法, 以提高垃圾回收的效率. 

以 Hot Spot 虚拟机为例, 它将所有的新建对象都放入*新生代*的内存区域, *新生代*的特点是对象会很快被回收. 因此, 在*新生代*就选择效率较高的*复制算法*. 当一个对象经过一次回收后依然存活, 它的年龄就加 1, 如果对象的年龄大于阈值(默认为 15, 通过 `-XXMaxTenuringThreshold` 参数设置, 最大为 15), 就会被放入*老年代*的内存空间. 在*老年代*中, 几乎所有的对象都是数次回收后依然得以幸存的, 因此, 可以认为这些对象在一段时期内, 甚至在应用程序的整个生命周期中, 将是常驻内存.  如果依然使用*复制算法*收集*老年代*, 将需要复制大量对象. 再加上*老年代*回收性价比要低于*新生代*, 因此这种做法也是不可取的. 根据**分代**的思想, 可以对*老年代*的回收使用与*新生代*不同的*标记-压缩算法*, 以提高垃圾回收效率.

### 垃圾收集器

> 如果说收集算法是内存回收的方法论, 垃圾收集器就是内存回收的具体实现.

#### Serial 收集器

*Serial 收集器*是最古老的以及效率最高的收集器, 只使用一个线程去回收, 但可能会产生较长的停顿.

*新生代*和*老年代*使用串行回收. *新生代*使用*复制算法*, *老年代*使用*标记-压缩算法*. 垃圾回收的过程中会*Stop The World(服务暂停)*.

参数控制:

- `-XX:+UseSerialGC`: Serial 收集器

![UTOOLS1556873397966.png](https://i.loli.net/2019/05/03/5ccc00b68a62b.png)

#### ParNew 收集器

*ParNew 收集器*其实就是 *Serial 收集器*的多线程版本.

*新生代*并行, *老年代*串行. *新生代*使用*复制算法*, *老年代*使用*标记-压缩算法*. 垃圾回收的过程中同样会*Stop The World(服务暂停)*.

参数控制: 

- `-XX:+UseParNewGC`: ParNew 收集器
- `-XX:ParallelGCThreads`: 限制线程数量

![UTOOLS1556873463849.png](https://i.loli.net/2019/05/03/5ccc00f6b8a4b.png)

#### Parallel Scavenge 收集器

*Parallel Scavenge 收集器*类似 *ParNew 收集器*, 其更关注系统的*吞吐量*. 可以通过参数来打开自适应调节策略, 虚拟机会根据当前系统的运行情况收集性能监控信息, 动态调整这些参数以提供最合适的*停顿时间*或最大的*吞吐量*; 也可以通过参数控制 *GC* 的时间不大于多少毫秒或者比例.

*新生代*使用*复制算法*, *老年代*使用*标记-压缩*算法. *新生代*并行, *老年代*串行.

参数控制:

- `-XX:+UseParallelGC`: 使用 Parallel 收集器

![UTOOLS1556874349353.png](https://i.loli.net/2019/05/03/5ccc046d06fb1.png)

#### Serial Old 收集器

![UTOOLS1556873475381.png](https://i.loli.net/2019/05/03/5ccc0101f0ee2.png)

#### Paralle Old 收集器

*Parallel Old 收集器*是*Paralle Scavenge 收集器*的*老年代*版本, 使用多线程和*标记-压缩算法*. 这个收集器是在 JDK1.6 才开始提供.

*新生代*使用*复制算法*, *老年代*使用*标记-压缩算法*. *新生代*和*老年代*并行.

参数控制:

- `-XX:+UseParallelOldGc`: 使用 *Parallel 收集器*

![UTOOLS1556873484636.png](https://i.loli.net/2019/05/03/5ccc010b382e4.png)

#### CMS 收集器

*CMS(Concurrent Mark Sweep)收集器*是一种以获取最短回收停顿时间为目标的收集器. 目前很大一部分的 Java 引用都集中在互联网站或 B/S 系统的服务端上, 这类引用尤其中是服务的响应速度, 希望系统停顿时间最短, 以给用户带来较好的体验.

*CMS GC* 采用了*标记-清除算法*, 它的运作过程相对于前面几种收集器来说要更加复杂一些, 整个过程分为以下步骤:

1. 初始标记(STW initial mark)

   需要 *Stop The World*. 这个过程从垃圾回收的 GC Roots 开始, 只扫描能够和 GC Roots 直接关联的对象, 并作标记. 所以这个过程虽然暂停了整个 JVM, 但是很快就完成了.

2. 并发标记(Concurrent marking)

   这个阶段紧随*初始标记*阶段, 在*初始标记*的基础上继续向下追溯标记. 此阶段, 引用程序的线程和*并发标记*的线程并发执行, 所以用于不会感受到停顿.

3. 并发预清理(Concurrent precleanig)

   *并发预清理*阶段仍是并发的. 这个阶段, 虚拟机查找在执行*并发标记*阶段新进入*老年代*的对象(可能会有一些对象从*新生代*晋升到*老年代*, 或者有一些对象被分配到*老年代*). 通过重新扫描, 减少下一个阶段*重新标记*的工作, 因为下一个阶段会 *STW*.

4. 重新标记(STW remark)

   需要 *Stop The World*, 时间较*初始标记*阶段要长一些. 为了修正*并发标记*期间, 因用于程序继续运作而导致标记产生变动的那一部分对象的标记记录. 

5. 并发清除(Concurrent sweeping)

   清理垃圾对象, 与应用程序线程并发执行.

6. 并发重置(Concurrent reset)

   这个阶段, 重置 *CMS 收集器*的数据结构, 等待下一次垃圾回收.

参数控制:

- `-XX:+UseConcMarkSweepGC`: 使用CMS收集器
- `-XX:+ UseCMSCompactAtFullCollection`: Full GC 后, 进行一次碎片整理; 整理过程是独占的, 会引起停顿时间变长
- `-XX:+CMSFullGCsBeforeCompaction`: 设置进行几次 Full GC  后, 进行一次碎片整理
- `-XX:ParallelCMSThreads`: 设定CMS的线程数量(一般情况约等于可用CPU数量)

![UTOOLS1556875164774.png](https://i.loli.net/2019/05/03/5ccc079c2a81f.png)

#### G1 收集器

*G1 收集器*是 JDK1.7 中正式投入使用的用于取代 CMS 的压缩回收器, 它虽然没有物理上隔断*新生代*与*老年代*, 但是仍然属于分代垃圾回收器. *G1 GC* 仍然会区分*新生代*与*老年代*, *新生代*依然有 *Eden区*与 *Survivor区*. *G1 GC* 首先将堆分为大小相等的 *Region*, 避免全区域的垃圾收集, 然后追踪每个 *Region* 垃圾堆积的价值大小, 在后台维护一个优先列表, 根据允许的手机时间优先回收价值最大的 *Region*; 同时 *G1 GC* 采用 Remembered Set 来存放 *Region* 之间的对象引用以及其他回收器中的*新生代*与*老年代*之间的对象引用, 从而避免全堆扫描.

G1 内存分布

![UTOOLS1556876238766.png](https://i.loli.net/2019/05/03/5ccc0bce87f1a.png)

一次 Young GC

![UTOOLS1556876313762.png](https://i.loli.net/2019/05/03/5ccc0c185b18c.png)

![UTOOLS1556876301977.png](https://i.loli.net/2019/05/03/5ccc0c0c8e67f.png)

随着 *G1 GC*的出现, Java 垃圾回收器通过引入 *Region* 概念, 从传统的连续堆内存布局设计, 逐步走向了**物理上不连续但逻辑上依旧连续的内存块**. 这样我们能够将某个 *Region* 动态地分配给 *Eden*, *Survivor*, *老年代*, *大对象空间*和*空闲区间*等任意一个. 每个 *Region* 都有一个关联的 Remembered Set(简称 RS), RS 的数据结构是 Hash 表, 里面的数据是 Card Table(堆中每 512byte 映射在 Card Table 1 byte). 简单地说 RS 里面存在的是 *Region* 中存活对象的指针. 当 *Region* 中数据发生变化时, 首先反映到 Card Table 中的一个或多个 Card 上, RS 通过扫描内部的 Card Table 得知 *Region* 中内存使用情况和存活对象. 在使用 *Region* 过程中, 如果 *Region* 被填满了, 分配内存的线程会重新选择一个新 *Region*, 空闲 *Region* 被组织到一个基于链表的组织结构(Linkedlist)中, 这样可以快速找到新的 *Region*.

总结 *G1 GC* 的特性如下:

- 并行性: *G1* 在回收期间, 可以有多个 GC 线程同时工作, 有效利用多核计算能力.
- 并发性: *G1* 拥有与应用程序交替执行的能力, 部分工作可以和应用程序同时执行, 因此, 一般来说, **不会再整个回收阶段发生阻塞应用程序的情况**.
- 分代 GC: *G1* 依然是一个分代回收器, 但是和之前的各类回收器不同, 它同时兼顾*年轻代*和*老年代.* 对比其他回收器, 或者工作在*年轻代*, 或者工作在*老年代*.
- 空间整理: *G1* 在回收过程中, 会进行适当的对象移动, 不像 *CMS* 只是简单地标记清理对象. 在若干次 GC 后, CMS 必须进行一次碎片整理. 而 *G1* butong , 它每次回收都会有效地复制对象, 减少空间碎片, 进而提升内部循环速度.
- 可预见性: 为了缩短停顿时间, *G1* 建立可预存停顿的模型, 这样在用户设置的停顿时间范围内, *G1* 会适当选择恰当的区域进行收集, 确保停顿时间不超过用户指定时间.

*G1 GC* 的工作步骤如下所示:

![UTOOLS1556877090244.png](https://i.loli.net/2019/05/03/5ccc0f22344f5.png)

- 初始标记: 标记一下 GC Roots 能直接关联的对象并修改 TAMS 值, 需要 STW 但耗时很短.
- 并发标记: 从 GC Roots 从堆中对象进行可达性分析寻找存活的对象, 耗时较长, 但可以与用户线程并发执行.
- 最终标记: 为了修正并发标记期间产生变动的那一部分标记记录, 这一期间的变化记录在 Remembered Set Log 里, 然后合并到 Remembered Set 里, 该阶段需要 STW 但是可并行执行.
- 筛选回收: 对各个 *Region* 回收价值排序, 根据用户期望的 GC 停顿时间制定回收计划来回收.

参数设置:

- `--XX:+UseG1GC` 使用 G1 收集器
- `-XX:MaxGCPauseMillis=n`: 设置最大GC停顿时间(GC pause time)指标(target). 这是一个软性指标(soft goal), JVM 会尽量去达成这个目标.
- `-XX:G1HeapRegionSize=n`: 此参数指定每个 *Region* 的大小. 默认值将根据 heap size 算出最优解. 最小值为 1Mb, 最大值为 32Mb.
- `-XX:ConcGCThreads=n`: 并发垃圾收集器使用的线程数量. 默认值随 JVM 运行的平台不同而不同.

### 常用的收集器组合

不同垃圾收集器适用的范围. 

![UTOOLS1556877928882.png](https://i.loli.net/2019/05/03/5ccc12684a9b3.png)

|      |   新生代GC策略    |  老年代GC策略  | 说明                                                         |
| :--: | :---------------: | :------------: | :----------------------------------------------------------- |
|  1   |      Serial       |   Serial Old   | Serial 和 Serial Old 都是单线程进行 GC. 特点就是 GC 时暂停所有应用线程. |
|  2   |      Serial       | CMS+Serial Old | CMS(Concurrent Mark Sweep)是并发 GC, 实现 GC 线程和应用线程并发工作, 不需要暂停所有应用线程. 另外, 当 CMS 进行 GC 失败时, 会自动使用 Serial Old 策略进行 GC. |
|  3   |      ParNew       |      CMS       | 使用 `-XX:+UseParNewGC `选项来开启. ParNew 是 Serial 的并行版本, 可以指定 GC 线程数, 默认 GC 线程数为 CPU 的数量. 可以使用 ``-XX:ParallelGCThreads` 选项指定 GC 的线程数. 如果指定了选项 `-XX:+UseConcMarkSweepGC` 选项, 则新生代默认使用ParNew GC 策略. |
|  4   |      ParNew       |   Serial Old   | 使用 `-XX:+UseParNewGC` 选项来开启. 新生代使用 ParNew GC 策略, 年老代默认使用 Serial Old 策略. |
|  5   | Parallel Scavenge |   Serial Old   | Parallel Scavenge 策略主要是关注一个可控的吞吐量: `应用程序运行时间 / (应用程序运行时间 + GC 时间)`, 可见这会使得 CPU 的利用率尽可能的高, 适用于后台持久运行的应用程序, 而不适用于交互较多的应用程序. |
|  6   | Parallel Scavenge |  Parallel Old  | Parallel Old 是 Serial Old 的并行版本.                       |
|  7   |        G1         |       G1       | `-XX:+UnlockExperimentalVMOptions` `-XX:+UseG1GC` 开启<br>`-XX:MaxGCPauseMillis =50` 暂停时间目标<br>`-XX:GCPauseIntervalMillis =200` 暂停间隔目标<br>`-XX:+G1YoungGenSize=512m` 年轻代大小<br>`-XX:SurvivorRatio=6` 幸存区比例 |

## JVM 常用参数

- `-XX:PrintFlagsFinal`: 显示所有可设置的参数及它们的值.

- `-XX:PrintFlagsInitial`: 显示在处理参数之前所有可设置的参数及它们的值, 然后直接退出程序.

- `-Xms`=`-XX:InitialHeapSize`: 初始堆大小, 默认物理内存的 1/64. 空余堆内存小于 40% 时, JVM 就会增大堆直到 `-Xmx` 的最大限制.

- `-Xmx`=`-XX:MaxHeapSize`: 最大堆大小, 默认物理内存的 1/4. 空余堆内存大于 70% 时, JVM 会减少堆直到 `-Xms` 的最小限制.

- `-Xss`: 每个线程的堆栈大小.

- `-Xmn`: *新生代*大小.

  Sun 官方推荐配置为整个*堆*的 3/8.

- `-XX:MetaspaceSize`: 初始元空间大小, 该值越大触发 Metaspace GC 的时机就越晚.

- `-XX:+PrintGCDetails`: 输出垃圾回收详情.

- `-XX:SurvivorRatio`: *Eden 区*与 *Survivor 区*的大小比值.

  如果设置为 4, 表示 EdenSpace:FromSpace:ToSpace = 4:1:1.

- `-XX:NewRatio`: *新生代*与*老年代*的比值.

  如果设置为 4, 表示 *新生代*:*老年代* = 4:1. 如果 Xms=Xmx 且设置了 Xmn 的情况下, 此参数不生效.

- `-XX:MaxTenuringThreshold`: 垃圾最大年龄. 最大为 15.

  如果设置为 0 的话, 则*新生代*对象不经过 *Survivor 区*, 直接进入*老年代*. 该参数只有在串行 GC 时才有效.
  
- `-XX:+UseStringDeduplication`: 在使用 G1 GC 时进行字符串排重, 将相同数据的字符串指向同一份数据.

## **参考**

- jvm-overview: <http://www.ityouknow.com/java/2017/03/01/jvm-overview.html>
- JVM总结--JVM体系结构: <https://blog.csdn.net/samjustin1/article/details/52215274>
- 重读 JVM: <https://juejin.im/post/59ad4cd56fb9a02477075780>
- 深入理解JVM之Java内存管理（基于JAVA8）: <https://www.jianshu.com/p/e90b5121ff45>
- 
- 

[^1]: 2个幸存区分别是*FromSpace*和*ToSpace*. 在 *GC* 时, 这两个内存区域的逻辑角色会交换.