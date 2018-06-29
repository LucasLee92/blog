---
title: 在Ubuntu上使用SSL/TLS加密ftp服务器
date: 2018-06-28 17:39:37
categories:
    - linux
    - ftp
tags: 
    - ftp
    - ubuntu
---

## 生成 SSL/TLS 证书

1. 创建子文件夹用于存储证书，`sudo mkdir /etc/ssl/private`
2. 生成证书和key，`sudo openssl req -x509 -nodes -keyout /etc/ssl/private/vsftpd.pem -out /etc/ssl/private/vsftpd.pem -days 365 -newkey rsa:2048`

## 配置vsftpd使用SSl/TLS加密通信

### 配置防火墙策略
```
sudo ufw allow 990/tcp
sudo ufw allow 40000:50000/tcp
sudo ufw status
```

### 修改配置文件
* `sudo vi /etc/vsftpd.conf`

```conf
ssl_enable=YES
ssl_tlsv1=YES
ssl_sslv2=NO
ssl_sslv3=NO
#rsa_cert_file=/etc/ssl/private/ssl-cert-snakeoil.pem
#rsa_private_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
rsa_cert_file=/etc/ssl/private/vsftpd.pem
rsa_private_key_file=/etc/ssl/private/vsftpd.pem
allow_anon_ssl=NO
force_local_data_ssl=YES
force_local_logins_ssl=YES
require_ssl_reuse=NO
ssl_ciphers=HIGH # 选择高加密的加密方式
pasv_min_port=40000 # 被动通信的最大和最小端口
pasv_max_port=50000
debug_ssl=YES # ssl连接的日志级别提升到debug
```

* 重启，`sudo systemctl restart vsftpd`

















