---
layout: post
title:  "微信网页授权－invalid code"
date:   2016-06-17 09:47:52
categories: wechat 微信
published: true
comments: true
thread: 20160617094752555
---

微信网页授权登录时, 总是报错40026， invalid code。
根据微信网页授权登录规范来拼取路径，然后从微信回调url中获取参数code，
根据该code 获取 access_token时报错, 由此可见不是程序的原因：
- 微信方面的原因
- 系统部署的原因


对故障的预期进行逐一排查：

第一个原因：
- 对饿了么，大众点评的微信H5界面进行测试，发现功能OK，该原因排查完毕。

第二个原因：
- 查看nginx后面的tomcat进程，发现居然有两个启动的进程。关闭后，重新，微信网页授权成功。
