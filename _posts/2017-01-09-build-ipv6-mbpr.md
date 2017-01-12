---
layout: post
title:  "使用MacBookPro构造ipv6网络"
date:   2017-01-09 10:58:52
categories: ipv6
published: true
comments: true
thread: 20170109101155555
---
使用MacBookPro构造ipv6网络
---

由于app store审核要求ios必须要支持ipv6，所以在应用提交审核之前我们需要构造ipv6环境。
开发ios应用必须要使用苹果系列的硬件。我们选用mac book pro作为ipv6构建的硬件。

1. 有网线的macbook Pro
第一种比较简单
    - 在 setting->share 中，右边‘共享以下来源的连接’处选择 有线宽带
    - 右边‘用以下端口共享给电脑’处  选择 wifi，如果需要对共享网络加密，点击wifi选项中设定加密方式和密码
    - 勾选 ‘创建Nat64网络’
    - 左边勾选 ‘互联网共享’
设定完毕后，用测试机连接共享出来的网络即可

2. 没有网线的macbook Pro
    - 首先想到的是把wifi网络，通过蓝牙PAN共享给iphone，结果设置好后死活连不上网络，google一下解释是802.1x类型网络安全限制。 [解释详情](http://apple.stackexchange.com/questions/98370/share-wifi-internet-via-bluetooth-pan)
    - 另外一种曲线救国方式。除了测试机器外，还需要提供一部提供网络的iphone连接到电脑上。类似于第一种方式。
把iphone USB网络通过 wifi方式共享给测试iphone设备。
