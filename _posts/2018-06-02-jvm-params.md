---
layout: post
title:  "Java虚拟机参数"
date:   2018-06-02 10:58:52
categories: jvm
published: true
comments: true
thread: 20180602101155555
---
java虚拟机参数
---



1. 一行命令获取当前JVM所有可设置的参数以及当前默认值
```shell
[root@localhost ~]$ java -XX:+PrintFlagsInitial  >>1.txt
```
然后查看这个1.txt即可
```log
uintx AdaptivePermSizeWeight                    = 20              {product}             
uintx AdaptiveSizeDecrementScaleFactor          = 4               {product}             
uintx AdaptiveSizeMajorGCDecayTimeScale         = 10              {product}             
uintx AdaptiveSizePausePolicy                   = 0               {product}             
uintx AdaptiveSizePolicyCollectionCostMargin    = 50              {product}    
```
具体的参数含义可以百度
2. 这条指令是默认执行在Path 环境 配置的jvm。
```bash
[root@localhost ~]$ java -XX:+PrintCommandLineFlags -version
-XX:InitialHeapSize=268435456 -XX:MaxHeapSize=4294967296 -XX:+PrintCommandLineFlags -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseParallelGC
java version "1.8.0_141"
Java(TM) SE Runtime Environment (build 1.8.0_141-b15)
Java HotSpot(TM) 64-Bit Server VM (build 25.141-b15, mixed mode)
```
