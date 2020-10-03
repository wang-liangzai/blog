---
title: "解除锐捷客户端对VMware NAT Service限制"
date: 2020-09-12T12:44:51+08:00
draft: false
tags: [虚拟机,网络,破解]
categories: [软件]
summary: 锐捷客户端会每30~60秒就停止VMware NAT Service的运行，这里通过修改8021x.exe来解除限制
author: "八荒山人"
authorlink: "https://www.shan-ren.cn"
---

> **转载自[山人的博客](https://www.shan-ren.cn)**

## VMware NAT Service停止运行原因

使用无线连接校园网时，只需要在认证网页上登录即可，VMware NAT Service不受影响；而使用有线连接校园网时，需要使用锐捷客户端来登录。

问题来了——客户端安装文件夹里的 **8021.exe** 程序大概每30~60秒就检查一次VMware NAT Service是否处于运行状态，如果是，就停止VMware NAT Service的运行，所以使用NAT模式的VMware虚拟机就无法上网。

<p align="center"><img src="https://cdn.jsdelivr.net/gh/BahuangShanren/picture@master/锐捷客户端版本.png" style="zoom:150%;" /></p>

## 解决方法

### 有效的方法

> 思路是修改含有 `VMware NAT Service` 的代码，使这个关闭VMware NAT Service的功能失去作用目标。

1. 在任务管理器中先结束 **RJSuService** ，然后再结束 **锐捷认证客户端** 。

<p align="center"><img src="https://cdn.jsdelivr.net/gh/BahuangShanren/picture@master/RJSuService.png" /></p>

<p align="center"><img src="https://cdn.jsdelivr.net/gh/BahuangShanren/picture@master/锐捷认证客户端.png" /></p>

2. 用 [UltraEdit](https://www.ultraedit.com/downloads/ultraedit-download/) 打开锐捷客户端安装文件夹里的8021.exe，发现显示的数据是16进制的，用网上的 **ASCII字符串转16进制工具** ，把 `VMware NAT Service` 转换为 `56 4d 77 61 72 65 20 4e 41 54 20 53 65 72 76 69 63 65` ，然后在UltraEdit中搜索 `56 4d 77 61 72 65 20 4e 41 54 20 53 65 72 76 69 63 65` ，发现只有一处，那么用其他不存在的服务来替换它，比如 `AAware NAT Service` ，转换为16进制为 `41 41 77 61 72 65 20 4e 41 54 20 53 65 72 76 69 63 65` ，如下图：

<p align="center"><img src="https://cdn.jsdelivr.net/gh/BahuangShanren/picture@master/%E6%90%9C%E7%B4%A2VMware%20NAT%20Service%E4%BD%8D%E7%BD%AE.png" style="zoom:80%;" /></p>

<p align="center"><img src="https://cdn.jsdelivr.net/gh/BahuangShanren/picture@master/%E6%9B%BF%E6%8D%A2%E4%B8%BAAAware%20NAT%20Service.png" style="zoom:80%;" /></p>



> 注意不要改变字符长度，比如改成 `AAA Service` 的16进制编码 `41 41 41 20 53 65 72 76 69 63 65` ，这样改动后启动不了锐捷客户端。

这样改好了以后，如果没有失误导致的意外，就会发现系统服务中的VMware NAT Service不会被停止运行了。

### 失效的方法

1. 更改8021x.exe对于VMware NAT Service的检测时间
2. 直接搜索 `VMware NAT Service` ，替换为其他字符。

## 虚拟机NAT模式联网

因为 `VMware NAT Service` 总是被锐捷客户端停止运行，所以处理好锐捷客户端之后才弄明白，只要 `VMware NAT Service` 、`VMware DHCP Service` 处于运行状态，不必在 **主机** 的 **网络共享中心** 里的 **更改适配器** 中共享主机的网卡到虚拟机的网卡：

<p align="center"><img src="https://cdn.jsdelivr.net/gh/BahuangShanren/picture@master/主机网卡共享给虚拟机.png" style="zoom: 67%;" /></p>

VMware中的 **编辑-虚拟网络编辑器** 中的NAT模式保持默认设置，不是默认就还原默认设置：

<p align="center"><img src="https://cdn.jsdelivr.net/gh/BahuangShanren/picture@master/NAT设置.png" style="zoom: 80%;" /></p>

**虚拟机-虚拟机设置-硬件-网络适配器** 中选择NAT模式：

<p align="center"><img src="https://cdn.jsdelivr.net/gh/BahuangShanren/picture@master/选择NAT模式.png" style="zoom:67%;" /></p>

 **主机** 的 **网络共享中心** 里的 **更改适配器** 中虚拟机网卡选择自动获取IP和DNS即可，到此为止，虚拟机已经能上网了，不必做更多的配置。