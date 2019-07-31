---
title: SpringMVC
date: 2019/03/16 12:15
categories:
- Java
tags:
- Java
- SpringMVC
---

## SpringMVC 简介

SpringMVC 框架是以请求为驱动, 围绕 Servlet 设计, 将请求发给控制器, 然后通过模型对象, 分析器来展示请求结果视图. 其中核心类是 **DispatcherServlet**, 它是一个 Servlet, 顶层是实现 Servlet 的接口.

## SpringMVC 使用

XML 方式需要在 web.xml 中配置 DispatcherServlet, 并且需要配置 Spring 监听器 ContextLoaderListener.

```xml
<listener>
	<listener-class>org.springframework.web.context.ContextLoaderListener
	</listener-class>
</listener>
<servlet>
	<servlet-name>springmvc</servlet-name>
	<servlet-class>org.springframework.web.servlet.DispatcherServlet
	</servlet-class>
	<!-- 如果不设置init-param标签，则必须在/WEB-INF/下创建xxx-servlet.xml文件，其中xxx是servlet-name中配置的名称。 -->
	<init-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>classpath:spring/springmvc-servlet.xml</param-value>
	</init-param>
	<load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
	<servlet-name>springmvc</servlet-name>
	<url-pattern>/</url-pattern>
</servlet-mapping>
```

## SpringMVC 工作原理

**基本流程**:

客户端发送请求 -> 前端控制器 DispatcherServlet 接收客户端请求 -> 找到处理器映射 HandlerMapping 解析请求对应的 Handler -> HandlerAdapter 根据 Handler 来调用具体的处理器来处理请求, 并处理响应的业务逻辑. -> 处理器返回一个模型视图 ModelAndView -> 视图解析器进行解析 -> 返回一个视图对象 -> 前端控制器 DispatcherServlet 渲染数据 -> 将得到的视图对象返回给用户

**如图所示**:

![UTOOLS1552713992515.png](https://i.loli.net/2019/03/16/5c8c890a061ff.png)

**流程说明**:

1. 客户端发送请求, 请求交由 DispatcherServlet 处理.
2. DispatcherServlet 根据请求信息调用 HandlerMapping, 解析请求对应的 Handler.
3. 解析到对应的 Handler (也就是平常所说的 Controller 控制器)后, 开始有 HandlerAdapter 适配器处理.
4. HandlerAdapter 会根据 Handler 来调用具体的处理器来处理请求, 并处理相应的处理逻辑.
5. 处理器处理完业务后, 会返回一个 ModelAndView 对象, Model 是返回的数据对象, View 是个逻辑上的 View.
6. ViewResolver 会根据逻辑 View 查找实际的 View.
7. DispatcherServlet 把返回的 Model 传给 View(视图渲染).
8. 把 View 返回给客户端

### SpringMVC 重要组件说明

#### 前端控制器 DispatcherServlet

**作用**: 

SpringMVC 的入口, 接收请求, 响应结果, 相当于转发器, 中央处理器. 有了 DispatcherServlet, 减少了其他组件之间的耦合度. 用户请求到达前端控制器, 它就相当于 MVC 模式中的 C, DispatcherServlet 是整个流程控制的中心. 由它来调用其他组件处理用户的请求, DispatcherServlet 的存在降低了组件之间的耦合度.

#### 处理映射器 HandlerMapping

**作用**:

根据请求的 URL 查找 Handler. HandlerMapping 负责根据用户请求寻找能够处理请求的 Handler(处理器 Controller). SrpingMVC 提供了不同的映射器实现不同的映射方式, 例如: 配置文件方式 - `web.xml` , 实现接口方式 - `implements Controller`, 注解方式 - `@RequestMapping` (常用).

#### 处理适配器 HandlerAdapter

**作用**:

按照特定的规则去执行 Handler.

通过 HandlerAdapter 对处理器进行执行, 这是适配器模式的应用, 通过扩展适配器, 可以对更多类型的处理器进行执行.

#### 处理器 Handler (Controller)

编写 Handler 时需要遵循 HandlerAdapter 的要求, 这样适配器才能正确执行 Handler

Handler 是继 DispatcherServlet 前端控制器的后端控制器, 在 DispatcherServlet 的控制下, Handler对具体的用户请求进行处理. 

由于 Handler 涉及到具体的业务请求, 所以一般情况下需要工程师根据业务需求开发 Handler

#### 视图解析器 View Resolver

**作用:**

进行视图解析, 根据逻辑视图名解析成真正的视图(View)

View Resolver 负责将处理结果生成 View 视图, View Resolver 首先根据逻辑视图名解析成物理视图名, 即具体的页面地址, 再生成 View 视图对象, 最后对 View 进行渲染并将处理结果通过页面展示给用户. 

#### 视图 View

View 是一个接口, 实现类支持不同的 View 类型. 如: JSP, Freemaker, pdf, excel...

**注意**: 处理器 Handler 以及视图层 View 都是需要我们自己手动开发的, 其他的一些组件比如: DispatcherServlet, HandlerMapping, HandlerAdapter 等等都是

### DispatcherServlet 详细解析

**源码**:

```java
package org.springframework.web.servlet;
 
@SuppressWarnings("serial")
public class DispatcherServlet extends FrameworkServlet {
 
	public static final String MULTIPART_RESOLVER_BEAN_NAME = "multipartResolver";
	public static final String LOCALE_RESOLVER_BEAN_NAME = "localeResolver";
	public static final String THEME_RESOLVER_BEAN_NAME = "themeResolver";
	public static final String HANDLER_MAPPING_BEAN_NAME = "handlerMapping";
	public static final String HANDLER_ADAPTER_BEAN_NAME = "handlerAdapter";
	public static final String HANDLER_EXCEPTION_RESOLVER_BEAN_NAME = "handlerExceptionResolver";
	public static final String REQUEST_TO_VIEW_NAME_TRANSLATOR_BEAN_NAME = "viewNameTranslator";
	public static final String VIEW_RESOLVER_BEAN_NAME = "viewResolver";
	public static final String FLASH_MAP_MANAGER_BEAN_NAME = "flashMapManager";
	public static final String WEB_APPLICATION_CONTEXT_ATTRIBUTE = DispatcherServlet.class.getName() + ".CONTEXT";
	public static final String LOCALE_RESOLVER_ATTRIBUTE = DispatcherServlet.class.getName() + ".LOCALE_RESOLVER";
	public static final String THEME_RESOLVER_ATTRIBUTE = DispatcherServlet.class.getName() + ".THEME_RESOLVER";
	public static final String THEME_SOURCE_ATTRIBUTE = DispatcherServlet.class.getName() + ".THEME_SOURCE";
	public static final String INPUT_FLASH_MAP_ATTRIBUTE = DispatcherServlet.class.getName() + ".INPUT_FLASH_MAP";
	public static final String OUTPUT_FLASH_MAP_ATTRIBUTE = DispatcherServlet.class.getName() + ".OUTPUT_FLASH_MAP";
	public static final String FLASH_MAP_MANAGER_ATTRIBUTE = DispatcherServlet.class.getName() + ".FLASH_MAP_MANAGER";
	public static final String EXCEPTION_ATTRIBUTE = DispatcherServlet.class.getName() + ".EXCEPTION";
	public static final String PAGE_NOT_FOUND_LOG_CATEGORY = "org.springframework.web.servlet.PageNotFound";
	private static final String DEFAULT_STRATEGIES_PATH = "DispatcherServlet.properties";
	protected static final Log pageNotFoundLogger = LogFactory.getLog(PAGE_NOT_FOUND_LOG_CATEGORY);
	private static final Properties defaultStrategies;
	static {
		try {
			ClassPathResource resource = new ClassPathResource(DEFAULT_STRATEGIES_PATH, DispatcherServlet.class);
			defaultStrategies = PropertiesLoaderUtils.loadProperties(resource);
		}
		catch (IOException ex) {
			throw new IllegalStateException("Could not load 'DispatcherServlet.properties': " + ex.getMessage());
		}
	}
 
	/** Detect all HandlerMappings or just expect "handlerMapping" bean? */
	private boolean detectAllHandlerMappings = true;
 
	/** Detect all HandlerAdapters or just expect "handlerAdapter" bean? */
	private boolean detectAllHandlerAdapters = true;
 
	/** Detect all HandlerExceptionResolvers or just expect "handlerExceptionResolver" bean? */
	private boolean detectAllHandlerExceptionResolvers = true;
 
	/** Detect all ViewResolvers or just expect "viewResolver" bean? */
	private boolean detectAllViewResolvers = true;
 
	/** Throw a NoHandlerFoundException if no Handler was found to process this request? **/
	private boolean throwExceptionIfNoHandlerFound = false;
 
	/** Perform cleanup of request attributes after include request? */
	private boolean cleanupAfterInclude = true;
 
	/** MultipartResolver used by this servlet */
	private MultipartResolver multipartResolver;
 
	/** LocaleResolver used by this servlet */
	private LocaleResolver localeResolver;
 
	/** ThemeResolver used by this servlet */
	private ThemeResolver themeResolver;
 
	/** List of HandlerMappings used by this servlet */
	private List<HandlerMapping> handlerMappings;
 
	/** List of HandlerAdapters used by this servlet */
	private List<HandlerAdapter> handlerAdapters;
 
	/** List of HandlerExceptionResolvers used by this servlet */
	private List<HandlerExceptionResolver> handlerExceptionResolvers;
 
	/** RequestToViewNameTranslator used by this servlet */
	private RequestToViewNameTranslator viewNameTranslator;
 
	private FlashMapManager flashMapManager;
 
	/** List of ViewResolvers used by this servlet */
	private List<ViewResolver> viewResolvers;
 
	public DispatcherServlet() {
		super();
	}
 
	public DispatcherServlet(WebApplicationContext webApplicationContext) {
		super(webApplicationContext);
	}
	@Override
	protected void onRefresh(ApplicationContext context) {
		initStrategies(context);
	}
 
	protected void initStrategies(ApplicationContext context) {
		initMultipartResolver(context);
		initLocaleResolver(context);
		initThemeResolver(context);
		initHandlerMappings(context);
		initHandlerAdapters(context);
		initHandlerExceptionResolvers(context);
		initRequestToViewNameTranslator(context);
		initViewResolvers(context);
		initFlashMapManager(context);
	}
}

```

**DispatcherServlet 类中的属性 beans**:

- HandlerMapping: 定义请求和处理程序对象之间的映射. 用于 Handlers 映射请求和一系列的对于拦截器的前处理和后处理, 大部分用 `@Controlelr` 注解.

  - SimpleUrlHandlerMapping: 通过配置文件把 URL 映射到 Controller 类.
  - DefaultAnnotationHandlerMapping: 通过注解把 URL 映射到 Controller 类.

- HandlerAdapter: 帮助 DispatcherServlet 处理映射请求的处理程序的适配器.

  - AnnotationMethodHandlerAdapter: 通过注解把 URL 映射到 Controller 类的方法上.

- ViewResolver: 根据配置解析实际的 View 类型.

  - UrlBaseViewResolver: 通过配置文件, 提供一种拼接 URL 的方式来解析视图.

- ThemeResolver: 解决 Web 应用程序可以使用的主题, 例如提供个性化布局.

- MultipartResolver: 解析多部分请求, 以支持从 HTML 表单上传文件.

- FlashMapManager: 存储并检索可用于将一个请求属性传递到另一个请求的 input 和 output 的 .FlashMap, 通常应用于重定向. 

  > 通过方法将会话中的数据发送为 flash attribute, flash 属性会一直携带这些数据直到下一次请求, 然后才会消失. 解决了重定向时会话数据丢失的问题.

- HandlerExceptionResolver: 异常处理的接口.

  - SimpleMappingExceptionResolver: 通过配置文件进行异常处理.
  - AnnotationMethodHandlerExceptionResolver: 通过注解进行异常处理.

在 Web MVC 框架中, 每个 DispatcherServlet 都拥有自己的 WebApplicationcontext, 它继承了 ApplicationContext. WebApplicationContext 包含了其上下文和 Servlet 实例之间共享的所有的基础框架 beans.

#### 时序图

![UTOOLS1552719017735.png](https://i.loli.net/2019/03/16/5c8c9cac81708.png)

![UTOOLS1552719267990.png](https://i.loli.net/2019/03/16/5c8c9da455e1c.png)

## 参考资料

- JavaGuide: <https://github.com/Snailclimb/JavaGuide>

