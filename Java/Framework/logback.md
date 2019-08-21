---
title: Logback学习笔记
date: 2019/02/15 19:24
categories:
- Java
- Framework
tags:
- Logback
- Java
---

# Logback使用

## 标签属性介绍

### 根标签 - configuration

#### 属性

- `scan`:布尔值.表示是否自动扫描`logback.xml`的文本变化.

- `scanPeriod`:字符串.表示每间隔多长时间对`logback.xml`进行扫描.格式:`数字 + 时间单位(如:seconds,minutes)`
- `debug`:布尔值.表示是否打印`logback`内部的日志信息.

> 常用配置为:`<configuration scan="true" scanPeriod="60 seconds" debug="false"></configuration>`
>
> 表示自动每60秒扫描一次`logback.xml`的配置有无变化,如果有就更新,且不打印`logback`的内部日志.这样可以热更新`logback`配置文件

#### 子标签

- `<property>`:定义参数常量
- `<appender`:
- `<root>`:

### configuration - property

#### 属性

- `name`:变量名
- `value`:变量值

> 常用来设置一些配置文件中需要使用的常量值,如默认日志等级,最大保存天数,日志存储位置等.

### configuration - appender

#### 属性

- `name`:变量名
- `class`:具体的实现类的全限定类名

> 用来定义日志的输出源的配置
>
> `class`的取值一般有两个:`ch.qos.logback.core.ConsoleAppender`和`ch.qos.logback.core.rolling.RollingFileAppender`
>
> 前者的功能是在输出到控制台,后者功能可以按时间分卷输出到文件

#### 子标签

- `<encoder>`:把日志转为字符串并将其输出到文件中

- `<encoding>`:编码方式
- `<filter>`:过滤器
- `<file>`:日志文件输出储存位置
- `<rollingPolicy>`:分卷模式

### configuration - appender - encoder

#### 子标签

- `<pattern>`:日志格式

### configuration - appender - rollingPolicy

#### 属性

- `class`:分卷模式的全限定类名
- `<append>`:布尔值.日志被追加到文件结尾.如果是`false`,清空现存文件,默认是`true`

> 用来定义日志的分卷模式
>
> `class`常用的类是`ch.qos.logback.core.rolling.TimeBasedRollingPolicy`
>
> 可以按照时间进行分卷

#### 子标签

- `<filenamePattern>`:分卷日志文件名格式
- `<MaxHistory>`:日志最大保存天数

### configuration - appender - filter

#### 属性

- `class`:实现过滤规则的类的全限定类名

#### 子标签

- `<level>`:要进行过滤的级别
- `<onMatch>`:等于`level`属性值时的操作.有`NEUTRAL`(有序列表里的下个过滤器过接着处理日志),`ACCEPT`和`DENY`可选
- `<onMismatch>`:不等于时的操作,类上

> 用来定义输出源要过滤的日志等级
>
> 示例:
>
> ```xml
>      <filter class="ch.qos.logback.classic.filter.LevelFilter">
>          <level>ERROR</level>
>          <onMatch>ACCEPT</onMatch>
>          <onMismatch>DENY</onMismatch>
>      </filter>
> ```
>
> 以上配置表示只保留`Error`等级的日志信息
>
> `ch.qos.logback.classic.filter.ThresholdFilter`: 临界值过滤器, 过滤掉低于指定临界值的日志. 当日志级别高于或等于临界值时, 过滤器返回 NEUTRAL, 否则拒绝.
>
> ```xml
>      <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
>          <level>INFO</level>
>      </filter>
> ```

### configuration - logger

#### 属性

- `name`:指定为哪个包或类添加`appender`
- `level`:指定为**哪个级别及以上**记录日志,如果不指定,**则默认继承`<root>`下的`level`**
- `additivity`:是否向上级传递打印信息.默认值为`true`

#### 子标签

- `<appender-ref>`:为`name`表示的包或类指定`appender`.用属性`ref`来指定

> 用来定义包或类的日志打印级别及指定`appender`
>
> 示例:
>
> ```xml
> <logger name="com.vnaso" additivity="false" level="INFO">
>     <!-- 设置日志输出 -->
>     <appender-ref ref="vnasoRss"/>
>     <appender-ref ref="console"/>
> </logger>
> ```
>
> 表示为`com.vnaso`这个包及下面所有类指定`name`为`vnasoRss`和`console`的`appender`来记录`INFO`及以上的日志,且不向上继承.
>
> 
>
> `Appender`是绑定在`logger`上的,而`logger`又**有继承关系**,因此一个`logger`打印信息时的目的地`Appender`需要参考它的父亲和祖先.在`logback`中,默认情况下,如果一个`logger`打印一条信息,那么这条信息首先会打印至它自己的`Appender`,然后打印至它的父亲和父亲以上的祖先的`Appender`,但如果它的父亲设置了 `additivity = false`,那么这个`logger`除了打印至它自己的`Appender`外,只会打印至其父亲的`Appender`,因为它的父亲的`additivity` 属性置为了`false`,开始变得忘祖忘宗了,所以这个`logger`只认它父亲的`Appender`;此外,对于这个`logger`的父亲来说,如果父亲的`logger`打印一条信息,那么它只会打印至自己的`Appender`中(如果有的话),因为父亲已经忘记了爷爷及爷爷以上的那些父辈了.

### configuration - root

#### 属性

- `level`(only):指定为**哪个级别及以上**记录日志

#### 子标签

- `<appender-ref>`:为`name`表示的包或类指定`appender`.用属性`ref`来指定

> `<root>`标签是一种特殊的`logger`,但是它不能特别指定包或类.也就是说,它只能够接收继承了它且`additivity`值为`true`的`logger`传来的打印信息,具体查看继承关系

## 继承关系

> 继承关系是通过`logger`的`name`属性来实现的.
>
> 示例
>
> `<root>`>`com.vnaso`>`com.vnaso.controller`>`com.vnaso.controller.UserController.java`
>
> 以上对应继承关系:祖>爷>父>子

### level 继承关系

![UTOOLS1551424430122.png](https://i.loli.net/2019/03/01/5c78dbb03a3dc.png)

### appender 继承关系

![UTOOLS1551424496592.png](https://i.loli.net/2019/03/01/5c78dbf1107a0.png)

## 相关知识

如果程序运行在 Tomcat 服务器上, 可以利用 `${catalina.base}` 来设置 log 文件持久化的根目录, 如: `<property name="log.filePath" value="${catalina.base}/..."/>`.

> Tomcat 有 `${catalina.base}` 和 `${catalina.home}` 两个变量, 容易混淆, 这里说一下区别. 
>
> - `${catalina.home}`
>
>   home 一般指的是 Tomcat 的**安装目录**, 即存放 bin 和 lib 这些包含创建 Tomcat 实例所必要的文件的目录.
>
> - `${catalina.base}`
>
>   base 一般指的是 Tomcat 的**工作目录**, 即存放 `conf`, `logs`, `temp`, `webapps`, `work` 这些保存和运行程序有关的文件的目录.
>
> 一般来说, 在普通*单实例单应用*的情况下, 这两者是没有区别的. 但如果启动多个实例时, home 相同, 但是 base 是根据实例而不同的. 因此, 可以通过设置多个 base 目录, 包含必要的那些文件夹, 修改监听的端口, 就可以启动多个 Tomcat. 详见: https://www.cnblogs.com/mafly/p/tomcat.html.
>
> Tomcat 安装目录下有:
>
> - `bin`: 存放一些脚本文件, 比如: `startup.sh` 和 `shutdown.sh`.
>
> - `conf`: 存放配置文件, 比如: `server.xml` 和 `web.xml`.
> - `lib`: 存放 Tomcat 依赖的包.
> - `logs`: 存放运行时产生的日志文件.
> - `temp`: 存放运行时产生的临时文件.
> - `webapps`: 部署 Web 应用程序的默认目录, 也就是 war 包所在默认目录.
> - `work`: 存放由 JSP 文件生成的 servlet.

## 整合 Spring Boot

### 引入依赖

在 pom.xml 中添加:

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
        </dependency>
```

在 Spring Boot 中有一些 starter 已经依赖了 logging, 则不需显式添加依赖. 如 `spring-boot-starter-web`, `spring-boot-starter-aop`.

### 配置 logback-spring.xml

> 文件名末尾添加 `-spring` 可以使用 `<springProfile>` 标签来配置不同环境日志策略.

**示例配置**:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- 每 60 seconds 进行一次 scan, 不开启 logback 的内部 debug -->
<configuration scan="true" scanPeriod="60 seconds" debug="false">
    <!-- 引入 Spring Boot 默认的一些设置 -->
    <include resource="org/springframework/boot/logging/logback/defaults.xml"/>
    <!-- 引入 Spring Boot 的参数 -->
    <!-- FIXME 日志目录 -->
    <springProperty scope="context" name="logback.path" source="logging.path" defaultValue="logs"/>
    <!-- 定义参数常量 -->
    <!-- 日志最大保存天数 -->
    <property name="log.maxHistory" value="30"/>
    <property name="log.filePath" value="${logback.path}"/>
    <!-- 日志格式 -->
    <property name="log.pattern" value="===[%d{HH:mm:ss.SSS}][%p][%c{40}][%t]=== - %m%n"/>
    <!-- 单日志最大大小 -->
    <property name="log.maxFileSize" value="10MB" />
    <!-- 异步存储文件阻塞队列最大处理 event 数量 -->
    <property name="log.queueSize" value="512" />
    <!-- 控制台输出配置 DEBUG -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>${log.pattern}</pattern>
            <charset>UTF-8</charset>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>DEBUG</level>
        </filter>
    </appender>
    <!-- 项目输出配置 DEBUG -->
    <appender name="DEBUG_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- FIXME 日志名 -->
        <!-- 正在记录的日志文件名 -->
        <File>${log.filePath}/debug.log</File>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <!-- 日志归档, 索引 i 从 0 开始 -->
            <fileNamePattern>${log.filePath}/debug-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <!-- 控制单日志最大大小 -->
            <maxFileSize>${log.maxFileSize}</maxFileSize>
            <maxHistory>${log.maxHistory}</maxHistory>
        </rollingPolicy>
        <!-- 追加方式记录 -->
        <append>true</append>
        <encoder>
            <pattern>${log.pattern}</pattern>
            <charset>UTF-8</charset>
        </encoder>
        <!-- 指定日志界别过滤器, 不匹配的直接拒绝 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>debug</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>
    
    <!-- 项目输出配置 INFO -->
    <appender name="INFO_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- FIXME 日志名 -->
        <!-- 正在记录的日志文件名 -->
        <File>${log.filePath}/info.log</File>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <!-- 日志归档, 索引 i 从 0 开始 -->
            <fileNamePattern>${log.filePath}/info-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <!-- 控制单日志最大大小 -->
            <maxFileSize>${log.maxFileSize}</maxFileSize>
            <maxHistory>${log.maxHistory}</maxHistory>
        </rollingPolicy>
        <!-- 追加方式记录 -->
        <append>true</append>
        <encoder>
            <pattern>${log.pattern}</pattern>
            <charset>UTF-8</charset>
        </encoder>
        <!-- 指定日志界别过滤器, 不匹配的直接拒绝 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>info</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>
    
    <!-- 项目日志输出配置 ERROR -->
    <appender name="FILE_ERROR" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- FIXME 日志名-->
        <!-- 正在记录的日志文件名 -->
        <File>${log.filePath}/error.log</File>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>${log.filePath}/error.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <maxHistory>${log.maxHistory}</maxHistory>
            <maxFileSize>${log.maxFileSize}</maxFileSize>
        </rollingPolicy>
        <append>true</append>
        <encoder>
            <pattern>${log.pattern}</pattern>
            <charset>UTF-8</charset>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>ERROR</level>
        </filter>
    </appender>
    
    <!-- 开发环境 profile 配置 -->
    <springProfile name="dev">
        <!-- FIXME 数据库日志打印 -->
        <!--<logger name="" level="DEBUG"/>-->
        <logger name="cn.cdutacm.onlinejudge.mapper" level="DEBUG"/>
        <logger name="cn.cdutacm.onlinejudge" level="INFO"/>
        <root level="INFO">
            <appender-ref ref="CONSOLE"/>
            <appender-ref ref="INFO_FILE"/>
        </root>
    </springProfile>
    
    <!-- 测试环境 profile 配置 -->
    <springProfile name="beta">
        <!-- FIXME 数据库日志打印 -->
        <!--<logger name="" level="DEBUG"/>-->
        <logger name="cn.cdutacm.onlinejudge.mapper" level="DEBUG"/>
        <root level="INFO">
            <appender-ref ref="CONSOLE"/>
        </root>
    </springProfile>
    
    <!-- 生产环境 profile 配置 -->
    <springProfile name="prod">
        <!-- FIXME 其他需要记录日志的 logger -->
        <!--<logger name="" level="INFO" additivity="false">-->
        <!--    <appender-ref ref="FILE_INFO"/>-->
        <!--</logger>-->
        <root level="info">
            <appender-ref ref="FILE_ERROR"/>
        </root>
    </springProfile>
    

</configuration>
```

如果需要引用 `application.yml` 文件中的属性, 可以使用:

```xml
<springProperty scope="context" name="{PROPERTY_NAME}" source="{PROPERTY_KEY}" defaultValue="{DEFAULT_VALUE}"/>
```

设置日志打印颜色:

格式: `%color(日志内容)`, 可以识别的颜色有: black, red, green, yellow, blue, magenta, cyan, white, gray, bold*, highlight.

> Grouping by [parentheses](https://logback.qos.ch/manual/layouts.html#Parentheses) as explained above allows coloring of sub-patterns. As of version 1.0.5, `PatternLayout` recognizes "%black", "%red", "%green","%yellow","%blue", "%magenta","%cyan", "%white", "%gray", "%boldRed","%boldGreen", "%boldYellow", "%boldBlue", "%boldMagenta""%boldCyan", "%boldWhite" and "%highlight" as conversion words. These conversion words are intended to contain a sub-pattern. Any sub-pattern enclosed by a coloring word will be output in the specified color.