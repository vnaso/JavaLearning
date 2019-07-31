---
title: Java基础笔记
date: 2019/03/11 21:34
categories:
- Java
tags:
- Java
---

## JVM JDK 和 JRE

- javac :包含于JDK中的将`.java`文件编译为字节码(`.class`)文件的编译器.
- JVM可以理解的代码就叫做`字节码`.  即扩展名为`.class`的文件.
- JSP 会被转换为 Java servlet.  并被 JDK 用 javac 编译器编译.

## 字符型常量和字符串常量

- 形式上:字符型常量是单引号引起的一个字符.如`'a'`.  字符串常量是双引号引起的若干个字符如`"Hello world"`.
- 含义上:字符型常量可以当做一个**整形值**(16位 Unicode 字符 0~65535)参与表达式运算.  字符串常量代表一个地址值(引用字符串常量池中的对象.  不会被垃圾回收)

## 重载(Overload)和重写(Override)

|                             重载                             |                             重写                             |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
|                       发生在同一个类中                       |                        发生在父子类中                        |
| 方法名必须相同<br>参数类型、个数、顺序、方法返回值和权限修饰符可以不同 | 方法名和参数列表必须相同.  权限修饰符范围大于等于父类<bt>抛出异常的范围小于等于父类.  返回值返回小于等于父类 |

## 封装 继承 多态

### 封装

封装把一个对象的属性**私有化**. 同时提供一些想让外界访问的属性的方法.

### 继承

继承是使用已存在的类的定义作为基础建立新类的技术. 新类的定义中可以增加新的成员变量和成员方法.  且同时拥有父类声明的可以被继承的变量和方法.

### 多态

多态就是指程序中定义的引用变量所指向的具体类型和通过该引用变量发出的方法调用在编程时并不确定. 只有在程序运行期间才能确定.

实现多态的方法:继承和接口

## String 及 String.intern()

### 前提知识

1. `new String("abc")` 的分析:
   1. `"abc"` 会在*字符串常量池*中查找是否有 `abc  `这个字符串, 如果有, 返回其引用, 如果没有, 将 `abc` 添加到字符串常量池, 再返回其引用.
   2. `new String("abc")` 会在堆内存中创建一个 `"abc"` 的字符串对象, 返回堆中该对象的引用. 无论常量池中是否存在 `"abc"` 字符串.

2. `String.intern()` 的分析:
   
    ```java
    String str = new String("abc");
    str.intern();
    ```
    
    以上面代码为例:
    
    - **在 JDK 6 以前**: 如果*字符串常量池*中存在 `"abc"`, 则返回其在常量池中的引用, 否则将 `"abc"` 拷贝到常量池中, 并返回常量池中的应用.
    
    - **从 JDK 7 开始**: 如果*字符串常量池*中存在 `"abc"`, 仍返回其在常量池中的引用, **但不存在时**, 不再将其拷贝到常量池中, 而是直接在常量池中生成一个对在堆中的原字符串的引用. 此例中即 `new String("abc")` 创建的对象的引用.
    
3. 使用 `Final String VALUE = "abc";` 时, 代码中所有使用 `VALUE` 变量的地方在编译时都会被直接替换成 `"abc"`.
   
      例如:
    ```java
      Final String a = "1";
        Final String b = "2";
        String c = a + b; // 编译后变为: String c = "1" + "2";
    ```
    
4. 字符串引用进行 `+` 运算时, 会创建 `StringBuilder/StringBuffer.append()`, 之后再转换成 `String`, 这种情况下会 `new` 一个 `String` 对象.
   
      例如:
      
      ```java
      String str1 = "a";
      String str2 = "b";
      String str12 = str1 + str2;
      String ab = "a" + "b";
      str12 == ab; // false
      ```

### 案例

**案例一**

```java
    public static void main(String[] args) {
        String str1 = "string";
        String str2 = new String("string");
        String str3 = str2.intern();
 
        System.out.println(str1==str2);// false
        System.out.println(str1==str3);// true
    }
```

对应 #1.

**案例二**

```java
public static void main(String[] args) {
        String baseStr = "baseStr";
        final String baseFinalStr = "baseStr";
 
        String str1 = "baseStr01";
        String str2 = "baseStr"+"01";
        String str3 = baseStr + "01";
        String str4 = baseFinalStr+"01";
        String str5 = new String("baseStr01").intern();
 
        System.out.println(str1 == str2);// true
        System.out.println(str1 == str3);// false
        System.out.println(str1 == str4);// true 
        System.out.println(str1 == str5);// true
    }
```

对应 #2 #3 #4

**案例三**

```java
 public static void main(String[] args) {
 
        String str2 = new String("str")+new String("01");
        str2.intern();
        String str1 = "str01";
        System.out.println(str2==str1);// true
    }
```

对应 #2.

**案例四**

```java
 public static void main(String[] args) {
        String str1 = "str01";
        String str2 = new String("str")+new String("01");
        str2.intern();
        System.out.println(str2 == str1);// false
    }
```

对应 #2.

## StringBuffer 和 StringBuilder

### 可变性

#### String

`String`类中使用`private final char value[]`来保存字符串. 所以`String`对象是不可变的

#### StringBuilder 与 StringBuffer

两者都继承自`AbstractStringBuilder`类. 该类同样使用`char[] value`来保存字符串. 但没有用`final`修饰.  所以这两种对象是可变的

### 线程安全性

#### String

`String `中的对象是不可变的. 在字符串常量池中只有一个. 线程安全.

#### StringBuffer 和 StringBuilder

`AbstractStringBuilder `是 `StringBuilder` 与 `StringBuffer `的父类.  定义了一系列字符串的基本操作. 如`expandCapacity`, `append`, `insert`, `indexOf `等公共方法.

##### StringBuffer

`StringBuffer` 对方法加了同步锁或者对调用的方法加了同步锁. 所以是线程安全的.

##### StringBuilder

`StringBuilder` 没有加锁. 所以是非线程安全的.

### 性能

- `String` 类型每次被改变的时候. 都会在字符串常量池中生成一个新的 `String` 对象. 然后将指针指向新的 `String `对象
- `StringBuffer `每次都会对对象本身进行操作. 不会生成新的对象并改变引用.
- `StringBuilder` 相比 `StringBuilder` 性能仅提升10%~15%左右. 但非线程安全.

### 使用

1. 操作少量数据 => String
2. 拼接字符串时 => StringBuilder StringBuffer
3. 单线程操作字符串缓冲区下大量数据 => StringBuilder
4. 多线程操作字符串缓冲区下大量数据 => StringBuffer

## 自动装箱与拆箱

### 装箱

将基本类型用他们对应的引用类型包装起来.如:`int`->`Integer`

自动装箱时编译器调用`valueOf`将原始类型值转换成对象

### 拆箱

将包装类型转换为基本数据类型.如:`Integer`->`int`

同时自动拆箱时，编译器通过调用类似intValue(). doubleValue()这类的方法将对象转换成原始类型值.

#### 发生场景

- 进行 = 赋值操作（装箱或拆箱）
- 进行+，-，*，/混合运算 （拆箱）
- 进行>.  <.  ==比较运算（拆箱）
- 调用equals进行比较（装箱）
- ArrayList.  HashMap等集合类 添加基础类型数据时（装箱）

#### 注意事项

##### 生成无用对象增加GC压力

因为自动装箱会隐式地创建对象.如果在一个循环体中.  会创建无用的中间对象.  这样会增加GC压力.  拉低程序的性能.所以在写循环时一定要注意代码.  避免引入不必要的自动装箱操作.

##### 对象相等比较

`==`可以用于基本类型进行比较.  也可以用于对象进行比较.  当用于对象与对象之间比较时.  比较的不是对象代表的值.  而是检查两个对象**是否是同一对象**.这个比较过程中没有自动装箱发生.进行对象值比较不应该使用`==`，而应该使用对象对应的`equals`方法.

```
public class AutoboxingTest {

    public static void main(String args[]) {

        // Example 1: == comparison pure primitive – no autoboxing
        int i1 = 1;
        int i2 = 1;
        System.out.println("i1==i2 : " + (i1 == i2)); // true

        // Example 2: equality operator mixing object and primitive
        Integer num1 = 1; // autoboxing
        int num2 = 1;
        System.out.println("num1 == num2 : " + (num1 == num2)); // true

        // Example 3: special case - arises due to autoboxing in Java
        Integer obj1 = 1; // autoboxing will call Integer.valueOf()
        Integer obj2 = 1; // same call to Integer.valueOf() will return same
                            // cached Object  cache range(-128~127)

        System.out.println("obj1 == obj2 : " + (obj1 == obj2)); // true

        // Example 4: equality operator - pure object comparison
        Integer one = new Integer(1); // no autoboxing
        Integer anotherOne = new Integer(1);
        System.out.println("one == anotherOne : " + (one == anotherOne)); // false

    }

}

Output:
i1==i2 : true
num1 == num2 : true
obj1 == obj2 : true
one == anotherOne : false
```

##### 缓存的对象

在Java中. 会对-128到127的Integer对象进行缓存. 当创建新的Integer对象时. 如果符合这个这个范围. 并且已有存在的相同值的对象. 则返回这个对象 否则创建新的Integer对象.

## 静态方法内不能调用非静态成员

静态方法可以不通过对象进行调用.  因此在静态方法里.  不能调用其他非静态变量.  也不可以访问费静态变量成员.

## 在 Java 中定义一个空的无参构造方法的作用

Java程序在执行子类的构造方法之前. 如果没有用`super()`来调用父类特定的构造方法. 就会**调用父类的无参构造方法**.因此. 如果父类中只定义了有参构造方法. 而子类的构造方法中没有用`super()`来调用父类中特定的构造方法. 则编译时将发生错误.

## 接口和抽象类的区别

|                             接口                             |                            抽象类                            |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| 方法默认是`public`. 所有方法都不能有实现且子类必须重写全部方法<br>Java 8开始.  接口提供默认方法`default`. 子类不需重写 |                       可以有非抽象方法                       |
|           实例变量必须是`public static final`类型            |                     可以不是`final`类型                      |
|                            多继承                            |                            单继承                            |
| 不能用`new`实例化. 但可以声明. 但是必须引用一个实现该接口的对象<br>接口是行为的抽象. 是一种行为的规范.适合对类的行为抽象 | 同样不能用`new`实例化. 但可以声明. 但是必须引用一个继承该类的对象<br>抽象是对类的抽象. 是一种模板设计.适合对事物抽象 |

#### 为什么接口属性必须是 public static final

> 接口是一种高度抽象的`模板`. 而接口中的属性也就是`模板`成员.就应当是所有实现`模板`的类的共有特性.
>
> 其次. 接口中如果可以定义非`final`变量的话. 而方法又都是`abstract`的. 这就会造成有可变成员变量.  但对应的方法却无法操作这些变量.接口是一种更高层面的抽象. 是一种规范. 功能定义的声明. 所有可变的东西都应归属到实现类中. 这样的接口才能起到标准化. 规范化的作用.所以接口中的属性必然是`final`的
>
> 最后, 接口只是对事物的属性和行为更高层次的抽象.对修改关闭. 对扩展开放.接口是对开闭原则的一种体现

#### 接口的静态方法

在Java 8中. 接口也可以定义静态方法. 可以直接用接口名调用. 实现类和实现四不可以调用的. 如果同时实现多个接口. 接口中定义了一样的默认方法. 必须重写. 否则会报错.

## 成员变量和局部变量的区别

|                        成员变量                         |                           局部变量                           |
| :-----------------------------------------------------: | :----------------------------------------------------------: |
| 成员变量属于类. 可以用public private static等修饰符修饰 | 局部变量在方法中定义或是方法的参数.  不能被权限修饰符修饰. 除了 final |
|             非静态时. 随对象存储于堆内存中              |                     随方法存储于栈内存中                     |
|       没有被赋初值. 则会自动以类型的默认值而赋值        |                        不会被自动赋值                        |

## == 与 equals

**==**: 判断两个对象的地址是不是相等. 即是不是同一个对象. 都是基本类型时比较的是值是否相等. 都是引用类型时比较的是内存地址是否相等.基本类型和它的封装类型时. 封装类型拆箱为基本类型然后比较值是否相等.

**equals**: 通过方法中定义的方式进行比较.所有类都继承了`Object`类的`equals()`方法. 其默认是用`==`来进行比较两个对象. 如果调用方法的对象重写了此方法. 按重写的方法中的方式进行比较.

## equals 与 hashCode

重写`equals`时必须重写`hashCode`方法

### hashCode()

`hashCode()`的作用是获取**哈希码**. 也称为**散列码**. 返回的是一个 int 整数. 这个哈希码的作用是确定该对象在哈希表中的索引位置. `hashCode()`定义在 JDK 的 `Object.java`中. 意味着任何内都包含有`hashCode()`函数.

散列表存储的是键值对(key-value). 特点是:能根据"键"快速检索出对应的"值". 这里就会用到散列码. 用来快速定位要查找的对象存储位置.

### 为什么要有 hashCode

当把对象加入 HashSet 时. HashSet会先计算对象的 hashCode 值来判断对象加入的位置, 同时会与其他加入的对象的 hashCode 值作比较, 如果没有相符的 hashCode , HashSet 会假设对象没有重复出现, 如果发现有相同的 hashCode 值的对象, 这时就会调用 equals() 方法来检查 hashCode 相等的对象是否真的相同. 如果两者相同, HashSet 就不会让其加入操作成功. 如果不同, 就会重新散列到其他位置. 这样就大大减少了 equals 的次数, 相应地就大大提高了执行速度. 

### hashCode() 与 equals() 相关规定

1. 如果两个对象相等, 则 hashCode 一定也是相同的
2. 两个对象相等, 调用 equals() 方法是返回一定是 true
3. 两个对象有相同的 hashCode 值, 它们不一定是相等的
4. 因此, equals 方法被重写时, hashCode 也一定要重写覆盖
5. hashCode 的默认行为是对堆上的对象产生独特值, 如果没有重写 hashCode , 则该 class 的两个对象无论如何都不会相等(即使这两个对象指向相同的数据)

## 线程 进程 程序

- **线程**

  线程与进程类似, 但线程是一个比进程更小的执行单位. 一个进程在其执行过程中可以产生多个线程. 同类的多个线程共享同一块内存和一组系统资源, 所以系统在产生一个线程, 或者在各个线程之间切换工作时, 负担要比进程小得多, 因此, 线程也被称为轻量级进程

- **进程**

  进程是程序的一次执行过程,是系统运行程序的基本单位,因此进程是动态的. 系统运行一个程序即使一个进程从创建, 运行到消亡的过程. 一个正在执行的程序就是一个进程, 它在计算机中一个一个指令地执行者, 同时每个进程占有某些系统资源, 如CPU时间, 内存空间, 文件, 输入输出设备的使用权等. 当程序被执行时, 会被操作系统载入到内存中.

  线程和进程的最大的不同在于基本上各进程是独立的, 而同一进程中的各线程极有可能会相互影响.

- **程序**

  程序是含有指令和数据文件, 被存储在硬盘或其他的数据存储设备中的静态代码

## 线程的基本状态

![UTOOLS1552377828574.png](https://i.loli.net/2019/03/12/5c8767e4c9af4.png)

Java 线程状态变化如下图

![UTOOLS1552377928385.png](https://i.loli.net/2019/03/12/5c876847460dd.png)

线程在创建之后处于 **NEW(新建)** 状态, 调用 `start()` 方法后开始运行后, 线程处于 **READY(可运行)** 状.可运行状态状态的线程获得了 CPU 时间片(timeslice)后就处于 **RUNNING(运行)** 状态.

> 操作系统隐藏 Java 虚拟机(JVM)中的 RUNNABLE 和 RUNNING 状态, 它只能看到 RUNNABLE 状态. 所以 Java 系统一般将这两个状态统称为 **RUNNABLE（运行中）** 状态.

![RUNNABLE-VS-RUNNING](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-3/RUNNABLE-VS-RUNNING.png)

当线程执行 `wait()` 方法之后, 线程进入 **WAITING(等待)** 状态. 进入等待状态的进程需要依靠其他线程的通知才能返回到运行状态. 而 **TIME_WATING(超时等待)** 状态相当于在等待状态的基础上增加了**超时限制**. 比如通过 `sleep(long millis)` 方法或 `wait(long millis)` 方法可以将 Java 线程置于 **TIME_WATING(超时等待)** 状态. 当超时时间到达后, Java 线程会返回到 **RUNNABLE** 状态. 当线程调用同步(synchronized)方法时, 在没有获取到锁的情况下, 线程将会进入到 **BLOCKED(阻塞)** 状态. 线程在执行 Runnable 的 `run()` 方法之后将会进入到 **TERMINATED(终止)** 状态

## final static this super 关键字

### final

final 关键字主要作用在三个地方: 变量 方法 类

1. 对于一个 final 变量, 如果是基本类型的变量, 则其一旦在初始化之后便不能修改. 如果是引用类型的变量, 则在对其进行初始化之后便不能在让其指向另一个对象.
2. 当用 final 修饰一个类时, 表明这个类不能被继承. final 类中的所有成员方法都会被隐式地指定为 final 方法.
3. 使用 final 方法的原因是: 把方法锁定, 以防任何继承类修改它的含义.

### static

static 关键字主要有以下四种使用场景:

1. **修饰成员变量和成员方法**: 被 static 修饰的成员属于类, 不属于单个这个类的某个对象, 被该类的所有对象共享, 建议通过类名调用. 被 static 声明的成员变量属于静态变量. 静态变量存放在 Java 内存区域的方法区. 调用格式: `类名.静态变量名` `类名.静态方法名()`

   > 方法区与 java 堆一样, 是各个线程共享的内存区域, 它用于存储已被虚拟机加载的类信息, 常量, 静态变量, 即时编译器编译后的代码等数据. 虽然 Java 虚拟机规范把方法区描述为堆的一个逻辑部分, 但是它却有一个别名叫做  `Non-Heap`(非堆), 目的应该是与 Java 堆区分开来

2. **静态代码块**: 静态代码块定义在类中方法外, 静态代码块在非静态代码之前执行(静态代码块 -> 非静态代码块 -> 构造方法). 该类不管创建多少对象, 静态代码只执行一次. 

   > ```java
   > static{
   >  静态代码块;
   > }
   > ```
   >
   > 一个类的静态代码块可以有多个, 位置任意. 它不处于任何方法体内, JVM 加载类时会执行这些静态的代码块, 如果静态代码块有多个, JVM 将按照他们在类中出现的先后顺序依次执行它们.
   >
   > 静态代码块对于**定义在它之后的静态变量**, 可以赋值, 但是不能访问.

3. **静态内部类(static 修饰类的话只能修饰内部类)**: 静态内部类与非静态内部类之间存在一个最大的区别: 非静态内部类在编译完成之后隐含地保存着一个引用, 该引用是指向创建它的外围类(`外围类名.this`), 但是静态内部类却没有. 

   > 1. 非静态内部类在外部类创建时, 会被隐式地初始化并绑定到外部类上, 而静态类则不会被初始化. 即, 静态内部类的创建是不需要依赖外围类的创建. 可以用于单例模式的延迟加载.
   > 2. 它不能够使用外围类的非 static 成员变量和方法. 

   静态内部类实现单例模式

   ```java
   public class Singleton {
       
       // 声明为 private 避免调用默认构造方法创建对象
       private Singleton() {
       }
       
       // 声明为 private 表明静态内部该类只能在该 Singleton 类中被访问
       private static class SingletonHolder {
           private static final Singleton INSTANCE = new Singleton();
       }
   
       public static Singleton getUniqueInstance() {
           return SingletonHolder.INSTANCE;
       }
   }
   ```

   当 Singleton 类加载时, 静态内部类 SingletonHolder 没有被加载进内存. 只有当调用 `getUniqueInstance()` 方法而触发 `Singleton.INSTANCE` 时 SingletonHolder 才会被加载, 此时初始化 INSTANCE 实例, 并且 JVM 能确保该实例只被实例化一次.

4. **静态导包(用来导入类中的静态资源)**: 格式为: `import static` 这两个关键字连用可以导入某个类中的指定静态资源, 并且不需要使用类名调用类中的静态成员, 可以直接使用类中静态变量和成员方法.

   ```java
   // Math. --- 将Math中的所有静态资源导入, 这时候可以直接使用里面的静态方法, 而不用通过类名进行调用
   // 如果只想导入单一某个静态方法，只需要将换成对应的方法名即可
   import static java.lang.Math.;
   // 换成import static java.lang.Math.max;具有一样的效果
    
   public class Demo {
     public static void main(String[] args) {
    
       int max = max(1,2);
       System.out.println(max);
     }
   }
   ```

#### static{} 静态代码块与 {} 非静态代码块(构造代码块)

**相同点**: 都是在 JVM 加载类时且在构造方法执行之前执行, 在类中都可以定义多个, 定义多个时按定义的顺序执行, 一般在代码块中对一些 static 变量进行赋值.

**不同点**: 静态代码块在非静态代码块之前执行(静态代码块 -> 非静态代码块 -> 构造方法). **静态代码块只在第一次 new 执行一次, 之后不再执行. 而非静态代码块在每 new 一次就执行一次**. 非静态代码块可以在普通方法中定义, 而静态代码块不行.

**非静态代码块与构造函数的区别是:** 

非静态代码块是给所有对象进行统一初始化, 而构造函数是给对应的对象初始化, 因为构造函数可以是多个的, 运行哪个构造函数就会建立什么样的对象

而无论建立哪个对象, 都会先执行相同的构造代码块. 也就是说, 构造代码块中定义的是不同对象共性的初始化内容.

### this

this 关键字用于引用类的当前实例, 如:

```
class Manager {
    Employees[] employees;
     
    void manageEmployees() {
        int totalEmp = this.employees.length;
        System.out.println("Total employees: " + totalEmp);
        this.report();
    }
     
    void report() { }
}
```

在以上示例中, this 关键字用于两个地方: 

1. `this.employees.length`: 访问 Manager 类的当前实例的变量
2. `this.report()`: 调用 Manager 类的当前实例的方法

### super

super 关键字用于从子类访问父类的变量和方法.

- `super.方法名()`: 调用父类的非 private 方法
- `super()`: 调用父类的构造方法
- `super.变量名`: 操作父类的非 private 变量

**使用 this 和 super 要注意的问题:**

- super 调用父类中的其他构造方法时, 要放在构造方法的首行. this 调用本类中的其他构造方法时, 也要放在首行
- this, super 不能用在 static 方法中

> 被 static 修饰的成员属于类, 不属于单个这个类的某个对象, 被类中所有对象共享. 而 this 代表对父类的引用, 指向父类对象. 所以, **this 和 super 是属于对象范畴的东西, 而静态方法是属于类范畴的东西**

## Java 中的异常处理

#### Java 异常类层次结构图

![UTOOLS1552380396521.png](https://i.loli.net/2019/03/12/5c8771edb9a8c.png)

在 Java 中, 所有的异常都有一个共同的祖先 `java.lang.Throwable`类. `Throwable`有两个重要的子类:`Exception(异常)`和`Error(错误)`. 二者都是 Java 异常处理的重要子类, 各自都包含大量的子类.

**Error**: 是程序无法处理的错误, 表示运行程序中较严重的问题. 大多数错误是 JVM 出现的问题. 如: Java虚拟机运行错误(VritualMachineError), 当 JVM 没有足够的内存资源执行操作时, 将出现 OutOfMemoryError  . 这些异常发生时, JVM 一般会选择线程终止.

这些错误表示故障发生于虚拟机自身, 或者发生在虚拟机试图执行应用时. 这些错误是不可查的, 错误通过 Error 的子类描述. 

**Exception**: 是程序本身可以处理的异常. Exception 类有一个重要的子类: **RuntimeException**. 该类异常由虚拟机抛出, 线程不会被终止. **NullPointerException**: 访问的变量没有任何引用对象时, 抛出该异常. **ArithmeticException**: 算数运算异常, 整数除以 0 时, 抛出该异常. **ArrayIndexOutOfBoundsException**: 数组下标越界时, 抛出该异常.

### Throwable 类常用方法

- **public String getMessage()**: 返回异常发生时的详细信息.
- **public String toString()**: 返回异常发生时的简要描述.
- **public String getLocalizedMessage()**: 返回异常对象的本地化信息. 使用 Throwable 的子类覆盖这个方法, 可以生成本地化信息. 如果子类没有覆盖该方法, 则该方法返回的信息与 `getMessage()` 返回的结果相同. 
- **public void printStackTrace()**: 在控制台上打印 Throwable 对象封装的异常信息.

### 异常处理

#### try catch

- `try`: 用于捕获异常. 其后可接任意个 `catch` 块. 如果没有 `catch` 块, 则必须跟一个 `finally` 块
- `catch`: 用于处理 `try` 捕获到的异常. 越具体的类必须越先捕获处理. 
- `finally`: 无论是否捕获或处理异常, **块中的语句都会被执行**. 当在 `try` 块或 `catch` 块中遇到 `return` 或 `throw` 语句时, `finally` 语句块将在方法返回之前执行. 

**以下 4 种特殊情况下, finally块不会被执行:**

1. 在 `finally` 语句块中第一行发生了异常. 如果在其他行发生异常, 之前行的代码仍会执行. 

2. 在前面的代码中使用了 `System.exit(int)` 退出虚拟机. 

3. 程序所在的线程死亡

   > 当所有的非守护线程中止时, 不论存不存在守护线程, 虚拟机都会kill掉守护线程从而中止程序. 
   >
   > 所以, 如果守护线程中存在 finally 代码块, 那么当所有的非守护线程中止时, 守护线程被 kill 掉, 其 finally 代码块是不会执行的. 

4. 关闭 CPU

**关于返回值:**

如果 `try` 于中中有 `return` , 返回的是 `try` 语句块中的变量值. 详细过程如下:

1. 如果有返回值, 就把返回值保存到局部变量中
2. 执行 jsr 指令跳转到 `finally` 语句中执行
3. 执行完 `finally` 语句后, 返回之前保存在局部变量中的值
4. 如果 `try` , `finally` 块中均有 `return` , 则忽略 `try` 中的 `return` , 而使用 `finally` 中的 `return`

#### throw

使用 `throw` 语句抛出异常, 在方法上声明 , 交由调用方法的对象来处理.

## 序列化

> Java 序列化技术是将对象编码成字节流. 反之, 将字节流重新构建成对象, 称之为反序列化. 实现对象的持久化.

### 实现

借助 `common-lang` 工具类

```java
import org.apache.commons.lang3.SerializationUtils;


public class Test {    
	public static void main(String[] args) {
		User user = new User();
		user.setUsername("Java");
		user.setAddress("China");
		byte[] bytes = SerializationUtils.serialize(user);
		User u = SerializationUtils.deserialize(bytes);
        System.out.println(u);
	}
}
```

**注意事项**:

1. 序列化对象必须实现序列化接口.
2. 序列化对象里面的属性是对象的话也要实现序列化接口.
3. 类的对象序列化后, 类的序列化 ID 不能轻易修改, 不然反序列化会失败.
4. 类的对象序列化后, 类的属性有增加或者删除不会影响序列化, 只是值会丢失.
5. 如果父类序列化了, 子类会继承父类的序列化, 子类无需添加序列化接口.
6. 如果父类没有序列化, 子类序列化了, 子类中的属性能正常序列化, 但父类的属性会丢失, 不能序列化.
7. 用 Java 序列化的二进制字节数据只能由 Java 反序列化, 不能被其他语言反序列化。如果要进行前后端或者不同语言之间的交互一般需要将对象转变成 Json/Xml 通用格式的数据, 再恢复原来的对象.
8. 如果某个字段不想序列化, 在该字段前加上 `transient` 关键字即可

## 反射

> Java 反射机制在程序运行时, 对于任意一个类, 都能够知道这个类的所有属性和方法. 对于任意一个对象, 都鞥能够调用它的任意一个方法和属性. 这种**动态调用对象的方法**的功能, 称为 **Java 的反射机制**
>
> 反射机制很重要的一点就是"运行时". 其使得我们可以在程序运行时加载, 探索以及使用编译期间完全未知的 `.class`文件. 也就是说, Java 程序可以在一个运行时才得知名称的 `.class` 文件, 然后获悉其完整构造, 并生成对象实体, 或对其变量 field 进行操作, 或调用其方法 method.

**获取Class类的三种方法**:

- 类名.class
- 对象名.getClass()
- Class.forName("要加载的类名")

### 主要方法

- `Class.getName()`: 获取类的名称
- `Class.getFields()`: 获取当前类及其所继承的父类的 `public` 变量
- `Class.getDeclaredFields()`: 获取当前类的所有成员变量, 不论访问权限
- `Class.getMethods()`: 获取当前类及其所继承的父类的 `public` 方法
- `Class.getDeclaredMethods()`: 获取当前类的所有成员方法, 不论访问权限
- `Method.setAccessible(true)`: 获取当前方法的访问权限, 操作私有方法时必须设置, 否则会报异常 `IllegalAccessException`
- `Field.setAccessible(true)`: 获取当前变量的访问权限, 操作私有方法时必须设置, 否则会报异常 `IllegalAccessException`
- `Method.invoke()`: 调用当前方法
- `Field.set()`: 修改当前变量的值

#### 修改常量的特殊情况

使用 static final 修饰的常量值, 在 JVM 编译时, 会在常量被使用的地方将**常量名**直接替换为**常量值**来优化代码. 而有些数据类型不会被优化.

要想避免上面出现的特殊情况, 有两种方法来修改常量的值

**方法一:**

```java
public class TestClass {

    //......
    private final String FINAL_VALUE;

    //构造函数内为常量赋值 
    public TestClass(){
        this.FINAL_VALUE = "FINAL";
    }
    //......
}
```

输出:

```java
Before Modify：FINAL_VALUE = FINAL
After Modify：FINAL_VALUE = Modified
Actually ：FINAL_VALUE = Modified
```

解释: 将赋值放在构造函数中, 构造函数只有在 new 对象时才会调用, 所以不会在**编译阶段**被直接优化为**常量值**, 而是指向**常量名**. 这样就可以在**运行阶段**来通过反射修改常量

**方法二**:

将声明常量的语句改为使用三目表达式赋值:

```java
private final String FINAL_VALUE
        = null == null ? "FINAL" : null;
```

因为 `null == null ? "FINAL" : null` 是在运行时刻计算的, 在编译时刻不会计算, 也就不会被优化.

##### 判断是否能修改

![UTOOLS1552490462913.png](https://i.loli.net/2019/03/13/5c891fe08022f.png)

## 大数值

BigInteger 类实现了任意精度的整数运算, BigDecimal 类实现了任意精度的浮点数运算.

应用在需要精确运算结果的场合, 如: 涉及金钱汇率以及不同表达式计算出的结果的比较

## 类加载及初始化顺序

1. 基类的静态代码块, 基类的静态成员字段.

   并列优先级, 按代码中出现的先后顺序执行. (只在第一次加载类时执行, 即 `<clinit>()` 方法.)

2. 派生类的静态代码块, 派生类的静态成员字段.

   并列优先级, 按代码中出现的先后顺序执行. (只在第一次加载类时执行, 即 `<clinit>()` 方法.)

3. 基类普通代码块, 基类普通成员字段.

   并列优先级, 按代码中出现的先后顺序执行. 

4. 基类构造函数.

   3, 4 在每次实例化对象时执行. 即在 `<init()>` 方法中.

5. 派生类普通代码块, 派生类普通成员字段.

   并列优先级, 按代码中出现的先后顺序执行. 

6. 派生类构造函数.

   5, 6 在每次实例化对象时执行. 即在 `<init()>` 方法中.

说明:

- 因为派生类的构造函数中会如果没有显示调用 `this()`, 会隐式调用 `super()`. 所以 5,6 会在 3,4 之后.
- 如果在初始化一个派生类的实例时, 使用到了重写的方法, 那么基类和派生类都会使用派生类重写的此方法.

### 示例

**基类**

```java
/**
 * Father
 */
public class Father {
    private static int j = setj();
    static {
        System.out.println("Father static block");
    }
    {
        System.out.println("Father non-static block");
    }
    private int i = seti();
    Father() {
        System.out.println("Father constructor");
    }
    private int seti() {
        System.out.println("Father non-static field ");
        return 0;
    }
    private static int setj() {
        System.out.println("Father static field");
        return 0;
    }
}
```

**派生类**

```java
/**
 * Son
 */
public class Son extends Father {
    private int i = seti();
    private static int j = setj();
    static {
        System.out.println("Son static block");
    }
    {
        System.out.println("Son non-static block");
    }
    Son() {
        System.out.println("Son constructor");
    }
    private int seti() {
        System.out.println("Son non-static field");
        return 0;
    }
    private static int setj() {
        System.out.println("Son static field");
        return 0;
    }
}
```

**测试代码及结果**

```java
/**
 * demo01
 */
public class demo01 {
    public static void main(String[] args) {
        Son s = new Son();
        System.out.println("----------------");
        Son s2 = new Son();
    }
}
```

```
Father static field
Father static block
Son static field
Son static block
Father non-static block
Father non-static field 
Father constructor
Son non-static field
Son non-static block
Son constructor
----------------
Father non-static block
Father non-static field 
Father constructor
Son non-static field
Son non-static block
Son constructor
```

注意: 在程序运行时, JVM 首先会试图访问 `main()` 方法, 并会加载其所在的类.

## Java 线程 join() 方法

`join()` 方法的源码如下:

```java

    /**
     * Waits at most {@code millis} milliseconds for this thread to
     * die. A timeout of {@code 0} means to wait forever.
     *
     * <p> This implementation uses a loop of {@code this.wait} calls
     * conditioned on {@code this.isAlive}. As a thread terminates the
     * {@code this.notifyAll} method is invoked. It is recommended that
     * applications not use {@code wait}, {@code notify}, or
     * {@code notifyAll} on {@code Thread} instances.
     *
     * @param  millis
     *         the time to wait in milliseconds
     *
     * @throws  IllegalArgumentException
     *          if the value of {@code millis} is negative
     *
     * @throws  InterruptedException
     *          if any thread has interrupted the current thread. The
     *          <i>interrupted status</i> of the current thread is
     *          cleared when this exception is thrown.
     */
    public final synchronized void join(long millis)
    throws InterruptedException {
        long base = System.currentTimeMillis();
        long now = 0;

        if (millis < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }

        if (millis == 0) {
            while (isAlive()) {
                wait(0);
            }
        } else {
            while (isAlive()) {
                long delay = millis - now;
                if (delay <= 0) {
                    break;
                }
                wait(delay);
                now = System.currentTimeMillis() - base;
            }
        }
    }
```

简单使用示例:

```java
public static void main(String[] args) throws InterruptedException{
    Thread ta = new Thread(() -> System.out.println("haha"));
    ta.start();
    ta.join()
}
```

刚开始不理解为什么 `a.join()` 会使 main 线程等待. 这里对源码 `isAlive()` 和 `wait(0)` 做下说明:

- `isAlive()` 判断的是判断调用 `join()` 方法的线程是否存活. 

  完整的方法签名是: 

  ```java
  /**
  * Tests if this thread is alive. A thread is alive if it has been started and has not yet died.
  */
  public final native boolean isAlive();
  ```

  这个方法调用的是 Thread 类中的本地方法, 是普通的方法调用, 所以判断的是当前对象线程是否存活.

- `wait()` 方法是 Object 类中的方法, 调用该方法会使获取当前对象锁的线程等待.

  在示例中, `ta.join()` 方法是在主线程中调用, `join()` 方法是**同步方法**, 所以当前线程(main)在调用 `ta.join()` 时, 获取了 `join()` 方法上的对象(ta)的锁. 所以调用 `join()` 执行到 `wait()` 方法时会使 main 线程等待, 直到 ta 线程执行结束后, main 线程才会继续执行.

## 参考资料

- JavaGuide: <https://github.com/Snailclimb/JavaGuide
- Java 核心技术 第十版