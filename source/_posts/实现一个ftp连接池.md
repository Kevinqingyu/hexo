---
layout: ruby
title: 实现一个ftp连接池
date: 2018-09-29 12:14:29
tags: [ruby, FTP]
---

### 需求背景：
##### 项目中使用Net::FTP上传文件到服务器，有时候会出现一次性上传几十到几百不等的文件，每次上传一个文件都要 执行：
1.创建client

2.连接

3.登录

4.上传文件

5.关闭client
<!--more-->

### 目标：

##### 希望能像ftp软件类似 filezilla一样，当同时上传10个文件时，只登陆一次，上传完毕后在下线（或者被服务器踢掉）

### 想法：
 1.实现一个连接池

```ruby
require 'net/ftp'

class FtpPool
  include Singleton

  attr_accessor :connections

  semaphore = Mutex.new

  def initialize
    @connections = {}
  end

  def get_connection(ftp_server)
    key = "#{ftp_server.address}#{ftp_server.port}#{ftp_server.username}"

    semaphore.synchronize{
      client = @connections[key] || @connections[key] = Net::FTP.new
    }

    begin
      client.noop
    rescue Exception => e
      reconnect(ftp_server, client)
    end
    client
  end

  def reconnect(ftp_server, client)
    client.connect(ftp_server.address, ftp_server.port)
    client.login(ftp_server.username, ftp_server.password)
  end

end
```


这里假设传入一个 ftpserver 附带四个属性 分别是地址，端口，账号和密码；
用前三个属性简单的生成一个key测试一下

首先生成一个 ftp_pool

```bash
pry(main)> ftp_pool = FtpPool.instance
=> #<FtpPool:0x007fba931af310 @connections={}>
```

然后调用get_connection 从连接池中获取一个连接，如果没有就新建一个

```bash
ftp_pool.get_connection(ftp_server)
=> #<Net::FTP:0x007fa1502147a8
 @binary=true,
 @debug_mode=false,
 @last_response="200 OK\n",
 @last_response_code="200",
 @logged_in=true,
 @mon_count=0,
 @mon_mutex=#<Thread::Mutex:0x007fa150214708>,
 @mon_owner=nil,
 @open_timeout=nil,
 @passive=true,
 @read_timeout=60,
 @resume=false,
 @sock=#<Net::FTP::BufferedSocket io=#<TCPSocket:0x007fa1511cf918>>,
 @welcome="230 Logged on\n">
```
这样我们就获得了一个 Net::FTP 的 client
可以使用然后我们用这个client 发送文件到ftp服务器，如果发送失败通过抓取异常，
如果异常是未连接或者未登录，就重新执行连接和登录名令后在发送文件即可。
这样一个简单的 ftp_pool 就可以运行了