---
title: Spring
date: 2019/03/16 15:27
categories:
- Java
- Framework
tags:
- Java
- Spring
---

## Spring IOC

### IOC 和 DI 简介

> IOC(Inversion of Control)控制翻转, 包含了两个方面: 控制 和 反转
>
> 简单理解:
>
> **控制**: 当前对象对内部成员的控制权
>
> **反转**:控制权不由当前对象管理, 由其他类 / 第三方容器来管理

IOC 不够开门见山, 于是 Martin Fowler 提出了 DI(Dependency Injection)来替代 IOC. 即让调用类对某一个接口实现类的依赖关系由第三方(容器或协作类)注入, 以移除调用类对某一个接口实现类的依赖.

> 通过 DI, 对象的依赖关系将有系统中负责协调各对象的第三方组件在创建对象的时候进行设定, 对象无须自行创建或管理他们的依赖关系. 依赖关系将被自动注入到需要它们的对象当中去.

使用 IOC 的好处:

1. 不用自己组装, 拿来就用
2. 单例, 效率高, 不浪费空间
3. 便于单元测试, 方便切换 mock 组件
4. 便于进行 AOP 操作, 对于使用者是透明的
5. 统一配置, 便于修改

### 原理

IOC 容器其实就是一个大工厂, 用来管理我们所有的对象以及依赖关系

- 通过反射获取类的所有信息
- 通过配置文件或注解来描述类与类之间的关系
- 结合配置信息和反射来构建出对应的对象和依赖关系

Spring IoC 容器实现对象的创建和依赖:

![1962542e-a76a-427a-a289-00f4315546cb.png](https://i.loli.net/2019/03/16/5c8cb48e17ac1.png)

1. 根据 bean 配置信息在容器内创建 bean定义注册表.
2. 根据注册表加载、实例化 bean, 建立 bean 与 bean 之间的依赖系.
3. 将这些准备就绪的 bean 放到 Map 缓存池中, 等待应用程序调用.

## AOP

> AOP(Aspect Oriented Programming)面向切面编程: 通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术. 利用 AOP 可以对业务逻辑的各个部分进行隔离, 从而使业务逻辑各部分之间的耦合度降低, 提高程序的可用性, 同时提高了开发的效率.

### 术语

- 连接点(Join point)

  连接点是程序执行过程中能够应用通知的所有点. 目前 Spring AOP 仅支持方法级.

- 通知(Advice)

  通知是指拦截到切点之后要做的事情就是通知, 包含了需要用于多个应用对象的横切行为.

  通知的 5 种类型:

  - `Before`: 在方法被调用之前调用
  - `After`: 在方法完成后调用通知, 无论方法是否执行成功
  - `After-returning`: 在方法成功执行之后调用通知
  - `After-throwing`: 在方法抛出异常后调用通知
  - `Aroud`: 通知包含了被通知的方法, 在被通知的方法调用之前和调用之后执行定义的行为

- 切点(Pointcut)

  切点定义了通知被应用的具体位置. 切点定义了那些连接点会得到通知.

- 切面(Aspect)

  切面是通知和切点的结合. 通知和切点共同定义了切面的全部内容--它是什么, 在何时和在何处完成其功能.

- 引入(Introduction)

  引入允许我们向现有的类添加新方法或属性.

- 织入(Weaving)

  织入是把切面应用到目标对象并创建新的代理对象的过程. 切面在指定的连接点被织入到目标对象中. 在目标对象的生命周期里有多个点可以进行织入.

- 顾问(Advisor)

  顾问是切面的一种, 能够将通知以更为复杂的方式织入到目标对象中, 是将通知包装为更复杂的切面的装配器.

## Bean

**Spring 容器(Bean 工厂)可简单分成两种**:

- BeanFactory: 这是 Spring 中较原始的 Factory, 无法支持 Spring 的许多插件, 如: AOP, Web 应用等.
- ApplicationContext: 这是 BeanFactory 派生而来, 还继承了其他许多接口, 提供了更多的功能. 因此, 大多数场合都是使用 ApplicationContext.

**BeanFactory 类继承体系**

![UTOOLS1552725831513.png](https://i.loli.net/2019/03/16/5c8cb74808c6e.png)

**ApplicationContext 类继承体系**

![UTOOLS1556241780181.png](https://i.loli.net/2019/04/26/5cc25d768b32f.png)

其中在 ApplicationContext 子类中还有一个比较重要的: WebApplicationContext: 专门为 Web 应用服务

**Spring 与 Web 应用的上下文融合**:

![UTOOLS1556241811898.png](https://i.loli.net/2019/04/26/5cc25d9443088.png)

### Bean 的生命周期

**BeanFactory 的生命周期**:

![UTOOLS1556241836670.png](https://i.loli.net/2019/04/26/5cc25dad1ab75.png)

**ApplicationContext 的生命周期**

![UTOOLS1556241851733.png](https://i.loli.net/2019/04/26/5cc25dbc2af6d.png)

上图中 **设置属性值** 的下一步应是: 调用 BeanNameAware 的 setBeanName() 方法

分类对方法进行解析:

- Bean 自身的方法: 如调用 Bean 的构造方法实例化 Bean, 调用 setter 设置 Bean 的属性值以及通过 init-method 和 destroy-method 所指定的方法
- Bean 级生命周期接口方法: 如 BeanNameAware, BeanFactory, InitializingBean 和 DisposableBean, 这些接口方法由 Bean 类直接实现. 
- 容器级生命周期接口方法: 图中带 "★" 的步骤是由 InstantiationAwareBeanPostProcessor 和 BeanPostProcessor 这两个接口实现, 一般称它们的实现类为 **后处理器**. 后处理器接口一般不由 Bean 本身实现, 它们独立于 Bean, 实现类以容器附加装置的形式注册到 Spring 容器中并通过接口反射为 Spring 容器预先识别. 当 Spring 容器创建任何 Bean 的时候, 这些后处理器都会发生作用, 所以这些后处理器的影响是全局性的. 用户可以通过合理地编写后处理器, 让其仅对特定的 Bean 进行加工处理.

**ApplicationContext 与 BeanFactory 不同之处**:

- ApplicationContext 会利用 Java 反射机制自动识别出配置文件中定义的 BeanPostProcessor, InstantiationAwareBeanPostProcessor 和 BeanFactoryPostProcessor 后置器, 并自动将它们注册到应用上下文中. 而 BeanFactory 需要在代码中通过手动调用 `addBeanPostProcessor()` 方法进行注册.
- ApplicationContext 在初始化应用上下文的时候就实例化所有单实例的 Bean, 而 BeanFactory 在初始化容器的时候并未实例化 Bean, 直到第一次访问某个 Bean 时才实例化目标 Bean.

**整个步骤详细描述**:

1. Spring 找到配置文件中 Spring Bean 的定义, 再利用 Java Reflection API 对 Bean 进行实例化.

2. Spring 将值和 Bean 的引用注入到 Bean 对应的属性中.

3. 如果 Bean 实现了 BeanNameAware 接口, Spring 将 Bean 的 ID 传递给 setBeanName() 方法, 设置 Bean 的名字.

4. 如果 Bean 实现了 BeanFactoryAware 接口, Spring 将调用 setBeanFactory() 方法, 将 BeanFactory 容器实例传入.

5. 如果 Bean 实现了 ApplicationContextAware 接口, Spring 将调用 setApplicationContext() 方法, 将 Bean 所在的应用上下文的引用传入进来.

6. 与上面的类似, 如果实现了其他 *Aware 接口, 就调用相应的方法.

   > - **ApplicationContextAware**: 获得 ApplicationContext 对象, 可以用来获取所有 Bean definition 的名字.
   > - **BeanFactoryAware**: 获得 BeanFactory 对象，可以用来检测 Bean 的作用域。
   > - **BeanNameAware**: 获得 Bean 在配置文件中定义的名字.
   > - **ResourceLoaderAware**: 获得 ResourceLoader 对象, 可以获得 classpath 中某个文件. 
   > - **ServletContextAware**: 在一个 MVC 应用中可以获取 ServletContext 对象, 可以读取 context 中的参数.
   > - **ServletConfigAware**: 在一个MVC应用中可以获取 ServletConfig 对象, 可以读取config中的参数.

7. 如果 Bean 实现了 BeanPostProcessor 接口, Spring 将调用它们的 postProcessBeforeInitialization() 方法.

8. 如果 Bean 实现了 InitializingBean 接口, Spring 将调用它们的 afterPropertiesSet() 方法. 类似地, 如果 Bean 使用 init-method 声明了初始化方法, 该方法也会被调用.

9. 如果 Bean 实现了 BeanPostProcessor 接口, Spring 将调用它们的 postProcessAfterInitialization() 方法.

10. 此时 Bean 已经准备就绪, 可以被应用程序使用了. 它们将一直驻留在应用上下文, 直到该应用上下文被销毁.

11. 如果 Bean 实现了 DisposableBean 接口, Spring 将调用它的 destroy() 接口方法. 同样, 如果 Bean 使用 destroy-method 声明了销毁方法, 该方法也会被调用.

### Bean 的作用域

**Spring Framework** 支持五种作用域, 分别阐述如下表:

|   作用域    | 描述                                                         |
| :---------: | ------------------------------------------------------------ |
|  singleton  | 整个 `Spring IoC `容器内只有一个这样的 `bean`, 生成后一直保持到容器销毁 |
|  prototype  | 每次从容器获取该 `bean `都会得到一个新的实例, 会被如何保持, 会被保持多长时间, 都由使用者决定, 容器不再管理 |
| application | 整个 `ServletContext` 视为一个应用, 在这个应用内只有一个这样的 `bean`, 创建之后一直保持到该 `ServletContext ` 销毁 |
|   session   | 整个 `HTTP Session` 对应只有一个这样的 `bean`, 创建之后一直保持到该 `HTTP session` 被销毁 |
|   request   | 整个 `HTTP request `请求处理过程对应只有一个这样的 `bean`, 创建之后一直保持到该 `HTTP request` 处理完被销毁 |

五种作用域中, 通过 `scope = "singleton"(xml)`, `@Scope("singleton")(annotation)`. 其中 request, session 和 global session 三种作用域仅在基于 web 的应用中使用. 只能用在基于 web 的 Spring ApplicationContext 环境.

#### singleton - 唯一 bean 实例

当一个 bean 的作用域为 singleton, 那么 Spring IoC 容器中只会存在一个共享的 bean 实例, 并且所有对 bean 的请求. 只要 id 与该 bean 定义相匹配, 则只会返回 bean 的同一实例. 可以指定 bean 节点的 `lazy-init = "true"` 来延迟初始化 bean. 这时候, 只有在第一次获取 bean 时才会初始化 bean, 即第一次请求该 bean 时才初始化. 

#### prototype - 每次请求都会创建一个新的实例

当一个 bean 的作用域为 prototype, 表示一个 bean 定义对应多个对象实例. prototype 作用域的 bean 会导致每次对该 bean 请求时都会创建一个新的 bean 实例. **prototype 是原型类型, 它在创容器时并没有实例化, 而是当我们获取 bean 时才会去创建一个对象. 

#### request - 每一次 HTTP 请求都会产生一个新的 bean, 仅在当前 HTTP request 内有效

每一次 HTTP 请求都会产生一个新的 bean, 同时该 bean 仅在当前 HTTP request 内有效, 当请求结束后, 该对象的生命周期即宣告结束.

#### session - 一次 HTTP 请求都会产生一个新的 bean, 仅在当前 HTTP session 内有效

每一次 HTTP 请求都会产生一个新的 bean, 同时该 bean 仅在当前 HTTP session 内有效. 当 HTTP session 最终被废弃的时候, 在该 HTTP session 作用域内的 bean 也会被废弃掉.

#### globalSession

作用域类似于标准的 HTTP session 作用域. 不过仅仅在基于 portlet 的 web 应用中才有意义. Portlet 规范定义了全局 Session 的概念, 它被所有构成某个 portlet web 应用的各种不同的 portlet 所共享. 在global session 作用域中定义的 bean 被限定于全局 portlet Session 的生命周期范围内.



## 参考资料

- Spring IOC知识点一网打尽: <https://segmentfault.com/a/1190000014979704>
- JavaGuide: <https://github.com/Snailclimb/JavaGuide>
- Spring 实战 第4版

