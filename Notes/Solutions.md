---
title: 工具踩坑记录
date: 2019/06/08 23:33
categories:
- Note
tags:
- Tools
---

## Cmder

### Cmder 进入 Windows Linux Subsystem 后, 进入 vim 界面方向键无法使用.

在 Startup -> Tasks -> bash::ubuntu -> 启动参数中添加 `%windir%\system32\bash.exe ~ -cur_console:p5`.

## Win10

### 电脑开机后, 亮度自动变为 50.

(任务管理器) -> 服务 -> 显示增强服务(DisplayEnhancementService), 把该服务禁用即可.


