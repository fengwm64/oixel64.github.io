---
title: [大数据] Spark-4 运行Spark程序
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
package cn.edu.ecnu.spark.examples.java.wordcount;

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
    conf.setMaster("local"); // 仅用于本地进行调试，如在集群中运行则删除本行
    JavaSparkContext sc = new JavaSparkContext(conf);

    /* 步骤2：按应用逻辑使用操作算子编写DAG，其中包括RDD的创建、转换和行动等 */
    // 读入文本数据，创建名为lines的RDD
    JavaRDD<String> lines = sc.textFile("src/main/resources/input/wordcount/words.txt");

    // 将lines中的每一个文本行按空格分割成单个单词
    JavaRDD<String> words =
        lines.flatMap(
            new FlatMapFunction<String, String>() {
              @Override
              public Iterator<String> call(String line) throws Exception {
                return Arrays.asList(line.split(" ")).iterator();
              }
            });
    // 将每个单词的频数设置为1，即将每个单词映射为[单词, 1]
    JavaPairRDD<String, Integer> pairs =
        words.mapToPair(
            new PairFunction<String, String, Integer>() {
              @Override
              public Tuple2<String, Integer> call(String word) throws Exception {
                return new Tuple2<String, Integer>(word, 1);
              }
            });
    // 按单词聚合，并对相同单词的频数使用sum进行累计
    JavaPairRDD<String, Integer> wordCounts =
        pairs
            .groupByKey()
            .mapToPair(
                new PairFunction<Tuple2<String, Iterable<Integer>>, String, Integer>() {
                  @Override
                  public Tuple2<String, Integer> call(Tuple2<String, Iterable<Integer>> t)
                      throws Exception {
                    Integer sum = Integer.valueOf(0);
                    for (Integer i : t._2) {
                      sum += i;
                    }
                    return new Tuple2<String, Integer>(t._1, sum);
                  }
                });
    // 合并机制
    /*JavaPairRDD<String, Integer> wordCounts =
    pairs.reduceByKey(
        new Function2<Integer, Integer, Integer>() {
          @Override
          public Integer call(Integer t1, Integer t2) throws Exception {
            return t1 + t2;
          }
        });*/

    // 输出词频统计结果
    wordCounts.foreach(t -> System.out.println(t._1 + " " + t._2));

    /* 步骤3：关闭SparkContext */
    sc.stop();
  }

  public static void main(String[] args) {
    run(args);
  }
}
```

----------------

## 1.1 打包 WordCount

本步将`WordCount`源码打包为`.jar`



![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202405021759128.png)

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202405021800838.png)

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202405021659339.png)

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202405021803426.png)

- `wget https://github.com/ymcui/Chinese-Cloze-RC/archive/master.zip`

- `unzip master.zip`

- `unzip ~/Chinese-Cloze-RC-master/people_daily/pd.zip`

- `~/hadoop-2.10.1/bin/hdfs dfs -put ~/Chinese-Cloze-RC-master/people_daily/pd/pd.test spark_input/pd.test`

- `~/hadoop-2.10.1/bin/hdfs dfs -rm -r spark_output`

- `~/spark-2.4.7/bin/spark-submit \
--master spark://ecnu01:7077 \
--class cn.edu.ecnu.spark.example.java.wordcount.WordCount \
/home/dase-dis/spark-2.4.7/myapp/spark-wordcount.jar hdfs://ecnu01:9000/user/dase-dis/spark_input hdfs://ecnu01:9000/user/dase-dis/spark_output`

![](https://cdn.jsdelivr.net/gh/oixel64/imgs/imgs/202405021816280.png)
