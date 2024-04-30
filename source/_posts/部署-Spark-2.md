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

请确保已经完成1

# 1.下载

地址：https://archive.apache.org/dist/hadoop/common/hadoop-2.10.1/hadoop-2.10.1.tar.gz

**在主节点上执行：**

- `wget https://archive.apache.org/dist/hadoop/common/hadoop-2.10.1/hadoop-2.10.1.tar.gz`

- `tar -zxvf hadoop-2.10.1.tar.gz`

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202404301317432.png)

- `cd ~/hadoop-2.10.1/`

- `./bin/hadoop version`

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202404301319210.png)

# 2.修改配置

**在主节点上执行：**

- 修改 slaves 文件: `vim ~/hadoop-2.10.1/etc/hadoop/slaves`

修改为：
```
ecnu02
ecnu03
```

---------------

- 修改 core-site.xml: `vim ~/hadoop-2.10.1/etc/hadoop/core-site.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!--
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->

<!-- Put site-specific property overrides in this file. -->

<configuration>
  <property>
    <name>hadoop.tmp.dir</name>
    <value>/home/dase-dis/hadoop-2.10.1/tmp</value>
  </property>
  <!--以下填写主节点主机名-->
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://ecnu01:9999</value>
  </property>
</configuration>
```

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202404301329297.png)

**使用云服务器的！！！！注意** ecnu01需要改成内网IP，详细看后文

---------------

- 修改 hdfs-site.xml: `vim ~/hadoop-2.10.1/etc/hadoop/hdfs-site.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!--
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->

<!-- Put site-specific property overrides in this file. -->

<configuration>
  <property>
    <name>dfs.replication</name>
    <value>2</value>
  </property>
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>file:/home/dase-dis/hadoop-2.10.1/tmp/dfs/name</value>
  </property>
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>file:/home/dase-dis/hadoop-2.10.1/tmp/dfs/name</value>
  </property>
</configuration>
```

--------------

- 修改 hadoop-env.sh: `vim ~/hadoop-2.10.1/etc/hadoop/hadoop-env.sh`
  
- 将`JAVA_HOME`改为：

```xml
export JAVA_HOME=/home/dase-dis/jdk1.8.0_202
```

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202404301434373.png)

----------------------

好了好了，终于改完了，接下来将改好的这份hadoop拷贝到其余三台机：

- 拷贝到从节点1：`scp -r /home/dase-dis/hadoop-2.10.1 dase-dis@ecnu02:/home/dase-dis/`

- 拷贝到从节点2：`scp -r /home/dase-dis/hadoop-2.10.1 dase-dis@ecnu03:/home/dase-dis/`

- 拷贝到客户端：`scp -r /home/dase-dis/hadoop-2.10.1 dase-dis@ecnu04:/home/dase-dis/`

其实打包一下拷贝会更加好的，这里偷懒了

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202404301352765.png)

# 3.启动HDFS服务

## 3.1 格式化

> **注意:** 仅在第一次启动 HDFS 时才需要格式化 NameNode，如果是重启HDFS那么跳过这步，直接执行下一步即可。
> 此外，在进行 NameNode 格式化之前，如果~/hadoop-2.10.1/tmp/文件夹已存在，那么需要删除该文件夹后再执行以下格式化命令。
>
> **如果启动时炸了，CTRL+C了，断电了，请参考后文解决办法，可能仍然需要格式化**

- 格式化命令: `~/hadoop-2.10.1/bin/hdfs namenode -format`

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202404301427369.png)

----------------

## 3.2 启动

- 启动：`~/hadoop-2.10.1/sbin/start-dfs.sh`

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202404301505902.png)

---------------

## 3.3 验证

- 验证：`jps`

- **主节点**

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202404301620529.png)

- **从节点**

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202404301509565.png)

----------------------

浏览器访问`http://主节点IP:50070/`，（**如果主节点是云服务器记得把防火墙打开**）

开防火墙：

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202404301515606.png)

集群工作正常：

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202404301622499.png)
----------------------------

## 3.4 集群异常解决

1. 如果因为一些情况导致集群第一次没有启动成功，请在四台机器：

- 停止集群：`~/hadoop-2.10.1/sbin/stop-dfs.sh`

- 删除运行生成文件：`cd ~/hadoop-2.10.1/tmp/dfs && rm -rf *`

- 删除日志：`cd ~/hadoop-2.10.1/logs && rm -rf *`

- 解决端口占用：`sudo reboot`

- 重新执行格式化命令：`~/hadoop-2.10.1/bin/hdfs namenode -format`

--------------------

2. **使用云服务器的请注意：**

- 错误日志：

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202404301613498.png)

- 提示绑定错误或`2024-04-30 16:06:52,547 INFO org.apache.hadoop.util.ExitUtil: Exiting with status 1: java.net.BindException: Problem binding to [ecnu01:9000] java.net.BindException: Cannot assign requested address; For more details see:  http://wiki.apache.org/hadoop/BindException`的，请将`core-site.xml`文件中的`encu01`改为**内网IP**。

- 获取内网IP：

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202404301614410.png)

- 参考：

1. https://blog.csdn.net/xiaosa5211234554321/article/details/119627974

2. https://cwiki.apache.org/confluence/display/HADOOP2/BindException






---------------------------

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