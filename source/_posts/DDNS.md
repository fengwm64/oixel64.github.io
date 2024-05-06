---
layout: '[部署]老毛子系统路由器配置dnspod'
title: DDNS
date: 2024-05-06 20:56:03
tags:
---

# 1.前言

家里宽带有公网，不能浪费了，做了一个家庭服务器。

但是公网ip是动态的，每48小时变动一次，故采用ddns方式将ip绑定到域名上，然后就可以动态更新。

之前是使用路由器的ddns功能的，但是不知道为什么老是不能自动更新，所以直接自己利用一个Github上的的一个shell脚本来实现定时更新。

# 2.开启ssh服务

路由器web管理界面

`高级设置` - `系统管理` - `服务` - `终端服务` - `启用 SSH 服务` - `是`

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202405062110413.png)

然后使用ssh工具连接，用户名`admin`、密码应该是登录web管理界面时的密码。

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202405062111423.png)



# 3.下载脚本

GitHub项目地址：https://github.com/rehiy/dnspod-shell

直接下载到本地，文本方式打开`ardnspod`文件，追加内容如下：

```sh
arToken=yourDnsPodToken 
arDdnsCheck yourDomain subDomain
```

上传到路由器，记住文件位置

然后在ssh终端执行：

- `cd yourFilePath`

- 给予运行权限`chmod +x ./ardnspod`

- 手动执行看看有没有报错`./ardnspod`

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202405062114696.png)

# 4.部署脚本

一切安好，在路由器web管理界面：

1. `高级设置` - `自定义设置` - `脚本` - `自定义 Crontab 定时任务配置`

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202405062114233.png)

最下面添加Crontab定时命令：`*/10 * * * * /home/root/ardnspod` 

2. `高级设置` - `系统管理` - `服务` - `其他服务` - `计划任务(Crontab)`

添加Crontab定时命令：`*/10 * * * * /home/root/ardnspod` 

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202405062118285.png)

> 理论上应该是设置一个地方即可，但是我发现按1设置后在系统执行`crontab -l`时定时任务为空，所以我又设置了2以保险。
> 
> emmm，我觉得可能1是老毛子系统的定时任务，2是linux自带的定时任务。了解不深，暂时这样了。