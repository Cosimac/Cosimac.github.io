---
title: 内网穿透
date: 2024-10-23 10:20:22
tags: web
---
# 这个可怜的前端终于学会了内网穿透

[九段刀客](https://juejin.cn/user/4001878055066237/posts)

## 什么是内网穿透

**内网穿透**这个词大家肯定都有所耳闻，毕竟都是干这行的嘛！没吃过猪肉还能没见过猪跑啊。不就是让自己电脑启的服务，让远在千里之外的人也能访问到吗？对对对就是这个东西。

## 什么场景下，让我需要这个东西呢

微信支付和小程序的图片安全校验的时候。

微信支付和图片安全校验都需要给一个notify\_url，处理成功后腾讯那边会调用这个接口来通知你结果和信息。

这个场景下内网穿透就非常重要了，如果你不弄内网穿透，每次都要传到测试服务器上去就会很繁琐，写一点传一次，有问题还得倒腾一次，如果能让你的本地就能实现测试服务器一样的功能，那岂不是完美。

有朋友会说服务啊，这不是后端的事情吗？这不是越来越卷了吗？

## 依赖安装和token配置

其实很简单哈哈😄,首先拿出我们的前端秘制配方 npm

`npm install ngrok -g`

它似乎不想我们一步到位，还需要配置一下token，到ngrok.com 注册一个账号 [ngrok官网地址](https://link.juejin.cn/?target=https%3A%2F%2Fngrok.com%2F "https://ngrok.com/")

`ngrok config add-authtoken xxxxxxxxxxxxxxx`

![image.png](https://p9-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/3204ca5c30e64855829632346f27bc88~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5Lmd5q615YiA5a6i:q75.awebp?rk3s=f64ab15b&x-expires=1729664096&x-signature=XG7gL17AA81XdT1yt5au66QNIcI%3D)

## 起飞🛫

先把你的nodejs服务搞起来，比如端口是3080,再运行内网穿透

`ngrok http http://localhost:3080`

## 看看效果

![image.png](https://p9-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/aee65decc61b4dbc9716b34f10c837e4~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5Lmd5q615YiA5a6i:q75.awebp?rk3s=f64ab15b&x-expires=1729664096&x-signature=8g90jOIZXwxqXWvEQ%2FVRZx%2FVs1k%3D)