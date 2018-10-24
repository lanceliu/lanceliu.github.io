---
layout: post
title:  "Insufficient space for shared memory file"
date:   2018-09-29 10:11:52
categories: jvm
published: true
comments: true
thread: 20180929101155555
---
Insufficient space for shared memory file解决办法
---
# 问题
启动Java应用时报错
> Java HotSpot(TM) 64-Bit Server VM warning: Insufficient space for shared memory file:
   /tmp/hsperfdata_work/10700
Try using the -Djava.io.tmpdir= option to select an alternate temp location.

# 解决步骤
1. df命令查看发现磁盘已经占满
```bash
➜  [~] ✗  df -h
Filesystem                                                1K-blocks          Used    Available         Use%    Mounted on
/dev/mapper/VolGroup01-LogVol00  49384248 49384248                  0         100%     /
```

2. 使用find命令查找系统的大文件, 发现出现了很多日志的大文件，删除后重新启动应用成功
```bash
➜  [~] ✗  find / -size +100M -exec ls -lh {} \;

```

# 分析
搜索学习这个 `/tmp/hsperfdata_$user/$number`的作用，这个文件存的应该是JVM进程当前的一些性能参数（或者说运行信息）
That directory is part of a Java performance counter.


jvmstat会生成一个目录文件叫hsperfdata_username：
  - 默认的是生成在 java.io.tmpdir目录下
  - java.io.tmpdir在linux下默认是/tmp下

故默认开启了jvm monitor的功能以后就会在/tmp目录下生成一个目录叫 hsperfdata_username ，然后这个目录中会有一个pid文件，可以利用strings查看里面的文件内容，一般就是jvm的进程信息而已。

而jps、jconsole、jvisualvm等工具的数据来源就是这个文件（/tmp/hsperfdata_userName/pid)。所以当该文件不存在或是无法读取时就会出现jps无法查看该进程号，jconsole无法监控等问题
/tmp/hsperfdata_userName/pid文件会在对应java进程退出后被清除。如果java进程非正常退出（如kill -9），那么pid文件会被保留，直到执行一次java命令或是加载了jvm程序的命令（如jps、javac、jstat），会将所有无用的pid文件都清除掉


### 参考
[解决方案](https://blog.csdn.net/u014039577/article/details/49148063/)
[分析](https://blog.csdn.net/levy_cui/article/details/51143101)
