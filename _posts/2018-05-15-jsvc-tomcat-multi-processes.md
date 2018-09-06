---
layout: post
title:  "JSVC启动tomcat有多个Daemon进程"
date:   2018-05-15 10:58:52
categories: jsvc daemon
published: true
comments: true
thread: 20180515101155555
---
JSVC启动tomcat有多个Daemon进程
---


原理：
使用jsvc来运行服务，没有了默认8005的shutdown端口；
主进程pid为1，fork 2个进程。

Jsvc允许应用程序（例如Tomcat）以root身份执行某些特权操作（例如，绑定到端口<1024），然后将身份切换到非特权用户。
  - 使用root用户启动，然后降级为普通用户

Jsvc使用3个进程：启动进程，控制进程和受控进程。

受控进程：也是主要的Java线程，如果JVM崩溃，控制器将在下一分钟内重新启动它。
控制进程：Jsvc是一个守护进程，因此它应该以root用户身份启动，并且-user参数允许降级为未经授权的用户，当使用-wait参数时，启动程序进程将等待，直到控制器显示“我准备好”，否则在创建控制器进程后返回。
[Jsvc详解](http://commons.apache.org/proper/commons-daemon/jsvc.html)


init命令是Linux下的进程初始化工具，init进程是所有Linux进程的父进程，它的进程号为1。init命令是Linux操作系统中不可缺少的程序之一，init进程是Linux内核引导运行的，是系统中的第一个进程。


Daemon进程是什么：
[Daemon的详解](https://blog.csdn.net/woxiaohahaa/article/details/53487602)
[Daemon的pid文件](http://dram.me/blog/2010/08/14/pidfile-in-daemon.html)

Daemon创建：
https://blog.csdn.net/woxiaohahaa/article/details/53487602



[僵尸进程和守护进程](http://www.cnblogs.com/xiaoyixy/archive/2006/02/11/328798.html)
[守护、服务和进程之间的区别](https://askubuntu.com/questions/192058/what-is-technical-difference-between-daemon-service-and-process)


非root用户可以使用1024以下端口的方式：
https://liquidat.wordpress.com/2018/01/04/howto-run-programs-as-non-root-user-on-privileged-ports-via-systemd/
https://superuser.com/questions/710253/allow-non-root-process-to-bind-to-port-80-and-443
