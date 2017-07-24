---
layout: post
title:  "FTP上传下载"
date:   2017-06-17 10:58:52
categories: ftp
published: true
comments: true
thread: 20170617101155555
---
FTP上传下载
---

使用 commons-net 框架

遇到了一些问题：
- 文件内容乱码
    - 保证上传文件编码格式不变，使用二进制方式：setFileType(FTPClient.BINARY_FILE_TYPE)
- 本地上传下载不了
    - 本地防火墙端口权限问题，使用本地被动方式：enterLocalPassiveMode
- 中文名称上传下载失败
    - 需要把文件名转码为 iso-8859-1
- web下载时命名乱码
    - 单纯的字符转码utf 转为  iso-8859-1 ，才适合浏览器


[参考FTP被动主动](http://www.cnblogs.com/xiaohh/p/4789813.html)
