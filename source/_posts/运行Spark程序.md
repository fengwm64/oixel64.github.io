---
title: '[大数据]Spark-4 运行Spark程序'
date: 2024-05-02 13:23:34
tags: 
- Spark
- 大数据
categories: 
- 大数据
---

# 1.经典的 WordCount 程序

程序源码如下:

```java
package cn.edu.ecnu.spark.example.java.wordcount;

import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaPairRDD;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.function.*;
import scala.Tuple2;

import java.util.Arrays;
import java.util.Iterator;

public class WordCount {
    public static void run(String[] args) {
        /* 步骤1：通过SparkConf设置配置信息，并创建SparkContext */
        SparkConf conf = new SparkConf();
        conf.setAppName("WordCount");
        JavaSparkContext sc = new JavaSparkContext(conf);

        /* 步骤2：按应用逻辑使用操作算子编写DAG，其中包括RDD的创建、转换和行动等 */
        // 读入文本数据，创建名为lines的RDD
        JavaRDD<String> lines = sc.textFile(args[0]);

        // 将lines中的每一个文本行按空格分割成单个单词
        JavaRDD<String> words = lines.flatMap(new FlatMapFunction<String, String>() {
            @Override
            public Iterator<String> call(String line) throws Exception {
                return Arrays.asList(line.split(" ")).iterator();
            }
        });
        // 将每个单词的频数设置为1，即将每个单词映射为[单词, 1]
        JavaPairRDD<String, Integer> pairs = words.mapToPair(new PairFunction<String, String, Integer>() {
            @Override
            public Tuple2<String, Integer> call(String word) throws Exception {
                return new Tuple2<String, Integer>(word, 1);
            }
        });
        // 按单词聚合，并对相同单词的频数使用sum进行累计
        JavaPairRDD<String, Integer> wordCounts = pairs.groupByKey().mapToPair(new PairFunction<Tuple2<String, Iterable<Integer>>, String, Integer>() {
            @Override
            public Tuple2<String, Integer> call(Tuple2<String, Iterable<Integer>> t) throws Exception {
                Integer sum = Integer.valueOf(0);
                for (Integer i : t._2) {
                    sum += i;
                }
                return new Tuple2<String, Integer>(t._1, sum);
            }
        });
        // 合并机制
        /*
        JavaPairRDD<String, Integer> wordCounts =
        pairs.reduceByKey(
            new Function2<Integer, Integer, Integer>() {
              @Override
              public Integer call(Integer t1, Integer t2) throws Exception {
                return t1 + t2;
              }
            });
         */

        // 输出词频统计结果
        wordCounts.saveAsTextFile(args[1]);

        /* 步骤3：关闭SparkContext */
        sc.stop();
    }

    public static void main(String[] args) {
        run(args);
    }
}
```

----------------

## 1.1 新建maven项目

- 在`idea`新建项目：

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202405021832587.png)

- `pom.xml`内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>spark-wordcount</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <spark.version>1.2.0</spark.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-core_2.11</artifactId>
            <version>2.4.7</version>
        </dependency>
    </dependencies>

</project>
```

- 更新依赖

## 1.2 新建 Java 代码

- 新建包`cn.edu.ecnu.spark.example.java.wordcount`，类`WordCount`: 

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202405021833109.png)

## 1.3 打包

- 打包为`.jar`:

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202405021759128.png)

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202405021800838.png)

## 1.4 传送到客户端

- 将打包好的`.jar`(位置`项目路径\out\artifacts\spark_wordcount_jar`)传到**客户端**的`/home/dase-dis/spark-2.4.7/myapp`

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202405021803426.png)

## 1.5 下载测试数据

- 下载：`wget https://github.com/ymcui/Chinese-Cloze-RC/archive/master.zip`

- 解压：`unzip master.zip`

- 解压：`unzip ~/Chinese-Cloze-RC-master/people_daily/pd.zip`

- 拷贝到集群：`~/hadoop-2.10.1/bin/hdfs dfs -put ~/Chinese-Cloze-RC-master/people_daily/pd/pd.test spark_input/pd.test`

## 1.6 提交jar任务

- 删除输出文件夹：`~/hadoop-2.10.1/bin/hdfs dfs -rm -r spark_output`

- 提交任务：`~/spark-2.4.7/bin/spark-submit \
--master spark://ecnu01:7077 \
--class cn.edu.ecnu.spark.example.java.wordcount.WordCount \
/home/dase-dis/spark-2.4.7/myapp/spark-wordcount.jar hdfs://ecnu01:9000/user/dase-dis/spark_input hdfs://ecnu01:9000/user/dase-dis/spark_output`

正在运行

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202405021816280.png)

**顺利跑完**

ssh：

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202405021831459.png)

webui：

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202405021830147.png)

查看`output`文件夹:

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202405021931447.png)

查看`part01`运行结果：

- `./hadoop-2.10.1/bin/hdfs dfs -cat /user/dase-dis/spark_output/part-00001`

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202405021934024.png)