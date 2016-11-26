---
layout: post
title:  "StartCom 证书申请"
date:   2016-11-25 09:50:52
categories: https
published: true
comments: true
thread: 20161125095055555
---
StartCom 证书申请
---

已经是全民https的时代了，对于个人站点来说如果上一个https成本还是略高的。别担心，良心的证书颁发机构还是有的。下面给大家介绍一下
[StartCom](https://startssl.com/)免费证书申请。

## 1.授权账号申请（比较简单不做介绍）

![chooseCertType](/assets/img/https/chooseCertType.png)

## 2.要ssl保护的域名列表 && csr生成

![firststep](/assets/img/https/StartSSL™_Certificates___Public_Key_Infrastructure.png)

## 3.获取crt文件
>在右侧菜单中的【SSL／TLS Server】中选择刚申请的证书，里面会包含Apache／Nginx／IIS／Other Server的证书，选择你需要的进行部署实施
    ![certfile](/assets/img/https/certfile.png)

## 4.服务器部署

> 露珠是nginx服务器部署的。

> - i. 第3步中的nginx文件夹中的crt文件copy到nginx所在server上
> - ii. 第2步生成csr中，的key做一个处理(拷贝一个不需要输入密码的密钥文件)`openssl rsa -in jebus.key -out jebus-stripped.key`,把处理过的keycopy到nginx所在服务器
> - iii. nginx服务器配置：

```shell
listen 443 ssl;
ssl on;
ssl_certificate [crt path];
ssl_certificate_key [jebus-stripped.key path];
```

> - iv. 测试配置是否生效
`./nginx -t -c [nginx.conf path]`

    the configuration file /etc/nginx/nginx.conf syntax is ok
    configuration file /etc/nginx/nginx.conf test is successful

## 5.重启nginx，成功
