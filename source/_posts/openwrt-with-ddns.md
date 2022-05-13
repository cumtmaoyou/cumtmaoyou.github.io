---
title: OpenWRT问题小记
categories:
  - 折腾小记
tags:
  - OpenWRT
abbrlink: 168ad1fd
date: 2022-05-13 11:53:59
---

## 0x00. DDNS 无法正常工作

对于远程工作，我之前一直是使用[向日葵](https://sunlogin.oray.com/)的，好处是简单易用，随时内网穿透。但是要说体验，那 windows 自身的远程桌面工具（RDP）是远超向日葵之类的工具的。想要 rdp 家庭网络，需要折腾几个东西：

1. 路由支持 DDNS。
   这个一般路由器都提供 DDNS 的功能，我自己使用的是 OpenWRT 的软路由，安装 ddns-scrip 插件就可以了
2. ddns 服务提供商增加一条解析记录
   我目前用的服务是 noip.com 提供的，可以免费使用，但是需要定期 renew 你的 hostname

<!--more-->

### 问题描述

安装了 openclash 后，ddns 无法更新，重启 ddns 服务也不行

### 解决过程

1. 查看 ddns 配置，检查密码是否错误
2. 重启 ddns 服务，发现还是无法更新
3. 重启路由器，第一次更新正常，但是再手动点击更新，发现还是更新失败
4. 查看 ddns 日志，发现 CURL 返回错误 28
5. 找到日志里更新的 url，在浏览器里访问，发现更新正常
6. ssh 到路由，ping 更新的 URL，发现正常 ping
7. 找到日志里更新的命令行，用执行改命令，发现失败
8. 移除--noproxy 参数，发现更新失败
9. 移除--interface 参数，发现更新成功，那应该就是--interface 参数引起的

_到这里大致知道是什么问题引起的了，接下来尝试找到解决方案_

1. 尝试在 luci 界面中修改接口设置，但是好像没有移除`--interface`选项的
2. 安装了`ddns-script_no-ip_com`包，想看看有没有关于`--interface`的选项，但是安装后在 luci 界面的服务商列表里没找到 noip
3. 尝试自己修改 ddns 的更新脚本，找到脚本所在位置，位于`/usr/lib/ddns`目录下
4. 查看`update_no-ip_com.sh`，发现里面调用了`do_trasfer`函数
5. 查看`update_dns_functions.sh`，查找`do_transfer`，查找`--interface`，找到后注释改行
6. 重启 ddns 服务，发现问题已经解决了

整个过程大约花了 2 个小时，但是具体是哪里设置错误导致不能更新还是不太明白，也不愿意深究了，能正常工作就行。
