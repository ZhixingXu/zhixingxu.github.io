---
layout: post
title:  "linux 配置和使用 service"
author: shuai
categories: [ linux,service ]
image: assets/images/1648375125154.png
---

今天在公司的云开发机上面把我们的一个 cpp 工程搭建了一个 websocket 服务.本地访问挺好的.通过 vscode 的端口转发在 mac 机器上也能正常访问.可是当我想让公司内网的其他同事访问该服务的时候却一直访问不通.公司的另一位同事说云开发机上搭建的服务是可以内网访问的,于是我只好找公司网络安全相关的同事咨询,费了一番功夫,最后终于发现,是由于我的程序里面监听的是`ws:://localhost:8080`,而实际应该把 localhost 换成机器的内网 ip 地址.挺简单的问题,在沟通过程中了解到 linux 的 service 相关的知识.

我的这个网络服务使用 cpp 写的,编译出来一个可执行程序,我一直以为直接运行就行或者再加个`nohup`和`&`放到后台和防止被杀后台.没想到还可以通过创建一个.service 文件来启动和管理服务.

Linux中.service文件是某项服务对应的配置文件，可用于systemd管理和控制的服务的设置。

.service 文件通常包含3个模块，
    即[Unit]控制单元，表示启动顺序和依赖关系；
    [Service]服务，表示服务的定义；
    [Install]安装，表示如何安装配置文件。

一般系统管理员手工创建的单元文件存放在/etc/systemd/system/目录下面。

我的程序需要创建的 service 服务是这样的:

```
[Unit]
Description=test_qqseg
After=network.target

[Service]
WorkingDirectory=/data2/lfe/assets
ExecStart=/data2/lfe/build/linux/debug/x86_64/test_qqseg /data2/zeroshot-models/common/ 8080
Restart=always
User=brightxxu

[Install]
WantedBy=multi-user.target
```

启动服务: `sudo systemctl enable test_qqseg.service`
运行服务: `sudo systemctl start test_qqseg.service`
检查状态: `systemctl status test_qqseg.service `
