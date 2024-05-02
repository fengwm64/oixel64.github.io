---
title: '[大数据]Spark-3 Spark部署'
date: 2024-05-02 09:16:36
tags:
- 部署 
- Spark
- 大数据
categories: 
- 大数据
- 部署
---

# 1.修改配置文件

## 1.1 修改.bashrc文件

**客户端**执行：

- `vi ~/.bashrc`

- 按`i`进入编辑模式，按方向键到文件最后一行，输入`export TERM=xterm-color`

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202404292240144.png)

- 按`Esc`键退出编辑模式，输入`:wq`保存退出

- 使`.bashrc`配置生效：`source ~/.bashrc`


## 1.2 下载 spark

在**主节点**执行:

- 启动HDFS服务(已经启动直接下一步):`~/hadoop-2.10.1/sbin/start-dfs.sh`

- 下载Spark安装包：`wget http://archive.apache.org/dist/spark/spark-2.4.7/spark-2.4.7-bin-without-hadoop.tgz`

- 解压安装包：`tar -zxvf spark-2.4.7-bin-without-hadoop.tgz`

- 改名：`mv ~/spark-2.4.7-bin-without-hadoop ~/spark-2.4.7`

上述步骤完成后：
![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202404292257700.png)


## 1.3 修改配置

在**主节点**执行以下修改:

### 1.3.1 spark-env

- `cp /home/dase-dis/spark-2.4.7/conf/spark-env.sh.template /home/dase-dis/spark-2.4.7/conf/spark-env.sh`

- `vi /home/dase-dis/spark-2.4.7/conf/spark-env.sh`

修改为:

```shell
# 因为下载的是Hadoop Free版本的Spark, 所以需要配置Hadoop的路径
export HADOOP_HOME=/home/dase-dis/hadoop-2.10.1
export SPARK_DIST_CLASSPATH=$($HADOOP_HOME/bin/hadoop classpath)
export LD_LIBRARY_PATH=$HADOOP_HOME/lib/native:$LD_LIBRARY_PATH

export SPARK_MASTER_HOST=ecnu01 #主节点主机名
export SPARK_MASTERPORT=7077    #端口号
```

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202405020927048.png)

### 1.3.2 slaves

- `cp spark-2.4.7/conf/slaves.template spark-2.4.7/conf/slaves`

- `vi spark-2.4.7/conf/slaves`

修改为:

```shell
# localhost
ecnu02
ecnu03
```

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202405020930661.png)

### 1.3.3 spark-defaults

- `cp spark-2.4.7/conf/spark-defaults.conf.template spark-2.4.7/conf/spark-defaults.conf`

- `vi spark-2.4.7/conf/spark-defaults.conf`

修改为:

```shell
spark.eventLog.enabled=true
spark.eventLog.dir=hdfs://ecnu01:9000/tmp/spark_history
spark.history.fs.logDirectory=hdfs://ecnu01:9000/tmp/spark_history
```

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202405020941828.png)

### 1.3.4 spark-config

- `vi spark-2.4.7/sbin/spark-config.sh`

追加:

```
export JAVA_HOME=/home/dase-dis/jdk1.8.0_202
```

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202405020945282.png)

# 2.安装spark

## 2.1 拷贝

本步骤将spark修改好的安装包拷贝到其他三台机:

- `scp -r spark-2.4.7 dase-dis@ecnu02:~/`

- `scp -r spark-2.4.7 dase-dis@ecnu03:~/`

- `scp -r spark-2.4.7 dase-dis@ecnu04:~/`

## 2.2 HDFS中建立日志目录

- `~/hadoop-2.10.1/bin/hdfs dfs -mkdir -p /tmp/spark_history`

## 2.3 启动 spark

千辛万苦, 终于到启动了
<img src="https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202405021003091.png" alt="" width="25%">

在**主节点**执行:

- 启动spark: `~/spark-2.4.7/sbin/start-all.sh`

- 启动日志服务器: `~/spark-2.4.7/sbin/start-history-server.sh`

- **主节点:**

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202405021043271.png)

- **从节点:**

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202405021047418.png)


**错误处理:**

```
dase-dis@ecnu01:~$ ~/spark-2.4.7/sbin/start-all.sh
starting org.apache.spark.deploy.master.Master, logging to /home/dase-dis/spark-2.4.7/logs/spark-dase-dis-org.apache.spark.deploy.master.Master-1-ecnu01.out
ecnu02: starting org.apache.spark.deploy.worker.Worker, logging to /home/dase-dis/spark-2.4.7/logs/spark-dase-dis-org.apache.spark.deploy.worker.Worker-1-ecnu02.out
ecnu03: starting org.apache.spark.deploy.worker.Worker, logging to /home/dase-dis/spark-2.4.7/logs/spark-dase-dis-org.apache.spark.deploy.worker.Worker-1-ecnu03.out
ecnu02: failed to launch: nice -n 0 /home/dase-dis/spark-2.4.7/bin/spark-class org.apache.spark.deploy.worker.Worker --webui-port 8081 spark://172.19.39.254:7077
ecnu02:         at io.netty.channel.AbstractChannel.bind(AbstractChannel.java:248)
ecnu02:         at io.netty.bootstrap.AbstractBootstrap$2.run(AbstractBootstrap.java:356)
ecnu02:         at io.netty.util.concurrent.AbstractEventExecutor.safeExecute(AbstractEventExecutor.java:164)
ecnu02:         at io.netty.util.concurrent.SingleThreadEventExecutor.runAllTasks(SingleThreadEventExecutor.java:472)
ecnu02:         at io.netty.channel.nio.NioEventLoop.run(NioEventLoop.java:500)
ecnu02:         at io.netty.util.concurrent.SingleThreadEventExecutor$4.run(SingleThreadEventExecutor.java:989)
ecnu02:         at io.netty.util.internal.ThreadExecutorMap$2.run(ThreadExecutorMap.java:74)
ecnu02:         at io.netty.util.concurrent.FastThreadLocalRunnable.run(FastThreadLocalRunnable.java:30)
ecnu02:         at java.lang.Thread.run(Thread.java:748)
ecnu02:   24/05/02 10:23:10 INFO util.ShutdownHookManager: Shutdown hook called
ecnu02: full log in /home/dase-dis/spark-2.4.7/logs/spark-dase-dis-org.apache.spark.deploy.worker.Worker-1-ecnu02.out
```

**请检查hosts文件设置, 文章`[大数据]Spark-1 SSH & JDK部署`**

## 2.4 验证

浏览器访问: `http://主节点IP:8080/`，（**如果主节点是云服务器记得把防火墙打开**）

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202405021116322.png)

可以看到有两个worker在线, **大功告成**

# 3.运行spark应用程序

好不容易搞好了, 来玩一下: 

## 3.1 创建文件夹&上传文件

- 创建`spark_input`文件夹: `~/hadoop-2.10.1/bin/hdfs dfs -mkdir -p spark_input`

- 上传文件`RELEASE`到`spark_input`: `~/hadoop-2.10.1/bin/hdfs dfs -put ~/spark-2.4.7/RELEASE spark_input/`

在`hadood`的页面可以看到, 文件`RELEASE`存储在两个节点中: 

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202405021152426.png)

## 3.2 启动 Spark Shell

- 启动`spark-shell`: `~/spark-2.4.7/bin/spark-shell --master spark://ecnu01:7077`

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202405021309487.png)

- 键入以下`Scala`代码:

```scala
val sc = spark.sparkContext
val textFile = sc.textFile("hdfs://ecnu01:9000/user/dase-dis/spark_input/RELEASE")
val counts = textFile.flatMap(line => line.split(" ")).map(word => (word, 1)).reduceByKey(_ + _)
counts.collect().foreach(println)
```

shell输出:

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202405021317280.png)


可以在网页查看到正在运行的任务信息:

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202405021310029.png)

到此Spark集群就已经顺利搭建完毕了

# 4.停止集群

如果你希望停止集群:

- 停止`Spark`: `/spark-2.4.7/sbin/stop-all.sh`

- 停止`Spark`日志服务: `/spark-2.4.7/sbin/stop-history-server.sh`

- 停止`HDFS`服务: `/hadoop-2.10.1/sbin/stop-dfs.sh`