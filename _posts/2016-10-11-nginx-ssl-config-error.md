---
layout: post
title:  "nginx ssl config error"
date:   2016-10-11 09:50:52
categories: https nginx
published: true
comments: true
thread: 20161011095055555
---

1. nginx.conf文件配置增加如下配置
```config
      listen       443 ssl;
      ssl on;
      ssl_certificate      /web/deploy/nginx/conf/ssl/1_91cfx.com_bundle.crt;
      ssl_certificate_key  /web/deploy/nginx/conf/jebus.key;
```

2. 测试配置文件
```bash
[ ~]$ ./nginx -t -c /web/deploy/nginx/conf/nginx.conf
the configuration file /etc/nginx/nginx.conf syntax is ok
configuration file /etc/nginx/nginx.conf test is successful
```
3. 但是在错误日志中
```log
2016/10/10 19:30:24 [emerg] 3981#0: SSL_CTX_use_PrivateKey_file("/web/deploy/nginx/conf/ssl/91cfx.key") failed (SSL: error:0906406D:PEM routines:PEM_def_callback:problems getting password error:0906A068:PEM routines:PEM_do_header:bad password read error:140B0009:SSL routines:SSL_CTX_use_PrivateKey_file:PEM lib)
```
4. 网上有一篇 [文章](https://snippets.aktagon.com/snippets/543-how-to-fix-pem-read-bio-no-start-line-error-nginx-error) 说是，生成crt文件时，没有把csr文件中的下面内容copy进去
```
-----BEGIN CERTIFICATE-----
...
-----END CERTIFICATE-----
-----BEGIN RSA PRIVATE KEY-----
...
-----END RSA PRIVATE KEY-----
```
5. 解决方案把现有的key提取private key
```bash
[ ~] openssl rsa -in jebus.key -out jebus-stripped.key
```
6. 把生成的新key更新到nginx.conf文件中，启动成功。
