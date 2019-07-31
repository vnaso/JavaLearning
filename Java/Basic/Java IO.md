---
title: Java IO
date: 2019/04/13 16:17
categories:
- Java
tags:
- Java
- NIO
---

## IO

### 简介

> IO(Input/Output), 在 Java 中可以分为 BIO, NIO, AIO 三个部分. **BIO** 是学习 Java 时最先接触到 IO 操作模块, 位于 `java.io` 包中. **NIO** 是在 JDK1.4 引入的**非阻塞IO**. **AIO** 是 NIO 的升级版, 也称为 NIO.2, 在 JDK1.7 中引入.

- **BIO**: 即传统的 `java.io` 包, 基于流模型实现, 是**同步阻塞式 IO**. 也就是说在操作输入输出流是, 在读写动作完成之前, 线程会一直阻塞在那, 它们之间的调用是可靠的线性顺序. 这种方式简单易用, 但容易造成系统资源的浪费, 虽然可以用线程池技术进行优化, 但当创建的线程数量过多时, 内存资源会被大量线程消耗殆尽, 且线程之间频繁切换也会造成资源浪费, 所以效率仍然不够高.
- **NIO**: 为解决 BIO 资源浪费问题而诞生的一种新的 IO 机制, 是**同步非阻塞式 IO**. 它提供了 Channel, Selector, Buffer 等核心组件, 用来构建**多路复用的、同步非阻塞的** IO 程序, 同时提供了更接近操作系统底层高性能的数据操作方式.
- **AIO**: 是 NIO 的升级版 NIO.2, 是一种 **异步非阻塞式 IO**, 所以常被称为 AIO(Asynchronous IO). 它基于事件和回调机制实现, 也就是说应用操作之后会直接返回, 不会阻塞, 当后台处理完成后, 操作系统会通知相应的线程进行后续操作.

![UTOOLS1556242076957.png](https://i.loli.net/2019/04/26/5cc25e9d79ce9.png)

### 同步与非同步

同步和异步关注的是**消息通信机制**

**同步(Synchronous)**: 在发出一个**调用**时, 在没有得到结果之前, 该**调用**就不会返回. 一旦调用返回, 就得到返回值. **调用**返回之前, 程序不会往下执行.

**异步(Asynchronous)**: **调用**在发出之后, 这个调用直接返回, 没有返回结果. **被调用者**通过状态, 通知来通知调用者, 或者通过回调函数处理这个调用. 因为调用直接返回, 程序会继续执行.

### 阻塞与非阻塞

阻塞和非阻塞关注的是**程序在等待调用结果(消息, 返回值)**时的状态.

**阻塞(Blocking)**: 指调用的结果返回之前, 当前线程会被挂起. 调用线程只有在得到结果之后才会返回.

**非阻塞(Nonblocking)**: 指在不能立刻得到结果之前, 当前线程不会被阻塞.

## BIO

按流分类可分为输入流和输出流:

- 输入流: 把文件中的数据读取到内存中, 只能进行读操作.
- 输出流: 将内存中的数据写入到文件中, 只能进行写操作.

按数据分类可分为字节流和字符流:

- 字节流: 以字节为单位, 每次读写都是 8位 数据(1byte), 可以操作任何类型的数据.
- 字符流: 以字符为单位, 每次读写都是按照指定编码读取一个字符, 只能操作字符类型数据.

按功能分类可分为节点流和处理流:

- 节点流: 字节与数据源相连进行读写操作.
- 处理流: 通过调用节点流类, 从而更加灵活方便地进行读写操作.

### 类结构分类

 ![按操作方式分类结构图](https://i.loli.net/2019/04/26/5cc25de67e163.png)

#### 字节流

**输入字节流 InputStream**:

- **ByteArrayInputStream**, **StringBufferInputStream**, **FileInputStream**: 是三种基本的介质流, 分别从 Byte数组, StringBuffer 和本地文件中读取数据.
- **PipeInputStream**: 是从与其它线程共用的管道中读取数据. PipedInputStream 的一个实例要和 PipedOutputStream 的一个实例共同使用, 共同完成管道的读取写入操作, 主要用于线程操作.
- **DataInputStream**: 将基础数据类型读取出来.
- **ObjectInputStream** 和所有 **FilterInputStream** 的子类都是装饰流. ObjectInputStream 用于对象反序列化.

**输出字节流 OutPutStream**:

- **ByteArrayOutputStream**, **FileOutputStream**: 是两种介质流, 分别向 Byte数组 和本地文件中写入数据.
- **PipedOutputStream**: 向与其他线程共用的管道中读取数据.
- **DataOutputStream**: 将基础数据类型写入到文件中.
- **ObjectOutputStream**, 和所有 **FilterOutputStream** 的子类都是装饰流.]

#### 字符流

**字符输入流 Reader**:

- **FileReader**, **CharReader**, **StringReader**: 是三种基本的介质流, 分别从本地文件, char数组, String 中读取数据.
- **PipedReader**: 从与其他线程共用的管道中读取数据.
- **BufferedReader**: 具有缓冲功能, 避免频繁读写.
- **InputStreamReader**: 字节流通向字符流的桥梁, 使用指定的编码读取直接并解码为字符.

**字符输出流 Writer**:

- **StringWriter**: 向 String 中写入数据.
- **CharArrayWriter**: 此类实现一个可用作 Writer 的字符缓冲区, 以字符为单位操作数据.
- **BufferedWriter**: 增加缓冲功能, 避免频繁读写.
- **PrintWriter** 和 **PrintStream**: 将对象的格式表示打印到文本输出流.
- **OutputStreamWriter**: 字符流通向字节流的桥梁, 使用指定的编码将要写入的字符编码为字节.

### 操作方式分类

![按操作对象分类结构图](https://i.loli.net/2019/04/26/5cc25e42ac015.png)

#### 节点流

**对文件进行操作(节点流)**:

- **FileInputStream** (字节输入流)
- **FileOutputStream** (字节输出流)
- **FileReader** (字符输入流)
- **FileWriter** (字符输出流)

**对管道进行操作(节点流)**:

- **PipedInputStream** (字节输入流)
- **PipedOutStream** (字节输出流)
- **PipedReader** (字符输出流)
- **PipedWriter** (字符输出流)
- **PipedInputStream** 的一个实例要和 **PipedOutputStream** 的一个实例共同使用, 共同完成管道的读取写入操作。主要用于线程操作. 

字节/字符数组流(节点流):

- **ByteArrayInputStream**
- **ByteArrayOutputStream**
- **CharArrayReader**
- **CharArrayWriter**

#### 处理流

**Buffered 缓冲流(处理流)**: 带缓冲区的处理流, 缓冲区的主要作用是: 一次性读取多个数据, 避免频繁读写磁盘, 提高数据访问效率.

- **BufferedInputStream**
- **BufferedOutputStream**
- **BufferedReader**
- **BufferedWriter**

**转化流(处理流)**:

- **InputStreamReader**: 把字节转化成字符
- **OutputStreamWriter**: 把字节转化成字符

**基本类型数据流(处理流)**: 用于操作基本数据类型值. 解决了输出数据类型转换的问题, 可以直接输出 float 或 long 类型, 提高效率.

- **DataInputStream**
- **DataOutputStream**

**打印流(处理流)**: 一般是打印到控制台, 可进行控制打印的位置.

- **PrintStream**
- **PrintWriter**

**对象流(处理流)**: 用于对象序列化.

- **ObjectInputStream**: 对象反序列化
- **ObjectOutputStream**: 对象序列化

**合并流(处理流)**:

- **SequenceInputStream**: 可以认为是一个工具类, 将两个或者多个输入流当成一个输入流依次读取.

|   分 类    |        字节输入流        |        字节输出流         |     字符输入流      |     字符输出流      |
| :--------: | :----------------------: | :-----------------------: | :-----------------: | :-----------------: |
|  抽象基类  |      *InputStream*       |      *OutputStream*       |      *Reader*       |      *Writer*       |
|  访问文件  |   **FileInputStream**    |   **FileOutputStream**    |   **FileReader**    |   **FileWriter**    |
|  访问数组  | **ByteArrayInputStream** | **ByteArrayOutputStream** | **CharArrayReader** | **CharArrayWriter** |
|  访问管道  |   **PipedInputStream**   |   **PipedOutputStream**   |   **PipedReader**   |   **PipedWriter**   |
| 访问字符串 |                          |                           |  **StringReader**   |  **StringWriter**   |
|   缓冲流   |   BufferedInputStream    |   BufferedOutputStream    |   BufferedReader    |   BufferedWriter    |
|   转换流   |                          |                           |  InputStreamReader  | OutputStreamWriter  |
|   对象流   |    ObjectInputStream     |    ObjectOutputStream     |                     |                     |
|  抽象基类  |   *FilterInputStream*    |   *FilterOutputStream*    |   *FilterReader*    |   *FilterWriter*    |
|   打印流   |                          |        PrintStream        |                     |     PrintWriter     |
| 推回输入流 |   PushbackInputStream    |                           |   PushbackReader    |                     |
|   特殊流   |     DataInputStream      |     DataOutputStream      |                     |                     |

> **注: 表中粗体字所标出的类代表节点流, 必须直接与指定的物理节点关联: 斜体字标出的类代表抽象基类，无法直接创建实.**

## NIO

### 简介

NIO 是 JDK1.4 之后引入的一套 IO 接口, NIO 所有 IO 操作都是非阻塞的. N 可以理解为 Non-blocking.和 New.

### 核心组件

#### Channel(通道)

Java NIO 中所有的 IO 操作都基于 Channel 对象, 就像流操作都要基于 Stream 对象一样.

> A channel represents an open connection to an entity such as a
> hardware device, a file, a network socket, or a program component that
> is capable of performing one or more distinct I/O operations, for
> example reading or writing.

简单来说, 一个 Channel(通道) 代表和某一实体的连接, 这个实体可以是硬件设备, 文件, 网络套接字等. 即通道是 Java NIO 提供的一座桥梁, 用于我们的程序和操作系统底层 IO 服务进行交互.

通道是一种基本的抽象的描述, 执行不同的 IO 操作, 实现各不相同, 常见的实现有:

- FileChannel: 从文件中读写数据.
- SocketChannel: 能通过 TCP 读写网络中的数据.
- ServerSocketChannel: 可以监听新进来的 TCP 连接, 像 Web 服务器那样, 对每一个新进来的连接都会创建一个 SocketChannel.
- DatagramChannel: 能通过 UDP 读写网络中的数据.

通道与流的不同:

- 既可以从同道中人读取数据, 又可以写数据到通道. 但流的读写通常是单向的.
- 通道可以异步地读写.
- 通道中的数据总是要先读到一个 Buffer, 或者总是要从一个 Buffer 中写入.

![UTOOLS1556242013872.png](https://i.loli.net/2019/04/26/5cc25e5e15702.png)

##### Channel 使用示例

###### FileChannel

```java
public class FileChannelTxt {
    public static void main(String args[]) throws IOException {
        //1.创建一个RandomAccessFile（随机访问文件）对象，
        RandomAccessFile raf=new RandomAccessFile("D:\\niodata.txt", "rw");
        //通过RandomAccessFile对象的getChannel()方法。FileChannel是抽象类。
        FileChannel inChannel=raf.getChannel();
        //2.创建一个读数据缓冲区对象
        ByteBuffer buf=ByteBuffer.allocate(48);
        //3.从通道中读取数据
        int bytesRead = inChannel.read(buf);
        while (bytesRead != -1) {
            System.out.println("Read " + bytesRead);
            //Buffer有两种模式，写模式和读模式。在写模式下调用flip()之后，Buffer从写模式变成读模式。
            buf.flip();
           //如果还有未读内容
            while (buf.hasRemaining()) {
                System.out.print((char) buf.get());
            }
            //清空缓存区
            buf.clear();
            bytesRead = inChannel.read(buf);
        }
        //关闭RandomAccessFile（随机访问文件）对象
        raf.close();
    }
}
```

1. 开启 FileChannel.

   > 由于 FileChannel 是抽象的, 所以需要通过 InputStream, OutputStream, RandomAccessFile 获取 FileChannel.

2. 通过 FileChannel 读取/写入数据.

   > 构造缓冲区 Buffer 来存放要读取或写入的数据, 然后通过方法 `read()` 和 `write()` 来读/写数据. 因为无法保证 `write()` 方法一次能向 FileChannel 中写入多少字节, 所以需要用 `while` 循环, 直到 Buffer 中已经没有尚未写入通道的字节.

3. 关闭 FileChannel.

FileChannel 中的方法:

- position(): 获取 FileChannel 的当前位置. 传入参数时可设置 position 的值.
- size(): 返回该实例所关联文件的大小.
- truncate(): 截取一个文件, 指定长度后面的部分将被删除.
- force(): 将通道里尚未写入磁盘的数据强制写到磁盘上, 处于性能方面考虑, 操作系统会将数据缓存在内存中, 所以无法保证写入到 FileChannel 里的数据一定会写到磁盘上, 要保证这一点, 需要调用 `force()` 方法. 例: `channel.fore(true)`

###### SocketChannel

SocketChannel 用于创建基于 TCP 协议的客户端对象, SocketChannel 中不存在 `accept()`方法, 通过 `connect()`, SocketChannel 对象可以连接到其他 TCP 服务器程序.

SocketChannel 中的方法:

- connect(): 如果 SocketChannel 在非阻塞模式下, 此时调用 `connect()`, 该方法可能在连接建立之前就返回了. 为了确定连接是否建立, 可以调用 `finishConnect()` 的方法:

  ```java
  socketChannel.configureBlocking(false);
  socketChannel.connect(new InetSocketAddress("http://jenkov.com", 80));
  
  while(! socketChannel.finishConnect() ){
      //wait, or do something else...
  }
  ```

- write(): 非阻塞模式下, `write()` 方法在尚未写出任何内容时可能就返回了. 所以需要在循环中调用 `write()`.

- read(): 非阻塞模式下, `read()` 方法在尚未读取到任何数据时就可能返回了, 所以需要关注它的 int 返回值, 会告诉我们读取了多少字节.

客户端:

```java
public class WebClient {
    public static void main(String[] args) throws IOException {
        //1.通过SocketChannel的open()方法创建一个SocketChannel对象
        SocketChannel socketChannel = SocketChannel.open();
        //2.连接到远程服务器（连接此通道的socket）
        socketChannel.connect(new InetSocketAddress("127.0.0.1", 3333));
        // 3.创建写数据缓存区对象
        ByteBuffer writeBuffer = ByteBuffer.allocate(128);
        writeBuffer.put("hello WebServer this is from WebClient".getBytes());
        writeBuffer.flip();
        //把数据写给服务器
        socketChannel.write(writeBuffer);
        //创建读数据缓存区对象
        ByteBuffer readBuffer = ByteBuffer.allocate(128);
        socketChannel.read(readBuffer);
        //String 字符串常量，不可变；StringBuffer 字符串变量（线程安全），可变；StringBuilder 字符串变量（非线程安全），可变
        StringBuilder stringBuffer=new StringBuilder();
        //4.将Buffer从写模式变为可读模式
        readBuffer.flip();
        while (readBuffer.hasRemaining()) {
            stringBuffer.append((char) readBuffer.get());
        }
        System.out.println("从服务端接收到的数据："+stringBuffer);

        socketChannel.close();
    }
}
```

1. 通过 SocketChannel 连接到远程服务器.
2. 创建Buffer 对象来存储读写的数据, 并通过 `read()` 和 `write()` 方法向服务端接收发送数据.
3. 关闭 SocketChannel.

###### ServerSocketChannel

ServerSocketChannel 允许我们监听 TCP 连接请求，`accept()` 方法返回一个新进来的连接的 SocketChannel. 

ServerSocketChannel 可以设置成非阻塞模式, 在非阻塞模式下, `accept()` 会立即返回, 如果还没有新进来的链接, 返回的将是 null. 因此, 需要检查返回的 SocketChannel 是否为 null.

服务器端:

```java
public class WebServer {
    public static void main(String args[]) throws IOException {
        try {
            //1.通过ServerSocketChannel 的open()方法创建一个ServerSocketChannel对象，open方法的作用：打开套接字通道
            ServerSocketChannel ssc = ServerSocketChannel.open();
            //2.通过ServerSocketChannel绑定ip地址和port(端口号)
            ssc.socket().bind(new InetSocketAddress("127.0.0.1", 3333));
            //通过ServerSocketChannel的accept()方法创建一个SocketChannel对象用户从客户端读/写数据
            SocketChannel socketChannel = ssc.accept();
            //3.创建写数据的缓存区对象
            ByteBuffer writeBuffer = ByteBuffer.allocate(128);
            writeBuffer.put("hello WebClient this is from WebServer".getBytes());
            writeBuffer.flip();
            socketChannel.write(writeBuffer);
            //创建读数据的缓存区对象
            ByteBuffer readBuffer = ByteBuffer.allocate(128);
            //读取缓存区数据
            socketChannel.read(readBuffer);
            StringBuilder stringBuffer=new StringBuilder();
            //4.将Buffer从写模式变为可读模式
            readBuffer.flip();
            while (readBuffer.hasRemaining()) {
                stringBuffer.append((char) readBuffer.get());
            }
            System.out.println("从客户端接收到的数据："+stringBuffer);
            socketChannel.close();
            ssc.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

1. 通过 `ServerSocketChannel.open()` 创建一个 ServerSocketChannel 的实例.

2. 通过 ServerSocketChannel 绑定 **ip地址** 和 **端口号**.

3. 通过 ServerSocketChannel 的 `accept()` 方法获取一个 SocketChannel.

   > `accept()` 方法会监听新进来的连接, 并在返回时包含一个新进来的连接的 SocketChannel, 所以在没有请求时会阻塞.

4. 创建Buffer 对象来存储读写的数据, 并通过 `read()` 和 `write()` 方法从客户端接收发送数据.

5. 关闭 SocketChannel 和 ServerSocketChannel(一般不关闭).

##### 通道之间的数据传输

在 Java NIO 中, 如果一个 Channel 是 FileChannel 类型的, 那么它可以直接把数据传输到另一个 Channel.

- transferFrom(): 把数据从通道源传输到 FileChannel.
- transferTo(): 把 FileChannel 数据传输到另一个 Channel.

#### Buffer(缓冲区)

Java NIO 中的 Buffer 用于和 NIO 通道进行交互.

缓冲区本质上是一块可以写入数据, 然后可以从中读取数据的内存. 数据是从通道读入缓冲区, 从缓冲区写入到通道中的. 这块内存被包装成 NIO Buffer 对象, 并提供了一组方法, 用来方便地访问这块内存.

在 Java NIO 中使用的核心缓冲区如下(覆盖了通过 IO 发送的基本数据类型: byte, char, short, int, long, float, double, long):

- ByteBuffer
- CharBuffer
- ShortBuffer
- IntBuffer
- FloatBuffer
- DoubleBuffer
- LongBuffer

##### 基本用法

使用 Buffer 读写数据一般遵循以下四个步骤;

1. 写入数据到 Buffer.
2. 调用 `flip()` 方法.
3. 从 Buffer 中读取数据.
4. 调用 `clear()` 或 `compact()` 方法.

当向 Buffer 写入数据时, Buffer 会记录写下了多少数据. 一旦要读取数据, 需要通过 `flip()` 方法将 Buffer 从写模式切换到读模式. 在读模式下, 可以读取之前写入到 Buffer 的所有数据.

> Buffer 对象有三个重要属性: position(指针位置), limit(有效数量), capacity(最大容量). Buffer 类似于一个数组, 只能从中读取索引从 position 到 limit 的数据. Buffer 在初始化后, position=0 ,limit=最大容量, capacity=最大容量. 在向其中写入数据后, position 等于当前指针的索引, limit 不变, capacity 不变. 如果此时读取 Buffer 中的数据, 只能读取当前指针的索引到有效数量的索引的无效数据. `flip()` 的作用就是将 position 赋值给 limit, 然后置 0, 这样就能获取到有效的数据, 起始索引就由 position 表示, 长度为 limit. 如图:
>
> ![UTOOLS1556242048067.png](https://i.loli.net/2019/04/26/5cc25e8076d0b.png)
>
> 其中绿色部分表示使用 `get()` 方法会获得的数据.

##### Buffer 的 capacity, position 和 limit

缓冲区本质上是一块可以写入数据, 然后可以从中读取数据的内存. 这块内存被包装成 NIO Buffer 对象, 并提供了一组方法, 用来方便地访问该块内存. 它的三个属性为:

- capacity

  作为一个内存块, Buffer 有一个固定的大小值 - capacity. 只能向里写 capacity 个 byte, long, char 等类型. 一旦 Buffer 满了, 需要将其清空(通过读数据或清除数据), 才能继续往里写数据.

- position

  当写数据到 Buffer 中时, position 表示当前位置. 初始的 position 值为 0. 当一个 byte, long 等数据写到 Buffer 后, position 就会向前移动到下一个可插入数据的 Buffer 单元. position 最大可为 capacity - 1.

  当读取数据时, 也是从某个特定位置读. 当前 Buffer 从写模式切换到读模式, position 会被置为 0. 当从 Buffer 的 position 处读取数据时(`get()`), position 向前移动到下一个可读的位置.

- limit

  在写模式下, Buffer 的 limit 表示你最多能往 Buffer 里写多少数据. 写模式下, limit 等于 Buffer 的 capacity.

  当切换 Buffer 到读模式时, limit 表示你最多能读到多少数据. **因此, 当切换 Buffer 到读模式时, limit 会被设置成写模式下的 position 值.** 也就是说, 你能读到之前写入的所有数据(limit 被设置成已写数据的数量, 这个值在写模式下就是 position).

##### Buffer 的获取与分配

要想获得一个 Buffer 对象, 首先要进行分配. 每个 Buffer 类都有一个静态的 `allocate()` 方法, 该方法会返回一个参数值指定大小的该类的实例.

```java
// 分配一个 48 字节 capacity 的 ByteBuffer
ByteBuffer buf = ByteBuffer.allocate(48);
// 分配一个 1024 字节的 CharBuffer
CharBuffer buf = CharBuffer.allocate(1024);
```

##### 操作数据

**写数据**:

- 从 Channel 中写到 Buffer.

  ```java
  int bytesRead = inChannel.read(buf); 
  ```

- 通过 Buffer 的 `put()` 方法写到 Buffer 里.

  ```java
  buf.put(123);
  ```

**读数据**:

- 从 Buffer 读取数据到 Channel.

  ```java
  int byteWritten = inChannel.write(buf);
  ```

- 使用 `get` 方法从 Buffer 中读取数据.

  ```java
  byte aByte = buf.get();
  ```

##### 其他重要方法

- flip(): 方法将 Buffer 从写模式切换到读模式. 调用方法会将 position 置 0, 并将 limit 设置成之前的 position 的值. 也就是说, position 现在用于标记读的位置, limit 表示之前写进了多少个 byte, char 等 -- 现在能读取多少个 byte, char 等.

- rewind(): 将 position 置 0, 所以可以重读 Buffer 中的所有数据, limit 保持不变.

- clear(): 一旦读完 Buffer 中的数据, 需要让 Buffer 准备好再次被写入, 可以通过此方法完成.  position 会被置 0, limit 被设置成 capacity 的值. 也就是说, Buffer 中的数据并未清除, 只是这些标记告诉我们可以从哪开始往 Buffer 中写数据. 会保留数据.

- compact(): 一旦读完 Buffer 中的数据, 需要让 Buffer 准备好再次被写入, 可以通过此方法完成. 将所有未读的数据拷贝到 Buffer 起始处, 然后将 position 设置为最后一个未读元素正后面. limit 设置为 capacity 的值. 会覆盖数据.

- mark(), reset(): 可以标记 Buffer 中的一个特定 position, 之后可以通过 Buffer.reset() 方法恢复到这个 position.

  ```java
  buffer.mark();
  //call buffer.get() a couple of times, e.g. during parsing.
  buffer.reset();	//set position back to mark.
  ```

- equals(): 可以用来比较两个 Buffer. 只比较 Buffer 中的剩余元素(position 后的元素).

  满足下列条件时, 返回真:

  1. 有相同的类型(byte, char 等).
  2. Buffer 中剩余的 byte, char 等的个数相等.
  3. Buffer 中所有的剩余的 byte, char 等都相同.

- compareTo(): 比较两个 Buffer 的剩余元素, 如果满足下列条件, 则认为前者小于后者:

  1. 第一个不相等的元素小于另一个 Buffer 中对应的元素.
  2. 所有元素都相等, 但第一个 Buffer 比另一个先耗尽.(前者 Buffer 的 capacity 小于 后者)

#### Scatter 和 Gather

Channel 提供了一种被称为 Scatter/Gather 的新功能, 也称为本地矢量 I/O. Scatter/Gather 是指在多个缓冲区上实现一个简单的 I/O 操作. 正确使用 Scatter / Gather可以明显提高性能.

大多数现代操作系统都支持本地矢量 I/O(native vectored I/O)操作. 当您在一个通道上请求一个Scatter/Gather 操作时, 该请求会被翻译为适当的本地调用来直接填充或抽取缓冲区, 减少或避免了缓冲区拷贝和系统调用.

> Scatter/Gather 应该使用直接的 ByteBuffers 以从本地 I/O 获取最大性能优势.

**Scatter/Gather 功能是通道(Channel)提供的 并不是Buffer.**

- **Scatter:** 从一个 Channel 读取的信息分散到 N 个缓冲区中(Buufer).
- **Gather:** 将 N 个 Buffer 里面内容按照顺序发送到一个 Channel.

##### Scattering Reads

Scattering reads 是把数据从单个 Channel 读取到多个 Buffer

![UTOOLS1556242030730.png](https://i.loli.net/2019/04/26/5cc25e6ee95a2.png)

示例代码:

```java
ByteBuffer header = ByteBuffer.allocate(128);
ByteBuffer body   = ByteBuffer.allocate(1024);

ByteBuffer[] bufferArray = { header, body };

channel.read(bufferArray);
```

`read()` 方法内部会负责把数据按顺序写进传入的 Buffer 数组内. 一个 Buffer 写满后, 会接着写到下一个 Buffer 中.

##### Gathering Writes

Gathering Writes 是指数据从多个 Buffer 写入到一个 Channel.

![UTOOLS1556242062285.png](https://i.loli.net/2019/04/26/5cc25e8e7f691.png)

示例代码:

```java
ByteBuffer header = ByteBuffer.allocate(128);
ByteBuffer body   = ByteBuffer.allocate(1024);

//write data into buffers

ByteBuffer[] bufferArray = { header, body };

channel.write(bufferArray);
```

`write()` 方法会按照 Buffer 在数组中的顺序, 将数据依次写入到 Channel 中, 注意只有 position 到 limit 之间的数据才会被写入. Gathering Writes 能较好地处理动态消息.

#### Selector(选择器)

Selector 是 Java NIO 中能够检测一到多个 NIO 通道, 并能够知道通道是否为读写操作做好准备的组件. 这样, 一个线程就可以管理多个通道, 从而管理多个网络连接.

##### 为什么要使用 Selector

仅用单个线程来处理多个 Channel 的好处是: 只需要更少的线程来处理通道, 事实上, 可以只用一个线程处理所有的通道. 这样对于操作系统来说, 减少了上下文切换的开销, 也减少了创建多个线程带来的资源消耗(如内存). 因此, 使用的线程越少越好.

![一个Selector管理多个Channel](https://i.loli.net/2019/04/26/5cc25e1407ad0.png)

##### 创建 Selector

通过调用 `Selector.open()` 方法可以创建一个 Selector.

```java
Selector selector = Selector.open()
```

##### 向 Selector 注册通道

为了将 Channel 和 Selector 配合使用, 必须将 Channel 注册到 Selector 上. 通过 `SelectableChannel.register()` 方法来实现.

```java
// 设置通道非阻塞
channel.configureBlocking(false);
SelectionKey key = channel.register(selector,SelectionKey.OP_READ);
```

与 Selector 一起使用时, Channel 必须处于非阻塞模式下. 这意味着不能将 FileChannel 与 Selector 一起使用, 因为 FileChannel 不能切换到非阻塞模式, 而套接字通道可以.

> SelectableChannel 抽象类的 `configureBlocking()` 方法是由 AbstractSelectableChannel 抽象类实现的, SocketChannel, ServerSocketChannel, DatagramChannel 都是直接继承了 AbstractSelectableChannel 抽象类.

`register()` 方法的第二个参数, 是一个 `interest集合`, 意思是在通过 Selector 监听 Channel 时, 对什么事件感兴趣. 可以监听的类型有: Connnect, Accept, Read, Write. 通道触发了一个事件也就意味着通道已为该事件准备就绪. 所以, 某个 Channel 成功连接到另一个服务器称之为"连接就绪"; 一个 ServerSocketChannel 准备好接收新进入的连接称之为"接收接续"; 一个通道有数据可读/可写称之为"读/写就绪".

这四个事件用 SelectionKey 的常量表示为:

- SelectionKey.OP_CONNECT

- SelectionKey.OP_ACCEPT

- SelectionKey.OP_READ

- SelectionKey.OP_WRITE

如果对不止一种事件感兴趣, 那么可以用"位或"操作符将常量连接起来

```java
int interestSet = SelectionKey.OP_READ | SelectionKey.OP_WRITE;
```

##### SelectionKey

一个 SelectionKey 键表示了一个特定的通道对象和一个特定的选择器对象之间的注册关系. 有如下方法:

- attachment(): 返回 SelectionKey 的 attachment.
- channel(): 返回该 SelectionKey 对应的 Channel.
- selector(): 返回该 SelectionKey 对应的 Selector.
- interestOps(): 返回代表需要 Selector 监控的 IO 操作的 bit mask.
- readyOps(): 返回一个 bit mask, 代表相应 Channel 上可进行的 IO 操作.

当向 Selector 注册 Channel 后, `register()` 方法会返回一个 SelectionKey 对象, 这个对象包含了一些你感兴趣的属性:

- interest set
- ready set
- Channel
- Selector
- attachment 附加的对象(可选)

**interest set**

interest 集合是你所选择的感兴趣的事件集合, 可以通过 SelectionKey 读写 interest 集合, 如:

```java
int interestSet = selectionKey.interestOps();
boolean isInterestedInAccept = (interestSet) & SelectionKey.OP_ACCEPT == SelectionKey.OP_ACCEPT;
boolean isInterestedInConnect = (interestSet) & SelectionKey.OP_CONNECT == SelectionKey.OP_CONNECT;
boolean isInterestedInRead = (interestSet) & SelectionKey.OP_READ == SelectionKey.OP_READ;
boolean isInterestedInWrite = (interestSet) & SelectionKey.OP_WRITE == SelectionKey.OP_WRITE;
```

用"位与"操作 interest 集合和给定的 SelectionKey 常量, 可以确定某个事件是否在 interest 集合中.

**ready set**

ready 集合是通道已经准备就绪的操作的集合, 在一次选择(Selection)之后, 你会首先访问这个 ready set, 如:

```java
int readySet = selectionKey.readyOps();
```

可以用像检测 interest 集合那样, 来检测 Channel 中什么事件或操作已经就绪, 也可以使用以下方法, 它们都会返回一个布尔类型:

- selectionKey.isAcceptable()
- selectionKey.isConnectable()
- selectionkey.isReadable()
- selectionKey.isWritable()

**Channel + Selector**

从 SelectionKey 访问 Channel 和 Selector 十分简单, 如:

```java
Channel channel = selectionKey.channel;
Selector selector = selectionKey.selector();
```

**附加的对象**

可以将一个对象或更多信息附着到 SelectionKey 上, 这样就能方便地识别某个给定的通道. 例如, 可以附加与通道一起使用的 Buffer, 或是包含聚集数据的某个对象:

```java
selectionKey.attach(theObject);
Object attachedObj = selectionKey.attachment();
```

还可以在用 `register()` 方法向 Selector 注册 Channel 时附加对象:

```java
SelectionKey key = channel.register(selector, SelectionKey.OP_READ, theObject);
```

##### 通过 Selector 选择通道

一旦向 Selector 注册了一个或多个通道, 就可以调用几个重载的 `select()` 方法, 这些方法返回对你所感兴趣的事件已经准备就绪的那些通道. 例如, 如果你对"读就绪"的通道感兴趣, `select()` 方法会返回读事件准备就绪的那些通道.

有以下方法可以使用:

- int select(): 阻塞到至少一个通道在你注册的事件上就绪.

  返回值表示有多少通道已经就绪, 即, **自上次调用 `select()` 后, 有多少通道编程就绪状态**. 如果调用 `select()` 方法, 因为有一个通道变成就绪状态, 返回了 1, 若再次调用 `select()` 方法, 如果另一个通道就绪了, 它会再次返回 1. 如果对第一个就绪的 Channel 没有任何操作, 现在就有两个就绪的通道, 但在每次 `select()` 方法调用之间, 只有一个通道就绪了.

- int select(long timeout): 在 `select()` 基础上增加超时等待.

- int selectNow(): 不会阻塞, 不管什么通道就绪, 都立刻返回(此方法执行非阻塞的选择操作, 如果指从前一次选择操作后, 没有通道变成可选择的, 则此方法直接返回 0).

一旦调用 `select()` 方法, 返回值不为 0 时, 则可以通过调用 Selector 的 `selectKeys()` 方法来访问已选择键集合(selected key set)中的就绪通道:

```java
Set selectedKeys = selector.selectedKeys();
```

可以遍历这个已选择的键集合来访问就绪的通道:

```java
Set selectedKeys = selector.selectedKeys();
Iterator keyIterator = selectedKeys.iterator();
while(keyIterator.hasNext()) {
    SelectionKey key = keyIterator.next();
    if(key.isAcceptable()) {
        // a connection was accepted by a ServerSocketChannel.
    } else if (key.isConnectable()) {
        // a connection was established with a remote server.
    } else if (key.isReadable()) {
        // a channel is ready for reading
    } else if (key.isWritable()) {
        // a channel is ready for writing
    }
    keyIterator.remove();
}
```

这个循环遍历已选择键集中的每个键, 并检测各个键对应的通道的就绪事件. 注意每次迭代末尾的 `keyIterator.remove()` 调用, Selector 不会自己从已选择键集中移除 SelectionKey 实例, 必须在处理完通道时自己移除. 下次该通道变成就绪时, Selector 会再次将其放入已选择键集中. 否则会重复.

`SelectionKey.channel()` 方法返回的通道需要转型成你需要处理的类型, 如 ServerSocketChannel, SocketChannel 等.

**wakeUp()**

某个线程调用 `select()` 方法后阻塞了, 即使没有通道已经就绪, 也有办法让其从 `select()` 方法返回. 只要让其他线程在第一个线程调用 `select()` 方法的那个对象上调用 `wakeUp()` 方法即可, 阻塞在 `select()` 方法上的线程会立即返回.

如果有其它线程调用了 `wakeUp()` 方法, 但当前 Selector 没有阻塞在 `select()` 方法上, 那么本次 `wakeUp()` 调用会在下一次`select()` 方法阻塞的时候生效.

> 参见: http://jm.taobao.org/2010/10/22/380/

**close()**

用完 Selector 后调用其 `close()` 方法会关闭该 Selector, 且使注册到该 Selector 上的所有 SelectionKey 实例无效, 通道本身并不会关闭.

##### 模板代码

**服务器端**

```java
ServerSocketChannel ssc = ServerSocketChannel.open();
ssc.socket().bind(new InetSocketAddress("localhost", 8080));
ssc.configureBlocking(false);

Selector selector = Selector.open();
ssc.register(selector, SelectionKey.OP_ACCEPT);

while(true) {
    int readyNum = selector.select();
    if (readyNum == 0) {
        continue;
    }

    Set<SelectionKey> selectedKeys = selector.selectedKeys();
    Iterator<SelectionKey> it = selectedKeys.iterator();
    
    while(it.hasNext()) {
        SelectionKey key = it.next();
        
    if(key.isAcceptable()) {
        // a connection was accepted by a ServerSocketChannel.
    } else if (key.isConnectable()) {
        // a connection was established with a remote server.
    } else if (key.isReadable()) {
        // a channel is ready for reading
    } else if (key.isWritable()) {
        // a channel is ready for writing
    }
        
    it.remove();
    }
}
```

##### 代码示例

**服务器端**

```java
public class NioServer {

public static void main(String[] args) throws IOException {
    // 创建一个selector
    Selector selector = Selector.open();

    // 初始化TCP连接监听通道
    ServerSocketChannel listenChannel = ServerSocketChannel.open();
    listenChannel.bind(new InetSocketAddress("127.0.0.1",9999));
    listenChannel.configureBlocking(false);
    // 注册到selector（监听其ACCEPT事件）
    listenChannel.register(selector, SelectionKey.OP_ACCEPT);

    // 创建缓冲区
    ByteBuffer buffer = ByteBuffer.allocate(1024);
    
    while (true) {
        selector.select(); //阻塞，直到有监听的事件发生
        Iterator<SelectionKey> keyIter = selector.selectedKeys().iterator();

        // 通过迭代器依次访问select出来的Channel事件
        while (keyIter.hasNext()) {
            SelectionKey key = keyIter.next();

            if (key.isAcceptable()) { // 有连接可以接受
                SocketChannel channel = ((ServerSocketChannel) key.channel()).accept();
                channel.configureBlocking(false);
                channel.register(selector, SelectionKey.OP_READ);

                System.out.println("与【" + channel.getRemoteAddress() + "】建立了连接！");

            } else if (key.isReadable()) { // 有数据可以读取
                buffer.clear();

                // 读取到流末尾说明TCP连接已断开，
                // 因此需要关闭通道或者取消监听READ事件
                // 否则会无限循环
                if (((SocketChannel) key.channel()).read(buffer) == -1) {
                    key.channel().close();
                    continue;
                } 

                // 按字节遍历数据
                buffer.flip();
                while (buffer.hasRemaining()) {
                    byte b = buffer.get();

                    if (b == 0) { // 客户端消息末尾的\0
                        System.out.println();

                        // 响应客户端
                        buffer.clear();
                        buffer.put("Hello, Client!\0".getBytes());
                        buffer.flip();
                        while (buffer.hasRemaining()) {
                            ((SocketChannel) key.channel()).write(buffer);
                        }
                    } else {
                        System.out.print((char) b);
                    }
                }
            }

            // 已经处理的事件一定要手动移除
            keyIter.remove();
        }
    }
}
}
```

**客户端**

```java
public class Client {

    public static void main(String[] args) throws Exception {
        SocketChannel socketChannel = SocketChannel.open();
        //设置为非阻塞
        socketChannel.configureBlocking(false); 
        socketChannel.connect(new InetSocketAddress("127.0.0.1", 9999));
        Selector selector = Selector.open();
        // 注册连接服务器socket的动作
        socketChannel.register(selector, SelectionKey.OP_CONNECT); 
        ByteBuffer writeBuffer = ByteBuffer.allocate(32);
        ByteBuffer readBuffer = ByteBuffer.allocate(32);

	while (true) { 
            //选择一组键，其相应的通道已为 I/O 操作准备就绪。  
            //此方法执行处于阻塞模式的选择操作。
            selector.select();
            //返回此选择器的已选择键集。
            Set<SelectionKey> keys = selector.selectedKeys(); 
            System.out.println("keys=" + keys.size()); 
            Iterator<SelectionKey> keyIterator = keys.iterator(); 
            while (keyIterator.hasNext()) { 
                SelectionKey key = keyIterator.next(); 
                // 判断此通道上是否正在进行连接操作。 
                if (key.isConnectable()) { 
                    socketChannel.finishConnect(); 
                    socketChannel.register(selector, SelectionKey.OP_WRITE); 
                    System.out.println("server connected..."); 
                    break; 
                } else if (key.isWritable()) { //写数据 
                    System.out.print("please input message:"); 
                    String message = scanner.nextLine(); 
                    //ByteBuffer writeBuffer = ByteBuffer.wrap(message.getBytes()); 
                    writeBuffer.clear();
                    writeBuffer.put(message.getBytes());
                    //将缓冲区各标志复位,因为向里面put了数据标志被改变要想从中读取数据发向服务器,就要复位
                    writeBuffer.flip();
                    socketChannel.write(writeBuffer); 
                    
                    //注册写操作,每个chanel只能注册一个操作，最后注册的一个生效
                    //如果你对不止一种事件感兴趣，那么可以用“位或”操作符将常量连接起来
                    //int interestSet = SelectionKey.OP_READ | SelectionKey.OP_WRITE;
                    //使用interest集合
                    socketChannel.register(selector, SelectionKey.OP_READ);
                    socketChannel.register(selector, SelectionKey.OP_WRITE);
                    socketChannel.register(selector, SelectionKey.OP_READ);
                    
                } else if (key.isReadable()){//读取数据
                    System.out.print("receive message:");
                    SocketChannel client = (SocketChannel) key.channel();
                    //将缓冲区清空以备下次读取 
                    readBuffer.clear();
                    int num = client.read(readBuffer);
                    System.out.println(new String(readBuffer.array(),0, num));
                    //注册读操作，下一次读取
                    socketChannel.register(selector, SelectionKey.OP_WRITE);
                }
                keyIterator.remove(); 
            } 
        }
    }
}
```

## AIO

AIO 也就是 NIO.2. 它是异步非阻塞的 IO 模型. 异步 IO 是基于事件和回调机制实现的. 在 NIO 的基础上引入了新的异步通道的概念, 并提供了异步文件通道和异步套接字通道的实现.

AIO 并没有采用 NIO 的多路复用器, 而是使用异步通道的概念. `read()` 和 `write()` 方法的返回类型都是 `Future`. 而 `Future` 模型都是异步的, 核心思想就是: 去主函数等待时间.

![UTOOLS1556241965130.png](https://i.loli.net/2019/04/26/5cc25e2d5d539.png)

### 为什么 Netty 使用 NIO 而不使用 AIO?

以下是作者原话:

- Not faster than NIO (epoll) on unix systems (which is true)
- There is no daragram suppport
- Unnecessary threading model (too much abstraction without usage)

## 参考

- 关于Java IO与NIO知识都在这里 <https://juejin.im/post/5af79bcc51882542ad771546>
- Java核心（五）深入理解BIO、NIO、AIO <https://juejin.im/post/5c048e0ff265da613a53c572>
- java IO体系的学习总结 <https://blog.csdn.net/nightcurtis/article/details/51324105>
- 一文让你彻底理解 Java NIO 核心组件 <https://segmentfault.com/a/1190000017040893>
- 并发编程网 NIO 系列教程 <http://ifeve.com>

 