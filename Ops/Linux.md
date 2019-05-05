---
title: Linux学习笔记
date: 2019/03/04 15:24
categories:
- Linux
tags:
- Linux
---

# Linux学习笔记

## 系统命令

#### tar 解压

解压到指定目录:`tar -zxvf 要解压的文件 -C 目的路径`

> tar 命令参数
>
> -x或--extract或--get:从备份文件中还原文件
>
> -Z或--compress或--uncompress:通过compress指令处理备份文件
>
> -f<备份文件>或--file=<备份文件>:指定备份文件
>
> -v:显示操作过程
>
> -C <目录>:这个选项用在解压缩,若要在特定目录解压缩,可以使用这个选项

#### 防火墙(使用`firewalld`)

开启关闭重启:`service firewalld start/stop/restart`

查看运行状态:`service firewalld status`/`sudo firewall-cmd --state`

查看防火墙开放的端口:`sudo firewall-cmd --list-ports`

删除端口:`firewall-cmd --zone=分区名 --remove-port=端口号/通讯协议 --permanent`

为防火墙添加端口:`sudo firewall-cmd --zone=分区名 --add-port=端口号/通讯协议 --permanent(永久生效)`

> - 防火墙分为多个区域,可用`sudo firewall-cmd --get-zones`查看,默认是`public`

#### 软件安装

查看软件是否安装: `rom -qa | grep 软件名>`

下载安装软件: `yum install 软件名1,2,3...`/`yum -y install 软件名1,2,3...`

安装`rpm`软件包: `rpm -ivh 软件包名`

卸载软件: `rpm -e 软件包名`/`yum remove 软件包名`

> rpm 命令参数
>
> -i: 安装过程中显示正在安装的文件信息
>
> -h: 安装过程中显示安装进度

#### 服务进程

查看服务进程: `ps -ef | grep <进程名>`

#### 修改权限

为文件添加可执行权限: `chmod +x file.name`,可以用通配符`*.后缀名`

#### 查看文件

查看文件最后n行: `tail -n <行数> <文件名>`

动态查看文件最后n行: `tail -n <行数> -f <文件名>`,使用`ctrl+c`来结束

#### 创建文件

创建指定大小文件: `fallocate -l <文件大小> <文件名>`

> 例:`sudo fallocate -l 4G /swapfile`(在`/`下创建一个大小为4G的文件`swapfile`)

#### 查看系统状态

查看内存使用情况: `free`

> free 命令参数
>
> `-m`:以`MB`为单位显示.默认单位为`KB`.向下取整

#### 用户操作

创建用户: `useradd 用户名`或`adduser 用户名`.后者创建的用户会自动创建主目录,系统shell版本和创建时输入密码,而前者没有.

修改用户密码: `passwd 用户名`

删除用户: `userdel 用户名`

#### 下载

下载: `wget 下载地址`

下载并指定名称: `wget -O 文件名称 下载地址`

## vim

- 搜索

在 `normal` 模式下按下 `/` 即可进入查找模式, 输入要查找的字符串并按下回车. Vim 会跳转到第一个匹配. 按下 `n `查找下一个, 按下 `N `查找上一个.

> - 注意查找回车应当用 `\n`,替换回车应当用 `\r`
>
> - vim默认采用大小写敏感.在查找模式中加入 `\c` 表示大小写不敏感 ,`\C `敏感.