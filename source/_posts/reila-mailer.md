---
title: rails-mailer
date: 2017-08-31 11:11:38
tags:
---

>app/mailer 文件夹创建mailer.rb文件

<!--more-->
```ruby
class UserMailer < ActionMailer::Base
  default :from => "测试邮件 <此处填写发件邮箱>"
  def send_email(email, subject)
     mail(to: email, subject: subject)
  end
end
```

>config/intializers 创建setup_mail.rb

发送邮件服务器的配置

```
# -*- encoding : utf-8 -*-

ActionMailer::Base.smtp_settings = {
    :address              => Settings.email.address,
    :port                 => Settings.email.port,
    :domain               => Settings.email.domain,
    :user_name            => Settings.email.user_name,
    :password             => Settings.email.password,
    :authentication       => Settings.email.authentication,
    :ssl => true,
    :enable_starttls_auto => true
}

ActionMailer::Base.default_url_options = { host: Setting.host }

# development
# ActionMailer::Base.default_url_options[:host] = "localhost:3000"
# ActionMailer::Base.delivery_method = :letter_opener
```

>settings.yml 添加配置

```
email:
    address: "smtp.qq.com"
    port: 25
    domain: "sinopr.org"
    user_name: "登录名"
    password: "密码"
    authentication: "login"
    host: '路径'
    mail_from: 'xxx@qq.com'
```

如果是smtp 服务要保证 邮箱开启了此类服务，部分邮箱开启此类服务后 会给予授权码，如果有授权码，在password 后要填写此授权码 。

> app/user_mailer 下创建 send_email.html.erb, 里面写入想要发送的内容。
