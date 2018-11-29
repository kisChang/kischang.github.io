---
title: SSH创建隧道Chrome访问内网服务
date: 2018-11-29 19:32:58
tags: SSH隧道 Chrome代理
---

SSH创建隧道，Chrome通过隧道链接到目标主机后，可以直接访问目标主机所在网络的服务。


### 1. 创建SSH隧道

``` bash
ssh -ND 8133 user@ip
```
    其中8133为本地的代理端口
    

### 2. 启动Chrome

Linux下
``` bash
google-chrome --proxy-server="socks5://localhost:8133" --host-resolver-rules=" MAP * 0.0.0.0, EXCLUDE localhost " --user-data-dir=/tmp/pro
```
Window 下
    
    可以采用创建快捷方式的方法带参数启动
