---
url: /blog/linux/HybPILE5U0z
title: "RabbitMQ三种Exchange模式(fanout,direct,topic)的性能比较"
date: 2018-05-14T04:45:01+08:00
description:
categories:
  - Linux
tags:
  - Linux
menu: main
---

> none

RabbitMQ中，所有生产者提交的消息都由Exchange来接受，然后Exchange按照特定的策略转发到Queue进行存储

RabbitMQ提供了四种Exchange：fanout,direct,topic,header

header模式在实际使用中较少，本文只对前三种模式进行比较。

性能排序：fanout > direct >> topic。比例大约为11：10：6

## 一.Direct Exchange

![r1.png](/static/uploads/images/20171103/6364532072265077778318349.png)

任何发送到Direct Exchange的消息都会被转发到RouteKey中指定的Queue。

1.一般情况可以使用rabbitMQ自带的Exchange：””(该Exchange的名字为空字符串，下文称其为default Exchange)。

2.这种模式下不需要将Exchange进行任何绑定(binding)操作

3.消息传递时需要一个“RouteKey”，可以简单的理解为要发送到的队列名字。

4.如果vhost中不存在RouteKey中指定的队列名，则该消息会被抛弃。

## 二.Fanout Exchange

![r2.png](/static/uploads/images/20171103/6364532073804675165663805.png)

任何发送到Fanout Exchange的消息都会被转发到与该Exchange绑定(Binding)的所有Queue上。

1.可以理解为路由表的模式

2.这种模式不需要RouteKey

3.这种模式需要提前将Exchange与Queue进行绑定，一个Exchange可以绑定多个Queue，一个Queue可以同多个Exchange进行绑定。

4.如果接受到消息的Exchange没有与任何Queue绑定，则消息会被抛弃。

## 三.Topic Exchange

![r3.png](/static/uploads/images/20171103/6364532074845526115373958.png)

任何发送到Topic Exchange的消息都会被转发到所有关心RouteKey中指定话题的Queue上

1.这种模式较为复杂，简单来说，就是每个队列都有其关心的主题，所有的消息都带有一个“标题”(RouteKey)，Exchange会将消息转发到所有关注主题能与RouteKey模糊匹配的队列。

2.这种模式需要RouteKey，也许要提前绑定Exchange与Queue。

3.在进行绑定时，要提供一个该队列关心的主题，如“#.log.#”表示该队列关心所有涉及log的消息(一个RouteKey为”MQ.log.error”的消息会被转发到该队列)。

4.“#”表示0个或若干个关键字，“*”表示一个关键字。如“log.*”能与“log.warn”匹配，无法与“log.warn.timeout”匹配；但是“log.#”能与上述两者匹配。

5.同样，如果Exchange没有发现能够与RouteKey匹配的Queue，则会抛弃此消息。

原文地址：http://www.gaort.com/index.php/archives/366