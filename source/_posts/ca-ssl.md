---
title: 自签证SSL证书
date: 2018-11-29 19:40:09
tags: CA SSL
---

文档并不完善，找时间再填补，先备份历史日志。

#### 1. 自建CA

ca 私钥
```bash
openssl genrsa -out ca.key 2048
```

ca 创建
```bash
openssl req -new -x509 -days 36500 -key ca.key -out ca.crt -subj \
"/C=CN/ST=Beijing/L=Beijing/O=Temp/OU=Temp"
```

#### 2. ssl证书

ssl私钥
```bash
openssl genrsa -out server.key 2048
```

创建证书
```bash
openssl req -new -key server.key -out server.csr -subj \
"/C=CN/ST=Beijing/L=Beijing/O=Temborn/OU=tem.com/CN=owncloud.tem.com"
```



#### 3. 签发ssl证书

准备工作

```bash
mkdir demoCA 
cd demoCA  
mkdir newcerts 
touch index.txt  
echo '01' > serial 
cd ..

openssl ca -in server.csr -out server.crt -cert ca.crt -keyfile ca.key
```

owncloud 的证书   
目录：/mnt/data/certs
```bash
cd /usr/local/share/ca-certificates/
sudo update-ca-certificates
```