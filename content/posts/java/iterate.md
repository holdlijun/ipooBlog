---
title: "Java List和Map遍历的方法,forEach()的使用"
date: 2020-07-15T16:49:16+08:00
description: Java List和Map遍历的方法，forEach()的使用
categories: ["Java"]
tags: ["Java"]
featured_image:
author: "ipoo"
KeyName : "Java,集合,集合遍历,List遍历,Map遍历"
---

**注意:**

> 不要在foreach循环里进行元素的remove/add操作。remove元素请使用Iterator方式，如果并发操作，需要对Iterator对象加锁。


## Java 8之前

### List
```java
        // List
		List<String> list = new ArrayList<>(6);
		list.add("1");
		list.add("2");
		for (Iterator<String> iterator = list.iterator();iterator.hasNext();){
			String item = iterator.next();
			System.out.println(item);
			if (删除元素的条件) {
				iterator.remove();
			}
		}
		
		
```

### Map

**规范:**

> 使用 entrySet遍历 Map类集合 KV，而不是 keySet方式进行遍历。

> 说明：keySet 其实是遍历了2 次，一次是转为 Iterator 对象，另一次是从 hashMap 中取出key所对应的 value。而 entrySet 只是遍历了一次就把 key和value都放到了entry中，效率更高。如果是 JDK8，使用 Map.forEach 方法。

> 正例：values()返回的是 V值集合，是一个 list 集合对象；keySet()返回的是K 值集合，是一个 Set 集合对 象；entrySet()返回的是K-V值组合集合。 

```java
        // Map 使用 entrySet
		HashMap<String,Integer> map = new HashMap<>(6);
		map.put("a",1);
		map.put("b",2);

		for (Map.Entry<String,Integer> entry : map.entrySet()){
			System.out.println("key:"+entry.getKey()+"\tvalue:"+entry.getValue());
		}
```
## Java 8 之后

### 使用forEach() + Lambda 表达式

```java
        // List
		List<String> list = new ArrayList<>(6);
		list.add("1");
		list.add("2");
		list.forEach(v -> System.out.println(v));
		
		// Map
		// Map
		HashMap<String,Integer> map = new HashMap<>(6);
		map.put("a",1);
		map.put("b",2);

		map.forEach((k,v) -> {
			System.out.println("key:"+k+"\tvalue:"+v);
		});
		
```

使用forEach + Lambda表达式之后，代码量减少了很多。


扫码关注公众号《ipoo》
![ipoo](http://oss.ipooli.com/images/%E5%85%AC%E4%BC%97%E5%8F%B7code.jpg)