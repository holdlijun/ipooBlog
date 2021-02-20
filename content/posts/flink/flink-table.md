---
title: "Flink Table Api & SQL 初体验,Blink的使用"
date: 2020-06-20T11:18:11+08:00
description: 介绍Flink Table Api & SQL和实现了两表连接的示例
categories: ["Flink"]
tags: ["Flink"]
featured_image: "https://bigdata-lijun.oss-cn-hangzhou.aliyuncs.com/uploads%2F2020%2F06%2F20%2F91PYjFhF_%E7%A8%BF%E5%AE%9A%E8%AE%BE%E8%AE%A1%E5%AF%BC%E5%87%BA-20200620-112750.png?Expires=1592623720"
author: "ipoo"
KeyName : "Flink,Flink Table,Flink SQL"
---

<!-- MarkdownTOC -->

- [概述](#%E6%A6%82%E8%BF%B0)
- [使用](#%E4%BD%BF%E7%94%A8)
    - [pom依赖](#pom%E4%BE%9D%E8%B5%96)
    - [模拟一个实时流](#%E6%A8%A1%E6%8B%9F%E4%B8%80%E4%B8%AA%E5%AE%9E%E6%97%B6%E6%B5%81)
    - [主程序](#%E4%B8%BB%E7%A8%8B%E5%BA%8F)
    - [结果打印](#%E7%BB%93%E6%9E%9C%E6%89%93%E5%8D%B0)
- [总结](#%E6%80%BB%E7%BB%93)

<!-- /MarkdownTOC -->



# 概述
Flink具有Table API和SQL-用于统一流和批处理。

Table API是用于Scala和Java的语言集成查询API，它允许以非常直观的方式组合来自关系运算符（例如选择，过滤和联接）的查询。

Flink的SQL支持基于实现SQL标准的Apache Calcite。无论输入是批处理输入（DataSet）还是流输入（DataStream），在两个接口中指定的查询都具有相同的语义并指定相同的结果。

Table API和SQL尚未完成所有功能，正在积极开发中，支持程度需查看 [官方文档](https://ci.apache.org/projects/flink/flink-docs-master/dev/table/#table-api-sql)

# 使用
多表连接案例
## pom依赖

flink 版本为:1.9.3

```java

    <dependencies>
        <!-- Apache Flink dependencies -->
        <!-- These dependencies are provided, because they should not be packaged into the JAR file. -->
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-java</artifactId>
            <version>${flink.version}</version>
            <scope>provided</scope>
        </dependency>

        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-streaming-java_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-table-api-java-bridge_2.11</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-table-planner-blink_2.11</artifactId>
            <version>${flink.version}</version>
        </dependency>

        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-table-api-java</artifactId>
            <version>${flink.version}</version>
        </dependency>
```

## 模拟一个实时流
```java
import lombok.Data;
@Data
public class Product {
    public Integer id;
    public String seasonType;
}

```
自定义Source
```java
import common.Product;
import org.apache.flink.streaming.api.functions.source.SourceFunction;

import java.util.ArrayList;
import java.util.Random;

public class ProductStremingSource implements SourceFunction<Product> {
    private boolean isRunning = true;

    @Override
    public void run(SourceContext<Product> ctx) throws Exception {
        while (isRunning){
            // 每一秒钟产生一条数据
            Product product = generateProduct();
            ctx.collect(product);
            Thread.sleep(1000);
        }
    }

    private Product generateProduct(){
        int i = new Random().nextInt(100);
        ArrayList<String> list = new ArrayList();
        list.add("spring");
        list.add("summer");
        list.add("autumn");
        list.add("winter");
        Product product = new Product();
        product.setSeasonType(list.get(new Random().nextInt(4)));
        product.setId(i);
        return product;
    }
    @Override
    public void cancel() {

    }
}
```

## 主程序

```java
public class TableStremingDemo {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment bsEnv = StreamExecutionEnvironment.getExecutionEnvironment();
        // 使用Blink
        EnvironmentSettings bsSettings = EnvironmentSettings.newInstance().useBlinkPlanner().inStreamingMode().build();
        StreamTableEnvironment bsTableEnv = StreamTableEnvironment.create(bsEnv, bsSettings);

        SingleOutputStreamOperator<Item> source = bsEnv.addSource(new MyStremingSource())
                .map(new MapFunction<Item, Item>() {
                    @Override
                    public Item map(Item value) throws Exception {
                        return value;
                    }
                });
        // 分割流
        final OutputTag<Item> even = new OutputTag<Item>("even") {
        };
        final OutputTag<Item> old = new OutputTag<Item>("old") {
        };

        SingleOutputStreamOperator<Item> sideOutputData = source.process(new ProcessFunction<Item, Item>() {
            @Override
            public void processElement(Item value, Context ctx, Collector<Item> out) throws Exception {
                if (value.getId() % 2 == 0) {
                    ctx.output(even,value);
                }else{
                    ctx.output(old,value);
                }
            }
        });

        DataStream<Item> evenStream = sideOutputData.getSideOutput(even);
        DataStream<Item> oldStream = sideOutputData.getSideOutput(old);
        // 注册两个 表 : evenTable,oddTable
        bsTableEnv.registerDataStream("evenTable",evenStream , "name,id");
        bsTableEnv.registerDataStream("oddTable", oldStream, "name,id");

        // 执行sql 输出Table
        Table queryTable = bsTableEnv.sqlQuery("select a.id,a.name,b.id,b.name from evenTable as a join oddTable as b on a.name = b.name");
        queryTable.printSchema();;
        // 获取流
        DataStream<Tuple2<Boolean, Tuple4<Integer, String, Integer, String>>> dataStream = bsTableEnv.toRetractStream(queryTable, TypeInformation.of(new TypeHint<Tuple4<Integer,String,Integer,String>>(){}));
        dataStream.print();

        bsEnv.execute("demo");
    }
}
```
## 结果打印
![](https://bigdata-lijun.oss-cn-hangzhou.aliyuncs.com/uploads%2F2020%2F06%2F20%2F37vPgQWk_table.gif?Expires=1592622921)
输出name相同的元素。

# 总结

简单的介绍了Flink Table Api & SQL和实现了两表连接的示例。
<!-- 

扫码关注公众号《ipoo》
![ipoo](http://oss.ipooli.com/images/%E5%85%AC%E4%BC%97%E5%8F%B7code.jpg) -->