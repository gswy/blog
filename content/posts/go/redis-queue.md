---
categories: ["Go", "Redis"]
date: "2022-08-08"
description: "Go使用Redis实现简单队列"
title: "Go使用Redis实现简单队列"
type: "post"
---

## 需求
在提供Web API接口的时候，经常遇到一些耗时操作，例如发送邮件，因为发送邮件的协议连接邮件服务器很慢，同步操作的话，会让用户等待很久。
此时就可以使用异步队列来完成这样的需求。

## 实现
队列本身就是先进先出原则，所以使用Redis自带的`List（列表）`最适合不过。此处案例采用的是[github.com/go-redis/redis/v8](github.com/go-redis/redis/v8)
这个包，它提供较为细致的Redis操作API，且跟Go的基本数据类型吻合。
