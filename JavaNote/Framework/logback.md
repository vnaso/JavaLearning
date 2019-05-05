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
>         <filter class="ch.qos.logback.classic.filter.LevelFilter">
>             <level>ERROR</level>
>             <onMatch>ACCEPT</onMatch>
>             <onMismatch>DENY</onMismatch>
>         </filter>
> ```
>
> 以上配置表示只保留`Error`等级的日志信息

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

### level继承关系

![UTOOLS1551424430122.png](https://i.loli.net/2019/03/01/5c78dbb03a3dc.png)

### appender继承关系

![UTOOLS1551424496592.png](https://i.loli.net/2019/03/01/5c78dbf1107a0.png)