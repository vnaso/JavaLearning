---
title: JavaEE
date: 2019/03/13 09:41
categories:
- Java
tags:
- Java
---

## Servlet

> 在 Java Web 程序中, `Servlet` 主要负责接收用户请求 `HttpServletRequest` , 在 `doGet()` , `doPost()` 中做相应的处理, 并将回应 `HttpServletResponse` 返回给用户. `Servlet` 可以设置初始化参数, 供 `Servlet` 内部使用. 一个 `Servlet` 类中只会有有一个实例, 在它初始化时调用 `init()` 方法, 销毁时掉哦用 `destroy()` 方法. `Servlet` 需要在 `web.xml` 中配置. **一个 Servlet 可以设置多个 URL 访问. ** 
>
> **Servlet 非线程安全, 多线程并发的读写会导致数据不同步的问题**. 解决的办法是尽量不要定义 name 属性, 而是把 name 变量分别定义在 `doGet() 和 doPost()` 方法中.
>
> 注意: 多线程的并发读写 Servlet 类属性会导致数据不同步. 但是如果只是并发地读取属性而不写入, 则不存在数据不同步的问题. 因此 Servlet 里的只读属性最好定义为 final 类型

### Servlet 和 CGI 区别

#### CGI 的不足:

1. 需要为每个请求启动一个 CGI 程序的系统进程. 如果请求频繁, 会带来很大开销.
2. 需要为每个请求加载和运行一个 CGI 程序, 带来很大开销. 
3. 需要重复编写处理网络协议的代码以及编码, 这些工作都是非常耗时的.

#### Servlet 的优点:

1. 只需要启动一个操作系统进程以及加载一个 JVM , 大大降低了系统的开销. 
2. 如果多个请求需要做同样处理的时候, 这时只需要加载一个类, 大大降低了开销.
3. 所有动态加载的类可以实现对网络协议以及请求解码的共享, 大大降低了工作量
4. Servlet 能直接和 Web 服务器交互, 而普通的 CGI 程序不能. Servlet 还能在各个程序之间共享数据, 使数据库连接池之类的功能很容易实现

一个基于 Java Web 的应用通常包含一个或多个 Servlet 类. Servlet 不能够自行创建并执行, 它是在 Servlet 容器中运行的, 容器将用户的请求传递给 Servlet 程序并将 Servlet 的响应回传给用户.

### Servlet 接口中的方法及生命周期

**Servlet 接口中定义了 5 个方法, 其中前三个方法与 Servlet 生命周期相关:**

1. **void init(ServletConfig config) throws ServletException**
2. **void service(ServletRequest req, ServletResponse resp) throws ServletException, java.io.IOException**  

3. **void destroy()**
4. String getServletInfo()
5. ServletConfig getServletConfig()

#### 生命周期

Web 容器加载 Servlet 并将其实例化后, Servlet 生命周期开始. 容器运行其 `init()` 方法进行 Servlet 的初始化. 请求到达时调用 Servlet 的 `service()` 方法, `service()` 方法根据需要调用与请求对应的 `doGet()` 或 `doPost()` 方法. 当服务器关闭或项目被卸载时, 服务器会将 Servlet 实例销毁. 此时会调用 Servlet 的 `destroy()` 方法. **init 和 destroy 方法只会执行一次, service 方法会在客户端每次请求 Servlet 时执行**. Servlet 中有时会用到一些需要初始化与销毁的资源. 因此可以把初始化资源的代码放入 init 方法中, 销毁资源的代码放入 destroy 中.

### 转发(Forward)和重定向的区别(Redirect)的区别

**转发是服务器行为, 重定向是客户端行为**

**转发(Forward)**

通过 RequestDispatcher 对象的 `forward (HttpServletRequest request, HttpServletResponse response)` 方法实现的. RequestDispatcher 可以通过 HttpServletRequest 的 `getRequestDispatcher()` 方法获得.

如: `request.getRequestDispatcher("login_sucess.jsp").forward(request, response);`

**重定向(Redirect)**

利用服务器返回的**状态码**实现的. 客户端浏览器请求服务器时, 服务器返回一个状态码. 服务器通过 `HttpServletResponse` 的 `setStatus(int status)` 方法设置状态码. 如果服务器返回 301 或者 302 , 则浏览器会到新的网址**重新请求该资源**

1. **从地址栏显示说**

   - forward 是服务器请求资源, 服务器直接访问目标地址的 URL , 把那个 URL 的响应内容读取过来, 然后把这些内容再发给浏览器. 浏览器端不知道服务器发送的内容来自哪里, 所以地址栏还是原来的地址

   - redirect 是服务端根据逻辑, 发送一个状态码, 告诉浏览器重新去请求哪个地址. 所以地址栏显示的是新的 URL

2. **从数据共享说**

   - forward: 转发的页面和转发到的页面可以共享 request 中的数据
   - redirect: 不能共享数据

3. **从运用方面说**

   - forward: 一般用于用户登录的时候, 根据角色转发到相应的模块
   - redirect: 一般用于用户注销登录返回主页面和跳转到其他页面等

4. **从效率说**

   - forward: 高
   - redirect: 低

### 实现会话跟踪

#### 使用Cookie

向客户端发送 Cookie

```java
Cookie c = new Cookie("name","value");	  // 创建Cookie
c.setMaxAge(60*60*24); 					// 设置最大时效，此处设置的最大时效为一天
response.addCookie(c); 				    // 把Cookie放入到HTTP响应中
```

从客户端读取 Cookie

```java
String name ="name"; 
Cookie[]cookies =request.getCookies(); 
if(cookies !=null){ 
   for(int i= 0;i<cookies.length;i++){ 
    Cookie cookie =cookies[i]; 
    if(name.equals(cookis.getName())) 
    //something is here. 
    //you can get the value 
    cookie.getValue(); 
       
   }
 }
```

**优点**: 数据可以持久保存, 不需要服务器资源. 基于文本的 key-value

**缺点**: 大小受限制, 用户可以禁用 Cookie 功能. 保存在本地, 有安全风险

#### URL 重写

在 URL 中添加用户会话的信息作为请求的参数, 或者将唯一的会话 ID 添加到 URL 结尾以标识一个会话.

**优点**: 在 Cookie 被禁用时依然可以使用

**缺点**: 必须对网站的 URL 进行编码, 所有页面必须动态生成. 不能用预先记录下来的 URL 进行访问.

#### 隐藏的表单

```
<input type = "hidden" name = "session" value = "..." />
```

**优点**: Cookie 被禁用时可以使用

**缺点**: 所有页面必须是表单提交之后的结果

#### HttpSession

最强大功能最多的会话跟踪技术. 当一个用户访问某个网站时会自动创建 HttpSession 分配一个 JSESSIONID. 每个用户可以访问他自己的 HttpSession . 可以通过 `HttpServletRequest` 对象的 `getSession()` 方法获得 HttpSession , 通过 HttpSession 对象的 `getAttribute()` 方法, 同时传入 `key` 就可以获取保存在 HttpSession 中的对象.

HttpSession 保存在服务器的内存中, 因此不要将过大的对象存放在里面. 虽然目前的 Servlet 容器可以在内存即将满时将 HttpSession 中的对象移动到其他存储设备中, 但也会影响性能. 添加到 HttpSession 中的值可以是任意 Java 对象. 这个对象最好**实现了 Serializable 接口**, 这样 Servlet 容器在必要的时候可以将其序列化到文件中, 否则在序列化时就会出现异常.

#### 使用

1. **和Cookie搭配使用**

   第一次创建 Session 的时候, 服务端会在 HTTP 协议中告诉客户端需要在 Cookie 里记录一个 Session

    ID . 以后在每次 HTTP 请求时, 客户端都会发送 Cookie 信息到服务端, Cookie 中就携带着 Session ID 来帮助服务端辨识当前请求的用户身份.

   如果客户端禁用了 Cookie 功能, 就会使用下面的方式.

2. **和URL重写技术搭配使用**

   在每次 HTTP 交互, URL 后面附带加上一个如 `sid=xxxx` 这样的参数, 服务端根据此来识别用户

**Cookie 和 Session** 的主要区别在于: Cookie 保存在客户端, Session 保存在服务端

### Get 和 Post 请求区别

|                             Get                              |                             Post                             |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
|         用来请求从服务器上获得的资源. 用于幂等的场景         |               操作服务器资源. 用户不幂等的场景               |
| 将表单中数据按照 name = value 的形式, 加上 ? 拼接到 URL 后, 各个变量之间用 & 连接. |        将表单中的数据放在 HTTP 协议的请求头或消息体中        |
| 传输的数据收到浏览器对 URL 长度的限制(最大长度为 2048 字符)  | 可以通过 request body 技术传输大量数据, 上传文件通常使用 post 方式 |
|                     参数会显示在地址栏上                     |                     数据不会显示在地址栏                     |
| 使用 MIME类型 application/x-www-form-urlencode 的 URL 编码文本的格式传递参数 |            支持更多的编码类型且不对数据类型做限制            |

### Servlet url-pattern

Servlet 对 url 进行匹配的规则分为**四类**, 优先级从高到低分别为: 

1. 精确匹配: `/user/login | /index.html`.

   请求的 url 与 url-pattern 完全一致, 则匹配成功, 否则进入下一级匹配.

2. 路径匹配: `/* | /user/*`.

   请求的 url 满足 url-pattern 的格式, 如 `/user/login` 会被匹配到 `/user/*` 的 Servlet, 匹配失败进入下一级匹配.

3. 扩展名匹配: `*.jpg | *.js`.

   请求的 url 满足 url-pattern 的格式, 如 `cat.jpg` 会被匹配到 `*.jpg` 的 Servlet, 匹配失败进入缺省匹配.

4. 缺省匹配: `/`.

   以上几个 url-pattern 都匹配失败就会进入缺省匹配的 Servlet, 缺省匹配的默认 Servlet 是 Tomcat 自身提供的 `default` Servlet.

## 参考资料

- JavaGuide: <https://github.com/Snailclimb/JavaGuide>

