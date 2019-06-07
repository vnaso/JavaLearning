---
title: Linux线上环境搭建
date: 2019/03/05 14:11
categories:
- Linux
tags:
- Linux
---

服务器环境搭建

## Java

#### 安装

1. 到[官网](https://www.oracle.com/technetwork/java/javase/downloads/index.html)下载 `jdk`.

   > 注意要使用带 Auth 的下载链接下载

2. 解压到 `tar -zxvf jdk_8u_201 -C /usr/java`

3. 编辑环境变量 `vim /etc/profile`, 使环境变量生效 `source /etc/profile`

   ```
   export JAVA_HOME=/usr/java/jdk...
   export JRE_HOME=/usr/java/jdk.../jre
   export CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
   export PATH=$PATH:$GIT_HOME/bin:$MAVEN_HOME/bin:$JAVA_HOME/bin:$JRE_HOME/bin:
   ```

4. 检查是否安装成功: `java -version`

## Mysql

#### 安装

1. 到[官网Yum仓库](https://dev.mysql.com/downloads/repo/yum/)下载 `rpm` 包.
2. `rpm -ivh <文件名> `或者 `yum localinstall <文件名> `安装刚才下载好的包
3. 此时便可以通过 `yum `安装最新的 `mysql`. 使用指令 `yum install mysql-community-server `安装

#### 配置

- 启动 mysql: `service mysqld start`
- 关闭 mysql: `service mysqld stop`

1. 查看 mysql 提供的初始 root 密码: `cat /var/log/mysqld.log | grep password`

   ![mysql_pwd.png](https://i.loli.net/2019/03/03/5c7b6d28c8abf.png)

2. 登录 mysql, 若出现提示 `Access denied for user 'root'@'localhost'`, 可能是密码错了.

3. 修改 root 密码: `ALTER USER 'root'@'localhost' IDENTIFIED BY 'vnaso943983';`

   > 如果不想使用强密码,需要先降低密码策略等级:
   >
   > `SET GLOBAL validate_password.policy=0;`

4. 创建用户 `CREATE USER 'vnaso'@'%' IDENTIFIED BY 'vnaso222';`, 并授权: `GRANT ALL ON *.* TO 'vnaso'@'%' WITH GRANT OPTION;`

5. 修改默认字符集. 打开 `/etc/my.cnf`, 添加如下配置:

   ```conf
   # 对本地的mysql客户端的配置
   [client]
   default-character-set = utf8mb4
   
   # 对其他远程连接的mysql客户端的配置
   [mysql]
   default-character-set = utf8mb4
   
   # 本地mysql服务的配置
   [mysqld]
   character-set-client-handshake = FALSE
   character-set-server = utf8mb4
   collation-server = utf8mb4_unicode_ci
   default-time-zone = UTC
   ```

   查看字符集: `SHOW VARIABLES WHERE Variable_name LIKE 'character\_set\_%' OR Variable_name LIKE 'collation%'; `

6. 查看时区: `show variables like '%zone%';`

   ![mysql_timezone.png](https://i.loli.net/2019/03/04/5c7d0e96800b4.png)

7. 修改时区为 UTC: `set time_zone = 'utc';flush privileges;`修改时区并刷新.

   - 如果报错 `Unknown or incorrect time zone: 'UTC'`, 在`shell`执行:  ` mysql_tzinfo_to_sql /usr/share/zoneinfo |mysql -u root mysql -p`, 有warning提示 `Unable to load xxx` 是正常的.

   - 如果修改了之后, 再次查询还是 `System`, 修改配置文件 `/etc/my.cnf`, 在 `[mysqld]` 下添加 `default-time-zone = UTC`

## Git

#### 安装

1. 到[官网](https://github.com/git/git/releases)获取下载链接. `wget <url>` 下载

2. 使用 `tar -zxvf <文件名>` 解压缩.

3. 安装 `git` 前置依赖: 

   ```bash
   yum -y install zlib-devel openssl-devel cpio expat-devel gettext-devel curl-devel perl-ExtUtils-CBuilder perl-ExtUtils-MakeMaker
   ```

4. 进入 `git `安装目录下, 进行安装: `./configure --prefix=/usr/local/git`-> `make && make install`

   如果报错 `no such file or directory: ./configure`. 使用: `yum install autoconf`然后在目录中键入 `autoconf`, 再次进行安装即可.

5. 添加环境变量. `vim /etc/profile`, 使环境变量生效`source /etc/profile`

6. 检查是否安装成功: `git --version`

#### 配置

1. 生成私钥: `ssh-keygen -t rsa -C "XXX@outlook.com"` 密钥名称可自定义

2. 告知系统来管理生成的密钥: `ssh-add ~/.ssh/id_rsa`

   >  如果报错 `could not open a connection to your authentication agent`
   >
   > 执行命令: eval \`ssh-agent\`(是 `~`, 而不是单引号). 然后再执行:`ssh-add ~/.ssh/id_rsa`

3. 在远程服务器中添加公钥: `cat ~/.ssh/id_rsa.pub`

## Nginx

#### 安装

1. 到[官网](https://nginx.org/en/download.html)下载压缩包并解压.
2. 安装 `nginx `编译文件及库文件 `yum -y install make zlib zlib-devel gcc-c++ libtool  openssl openssl-devel pcre pcre-devel`
3. 进入解压后的目录进行安装: `./configure --prefix=/usr/local/nginx`->`make && make install`
4. 测试: `/usr/local/nginx/sbin/nginx -t`

#### 配置

参考[Nginx配置文件说明](./Nginx.md)

## Tomcat

#### 安装

1. 到[官网](https://tomcat.apache.org/download-90.cgi)下载压缩包, 解压. 一般选择 `core` 就行
2. 防火墙打开 `8080` 端口, 启动 tomcat: `$CATALINA_HOME/bin/startup.sh`
3. 测试: 访问 `<服务器地址>:8080`

#### 配置

1. 修改默认编码. `vim /opt/Develop/apache-tomcat-9.0.16/conf/server.xml`. 搜索`8080`, 在 `<Connector> `节点中添加 `URIEncoding="UTF-8"`

   ![tomcatEncoding.png](https://i.loli.net/2019/03/03/5c7b93f19dc78.png)

2. 编辑环境变量.`vim /etc/profile`,添加`export CATALINA_HOME = /opt/apache-tomcat-9.0.1`.然后`source /etc/profile`使配置生效

## Maven

#### 安装

1. 到[官网](https://maven.apache.org/download.cgi)下载压缩包,解压.
2. 添加环境变量:`vim /etc/profile`,添加`export $MAVEN_HOME=...`.并将`$MAVEN_HOME/bin`添加到`$PATH`中
3. `source /etc/profile`使配置生效
4. 测试:`mvn -v`

#### 配置

> 将下载镜像更换为阿里云中央仓库,解决依赖从境外网站下载过慢的问题

1. 打开`maven安装根目录`->`conf`->`settings.xml`

2. 在`<settings>`标签下找到`<mirrors>`标签,添加如下代码

   ```xml
   <mirror>
       <id>nexus-aliyun</id>
       <mirrorOf>central</mirrorOf>
       <name>Nexus aliyun</name>
       <url>http://maven.aliyun.com/nexus/content/groups/public</url>
   </mirror>
   ```

3. 保存即可

## Vstpd

#### 安装

1. 下载安装:`yum install vstpd`

2. 创建接收上传文件的文件夹:`mkdir /ftpfile`

3. 创建只有上传权限不能登录的用户:`useradd ftpuser -d /ftpfile/ -s /sbin/nologin`

4. 给创建的`ftpuser`此用户赋予`/ftpfile`的权限:`chown -R ftpuser.ftpuser /ftpfile/`.此时用户名和用户组都是`ftpuser`

   ![ftpuser.png](https://i.loli.net/2019/03/03/5c7b807a8ab63.png)

5. 给`ftpuser`设置密码:`passwd ftpuser`

6. 编辑配置文件,让上传目录指向之前创建的目录:`vim /etc/vsftpd/vsftpd.conf`

   > 搜索`banner`节点,此处配置`ftp`的欢迎信息.`ftpd_banner=xxxxx`
   >
   > ![ftp_banner.png](https://i.loli.net/2019/03/03/5c7b8283e0357.png)
   >
   > 可以选择添加`use_localtime=yes`表示使用服务器时间
   >
   > ![ftp_root.png](https://i.loli.net/2019/03/03/5c7b83014f41b.png)
   >
   > 添加`anonymous_enable=NO`关闭匿名用户访问
   >
   > 配置FTP被动模式的端口
   >
   > `pasv_min_port=30000`
   > `pasv_max_port=30000`

7. ftp配置

   ```
   # 修改为NO，关闭匿名用户访问
   anonymous_enable=NO
   # 将所有本地用户限制在自家目录中。
   chroot_local_user=YES 
   # 设置系统用户FTP主目录
   local_root=/data
   # 开启charoot写权限
   allow_writeable_chroot=YES
   #配置可以登录ftp的用户目录
   userlist_deny=NO
   userlist_file=/etc/vsftpd/user_list
   #配置ftp用户访问目录配置目录
   user_config_dir=/etc/vsftpd/userconfig
   # 配置FTP被动模式的端口
   pasv_min_port=30000
   pasv_max_port=30000
   ```

8. 配置`ftp`用户登录后访问的目录

   > 在`/etc/vsftpd`目录下新建`userconfig`目录
   >
   > 在目录下配置用户的登录目录,文件名即对应的用户名
   >
   > `vim /etc/vsftpd/userconfig/ftpuser`
   >
   > 在创建的文件中添加`local_root=/ftpfile/ftpuser`  `/ftpfile`表示对应用户登录ftp时的根目录
   >
   > **注意**:路径前不能有空格,否则不识别!!!!会报错`unrecognised variable in config file: local_root `

9. 重启`vsftpd`

   > 启动:`systemctl start vsftpd.service(service vsftpd start)`
   >
   > 停止:`systemctl stop vsftpd.service(service vsftpd stop)`
   >
   > 重启:`systemctl restart vsftpd.service(service vsftpd restart)`

10. 打开防火墙,添加`21`端口和`30000`端口.

    > 添加端口:`firewall-cmd --add-port=21/tcp --permanent`,`firewall-cmd --add-port=21/tcp --permanent`
    >
    > 重启防火墙:`firewall-cmd --reload`

11. 测试.浏览器打开 `ftp://服务器地址` 登录.如果只能下载不能上传,请查阅**开启关闭SELinux**相关信息

## Redis

1. [官网](<https://redis.io/>)下载压缩包, 并解压.
2. 进入解压后的 redis 根目录.
3. 安装 redis:
   1. 编译: `make`
   2. 指定安装目录: `make PREFIX=/usr/local/redis install`
4. 测试:
   1. 进入 `/usr/local/redis/bin` 运行 `./redis-server`
   2. 另起一个窗口, 进入 `/usr/local/redis/bin` 运行 `./redis-cli`

## zsh

1. 使用 yum 安装: `sudo yum update && sudo yum -y install zsh`
2. 检测是否安装成功: `zsh --version`
3. 将 shell 切换至 zsh: `chsh -s $(which zsh)`
4. 重新登录, 检查 shell 是否切换: `echo $SHELL`

### 配置

1. 从[官网](<https://ohmyz.sh/>)下载插件

   ```bash
   git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
   git clone git://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
   ```

   插件需要放在 `${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/` 目录下, 否则每次重启都需要重新安装插件.

2. 安装插件

   1. 进入到下载的插件的根目录下, 分别 `source <插件名.zsh>`.

   2. 进入 zsh 安装目录(默认为 `/root/` 下)

   3. `vim .zshrc` 编辑 zsh 配置文件:

      ```.zshrc
      ZSH_THEME="agnoster"
      plugins=(
      		git
      		zsh-autosuggestions
      		zsh-syntax-highlighting
      		...# 插件名, 需要先下载并安装插件
      )
      ```

   4. 为 autosuggestions 的自动提示绑定快捷采纳键, 默认为 <kbd>→</kbd>. 替换为 <kbd>Ctrl + Space</kbd>.

      `bindkey '^ ' autosuggest-accept`.

      可以通过 `cat > /dev/null` 来查看组合键的转换序列.

## swap交换缓存

#### 创建

1. 创建`swap`文件:`sudo fallocate -l 4G /swapfile`(在`/`下创建一个大小为4G的文件`swapfile`)
2. 授权`swap`文件:`chmod 600 /swapfile`(该文件的读写只能`root`操作)
3. 告知系统将文件用于`swap`:`mkswap /swapfile`
4. 启用`swap`文件:`swapon /swapfile`
5. 确认:`free`

> 至此已经在系统中启用了`swap`交换缓存,但是一旦系统重启后,服务器还不能自动启用该文件.

#### 使swap永久生效

1. 打开文件`nano /etc/fstab`,在文件末尾添加`/swapfile   swap    swap    sw  0   0`
2. `^X`表示`Ctrl + X`,按下选择`yes`保存退出.

## 环境变量配置

在腾讯云服务器中的配置

![profile.png](https://i.loli.net/2019/03/03/5c7b96f1831a8.png)

```
export CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
export PATH=$PATH:$GIT_HOME/bin:$MAVEN_HOME/bin:$JAVA_HOME/bin:$JRE_HOME/bin:
export GIT_HOME=/usr/local/git
export MAVEN_HOME=/opt/Develop/apache-maven-3.6.0
export JAVA_HOME=/usr/java/jdk1.8.0_201
export JRE_HOME=/usr/java/jdk1.8.0_201/jre
export CATALINA_HOME=/opt/Develop/apache-tomcat-9.0.16
```

