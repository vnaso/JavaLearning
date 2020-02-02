---
title: Dev Note
date: 2019/06/08 23:33
categories:
- Note
tags:
- Dev
---

## DB

### MySQL 8.x jdbc 连接参数

```
jdbc:mysql://{host}:{port}/{schema}?useUnicode=true&characterEncoding=utf8&useSSL=false&serverTimezone=GMT%2B8
```

不设置时区会导致数据库无法连接, 报错: `The server time zone value 'ÖÐ¹ú±ê×¼Ê±¼ä' is unrecognized or represents more than one time zone`.

### Docker 创建的 MySQL 连接时报错：Client does not support authentication protocol requested by server; consider upgrading MySQL client

执行以下命令。[参考连接](https://stackoverflow.com/questions/50093144/mysql-8-0-client-does-not-support-authentication-protocol-requested-by-server)

```mysql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'YourRootPassword';
-- or
CREATE USER 'foo'@'%' IDENTIFIED WITH mysql_native_password BY 'bar';
-- then
FLUSH PRIVILEGES;
```

## Spring

### SpringEL 获取嵌套对象，避免 NPE

在使用 Spring EL 时，可以使用 Safe Navigation operator(`.?`) 来避免产生 NPE，如：`{Object}.?{field1}.?{field2}`。 当 `{Object}` 不存在时，不会进一步获取 `{field1}`。

## Spring Boot

### Spring Boot profile 拆分

通过 `application-{profile}` 的形式, 可实现 profile  的环境隔离和拆分.

#### 环境隔离

比如有 `prod`, `dev` 和 `beta` 三个环境. 可将环境特有的配置分别写在 `application-prod.yml`, `application-dev.yml` 和 `application-beta.yml` 中.

然后在主配置文件(通用)中通过 `spring.profiles.active : ${env}` 来切换当前环境, 达到环境隔离的效果.

#### 拆分

如果一个 profile 中配置过多, 想让文件更加简洁, 或按功能拆分, 比如配置了 redis, mybatis, 可以单独创建这两个的配置文件: `application-devRedis.yml`, `application-devMybatis.yml`. 然后在主配置文件中通过 `spring.profiles.include: devRedis,devMybatis` 来引入这两个配置.

### SpringBoot 序列化时忽略 null 字段.

添加注解的方式无法生效, 需要在 `application.yml` 中通过 `spring.jackson.default-property-inclusion: non_null` 来指定.

### Swagger @ApiImplicitParam 和 @ApiParam 的使用

`@ApiImplicParam` 用于描述接口方法参数对象的属性.

`@ApiParam` 用于描述接口方法所需的参数对象.

例:

```java
@ApiImplicitParams({
    @ApiImplicitParam(name = "people 的 name", value = "name"),
    @ApiImplicitParam(name = "people 的 age", value = "age)
    })
public void updatePeople(@ApiParam People p, @ApiParam String type){}
```

### 配置未生效？

检查配置类所在的包是否被扫描到，与 bootstrap 类不在同一包下的包需要手动添加扫描路径。

### 使用 Nginx 反向代理后获取不到真实客户端 IP

#### 起因

由于配置了 Nginx 反向代理, 使用 proxy_pass 代理了客户端请求, 导致通过 `HttpServletRequest.getRemoteAddr()` 获取到的 IP 地址为 Nginx 服务器所在的地址.

#### 解决方案

Nginx 添加配置:

```nginx
location / {
    proxy_set_header Host				$host;
    proxy_set_header X-Real-IP			$remote_addr;
    proxy_set_header X-Forwarded-For	$proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto	$scheme;
    proxy_redirect                      off;
}
```

SpringBoot 添加配置

```application.xml
server:
  use-forward-headers: true
  tomcat:
    remote-ip-header: X-Real-IP
    protocol-header: X-Forwarded-Proto
```

## Spring Security

### 使用 Spring Security 自定义 Authorization Server 时报错：`Authorization Provider not found for UsernamePasswordAuthenticationToken`。

> 情景：
>
> 自定义了一个 `AuthorizationServerConfig` 继承 `AuthorizationServerConfigurerAdapter` 来实现自定义的授权服务器并进行配置，使其支持返回自定义的 Token。在 `configure()` 方法中配置了授权类型为 `password`，所以需要提供一个 `AuthenticationManager` 的 Bean。但程序运行时，该 Bean 无法通过 `UsernamePasswordAuthenticationToken` 找到相应的 `AuthenticationProvider` 来处理登录请求。
>
> 程序还自定义了一个使用手机验证码登录的配置，提供了  `AuthenticationFilter`、`AuthenticationConfig`、`AuthenticationProvider`、`AuthenticationToken` 等相关类。

#### 问题产生原因

在编写使用手机验证码登录的相关类时，将 `AuthenticationProvider` 用 `Component` 注解标注了，意味着我们提供了 Provider，Spring Security 则不会再帮我们创建默认提供的 `UsernamePasswordAuthenticationProvider` 的 Bean，也不会向 `AuthenticationManager` 注册，所以在使用 `password` 授权模式请求时，会提示错误。这里只需要将 `@Component` 注解取消掉即可，然后手动将我们自己实现的 `AuthenticationProvider` 向 `AuthenticationManager` 注册即可。或者我们也可以自己创建一个 `DaoAuthenticationProvider`，然后向 `AuthenticationManager` 注册。

#### 源码追踪

在 `org.springframework.security.config.annotation.authentication.configuration.InitializeAuthenticationProviderBeanManagerConfigurer` 类中，有一个内部类 `InitializeUserDetailsManagerConfigurer`，它会通过 `configure(AuthenticationManagerBuilder auth)` 方法向我们的 `AuthenticationManager` 注册 `AuthenticationProvider`。其代码如下。

```java
class InitializeUserDetailsManagerConfigurer
			extends GlobalAuthenticationConfigurerAdapter {
		@Override
		public void configure(AuthenticationManagerBuilder auth) {
      // 判断 Builder 是否已经被建造配置过了
			if (auth.isConfigured()) {
				return;
			}
      // 这一行代码会获取所有 AuthenticationProvider 类型的 Bean。如果我们给自定义的 AuthenticationProvider 添加了注解，在这就会被获取。 
			AuthenticationProvider authenticationProvider = getBeanOrNull(
					AuthenticationProvider.class);
      
			if (authenticationProvider == null) {
				return;
			}

			// 如果有自定义的 AuthenticationProvider，就会注册
			auth.authenticationProvider(authenticationProvider);
		}

		/**
		 * @return
		 */
		private <T> T getBeanOrNull(Class<T> type) {
			String[] userDetailsBeanNames = InitializeAuthenticationProviderBeanManagerConfigurer.this.context
					.getBeanNamesForType(type);
			if (userDetailsBeanNames.length != 1) {
				return null;
			}

			return InitializeAuthenticationProviderBeanManagerConfigurer.this.context
					.getBean(userDetailsBeanNames[0], type);
		}
	}
```

上面的类继承自一个叫做 `GlobalAuthenticationConfigurerAdapter` 的类，该类用来对 `AuthenticationManger` 进行配置，它一共有 5 个实现类。比较关键的是以下两个类。

- `InitializeAuthenticationProviderBeanManagerConfigurer`
- `InitialUserDetailsBeanManagerConfigurer`

以上两个类中都有一个内部类 `InitializeUserDetailsManagerConfigurer` 实现了  `GlobalAuthenticationConfigurerAdapter`。`InitializeAuthenticationProviderBeanManagerConfigurer` 的代码如上面所示，会去寻找程序中自定义的 `AuthenticationProvider` 的 Bean，并将其向 `AuthenticationManager` 注册，而 `InitialUserDetailsBeanManagerConfigurer` 则会寻找程序中是否有 `UserDetailsService` 的 Bean，如果有，则使用它创建一个 `DaoAuthenticationProvider`，并向 `AuthenticationManager` 注册。具体代码如下。

```java
class InitializeUserDetailsManagerConfigurer
			extends GlobalAuthenticationConfigurerAdapter {
		@Override
		public void configure(AuthenticationManagerBuilder auth) throws Exception {
			if (auth.isConfigured()) {
				return;
			}
			UserDetailsService userDetailsService = getBeanOrNull(
					UserDetailsService.class);
			if (userDetailsService == null) {
				return;
			}

			PasswordEncoder passwordEncoder = getBeanOrNull(PasswordEncoder.class);
			UserDetailsPasswordService passwordManager = getBeanOrNull(UserDetailsPasswordService.class);

			DaoAuthenticationProvider provider = new DaoAuthenticationProvider();
			provider.setUserDetailsService(userDetailsService);
			if (passwordEncoder != null) {
				provider.setPasswordEncoder(passwordEncoder);
			}
			if (passwordManager != null) {
				provider.setUserDetailsPasswordService(passwordManager);
			}
			provider.afterPropertiesSet();

			auth.authenticationProvider(provider);
		}

		/**
		 * @return
		 */
		private <T> T getBeanOrNull(Class<T> type) {
			String[] userDetailsBeanNames = InitializeUserDetailsBeanManagerConfigurer.this.context
					.getBeanNamesForType(type);
			if (userDetailsBeanNames.length != 1) {
				return null;
			}

			return InitializeUserDetailsBeanManagerConfigurer.this.context
					.getBean(userDetailsBeanNames[0], type);
		}
	}
```

以上代码看似 Spring Security 会帮我们将 `AuthenticationProvider` 都向 `AuthenticationManager` 中注册，并在有 `UserDetailsService` Bean 的情况下在帮我们创建并注册一个 `DaoAuthenticationProvider`。但所有的 Configurer 中，只会有一个 Configurer 会起作用，原因是 Configurer 中的如下代码。

```java
if (auth.isConfigured()) {
  return;
}

public boolean isConfigured() {
  return !authenticationProviders.isEmpty() || parentAuthenticationManager != null;
}
```

这几行代码意味着只要任何一个 Configurer 对 `AuthenticationManager` 进行配置了，那么后面的 Configurer 都不再进行配置。那么 Configurer 的顺序就十分重要了，通过源码可以发现，`InitializeAuthenticationProviderBeanManagerConfigurer` 的优先级要比 `InitialUserDetailsBeanManagerConfigurer` 的优先级高，代码如下。

```java
@Order(InitializeAuthenticationProviderBeanManagerConfigurer.DEFAULT_ORDER)
class InitializeAuthenticationProviderBeanManagerConfigurer
		extends GlobalAuthenticationConfigurerAdapter {

	static final int DEFAULT_ORDER = InitializeUserDetailsBeanManagerConfigurer.DEFAULT_ORDER
			- 100;
}

-------------------------
  
@Order(InitializeUserDetailsBeanManagerConfigurer.DEFAULT_ORDER)
class InitializeUserDetailsBeanManagerConfigurer
		extends GlobalAuthenticationConfigurerAdapter {

	static final int DEFAULT_ORDER = Ordered.LOWEST_PRECEDENCE - 5000;
}

// 值越小优先级越高
InitializeUserDetailsBeanManagerConfigurer.DEFAULT_ORDER - 100 < Ordered.LOWEST_PRECEDENCE - 5000
```

即，在同时配置了自定义 `AuthenticationProvider` 类型的 Bean 和 `UserDetailsService` 的情况下，程序会优先使用我们自定义的 `AuthenticationProvider`，而不会自动创建 `DaoAuthenticationProvider`。

#### 总结

在通过继承 `AuthorizationServerConfigurerAdapter` 来实现自定义授权服务器时，如果要使用 `password` 授权模式，同时想要 Spring Security 帮我们自动创建一个 `DaoAuthenticationProvider` 并向 `AuthenticationManager` 注册，我们需要保证程序中没有 `AuthenticationProvider` 相关的 Bean。

**源码追踪的心得**
这次源码追踪，先是通过 `OAuth2AuthorizationServerConfiguration` 这个自动装配的配置类作为起点，观察到它会调用 `authenticationConfiguration.getAuthenticationManager();` 来获取 `AuthenticationManager` 对象。追踪 `getAuthenticationManager()` 方法，可以看到 `AuthenticationConfiguration` 会使用所有（一个列表） `GlobalAuthenticationConfigurerAdapter` 的 Configurer 来构造一个 `AuthenticationManager` 对象。代码如下。

```java
for (GlobalAuthenticationConfigurerAdapter config : globalAuthConfigurers) {
  authBuilder.apply(config);
}

authenticationManager = authBuilder.build();
```

然后进入 `GlobalAuthenticationConfigurerAdapter` 类，发现它是一个接口，通过寻找它的实现类和观察实现类的方法实现，并通过模拟不同的情景（提供自定义 `AuthenticationProvider` 的 Bean），观察代码执行的不同结果，明白了其中的逻辑。

### WebSecurityConfigurerAdapter 和 ResourceServerConfigurerAdapter 之间的关系

这两个类的都将作为 `FilterChainProxy` 中的 `Filter`。而这两个类的作用类似，都用于对请求进行权限验证。所以，对于两个配置中都匹配的请求，使用哪个 `Filter` 来进行处理就取决于他们定义的 `Order` 的值。

在 `WebSecurityConfigurerAdapter` 中，其 `Order` 值为 `100`。

```java
@Order(100)
public abstract class WebSecurityConfigurerAdapter implements WebSecurityConfigurer<WebSecurity> {}
```

`ResourceServerConfigurerAdapter` 需要使用 `@EnableResourceServer` 注解标注，其 `Order` 值为 `3`。

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import({ResourceServerConfiguration.class})
public @interface EnableResourceServer {
}
// --------------------------------------
@Configuration
public class ResourceServerConfiguration extends WebSecurityConfigurerAdapter implements Ordered {
    private int order = 3;
}
```

所以，`ResourceServerConfigurerAdapter` 类型的 `Filter` 会先于 `WebSecurityConfigurerAdapter` 执行。

就 OAuth2 的逻辑来说，一般是用户请求受保护的资源，资源服务器获取请求中的 `Token`，如果没有 `Token` 进行失败的逻辑。然后资源服务器拿 `Token` 去授权服务器获取用户权限相关信息，然后根据结果判断是否返回资源。

因此，代码的逻辑为：在 `WebSecurityConfigurerAdapter` 中配置对 `Token` 保护的相关配置，而在 `ResourceServerConfigurerAdapter` 中配置对资源保护的相关配置。

这样用户的请求最先被资源服务器拦截处理（如进行 `Bearer` 认证），如果认证不通过，返回失败，或导向认证服务器进行登录。同时也避免了因为在 `WebSecurityConfigurerAdapter` 中配置了 `anyRequest().authenticated()` 而将所有请求（包括资源请求）拦截并当做登录请求处理（如进行 `Basic` 认证），导致资源请求无法被正确处理。

参考：[WebSecurityConfigurerAdapter与ResourceServerConfigurerAdapter](https://www.jianshu.com/p/fe1194ca8ecd)

## Thymeleaf

### Thymeleaf 结合 SpringEL 使用正则表达式

在 Thymeleaf 中使用 SpringEL 进行正则表达式判断时，像 `${'key' matches 'RegExKey'}` 这种动态的表达式中的键（`key`，`RegExKey`）不会被 Thymeleaf 解析处理，而会直接当做 SpringEL 表达式处理。所以往往结果都是 `false`。需要使用预处理操作符 `__${...}__` 来让 Thymeleaf 预处理需要动态设置的值，如：`${'__${key}__' matches '__${RegExKey}__'}` 然后就可以正确被 SpringEL 计算得到结果。



## Code

### 使用正则匹配时，匹配的正文内容如含有正则相关字符，因未转义而引发错误

以如下代码为例，替换 `str` 中的 `[abc]` 为 `ABC`。

```java
String str = "[abc]def。".replaceFirst("[abc]","ABC");
```

结果可能与预期不符，如下。

```
[ABCbc]def
```

这是因为 `[` 和 `]` 都是正则表达式中的语法，传入的内容未经转义，所以造成此结果。解决办法有：

1. 将内容转义，如 `replaceFirst("\\[abc\\]","ABC");`。
2. 使用 `Pattern.quote()` 方法，该方法会将返回参数转义后的字符串（在首部加上 `\Q`，在尾部加上 `\E`）。



