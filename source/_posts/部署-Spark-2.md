---
title: '[部署]Spark-2 Hadoop 2.x部署'
date: 2024-04-30 11:09:16
tags: 
- 部署 
- Hadoop
- SSH
categories: 
- 部署
---




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