---
title: "Kubernetes 是什么"
date: 2020-06-02T12:04:52+08:00
description: kubernetes是什么？都9102年了,你再不了解Kubernetes就真的out了
categories: ["K8S"]
tags: ["K8S"]
featured_image: https://bigdata-lijun.oss-cn-hangzhou.aliyuncs.com/images/%E4%B8%BB.png
author: "ipoo"
KeyName : "k8s,Kubernetes,Kubernetes理解,Kubernetes是什么"
---

<!-- MarkdownTOC -->

- [Kubernetes 是什么](#kubernetes-%E6%98%AF%E4%BB%80%E4%B9%88)
- [分别解释下 k8s 的几个关键字](#%E5%88%86%E5%88%AB%E8%A7%A3%E9%87%8A%E4%B8%8B-k8s-%E7%9A%84%E5%87%A0%E4%B8%AA%E5%85%B3%E9%94%AE%E5%AD%97)
  - [生产级别](#%E7%94%9F%E4%BA%A7%E7%BA%A7%E5%88%AB)
  - [容器](#%E5%AE%B9%E5%99%A8)
  - [编排系统](#%E7%BC%96%E6%8E%92%E7%B3%BB%E7%BB%9F)
  - [k8s 架构概览](#k8s-%E6%9E%B6%E6%9E%84%E6%A6%82%E8%A7%88)
    - [Master 组件](#master-%E7%BB%84%E4%BB%B6)
    - [Node 组件](#node-%E7%BB%84%E4%BB%B6)
  - [概念](#%E6%A6%82%E5%BF%B5)
    - [计算资源](#%E8%AE%A1%E7%AE%97%E8%B5%84%E6%BA%90)
    - [网络通信](#%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1)
    - [工作模型](#%E5%B7%A5%E4%BD%9C%E6%A8%A1%E5%9E%8B)
      - [ReplicaSet](#replicaset)
      - [DaemonSet](#daemonset)
      - [Job](#job)
      - [CronJob](#cronjob)
      - [StatefulSets](#statefulsets)
    - [更新部署](#%E6%9B%B4%E6%96%B0%E9%83%A8%E7%BD%B2)
    - [数据持久化](#%E6%95%B0%E6%8D%AE%E6%8C%81%E4%B9%85%E5%8C%96)
      - [Persistent Volumes](#persistent-volumes)
      - [Persistent Volumes Claim](#persistent-volumes-claim)
  - [小结](#%E5%B0%8F%E7%BB%93)
- [k8s 与 大数据](#k8s-%E4%B8%8E-%E5%A4%A7%E6%95%B0%E6%8D%AE)

<!-- /MarkdownTOC -->



# Kubernetes 是什么

> 原文：  [Kubernetes 是什么](https://blog.didiyun.com/index.php/2019/12/11/kubernetes-%e6%98%af%e4%bb%80%e4%b9%88/)
作者：厚之成

都9102年了，你再不了解 Kubernetes 就真的 out 了！！！（贩卖焦虑体

kubernetes 是什么？Kubernetes 这个词来源于希腊语，有主管、舵手、船长的意思，我们从中能听到一丝管理的意味，从图标中也能看出来。

![container_engine](https://bigdata-lijun.oss-cn-hangzhou.aliyuncs.com/images/Kubernetes_(container_engine).png)

在 [kubernetes](https://kubernetes.io/zh/) 的网站上，描述 kubernetes 是:
> 生产级别的容器编排系统

从这个定义我们可以提炼出三个关键字：

1. 生产级别
2. 容器
3. 编排系统

>k8s是kubernetes的缩写，中间的8代表k与s间省略的8个字母，类似的缩写方式还有 internationalization 缩写为 i18n

# 分别解释下 k8s 的几个关键字

## 生产级别
说 k8s 是生产级别有如下几个原因:
- k8s 是 Google 开源的系统，基于 Google 的 Borg 和 Omega 系统设计，这套系统已经在 Google 内部运行了10年以上，并还在支持Google 每周数十亿容器的运行。
- k8s 是 CNCF （Cloud Native Computing Foundation）的首个毕业项目。
- k8s 在2015年6月发布首个生产级成熟版本1.0后，目前（19年8月）已经进展到1.15版本，已经被各大公司广泛使用。

## 容器

*没有集装箱，就没有全球化*

什么是容器？容器的英文为 container，这个词除了有容器的意思，还有集装箱的意思。对于运输，集装箱有着重大的意义，在[《集装箱改变世界》](https://book.douban.com/subject/2354988/) 这本书中，提出了“没有集装箱，就没有全球化”的观点，下面引用其中一段文字：
>经济全球化的基础就是现代运输体系,而一个高度自动化、低成本和低复杂性的货物运输系统的核心就是集装箱。在1956年集装箱出现之前,人们很难想象美国的沃尔玛能够遍地开花。而在集装箱出现之后,以至于某件东半球的产品运至纽约销售,远比在纽约近郊生产该产品更划算。毫不起眼的集装箱降低了货物运输的成本,实现了货物运输的标准化,以此为基础逐步建立全球范围内的船舶、港口、航线、公路、中转站、桥梁、隧道、多式联运相配套的物流系统,世界经济形态因此而改变。

集装箱运输能够获得成功，可以概括出如下几个特点：

- 可移植性：集装箱可以被任何类型支持的船舶使用
- 包容性：支持多种类型的货物，这些货物都可以被打包在集装箱内
- 标准大小：标准大小的集装箱可以被完美的组合在一起
- 共存：多个集装箱可以放到同一个船上
- 隔离：不同集装箱的货物间彼此隔离

这些特点同样适用于软件领域的容器：

- 可移植性：容器可以被任何类型支持的操作系统安装使用
- 包容性：支持多种类型的软件，这些软件都可以被打包在容器内
- 标准格式
- 共存：多个容器可以运行在同一个物理机上
- 隔离：不同容器的软件间彼此隔离

因此，可以说容器是集装箱思路在软件领域的实现，容器是软件的一个标准单元，可以将代码和依赖打包在一起、能够运行在多种操作系统和环境之上、多个容器能够在同一台物理机器上运行。

![container-what-is-container](https://bigdata-lijun.oss-cn-hangzhou.aliyuncs.com/images/container-what-is-container.png)

目前一说到容器，大家最容易想到的就是 Docker，但其实在 Docker 出现之前，容器技术也已经存在：

- Cgroups：在2006年，通过 Cgroups 技术可以实现进程级别的隔离，能够对进程实现资源的限制
- Linux Containers（lxc）：在 2008年，通过 lxc 实现操作系统级别的虚拟化，能够提供单独的进程和网络空间
- Docker：在2013年，docker 出现，docker 是一套容器执行和管理系统，结合之前的 lxc 等技术，提供了更完整的方案，使容器技术被广泛使用。

此外与虚拟机不同，容器只包括了应用和应用相关的库，不同的容器共用宿主机的内核。容器也更轻量化，带来了更小的overhead。

**没有容器，就没有微服务化**

容器技术带来的最显著的影响就是使软件更加的模块化，引领软件由单体架构转变向微服务架构：
![microservice](https://bigdata-lijun.oss-cn-hangzhou.aliyuncs.com/images/microservice.png)

容器和微服务化后，带来了一些好处，比如：

- 模块间更加独立，可以独立的部署和发布，加快了发布和更新的速度
- 隔离的运行环境，可以为不同模块定制不同的运行环境

但同时微服务化带来的问题就是，以前的一个单独的服务，现在变成了多个服务，服务的个数骤增，增大了部署和运维的工作量。同时，由于不同模块的软件依赖、网络配置、资源需求等各不相同，如果都直接使用 docker cli 来进行管理和配置，无疑是一件困难且容易出错的事，不能管理大规模的容器系统，因此就需要容器编排系统来处理。
## 编排系统 

容器的编排系统需要能够管理和组织在一个集群上的运行的宿主机和容器，需要能完成如下任务：

- 管理网络和访问
- 跟踪容器的状态
- 增大或缩小服务的规模
- 实现负载平衡
- 宿主机无响应后实现容器的重新分配
- 服务发现
- 管理容器的存储
- 等等…
- 
k8s 是这样一个容器编排系统，但除了 k8s 还有一些其他的容器编排系统，比如：

- Apache Mesos：是一个集群管理工具，支持容器的编排
- Docker Swarm：docker 容器平台自带的编排系统

这三个不同的容器编排系统，在不同的场景下有各自的优势，具体的对比可以参考[文章](https://www.sumologic.com/insight/kubernetes-vs-mesos-vs-swarm/)，简单来说：

- 入门或者测试使用：Docker Swarm
- 企业级使用：k8s
- 需要除了容器编排外的更多集群管理功能：Apache Mesos

不过，不管是Docker 还是 Mesos 都支持与 k8s 的集成，所以 k8s 是目前被使用的最为广泛的容器编排系统。

## k8s 架构概览
k8s 的架构图如下：
![arch](https://bigdata-lijun.oss-cn-hangzhou.aliyuncs.com/images/arch.jpg)
构成一个 k8s 集群，有两种节点角色，Master 和 Node。Master 节点用于集群控制，一般不运行用户的应用。Node 节点则是用户应用运行的地方。

### Master 组件

Master 组件提供集群控制功能，对集群作出全局性决策（比如调度），以及检测和响应集群事件。Master 组件有如下几个部分：

- kube-apiserver：对外暴露 API，是前端控制层，其他组件都与它进行通信，也是负责鉴权和授权的守门员角色。
- etcd：是 k8s 的后端存储，所有的集群数据存放在此处。
- kube-controller-manager：处理集群中常规任务的后台线程，包括节点控制器（负责节点移除响应）、副本控制器（维护正确数量的 Pod）、端点控制器、和服务账号和令牌控制器
- cloud-controller-manager：用于与底层云提供商交互的控制器。
- kube-scheduler：监视没有分配节点的新创建的 Pod，选择一个节点供他们运行。
- Additional Services：一些额外的插件，包括DNS、用户界面、容器资源监控、集群层面日志等

### Node 组件

Node 组件在每个Node上运行，Node 是工作容器的运行节点，维护运行时的 Pod 并提供运行时的环境。Node 组件有如下几个部分：

- kubelet：是主要的节点代理，管理其 host 上 Pod 的生命周期。
- kube-proxy：管理节点上的网络规则并执行连接转发。
- Container Runtime：提取容器运行的服务，包括 docker、rkt 等
- supervisord：来提供进程监控，保证kubelet 和 docker 运行
- fluentd：守护进程，有助于提供集群日志

## 概念

不得不说，k8s 不是一个简单的系统，包含了很多概念，下面将从应用如何使用的角度来进行说明，介绍其中的各种概念。

一个应用要运行，需要如下几方面的组件：

- 计算资源
- 网络通信
- 工作模型
- 更新部署
- 数据持久化

下面将分别介绍

### 计算资源

以 docker 为例，部署一个应用需要将软件打包在 docker image 内，然后将 docker image 部署在容器中，容器提供运行应用所需要的资源，包括 CPU/RAM、基础的文件系统，网络等等。但是不能直接部署一个容器在 k8s 上，因为 k8s 中可部署的最小单元是 Pod。

Pod 有如下几个特点：

- k8s 的最小计算单元
- 包括了一个容器或者多个容器，所有容器共享同样的独立IP，同样的hostname，同样的存储资源
- 是调度的最小单位，一个pod会放到一台宿主机上，即使包括多个容器
- 一个容器一个Pod 是 k8s中最常见的场景

此外 k8s 还负责对 Pod 进行调度和健康检查。以下是一个 Pod 定义的例子：

```java
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', 'echo Hello Kubernetes! && sleep 3600']
```

此外还有一些与 Pod 相关的概念也需要了解：

- ConfigMap：一个 kv 的配置格式，可以用来配置 pods，实现 image 重用，支持在线修改
- Secret：功能和 ConfigMap 相同，但是以 base64 加密存储
- Label：可以挂载到任意对象的标签，提供分组的基础

### 网络通信

在微服务架构里，多个微服务间需要进行通信，当每个应用部署到 pod 上之后，这里存在两个挑战：

- pod 间：不能使用写死的 IP，因为 Pod 可能会漂移到其他节点上，更换IP
- 对外：希望使用所有能够提供服务的 Pod，同时支持负载均衡

因此 k8s 提供了 Service 的概念，Service 定义了一个 Pod 的逻辑组，这些 Pod 提供相同的功能服务。Service 根据类型不同分为如下几类：

- ClusterIP Service：这是默认的 Service 类型，该类型 Service 对外暴露一个集群内部 IP，可以用于集群内部服务间的通信
- NodePort Service：该类型 Service 对外暴露集群上所有节点的固定端口，支持外部通过 : 进行访问
- LoadBalancer：该类 Service 通过云平台提供的 load balancer 向外提供服务
- ExternalName：将服务映射到一个 externalName （比如 foo.bar.example.com)

此外，还可以使用 Ingress 来向外暴露服务，Ingress 虽然不是一个 Service 的类型，但也可以充当你应用集群的入口，通过编写路由规则实现流量路由。

下面是一个 ClusterIP Service 的例子：

```java
apiVersion: v1 
kind: Service 
metadata:
    name: example-prod 
spec:
    ports:
    - protocol: TCP
        port: 80 
        targetPort: 80
 
```

### 工作模型

为了方便管理多个 Pod，k8s 抽象出了几种工作模型，通过保持模型的状态，实现对 Pod 的管理。

k8s 对于模型状态的保持，采用如下图所示的机制：从目标状态开始，不断追踪集群现在的状态，并使其满足目标状态。如此循环，可以得到一个目标驱动的、有自治愈效果的系统。

目前 k8s 有如下几种工作模型：

#### ReplicaSet

- 保证总有指定个数的 Pod 在运行
- 通过 pod selector 和 pod label 进行选择，将哪些 Pod 包含进来
- 适用于几乎无状态的服务
- 例子：静态 web 服务器

#### DaemonSet

- 保证每个宿主机上运行一个 Pod
- 通过 node selector 和 node label 进行选择，将哪些 Node 包含进来
- 适用于一些节点常驻服务
- 例子：各类 agent、log 收集、监控服务

#### Job

- 在 ReplicaSet 和 DaemonSet 中，如果 Pod 的进程退出了（即使正确退出），进程也会重启
- Job 则允许 Pod 中的进程正常退出，且不会再重启
- 适用于指定次数执行任务的场景
- 例子：数据备份操作

#### CronJob

- 基于时间来调度和创建 Job

#### StatefulSets

- 有稳定的网络标示和数据存储
- Pod 创建按顺序：索引从低到高
- Pod 销毁按顺序：索引从高到低
- 适用于有状态的服务
- 例子：数据库

下面是一个 ReplicaSet 的例子:
```java
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  # modify replicas according to your case
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google_samples/gb-frontend:v3
```

### 更新部署

当需要更新应用的时候，一版希望做到如下两点：

- 不停服务，不影响正在使用的用户
- 支持灰度发布，可以在全量更新前验证新版本时候工作正常

对此，k8s 中抽象出了 Deployment 的概念，对 ReplicaSet 模型中 Pod 的更新进行支持。目前支持两种更新策略：

- Recreate：销毁所有在运行的 Pod 后进行重建，这种策略的问题是会带来 downtime
- RollingUpdate：滚动升级，符合期望的两点目标

k8s 中的更新策略为 RollingUpdate，通过两个参数来配置具体的更新方法：

max unavailable：指定能够同时进行更新的 Pod 的个数
max surge：在更新过程中能够使用的额外的 Pod 的个数

下面是一份 Deployment 的例子：

```java

apiVersion: apps/v1 
kind: Deployment 
metadata:
    name: deploy-example 
spec:
    replicas: 3 
    revisionHistoryLimit: 3 
    selector:
        matchLabels: 
            app: nginx 
            env: prod
    strategy:
        type: RollingUpdate 
        rollingUpdate:
            maxSurge: 1
            maxUnavailable: 0
 ```
 
### 数据持久化

很多服务并不是无状态的，要支持有状态的服务，出了上述的 Statefull 工作模型，还需要数据持久化的支持。k8s 通过 Persistent Volumes 子系统来实现支持。

#### Persistent Volumes

- 是对集群中存储的抽象，提供存储服务的后端可以是：NFS、RBD等
- 生命周期与使用它的 Pod 的生命周期解耦
- 可以被手动添加也可以动态添加
- 不能直接和 Pod 关联，需要 Persistent Volumes Claim

#### Persistent Volumes Claim

- 是一个用户对存储的请求
- 在存储的角色类似于 Pod，Pod 消耗 Node 资源，而 Persistent Volumes Claim 消耗 Persistent Volumes 资源
- 提供抽象的存储资源，列出需要存储满足的要求，而不是直接使用 Persistent Volumes

下面是一个 Persistent Volume Claims 的例子：

```java
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 8Gi
  storageClassName: slow
  selector:
    matchLabels:
      release: "stable"
    matchExpressions:
      - {key: environment, operator: In, values: [dev]}
```
## 小结
将上述的概念做一个小结：

- 最小计算单元 = Pod
- 最小通信单元 = Service
- 分组 = Labels
- 配置 = ConfigMap 和 Secrets
- 无状态 Pod 调度 = ReplicaSet、DaemonSet、Job
- 更新部署 = Deployment
- 有状态 Pod 调度 = StatefulSet
- 存储 = Persistent Volumes 以及之上的 Persistent Volumes Claim

# k8s 与 大数据
我们看到很多企业的业务应用都已经部署到了 k8s 的集群之上，尤其是一些无状态的 Web 服务和 API 服务，特别适合 k8s 的场景。但是以 Hadoop 为代表的大数据系统能够运行在 k8s 之上吗？

如果将 Hadoop 部署在 k8s 之上，会带来哪些好处呢：

- 更好的资源利用：现在使用k8s的在线业务集群与离线大数据集群一般不在一起，而 Hadoop 部署在 k8s 之上就可以共用一套集群，更好的利用业务白天高峰和数据夜里高峰的特点
- 按需获取资源：就像 AWS 的 EMR，可以根据需要获取任意规模和任意时长的 Hadoop 集群，按需付费
- 更好的运维与升级：可以利用 k8s 的工具对 Hadoop 集群进行运维，不用再建设 Hadoop 运维工具
- 更灵活的资源隔离：使用 k8s 实现多个集群的部署，集群间互不影响

但是也存在一些挑战：

- IO性能：首当其冲的就是 IO，大数据系统是一类重 IO 操作的应用，容器化后需要将存储从本地 IO 变为网络 IO，这必然会引起性能的下
- 数据本地化：变为网络 IO 后，数据本地化的特点将不能利用
- 安全：因为不同容器共享一个内核，会有一定的安全隐患

但应对这些挑战，已经有一些企业和组织在进行优化了，比如 bluedata 提供了一套 spark on docker 的方案，并且获得了不错的性能，如下图：
![Spark-in-Docker-BlueData](https://bigdata-lijun.oss-cn-hangzhou.aliyuncs.com/images/Spark-in-Docker-BlueData.png)

总的来说，使用 k8s 可以快速构建起一套私有云的方案，降低了自建运维体系的成本。如果未来大数据等类型的应用也能在 k8s 上运行的很好，那 k8s 将会发挥更大的作用，我对此比较乐观。



扫码关注公众号《ipoo》
![ipoo](http://oss.ipooli.com/images/%E5%85%AC%E4%BC%97%E5%8F%B7code.jpg)