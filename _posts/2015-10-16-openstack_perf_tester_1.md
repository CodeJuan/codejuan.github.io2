---
layout: post
title: openstack性能测试器(1):AMQP
date: 2015-10-16 00:00:00
categories:
- code
tags: 
- openstack
- perfmance
- AMQP
mathjax: true
description: 
---

# 背景
我司的公有云产品是基于OpenStack，一直以来都有做性能测试，但以前的性能测试方法比较老土。

- 有一部分基于http的消息是通过自己写的测试器来测试，即模拟真实场景的消息收发，测试各组件在高并发下的性能。
- 另外一些基于AMQP的消息则还是通过一堆虚拟机来做测试，需要耗费大量资源。

有鉴于此，需要再把测试器完善一下，使其能模拟OpenStack的各种组件，用有限的几台虚拟机，就能完成所有组件的性能测试。
而作为什么都会一点的牛X人物，自然少不了被派来开荒。正好可以借此机会深入了解一下OpenStack，以前只是久闻其名，无缘深入探究，这次终于得偿所望。
嘎嘎嘎！！
<!--more-->

# AMQP
要想做好模拟器，就需要了解AMQP协议，以及此协议在OpenStack中的应用场景。

## 概念
AMQP是一种异步消息协议，在分布式系统就被大量使用。其传递流程可以大致理解成下图

![](https://github.com/CodeJuan/codejuan.github.io/raw/master/images/blog/amqp/hello-world-example-routing.png)
```
producer->broker->consumer
```
producer连接broker，broker可以理解为一个消息服务器，所有的消息都是通过它来中转。
还有几个概念需要注意：exchange channel topic routing-key


等有空的时候，照着wireshark抓的包来对着讲解一下消息收发过程。
以login为例，

clien|direct|server
-------|-------|----
建立TCP连接| ==>|
发送heade| ==>|
收| <==|connection
connection ok| ==>|
收| <==|connection.tune
connection.tune ok| ==>|
open connection| ==>|
收| <==|open connection ok


## openstack
openstack用的就是AMQP，具体实现有两种，是rabbitMQ和qpid，二者皆可使用。曾描过一眼，说ubuntu用rabbit，centos用qpid。我司用的是ubuntu，那么就看rabbitMQ好了。
发送分为三类
- cast:发送给特定的consumer，且不等待response
- call：发送给特定consumer，需要等待response
- fanout：发送给订阅了此消息的一组consumer，不等待response

### oslo.messaging
这个组件专门负责消息。其中的/_drivers/impl_rabbit.py就是rabbitMQ的具体实现。

## 吐槽
在折腾的过程中，在oslo.messaging的tools里发现了一个`simulator.py`，就是一个消息模拟器，然而由于原有框架是C写的，历史原因只能用C再做一个模拟器。
其实openstack就有一个性能测试工具，名叫rally，也很不错，但是也不让用，sigh～～


欲知后事如何，请看下集openstack的消息流程



----------------------------

`本博客欢迎转发,但请保留原作者信息`
github:[codejuan](https://github.com/CodeJuan)
博客地址:http://blog.decbug.com/

