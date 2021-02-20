---
title: "Linux 集群免密登录配置,双向免密"
date: 2020-04-25T11:18:55+08:00
description: "Linux 集群免密登录配置,双向免密"
categories: ["linux",]
tags : [
"linux"]
featured_image: http://oss.ipooli.com/images/blog/linx.png
author: "ipoo"
KeyName : "Linux 集群免密登录配置,Linux"
---

<!-- MarkdownTOC -->

- [linux 集群免密登录配置](#linux-%E9%9B%86%E7%BE%A4%E5%85%8D%E5%AF%86%E7%99%BB%E5%BD%95%E9%85%8D%E7%BD%AE)
    - [第一种方法](#%E7%AC%AC%E4%B8%80%E7%A7%8D%E6%96%B9%E6%B3%95)
    - [第二种方法](#%E7%AC%AC%E4%BA%8C%E7%A7%8D%E6%96%B9%E6%B3%95)

<!-- /MarkdownTOC -->



## linux 集群免密登录配置

### 第一种方法

- 我这边准备三台主机 <br>
分别为bigdata1,bigdata2,bigdata3

![hosts](http://oss.ipooli.com/images/blog/linuxhosts.png)
- 在bigdata1上生成秘钥对
```java
ssh-keygen -t rsa
```
之后根据提示,回车
进入.ssh目录会看见`id_rsa`(私钥) 和 `id_rsa.pub` (公钥)两个文件。
- 将公钥文件拷贝到另外两台主机
使用 `ssh-copy-id` 命令

```java
#拷贝到bigdata2主机
ssh-copy-id -i ~/.ssh/id_rsa.pub root@bigdata2
# 拷贝到bigdata3主机
ssh-copy-id -i ~/.ssh/id_rsa.pub root@bigdata3
```
如图
![copy_id_rsa](http://oss.ipooli.com/images/blog/copyidrsa.png)
即为成功。
- 验证登录
```java
ssh root@bigdata2
```
如果不提示输入密码则成功。
此方式另外两台主机登录bigdata1需要密码。
如需三台互相登录不需要密码见下面方法。

### 第二种方法

- 我这边准备三台主机 <br>
分别为bigdata1,bigdata2,bigdata3

![hosts](http://oss.ipooli.com/images/blog/linuxhosts.png)
- 在**每台主机**上都生成秘钥对
```java
ssh-keygen -t rsa
```
之后根据提示,回车
进入.ssh目录会看见`id_rsa`(私钥) 和 `id_rsa.pub` (公钥)两个文件。
- 在bigdata1上创建一个authorized_keys
```java
cp ~/.ssh/id_rsa.pub~/.ssh/authorized_keys
```
然后对集群中每台需要生成密钥对的机器都执行以上操作。
- 把所有机器上的`authorized_keys` 合并
1. 简单方法 直接查看每台机器的`authorized_keys`文件,然后直接拷贝到一个文件中。
```java
cat ~/.ssh/authorized_keys
```
2. 把合并后的一个`authorized_keys`文件,覆盖到所有机器的`~/.ssh/`的目录下。即可完成。


---
另外一种方法稍显麻烦
1. 把所有机器上的`authorized_keys`远程拷贝到一个机器上,这这台机器上合并。在去远程拷贝覆盖，具体操作如下。
    - 在bigdata2机器上执行:
    ```java
    scp ~/.ssh/authorized_keys root@bidata1:~/.ssh/bigdata2_rsa
    ```
    - 在bigdata3机器上执行:
    ```java
    scp ~/.ssh/authorized_keys root@bidata1:~/.ssh/bigdata3_rsa
    ```
    - 然后在bigdata1上合并
    ```java
    cat ~/.ssh/bigdata2_rsa >> ~/.ssh/authorized_keys
    cat ~/.ssh/bigdata3_rsa >> ~/.ssh/authorized_keys
    ```
    - 然后将bigdata1上合并好的`authorized_keys`远程拷贝到其他机器
    ```java
    scp ~/.ssh/authorized_keys root@bigdata2:~/.ssh
    scp ~/.ssh/authorized_keys root@bigdata3:~/.ssh
    ```
2. 验证登录
 ```java
ssh root@bigdata2
```
如果不提示输入密码则成功。

