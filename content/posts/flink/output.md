---
title: "Flink 如何分流数据"
date: 2020-06-11T17:16:05+08:00
description: Flink 如何分流数据,3种分流
categories: ["Flink"]
tags: ["Flink"]
featured_image: "https://bigdata-lijun.oss-cn-hangzhou.aliyuncs.com/uploads%2F2020%2F06%2F11%2FilU71EZb_%E7%A8%BF%E5%AE%9A%E8%AE%BE%E8%AE%A1%E5%AF%BC%E5%87%BA-20200611-173454.png?Expires=1591868304"
author: "ipoo"
KeyName : "Flink,DataStream,Flink,max"
---


<!-- MarkdownTOC -->

- [场景](#%E5%9C%BA%E6%99%AF)
- [分流方式](#%E5%88%86%E6%B5%81%E6%96%B9%E5%BC%8F)
- [如何分流](#%E5%A6%82%E4%BD%95%E5%88%86%E6%B5%81)
	- [使用Filter分流](#%E4%BD%BF%E7%94%A8filter%E5%88%86%E6%B5%81)
	- [使用Split分流](#%E4%BD%BF%E7%94%A8split%E5%88%86%E6%B5%81)
	- [使用Side Output分流](#%E4%BD%BF%E7%94%A8side-output%E5%88%86%E6%B5%81)

<!-- /MarkdownTOC -->


## 场景

获取流数据的时候,通常需要根据所需把流拆分出其他多个流,根据不同的流再去作相应的处理。

举个例子:创建一个商品实时流,商品有季节标签,需要对不同标签的商品做统计处理,这个时候就需要把商品数据流根据季节标签分流。

## 分流方式

- 使用Filter分流
- 使用Split分流
- 使用Side Output分流

## 如何分流

先模拟一个实时的数据流

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

输出:

![image](https://bigdata-lijun.oss-cn-hangzhou.aliyuncs.com/uploads%2F2020%2F06%2F11%2FuET80ByA_strem.gif?Expires=1591864826)

### 使用Filter分流
使用 filter 算子根据数据的字段进行过滤。

```java
import common.Product;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import source.ProductStremingSource;

public class OutputStremingDemo {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        DataStreamSource<Product> source = env.addSource(new ProductStremingSource());

        // 使用Filter分流
        SingleOutputStreamOperator<Product> spring = source.filter(product -> "spring".equals(product.getSeasonType()));
        SingleOutputStreamOperator<Product> summer = source.filter(product -> "summer".equals(product.getSeasonType()));
        SingleOutputStreamOperator<Product> autumn  = source.filter(product -> "autumn".equals(product.getSeasonType()));
        SingleOutputStreamOperator<Product> winter  = source.filter(product -> "winter".equals(product.getSeasonType()));
        source.print();
        winter.printToErr();

        env.execute("output");
    }
}
```

结果输出(红色为季节标签是winter的分流输出):

![image](https://bigdata-lijun.oss-cn-hangzhou.aliyuncs.com/uploads%2F2020%2F06%2F11%2F4UDV6kEW_productStrem.gif?Expires=1591864980)

### 使用Split分流

重写OutputSelector内部类的select()方法,根据数据所需要分流的类型反正不同的标签下,返回SplitStream,通过SplitStream的select()方法去选择相应的数据流。

只分流一次是没有问题的，但是不能使用它来做连续的分流。

**SplitStream已经标记过时了**

```java
public class OutputStremingDemo {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        DataStreamSource<Product> source = env.addSource(new ProductStremingSource());

        // 使用Split分流
        SplitStream<Product> dataSelect = source.split(new OutputSelector<Product>() {
            @Override
            public Iterable<String> select(Product product) {
                List<String> seasonTypes = new ArrayList<>();
                String seasonType = product.getSeasonType();
                switch (seasonType){
                    case "spring":
                        seasonTypes.add(seasonType);
                        break;
                    case "summer":
                        seasonTypes.add(seasonType);
                        break;
                    case "autumn":
                        seasonTypes.add(seasonType);
                        break;
                    case "winter":
                        seasonTypes.add(seasonType);
                        break;
                    default:
                        break;
                }
                return seasonTypes;
            }
        });
        DataStream<Product> spring = dataSelect.select("machine");
        DataStream<Product> summer = dataSelect.select("docker");
        DataStream<Product> autumn = dataSelect.select("application");
        DataStream<Product> winter = dataSelect.select("middleware");
        source.print();
        winter.printToErr();

        env.execute("output");
    }
}

```

### 使用Side Output分流
推荐使用这种方式

首先需要定义一个OutputTag用于标识不同流

可以使用下面的几种函数处理流发送到分流中:

- ProcessFunction
- KeyedProcessFunction
- CoProcessFunction
- KeyedCoProcessFunction
- ProcessWindowFunction
- ProcessAllWindowFunction

之后再用getSideOutput(OutputTag)选择流。
```java
public class OutputStremingDemo {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        DataStreamSource<Product> source = env.addSource(new ProductStremingSource());

        // 使用Side Output分流
        final OutputTag<Product> spring = new OutputTag<Product>("spring");
        final OutputTag<Product> summer = new OutputTag<Product>("summer");
        final OutputTag<Product> autumn = new OutputTag<Product>("autumn");
        final OutputTag<Product> winter = new OutputTag<Product>("winter");
        SingleOutputStreamOperator<Product> sideOutputData = source.process(new ProcessFunction<Product, Product>() {
            @Override
            public void processElement(Product product, Context ctx, Collector<Product> out) throws Exception {
                String seasonType = product.getSeasonType();
                switch (seasonType){
                    case "spring":
                        ctx.output(spring,product);
                        break;
                    case "summer":
                        ctx.output(summer,product);
                        break;
                    case "autumn":
                        ctx.output(autumn,product);
                        break;
                    case "winter":
                        ctx.output(winter,product);
                        break;
                    default:
                        out.collect(product);
                }
            }
        });

        DataStream<Product> springStream = sideOutputData.getSideOutput(spring);
        DataStream<Product> summerStream = sideOutputData.getSideOutput(summer);
        DataStream<Product> autumnStream = sideOutputData.getSideOutput(autumn);
        DataStream<Product> winterStream = sideOutputData.getSideOutput(winter);

        // 输出标签为:winter 的数据流
        winterStream.print();

        env.execute("output");
    }
}
```

结果输出:

![image](https://bigdata-lijun.oss-cn-hangzhou.aliyuncs.com/uploads%2F2020%2F06%2F11%2F5QeRqaJ0_outstrem.gif?Expires=1591866836)


<!-- 

扫码关注公众号《ipoo》
![ipoo](http://oss.ipooli.com/images/%E5%85%AC%E4%BC%97%E5%8F%B7code.jpg) -->