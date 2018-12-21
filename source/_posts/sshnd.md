---
title: SSH打洞 使用Chrome访问内网服务
date: 2018-11-29 19:32:58
tags: SSH隧道 Chrome代理
---

SSH创建隧道，Chrome通过隧道链接到目标主机后，可以直接访问目标主机所在网络的服务。


### 1. 创建SSH隧道

``` bash
ssh -ND 8133 user@ip
```
    其中8133为本地的代理端口
    

### 2. Chrome接入目标服务器

可以直接配置Chrome的代理服务器为以下配置：   
协议：Socket5   
服务器：127.0.0.1   
端口：8133   
此处推荐使用[SwitchyOmega](https://github.com/FelisCatus/SwitchyOmega/)管理代理服务器

在Linux下推荐使用命令行启动，参数中包含代理服务配置和用户目录，用户目录是临时目录，这样可以在每次使用时，有一个干净的环境
``` bash
google-chrome --proxy-server="socks5://localhost:8133" --host-resolver-rules=" MAP * 0.0.0.0, EXCLUDE localhost " --user-data-dir=/tmp/go-chrome
```

