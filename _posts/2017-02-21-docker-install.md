---
layout: post
title:  "安装Docker"
date:   2017-02-21 14:50:52
categories: docker
published: true
comments: true
thread: 20170221145055555
---

# Docker安装

安装环境MacOS 10.12.3, 下载完docker-toolkit后点击安装，完成打开时打开 Docker Terminal时询问使用 terminal还是iterm2，选择完毕后输入命令
```shell
$ docker info
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
```
google了一下也没有解决方案，既然没有解决问题的钥匙，那就自己配一把。
进入/Application/Docker Terminal 下

```shell
➜  ~ ll /Applications/Docker/Docker\ Quickstart\ Terminal.app/Contents/Resources/Scripts/
iterm.scpt     main.scpt      start.sh*      terminal.scpt
```

start.sh从文件名看就感觉是初始化脚本，执行后

```shell
~ /Applications/Docker/Docker\ Quickstart\ Terminal.app/Contents/Resources/Scripts/start.sh
Creating CA: /Users/liufei/.docker/machine/certs/ca.pem
Creating client certificate: /Users/liufei/.docker/machine/certs/cert.pem
Running pre-create checks...
Creating machine...
(default) Copying /Users/liufei/.docker/machine/cache/boot2docker.iso to /Users/liufei/.docker/machine/machines/default/boot2docker.iso...
(default) Creating VirtualBox VM...
(default) Creating SSH key...
(default) Starting the VM...
(default) Check network to re-create if needed...
(default) Found a new host-only adapter: "vboxnet0"
(default) Waiting for an IP...
Waiting for machine to be running, this may take a few minutes...
Detecting operating system of created instance...
Waiting for SSH to be available...
Detecting the provisioner...
Provisioning with boot2docker...
Copying certs to the local machine directory...
Copying certs to the remote machine...
Setting Docker configuration on the remote daemon...
Checking connection to Docker...
Docker is up and running!
To see how to connect your Docker Client to the Docker Engine running on this virtual machine, run: /usr/local/bin/docker-machine env default

```

启动成功
