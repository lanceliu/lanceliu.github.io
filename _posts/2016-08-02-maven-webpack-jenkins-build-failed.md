---
layout: post
title:  "Jenkins－Maven，Webpack构建踩过的坑"
date:   2016-08-02 17:35:52
categories: jenkins maven webpack
published: true
comments: true
thread: 20160802171852555
---

上线前推到生产时，测试爆出来构建速度超级缓慢，具体现象：
- 1. maven构建直接爆错 “Java heap space”
- 2. webpack 构建花费20分钟


## 错误一
1. 由于对Java构建比较熟悉，发现由于webpack构建的依赖node包有一部分是通过 cnpm安装的（cnpm安装时，即使是保留历史版本的，通过一个软连接形式对外）。
由于Maven对软连接（symlink），隐藏文件夹处理时会陷入循环引用的深渊，继而耗尽maven分配的jvm，引发 “Java heap space”异常。
2. 删除cnpm已经安装的node依赖，并删除node_modules/.npminstall 文件夹
3. 运行mvn -X package -Dmaven.test.skip=true, 很快完成构建。


## 错误二
1. webpack构建是由于一个前端迷糊同学引入了一个node依赖“vue-echarts”, 这个依赖的配置项中的入口  "main":"dist/vue-echarts.js", 明显不符合常理，一般该处会引用“src”文件夹下的入口文件
2. node-sass 有一个包死活install失败，具体情况见github上一个issue：https://github.com/sass/node-sass/pull/1117
［libsass seems to compile just fine with the default gcc 4.6.3］，构建机器gcc才是4.4.x的，yum update后。重新安装成功。
3. 本地修复后，前端同学们已经提交issue给百度的同学们了。
4. 重新安装此依赖包，webpack构建花费一分钟不到就完成了。


以上就是填坑过程。
