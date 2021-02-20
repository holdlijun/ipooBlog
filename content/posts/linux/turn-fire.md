---
title: "Linux关闭防火墙 Centos7如何关闭防火墙"
date: 2020-05-28T17:36:37+08:00
description: Linux关闭防火墙,Centos7怎么关闭防火墙
categories:
 ["linux"]
tags : ["linux"]
featured_image:
author: "ipoo"
KeyName : "linux,防火墙"
---

## 查看防火墙状态

`systemctl status firewalld`

centos7：`service  iptables status`

## 即使生效，临时关闭

`systemctl stop firewalld`

`service iptables stop`

## 永久关闭防火墙

`chkconfig iptables off`

centos7：`systemctl disable firewalld` (停止并禁用开机启动)

## 启动防火墙

`systemctl start firewalld`

## 重启防火墙

`systemctl enable firewalld` (开机启动)

`service iptables restart`


---

