---
title: "Flink DataStream中min(),minBy(),max(),max()之间的区别"
date: 2020-06-10T11:31:25+08:00
description: Flink DataStream中min(),minBy(),max(),max()之间的区别,举例论证
categories: ["Flink"]
tags: ["Flink"]
featured_image:
author: "ipoo"
KeyName : "Flink,DataStream,min(),max"
---


## 解释

官方文档中:
> The difference between min and minBy is that min returns the minimum value, whereas minBy returns the element that has the minimum value in this field (same for max and maxBy).

翻译:
> min和minBy之间的区别是min返回最小值，而minBy返回在此字段中具有最小值的元素（与max和maxBy相同）。

**但是事实上,min与max 也会返回整个元素。**

不同的是min会根据指定的字段取最小值，并且把这个值保存在对应的位置上，对于其他的字段取了最先获取的值，不能保证每个元素的数值正确，max同理。

而minBy会返回指定字段取最小值的元素，并且会覆盖指定字段小于当前已找到的最小值元素。maxBy同理。

## 示例论证

先拿min()与minBy()举例:

取第三个元素的最小值

```java
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        //获取数据源
        List data = new ArrayList<Tuple3<Integer,Integer,Integer>>();
        data.add(new Tuple3<>(0,2,2));
        data.add(new Tuple3<>(0,1,1));
        data.add(new Tuple3<>(0,5,6));
        data.add(new Tuple3<>(0,3,5));
        data.add(new Tuple3<>(1,1,9));
        data.add(new Tuple3<>(1,2,8));
        data.add(new Tuple3<>(1,3,10));
        data.add(new Tuple3<>(1,2,9));

        DataStreamSource<Tuple3<Integer,Integer,Integer>> items = env.fromCollection(data);
        items.keyBy(0).min(2).print();
        
        env.execute("defined streaming source");
    }
```
输出结果:
```java
(0,2,2)
(0,2,1)
(0,2,1)
(0,2,1)
(1,1,9)
(1,1,8)
(1,1,8)
(1,1,8)
```
可以看到返回的元素第二个字段取的是获取到第一个元素的字段值; 往下找,第二个元素的指定值是最小的，则把这个值保存的对应位置。

接下来再看minBy()的运行结果:
```java
(0,2,2)
(0,1,1)
(0,1,1)
(0,1,1)
(1,1,9)
(1,2,8)
(1,2,8)
(1,2,8)
```
返回的是指定字段最小值的元素。可以看到元素数值的正确。

当然max(),maxBy同理。

扫码关注公众号《ipoo》
![ipoo](http://oss.ipooli.com/images/%E5%85%AC%E4%BC%97%E5%8F%B7code.jpg)