---
title: '[部署]Spark部署'
date: 2024-04-28 19:48:24
tags: 
- 部署 
- Spark
categories: 
- 部署
---

# 1.环境准备

在哥们(hhh)的帮助下，搞到四台阿里云服务器，不过挺寒碜的配置：

| 编号 | IP             | 用户名 | 密码         | 系统配置         | 备注 |
| ---- | -------------- | ------ | ------------ | ------------ | ---- |
| 1    | 8.219.108.46 | root   | qwe157258. | Ubuntu 22.04 \| 2vCPU/4GiB| 冯 主节点  |
| 2    | 47.236.20.161  | root   | qwe157258.   | Ubuntu 22.04 \| 2vCPU/1GiB | 庄 计算节点  |
| 3    | 47.236.157.142 | root   | qwe157258. | Ubuntu 22.04 \| 2vCPU/2GiB| 冯  计算节点 |
| 4    | 47.236.115.157 | root   | qwe157258. | Ubuntu 22.04 \| 2vCPU/1GiB| 冯 客户机  |

写了一个shell脚本，在四台服务器上创建用户`dase-dis`（注意确保四台服务器的用户名和密码一致才可以使用）:

先`sudo apt install sshpass`，在Linux下执行脚本：

```sh
#!/bin/bash

# 服务器IP地址列表
ip_list=("xxxx" "xxxx" "xxxx" "xxxx")

# 设置统一的密码
password="admin"
ssh_password="qwe157258."

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

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202404282346549.png)

-------------------------------------------

```sh
#!/bin/bash

# 服务器IP地址列表
ip_list=("server1_ip" "server2_ip" "server3_ip" "server4_ip")

# 循环遍历IP地址列表
for ip in "${ip_list[@]}"
do
    echo "Configuring SSH key for $ip..."

    # 生成SSH密钥对（如果不存在）
    if [ ! -f ~/.ssh/id_rsa ]; then
        ssh-keygen -t rsa -N "" -f ~/.ssh/id_rsa
    fi

    # 将公钥复制到目标服务器上
    ssh-copy-id -i ~/.ssh/id_rsa.pub root@$ip

    # 嵌套循环，将该公钥发送到除自己外的每个服务器
    for target_ip in "${ip_list[@]}"
    do
        if [ "$ip" != "$target_ip" ]; then
            echo "Copying SSH key to $target_ip..."
            ssh-copy-id -i ~/.ssh/id_rsa.pub root@$target_ip
        fi
    done

    echo "SSH key configured for $ip."
done

echo "SSH key configured for all servers successfully."
```