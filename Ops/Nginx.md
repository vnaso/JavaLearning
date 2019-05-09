---
title: Nginx学习笔记
date: 2019/03/04 15:24
categories:
- Tool
- Nginx
tags:
- Nginx
---
标记
# Nginx使用

## Nginx配置 

> 在`···/nginx/conf`目录下的`nginx.conf`

```nginx.conf
# 全局块
...              
# events块
events {         
   ...
}
# http块
http      
{
    # http全局块
    ...   
    # 虚拟主机server块
    server        
    { 
        # server全局块
        ...       
        # location块
        location [PATTERN]   
        {
            ...
        }
        location [PATTERN] 
        {
            ...
        }
    }
    server
    {
      ...
    }
    # http全局块
    ...     
}
```

### 全局块

> 配置影响nginx全局的指令.一般有运行nginx服务器的用户组,nginx进程pid存放路径,日志存放路径,配置文件引入,允许生成worker process数等.

```
########### 每个指令必须有分号结束。#################
#user administrator administrators;  #配置用户或者组，默认为nobody nobody。
#worker_processes 2;  #允许生成的进程数，默认为1
#pid /nginx/pid/nginx.pid;   #指定nginx进程运行文件存放地址
error_log log/error.log debug;  #制定日志路径，级别。这个设置可以放入全局块，http块，server块，级别以此为：debug|info|notice|warn|error|crit|alert|emerg
```

### events块

> 配置影响nginx服务器或与用户的网络连接.有每个进程的最大连接数,选取哪种事件驱动模型处理连接请求,是否允许同时接受多个网路连接,开启多个网络连接序列化等.

```
events {
    accept_mutex on;   #设置网路连接序列化，防止惊群现象发生，默认为on
    multi_accept on;  #设置一个进程是否同时接受多个网络连接，默认为off
    #use epoll;      #事件驱动模型，select|poll|kqueue|epoll|resig|/dev/poll|eventport
    worker_connections  1024;    #每个worker process最大连接数
}
```

### http块

> 可以嵌套多个server,配置代理,缓存,日志定义等绝大多数功能和第三方模块的配置.如文件引入,mime-type定义,日志自定义,是否使用sendfile传输文件,连接超时时间,单连接请求数等.

```
http {
    include       mime.types;   #文件扩展名与文件类型映射表
    default_type  application/octet-stream; #默认文件类型，默认为text/plain
    #access_log off; #取消服务日志    
    log_format myFormat '$remote_addr–$remote_user [$time_local] $request $status $body_bytes_sent $http_referer $http_user_agent $http_x_forwarded_for'; #自定义格式
    access_log log/access.log myFormat;  #combined为日志格式的默认值
    sendfile on;   #允许sendfile方式传输文件，默认为off，可以在http块，server块，location块。
    sendfile_max_chunk 100k;  #每个进程每次调用传输数量不能大于设定的值，默认为0，即不设上限。
    keepalive_timeout 65;  #连接超时时间，默认为75s，可以在http，server，location块。
    
    # 定义常量
    # 负载均衡,按权重或规则将请求转发到某个服务器
    upstream mysvr {   
      server 127.0.0.1:7878;
      server 192.168.10.121:3333 backup;  #热备
    }
    error_page 404 https://www.baidu.com; #请求失败时跳转的错误页 
```

### server块

> 配置虚拟主机的相关参数,一个http中可以有多个server.
>
> 并且可以过滤有人恶意将某些域名指向自己的主机服务器.

```
    #定义某个负载均衡服务器   
    server {
        keepalive_requests 120; #单连接请求上限次数。
        listen       4545;   #监听端口
        server_name  127.0.0.1;   #监听地址       
    }
```

- `server_name`

  `server_name`是监听你的HTTP请求头中的`host`.

  > 默认情况下，Nginx 允许直接以 IP 的方式就能直接访问到网站，或者通过未设置的域名访问(比如有人把他自己的域名指向了你的服务器 IP),可通过如下设置进行防护
  >
  > 例如:
  >
  > ```
  > server {
  > 	listen 80 default_server;
  > 	server_name _;
  > 	return 444; # 过滤其他域名的请求，返回444状态码
  > }
  > server {
  > 	listen 80;
  > 	server_name www.aaa.com; # www.aaa.com域名
  > 	location / {
  > 		proxy_pass http://localhost:8080; # 对应端口号8080
  > 	}
  > }
  > server {
  > 	listen 80;
  > 	server_name www.bbb.com; # www.bbb.com域名
  > 	location / {
  > 		proxy_pass http://localhost:8081; # 对应端口号8081
  > 	}
  > }
  > ```
  >
  > `server_name`的值为`www.aaa.com`.在浏览器中输入`www.aaa.com`,那么匹配到了对应`server_name`,请求就会被转发到`http://localhost:8080`.这里`www.aaa.com`和`www.bbb.com`都绑定到了服务器
  >
  > 而对于未绑定的域名指向服务器时,匹配不到配置的虚拟主机域名,就会使用默认的虚拟主机,然会返回444.
  >
  > `listen 80 default_server`:指定该`server`配置段为`80`端口的默认主机，即对于未绑定的域名指向你的服务器时,匹配不到你配置的虚拟主机域名后,会默认使用这个虚拟主机
  >
  > `server_name _`:此处的`_`可以换成任意其他无效字符或无效的域名,表示该`server`配置不会被正常访问到

- `access_log`:日志

  格式:`access_log logs/aaa.access.log main`-`日志类型 日志存放路径 日志格式`


### location块

[详细参考-知否](https://segmentfault.com/a/1190000013267839)

[详细参考-本地](./知否-nginx.md)

[nginx跨域代理的一些设置](https://www.jianshu.com/p/cc5167032525)

> 配置请求的路由,以及各种页面的处理情况

```
        location  ~*^.+$ {       #请求的url过滤,正则匹配
           #root path;  #根目录
           #index vv.txt;  #设置默认页
           proxy_pass  http://mysvr;  #请求转向mysvr 定义的服务器列表
           deny 127.0.0.1;  #拒绝的ip
           allow 172.18.5.54; #允许的ip           
        } 
```

- 通配符
  - `=`表示精确匹配.只有请求的url路径与后面的字符串完全相等时,才会命中
  - `~ `表示该规则是使用正则定义的,区分大小写
  - `~*`表示该规则是使用正则定义的,不区分大小写
  - `^~`表示如果该符号后面的字符是最佳匹配,采用该规则,不再进行后续的查找

- 属性解释

  `root`表示`url`匹配上了此`location`定义的正则后,就将这个`url`映射到定义的根目录.

  > 例如 `location /user/img`,`root /usr/local/resource;`.
  >
  > 那么如果请求为`http://www.aaa.com/user/img/aaa.jpg`,`/user/img`对应`/usr/local/resource`.该请求得到的结果就是`/usr/local/resource`下的`aaa.jpg`

  `index`就是设置默认的欢迎页面.页面需要存在于定义的`root`目录下.

  > 接上例`index index.html;`.
  >
  > 那么如果请求为`http://www.aaa.com/user/img/`,请求的结果就是`/usr/local/resource/user/img`下的`index.html`,有点类似于相对路径.而如果要让`index.html`为`/usr/local/resource/`路径下的`index.html`的话,需要把`root`换为`alias`,具体参照`root`和`alias`的区别.

### 注意

#### unknown directive

所有关键词后都必须加一个空格,否则会报错`unknown directive xxx`

## Linux下命令

> 在`···/nginx/sbin/`目录下

- 启动nginx:`nginx -s start`

- 停止nginx:`nginx -s stop`
- 重启nginx:`nginx -s reload`
- 测试配置:`nginx -t`

## Nginx日志目录

> 在`···/nginx/logs/`目录下

目录下通常存放有`access.log`,`error.log`以及`nginx.pid`.

`nginx.pid`中保存着`nginx`的进程号,可以通过查看此文件获取进程号来kill nginx.

## root和alias的区别

#### root 示例1

```
location / {
	root /data/www/;
	index index.html;
}
```

请求`http://example.com`这个地址,那么在服务器中真实对应的地址为`/data/www/index.html`

|            请求            |        真实地址        |
| :------------------------: | :--------------------: |
|    `http://example.com`    | `/data/www/index.html` |
| `http://example.com/a.png` |   `/data/www/a.png`    |

#### root 示例2

```
location /aaa/ {
	root /data/www/;
	index index.html;
}
```

|              请求              |         真实地址          |
| :----------------------------: | :-----------------------: |
|   `http://example.com/aaa/`    | `/data/www/aaa/indexhtml` |
| `http://example.com/aaa/a.gif` |   `/data/www/aaa/a.gif`   |

#### alias 示例1

```
location /{
    alias /data/www/;
    index index.html;
}
```

|            请求            |        真实地址        |
| :------------------------: | :--------------------: |
|    `http://example.com`    | `/data/www/index.html` |
| `http://example.com/b.jpg` |   `/data/www/a.jpg`    |

#### alias 示例2

```
location /bbb/{
    alias /data/www/;
    index index.html;
}
```

|              请求              |        真实地址        |
| :----------------------------: | :--------------------: |
|   `http://example.com/bbb/`    | `/data/www/index.html` |
| `http://example.com/bbb/b.jpg` |   `/data/www/b.jpg`    |

对比上面4个示例可以发现,使用`root`的真实地址是`root`+`location`+(`文件名`),而使用`alias`的真实地址是`alias`+(`文件名`).

> **注意**
>
> - `alias`只能作用在`location`中,而`root`可以存在`server`,`http`,`location`.
>
> - `alias`后面必须要用`/`结束.

## Nginx实现解决前后端分离的跨域问题

[参考](https://segmentfault.com/a/1190000011145364#articleHeader2)

> 在前端使用`ajax`向从后端获取数据发送跨域请求,由于浏览器的**同源策略限制**,会报403错误.

### 同源策略

> 同源是指"协议+域名+端口"三者相同,即便两个不同的域名指向同一个ip地址,也非同源

#### 同源策略限制行为

- Cookie,LocalStorage和IndexDB无法读取
- DOM和JS对象无法获得
- Ajax请求不能发送

#### 常见的跨域场景

| URL                                                          |          说明           |             是否允许通信             |
| :----------------------------------------------------------- | :---------------------: | :----------------------------------: |
| `http://www.domain.com/a.js`<br>`http://www.domain.com/b.js`<br>`http://www.domain.com/lab/c.js` | 同一域名,不同文件或路径 |                 允许                 |
| `http://www.domain.com:8000/a.js`<br> `http://www.domain.com/b.js ` |    同一域名,不同端口    |                不允许                |
| `http://www.domain.com/a.js`<br>`https://www.domain.com/b.js` |    同一域名,不同协议    |                不允许                |
| `http://www.domain.com/a.js`<br>`http://192.168.4.12/b.js `  |  域名和域名对应相同ip   |                不允许                |
| `http://www.domain.com/a.js`<br>`http://x.domain.com/b.js`<br>`http://domain.com/c.js` |    主域相同,子域不同    | 不允许(cookie这种情况下也不允许访问) |
| `http://www.domain1.com/a.js`<br>`http://www.domain2.com/b.js` |        不同域名         |                不允许                |

### 跨域解决方案

#### Nginx代理跨域

> 一种比较简单的解决方案,无需动后端代码

##### Nginx配置

```
    server {
        listen 80;
        server_name vnaso.live;
        location / {
                if ($http_origin ~* (http://vnaso\.live$) ) {
                        add_header Access-Control-Allow-Origin  '$http_origin';
                        add_header Access-Control-Allow-Methods 'GET, POST, OPTIONS';
                        add_header Access-Control-Allow-Headers 'DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization';
                        add_header 'Access-Control-Allow-Credentials' 'true';
                }
                if ($http_origin ~* (http://127.0.0.1:8080$) ) {
                        add_header Access-Control-Allow-Origin  '$http_origin';
                        add_header Access-Control-Allow-Methods 'GET, POST, OPTIONS';
                        add_header Access-Control-Allow-Headers 'DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization';
                        add_header 'Access-Control-Allow-Credentials' 'true';
                }

                proxy_pass               http://127.0.0.1:8080/;
                #proxy_cookie_path       / /;
                proxy_set_header         Host  $host;
                proxy_set_header         X-Real-IP  $remote_addr;
                proxy_set_header         X-Forwarded-For  $proxy_add_x_forwarded_for;
                proxy_set_header         Cookie $http_cookie;
                root /opt/Develop/apache-tomcat-9.0.16/webapps/ROOT/;
        #       index index.html;
        }
    }

```

- `if`语句来决定匹配跨域请求,并为这些请求添加`RequestHeader`,上面配置的`add_header`
- `Access-Control-Allow-Origin`:服务器默认是不允许跨域的,这里可以添加接受跨域的`请求源(origin)`.
- `Access-Control-Allow-Methods`:设置允许接收哪些方法的跨域请求
- `Access-Control-Allow-Headers`:不添加此属性会报错:`Request header field Content-Type is not allowed by Access-Control-Allow-Headers in preflight response.`原因是当前的`Content-Type`不被支持.详情看下方的`预检请求`.
- `proxy_pass`:表示将匹配的请求转发到该`url`下
- `proxy_set_header`:为转发的请求添加`RequestHeader`

[相关知识CORS](../Web/CORS跨域资源共享)

