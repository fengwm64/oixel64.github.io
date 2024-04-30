---
title: '[部署]Spark部署'
date: 2024-04-28 19:48:24
tags: 
- 部署 
- Spark
categories: 
- 部署
---

# 1.互联互通

在哥们(hhh)的帮助下，搞到四台阿里云服务器，不过挺寒碜的配置：

| 编号 | IP             | 用户名 | 密码         | 系统配置         | 备注 |
| ---- | -------------- | ------ | ------------ | ------------ | ---- |
| 1    | 8.219.108.46 | root   | qwe157258. | Ubuntu 22.04 \| 2vCPU/4GiB| 冯 主节点  |
| 2    | 47.236.20.161  | root   | qwe157258.   | Ubuntu 22.04 \| 2vCPU/1GiB | 庄 从节点  |
| 3    | 47.236.157.142 | root   | qwe157258. | Ubuntu 22.04 \| 2vCPU/2GiB| 冯  从节点 |
| 4    | 47.236.115.157 | root   | qwe157258. | Ubuntu 22.04 \| 2vCPU/1GiB| 冯 客户端  |

## 1.1 创建用户

写了一个shell脚本，在四台服务器上创建用户`dase-dis`（注意确保四台服务器的用户名和密码一致才可以使用）:

先`sudo apt install sshpass`，在Linux下执行脚本：

```sh
#!/bin/bash

# 服务器IP地址列表
ip_list=("xxxx" "xxxx" "xxxx" "xxxx")

# 设置统一的密码
password="admin"

# 循环遍历IP地址列表
for ip in "${ip_list[@]}"
do
    echo "Connecting to $ip..."

    # 连接服务器并创建用户
    sshpass -p "$ssh_password" ssh -o StrictHostKeyChecking=no root@$ip << EOF
        # 创建用户并设置密码
        useradd -m -s /bin/bash dase-dis
        echo "dase-dis:$password" | chpasswd
        # 添加用户到sudo组
        usermod -aG sudo dase-dis
EOF

    echo "User dase-dis created on $ip with password $password"
done
```

执行结果：

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202404282346549.png)

-------------------------------------------

## 1.2 服务器之间免密登录
实现四台服务器之间ssh免密登录


### 1.2.1 安装openssh

在四台服务器上执行

- `sudo apt-get install openssh-server` 安装openssh


### 1.2.2 更改主机名

在1号机（主节点，在文章开头编号1）执行：
- `sudo hostnamectl set-hostname ecnu01` 更改主机名

在2号机（从节点，在文章开头编号2）执行：
- `sudo hostnamectl set-hostname ecnu02`

...以此类推
- `sudo hostnamectl set-hostname ecnu03`

- `sudo hostnamectl set-hostname ecnu04`

四台服务器都执行完毕后，断开ssh重新连接，观察到主机名字已经成功更改

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202404292322849.png)


### 1.2.3 更改hosts

原理：

> `Hosts` 文件是本地计算机上的文本文件，用于将主机名与 `IP` 地址关联起来,绕过 `DNS` 解析。`Linux hosts` 文件的格式通常是：
> 
> `IP地址    主机名    [别名...]`
> 
> 在 `/etc/hosts` 路径下，每行代表一个主机名到 `IP` 地址的映射。例如：
> ```hosts
> 127.0.0.1   localhost
> ::1         localhost
> 192.168.1.2 example.com
> ```
> 其中，`127.0.0.1` 和 `::1` 映射到 `localhost`，`192.168.1.2` 映射到 `example.com`。`hosts` 文件允许手动指定主机名与 `IP` 地址的对应关系，用于特定网络配置和测试。

**开始修改：**

在四台机上执行以下操作：

- `sudo vim /etc/hosts`

在`hosts`文件后追加（**ip需要改成自己的哇**）：

```sh
# IP地址 主机名
8.219.108.46 ecnu01
47.236.20.161 ecnu02
47.236.157.142 ecnu03
47.236.115.157 ecnu04
```

### 1.2.4 拷贝ssh公钥

**除主机外的三台机依次执行下面命令：**

作用是将除主机外的三台机的ssh公钥拷贝到主中，实现其余三台机器到主机的ssh免密登录

- `ssh-keygen -t rsa` 生成ssh密钥

- `ssh dase-dis@ecnu01 'mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys' < ~/.ssh/id_rsa.pub'` 发送公钥到主机

- `sudo service ssh restart && chmod 700 ~/.ssh  && chmod 600 ~/.ssh/authorized_keys` 重启本机ssh服务+解决ssh文件夹的权限问题

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202404292330151.png)

---------------------

**主机执行：**

作用是将主机的ssh公钥拷贝到其余三台机中，实现主机到其余三台机器的ssh免密登录

- `scp ~/.ssh/authorized_keys dase-dis@ecnu02:/home/dase-dis/.ssh/authorized_keys`

- `scp ~/.ssh/authorized_keys dase-dis@ecnu03:/home/dase-dis/.ssh/authorized_keys`

- `scp ~/.ssh/authorized_keys dase-dis@ecnu04:/home/dase-dis/.ssh/authorized_keys`

上面的三条命令等价于命令：`for host in ecnu02 ecnu03 ecnu04; do scp ~/.ssh/authorized_keys dase-dis@$host:/home/dase-dis/.ssh/; done`

然后主机执行：

`sudo service ssh restart && chmod 700 ~/.ssh  && chmod 600 ~/.ssh/authorized_keys`

运行结果：

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202404292335551.png)

-------------------------

**验证：**

互相ssh过去看看要不要输入密码
```shell
ssh dase-dis@ecnu01
exit
ssh dase-dis@ecnu02
exit
ssh dase-dis@ecnu03
exit
ssh dase-dis@ecnu04
exit
```

## 1.3 配置Java环境


## 2.4 修改.bashrc文件

登录**客户机**，用户`dase-dis`：

- `vi ~/.bashrc`

- 按`i`进入编辑模式，按方向键到文件最后一行，输入`export TERM=xterm-color`

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202404292240144.png)

- 按`Esc`键退出编辑模式，输入`:wq`保存退出

- 使`.bashrc`配置生效：`source ~/.bashrc`


## 1.4 下载并安装Spark

在四台机上下载并安装Spark:

- 下载安装包：`wget http://archive.apache.org/dist/spark/spark-2.4.7/spark-2.4.7-bin-without-hadoop.tgz`

- 解压安装包：`tar -zxvf spark-2.4.7-bin-without-hadoop.tgz`

- 改名：`mv ~/spark-2.4.7-bin-without-hadoop ~/spark-2.4.7`

下载：
![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202404292248585.png)

上述步骤完成后：
![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202404292257700.png)


## 1.5 修改配置

- `cp /home/dase-dis/spark-2.4.7/conf/spark-env.sh.template /home/dase-dis/spark-2.4.7/conf/spark-env.sh`

- `vim /home/dase-dis/spark-2.4.7/conf/spark-env.sh`

```shell
# 因为下载的是Hadoop Free版本的Spark, 所以需要配置Hadoop的路径
HADOOP_HOME=/home/dase-dis/hadoop-2.10.1
export SPARK_DIST_CLASSPATH=$($HADOOP_HOME/bin/hadoop classpath)
export LD_LIBRARY_PATH=$HADOOP_HOME/lib/native:$LD_LIBRARY_PATH
```


- sudo apt-get install openssh-server
- sudo hostnamectl set-hostname ecnu01
- sudo hostnamectl set-hostname ecnu02
- sudo hostnamectl set-hostname ecnu03
- sudo hostnamectl set-hostname ecnu04

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202404292322849.png)


-----------

sudo vim /etc/hosts

```sh
# IP地址 主机名

8.219.108.46 ecnu01
47.236.20.161 ecnu02
47.236.157.142 ecnu03
47.236.115.157 ecnu04
```

除主机外的三台机：

- ssh-keygen -t rsa
- ssh dase-dis@ecnu01 'mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys' < ~/.ssh/id_rsa.pub'
- sudo service ssh restart && chmod 700 ~/.ssh  && chmod 600 ~/.ssh/authorized_keys

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202404292330151.png)

主机：

scp ~/.ssh/authorized_keys dase-dis@ecnu02:/home/dase-dis/.ssh/authorized_keys
scp ~/.ssh/authorized_keys dase-dis@ecnu03:/home/dase-dis/.ssh/authorized_keys
scp ~/.ssh/authorized_keys dase-dis@ecnu04:/home/dase-dis/.ssh/authorized_keys

= for host in ecnu02 ecnu03 ecnu04; do scp ~/.ssh/authorized_keys dase-dis@$host:/home/dase-dis/.ssh/; done

sudo service ssh restart && chmod 700 ~/.ssh  && chmod 600 ~/.ssh/authorized_keys


![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202404292335551.png)


验证：互相ssh过去看看要不要输入密码
ssh dase-dis@ecnu01
exit
ssh dase-dis@ecnu02
exit
ssh dase-dis@ecnu03
exit
ssh dase-dis@ecnu04
exit