---
title: rails项目生成二维码
date: 2018-01-19 15:29:34
tags: [ruby, rails, 二维码]
---

### Rails 项目中将连接生成二维码
<!--more-->
#### 准备
>`Gemfile`中添加

```ruby
gem 'rqrcode_png'
```
```ruby
# 生成二维码
  def print_rqrcode
    html = ""
    require 'rqrcode'
    host = "https://www.baidu.com" # 链接地址
    qr = RQRCode::QRCode.new(host, :size => 10, :level => :h )
    html <<
    "#{qr.as_svg(offset: 0, color: '000',
                 shape_rendering: 'crispEdges',
                 module_size: 2)}"
   end
```