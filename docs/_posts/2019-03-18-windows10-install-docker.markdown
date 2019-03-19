---
layout: post
title:  "Docker入门系列 - Windows系统上安装Docker"
date:   2019-03-18 21:37:58 -0500
categories: posts
author: jasydong
---
本文介绍如何在`Windows`系统下安装`Docker`。关于`Docker`的详细介绍请访问[Doker官方文档](https://docs.docker.com/)

本教程安装环境：`Windows 10` 家庭版, 当然`Winodws 7`, `Windows 8`等早期版本安装本教程同样适用)

Docker引擎使用了一个定制的Linux内核，所以要在Windows下运行Docker我们需要用到一个轻量级的`虚拟机`(VM)，我们使用Windows Docker客户端以控制Docker引擎，来创建，运行和管理我们的Docker容器。

## 安装步骤

1. 首先下载`Docker Toolbox`安装软件, 你可以点击 [这里](https://download.docker.com/win/stable/DockerToolbox.exe) 立即下载`DockerToolbox`。

2. 等下载完成之后双击`DockerToolbox.exe`进行安装，一路点击下一步完成安装后，桌面上会多出出三个图标，如下图所示：

    ![Install 01](https://jasydong.github.io/assets/images/docker/install_01.png)

3. 双击 `Docker QuickStart` 图标来启动`Docker QuickStart`终端, 如下图所示：

    ![Install 02](https://jasydong.github.io/assets/images/docker/install_02.png)

4. 在`Docker QuickStart`终端窗口`$`提示符输入`docker`命令按回车会看到很多可执行的命令列表, 至此Docker算是成功安装到你的电脑上了
