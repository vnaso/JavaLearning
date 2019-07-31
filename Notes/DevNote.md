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

## Spring Boot

### Spring Boot profile 拆分

通过 `application-{profile}` 的形式, 可实现 profile  的环境隔离和拆分.

#### 环境隔离

比如有 `prod`, `dev` 和 `beta` 三个环境. 可将环境特有的配置分别写在 `application-prod.yml`, `application-dev.yml` 和 `application-beta.yml` 中.

然后在主配置文件(通用)中通过 `spring.profiles.active : ${env}` 来切换当前环境, 达到环境隔离的效果.

#### 拆分

如果一个 profile 中配置过多, 想让文件更加简洁, 或按功能拆分, 比如配置了 redis, mybatis, 可以单独创建这两个的配置文件: `application-devRedis.yml`, `application-devMybatis.yml`. 然后在主配置文件中通过 `spring.profiles.include: devRedis,devMybatis` 来引入这两个配置.

### springBoot 序列化时忽略 null 字段.

添加注解的方式无法生效, 需要在 `application.yml` 中通过 `spring.jackson.default-property-inclusion: non_null` 来指定.

### Swagger @ApiImplicParam 和 @ApiParam 的使用

`@ApiImplicParam` 用于描述基本数据类型.

`@ApiParam` 用于描述实体类, 如 pojo 等. 否则在生成的接口文档中会把本来只是作为参数的实体类的变量名也作为一个接口的参数.

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

