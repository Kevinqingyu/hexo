---
title: Rails项目搭建rabbitmq消息中间件
date: 2019-01-18 14:01:27
tags: [ruby, rails, rabbitmq, sneakers, bunny]
---

### 背景
![](https://rails365.oss-cn-shenzhen.aliyuncs.com/uploads/photo/image/1111/2019/243dcedfb3893c477a585710f2918e86.jpg)
<!--more-->
### 起步
首先 安装 rabbitmq 到本机
Mac下 执行
```shell
brew install rabbitmq
```
启动
```shell
rabbitmq-server
```

默认可以访问http://localhost:15672/#/ web服务界面 会有默认的用户名guest和密码guest

如果想要使用其他用户 可以创建和修改并赋予权限

在本地项目启动 redis-server 启动 sidekiq 这里sidekiq 只是作为队列使用 有大量消息时 使用队列发送到mq

在生产消息的项目中安装gem  bunny 他是一个生产者可以帮你使用mq并创建信道和发送消息

在需要消费的项目中安装 sneaker 他是一个消费者 可以接收

rabbitmq的消费队列和exchange对应关系需在web 上手动操作
也可以在创建消费队列的时候配置

```ruby
from_queue 'q.message.service.user_message.create',
            durable: true,
            exchange: 'ex.message_service.user_message',
            exchange_type: 'topic',
            routing_key: 'message.create'
```

这里配置好队列于exchange的关系

关于bunny和sneakers 的使用[这里](https://ruby-china.org/topics/35230)有教程

### 配置
```yml
rabbitmq:
   amqp: 'amqp://staging:rabbitdev@localhost:5672'  # 用户名:密码@本地的服务地址
   vhost: 'msgbus.staging.yiqiyin.com' #  vhost 虚拟主机名称
   timeout_job_after: 180 # 3.minutes 超时时间
   daemonize: true   # 守护进程
```

Sneakers worker 有修改的话每次都要重新启动一次 sneakers 服务

项目下 执行
```shell
rake sneakers:run
```

### 总结
首先消息队列和中间件可以将很多不需要及时同步的操作在队列中执行，降低服务器等待时间和负载，并且微服务的模块保证自己模块做更简单的事情解耦，并且可以给对应的站点添加还是不添加此功能提供了很多的可选择性，比如注册成功邮件，激活邮箱邮件，发放优惠券等操作，
同时又要注意不要为了微服务而做微服务，还要考虑这样的架构如果在某一步消息丢失了或者执行失败了如何处理，sneakers有对应的异常处理机制，比如说有消息一直消费失败在重试一定次数后可以选择丢入dieline，等待后续查看失败原因等等。