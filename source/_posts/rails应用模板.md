---
title: rails应用模板
date: 2019-06-24 16:15:03
tags:
---

### 背景
应用模板是包含 DSL 的 Ruby 文件，作用是为新建的或现有的 Rails 项目添加 gem 和初始化脚本等。
[模板指南连接](https://ruby-china.github.io/rails-guides/rails_application_templates.html)

<!--more-->

### 以下为api模板实例

>api-service-template(接口服务模板)
>模板将会生成的文件

```
Gemfile
002-master_key.rb
bigint_json.rb
assets.rb
/grape
zh-CN.yml
uid_service.rb
app_config.rb
.ruby-version
.ruby-gemset
README.md
```

### 执行方法：

```
$ rails new blog -m ~/template.rb
$ rails new blog -m http://example.com/template.rb
```

### template.rb 中的内容

```ruby
# Gemfile
file 'Gemfile', <<~CODE
  source 'https://gems.ruby-china.com'
  ruby '2.5.0'
  # Bundle edge Rails instead: gem 'rails', github: 'rails/rails'
  gem 'rails', '~> 5.2.0'
  # Use mysql as the database for Active Record
  gem 'mysql2', '>= 0.4.4', '< 0.6.0'
  # Use Puma as the app server
  gem 'puma', '~> 3.11'
  # Use SCSS for stylesheets
  gem 'sass-rails', '~> 5.0'
  # Use Uglifier as compressor for JavaScript assets
  gem 'uglifier', '>= 1.3.0'
  # See https://github.com/rails/execjs#readme for more supported runtimes
  # gem 'mini_racer', platforms: :ruby
  # Use CoffeeScript for .coffee assets and views
  gem 'coffee-rails', '~> 4.2'
  # Turbolinks makes navigating your web application faster. Read more: https://github.com/turbolinks/turbolinks
  gem 'turbolinks', '~> 5'
  # Build JSON APIs with ease. Read more: https://github.com/rails/jbuilder
  gem 'jbuilder', '~> 2.5'
  # Use Redis adapter to run Action Cable in production
  gem 'kaminari', '~> 0.16.3'
  gem "snowflake-rb", '~>0.0.2'
  gem 'rack-cors', :require => 'rack/cors'
  gem 'config', '~> 1.7.0'
CODE
# 创建grape文件
if yes?("是否想要安装gem grape并创建grape API 默认文件?")
  gem 'grape', '~> 1.0.2'
  gem 'grape-entity', '~> 0.7.1'
  gem 'grape-swagger', '~> 0.28.0'
  gem 'grape-swagger-entity', '~> 0.2.3'
  gem 'grape-swagger-representable', '~> 0.1.5'
  gem 'grape-swagger-ui', '~> 2.2.8'
  gem 'grape_logging', '~> 1.7.0'

  # routes
  route "mount API::Base => '/'"

  initializer "assets.rb" do
    <<~CODE
      Rails.application.config.assets.version = '1.0'
      Rails.application.config.assets.paths << Rails.root.join('node_modules')
      Rails.application.config.assets.precompile += %w( swagger_ui.js swagger_ui_screen.css swagger_ui_print.css )
    CODE
  end
  file 'app/grape/api/error.rb', <<~CODE
    module API
      class Error < Grape::Entity
        expose :code
        expose :message
      end
    end
  CODE
  file 'app/grape/api/api_error_handler.rb', <<~CODE
    # trap all exceptions and fail gracefuly with a 500 and a proper message
    module API
      class ApiErrorHandler < Grape::Middleware::Base
        def call!(env)
          @env = env
          begin
            @app.call(@env)
          rescue Exception => e
            throw :error, :message => e.message || options[:default_message], :status => 500
          end
        end
      end
    end
  CODE
  file 'app/grape/api/base.rb', <<~CODE
    module API
      class Base < Grape::API
        use GrapeLogging::Middleware::RequestLogger, logger: logger, log_level: 'debug'#, formatter: MyFormatter.new
        version 'v1', using: :path, vendor: 'InvoiceService'
        prefix :api
        format :json
        rescue_from ActiveRecord::RecordNotFound do  |e|
          server_error!(404, '数据未找到')
        end
        # rescue最低优先级
        rescue_from :all do |e|
          puts e.message
          puts e.backtrace
          server_error!(500, e.message)
        end
        formatter :json, lambda { |object, env|
          if object.class == Hash && object[:swagger] # 适配swagger输出
            data = object.to_json
          else
            data = { code: 0, msg: "success", :data => object }.to_json
          end
        }
        error_formatter :json, lambda { |message, backtrace, options, env, original_exception|
            { code: 500, msg: options }.to_json
        }
        helpers do
          def current_user
            @current_user ||= User.authorize!(env)
          end
          def authenticate!
            error!('401 Unauthorized', 401) unless current_user
          end
          def server_error!(code = 500, message = 'Server error.')
          # failure [[code, message, API::Error]]
            error!(message, code)
          end
        end
        # 配置完全局helper再mount api，否则helper不在api中生效
        # use API::ApiErrorHandler
        # 例如：mount API::Finance::Invoice
        add_swagger_documentation base_path: '/',
                                  title: "xxAPI",
                                  doc_version: '0.0.1',
                                  hide_documentation_path: true,
                                  hide_format: true
      end
    end
  CODE
end
gem_group :development, :test do
  # Call 'byebug' anywhere in the code to stop execution and get a debugger console
  gem 'byebug', platforms: [:mri, :mingw, :x64_mingw]
  gem 'pry-rails', '0.3.2'
  gem 'pry-byebug'
  gem 'capistrano-rails'
  gem 'capistrano-rvm', '~> 0.1.2'
  gem 'capistrano3-puma', '~> 1.2.1'
end
gem_group :development do
  # Access an interactive console on exception pages or by calling 'console' anywhere in the code.
  gem 'web-console', '>= 3.3.0'
  gem 'listen', '>= 3.0.5', '< 3.2'
  # Spring speeds up development by keeping your application running in the background. Read more: https://github.com/rails/spring
  gem 'spring'
  gem 'spring-watcher-listen', '~> 2.0.0'
end
gem_group :test do
  # Adds support for Capybara system testing and selenium driver
  gem 'capybara', '>= 2.15', '< 4.0'
  gem 'selenium-webdriver'
  # Easy installation and use of chromedriver to run system tests with Chrome
  gem 'chromedriver-helper'
end
if yes?("是否锁定ruby版本为2.5.0?")
  file '.ruby-version', "ruby-2.5.0"
end
file '.ruby-gemset', "#{self.app_path}"
# 初始化文件
file 'app/services/uid_service.rb', <<~CODE
  class UidService
    class << self
      attr_accessor :worker_id, :snowflake
      def next_id
        @snowflake ||= Snowflake::Rb.snowflake(@worker_id||get_worker_id, region_id)
        @snowflake.next_id
      end
      private
      def get_worker_id
        hostname = Socket.gethostname
        id = (Socket.gethostname[0].ord*100 + Socket.gethostname[1].ord)%512
        @worker_id = id
      end
      def region_id
        1
      end
    end
  end
CODE
file 'config/initializers/002-master_key.rb', <<~CODE
  ENV['RAILS_MASTER_KEY'] = ENV['payment_service_STAG_RAILS_MASTER_KEY']
CODE
file 'config/initializers/bigint_json.rb', <<~CODE
  # json输出bigint时转为字符串
  # 否则grape输出时以int导致数值错误
  # 且某些json解析也需要适配bigint
  class Integer
    def as_json(options = nil)
      self > 2147483647 ? self.to_s : self
    end
  end
CODE
file 'config/initializers/app_config.rb', <<~CODE
  Rails.application.config.assets.version = '1.0'
  Rails.application.config.time_zone = 'Beijing'
  # # The default locale is :en and all translations from config/locales/*.rb,yml are auto loaded.
  # # config.i18n.load_path += Dir[Rails.root.join('my', 'locales', '*.{rb,yml}').to_s]
  Rails.application.config.i18n.default_locale = 'zh-CN'
  # # Do not swallow errors in after_commit/after_rollback callbacks.
  # Rails.application.config.active_record.raise_in_transactional_callbacks = true
  # Cors支持， 兼容Grape
  Rails.application.config.middleware.use Rack::Cors do
    allow do
      origins "*"
      resource "*", headers: :any, methods: [:get, :post, :put, :delete, :options]
    end
  end
CODE
# 添加国际化文件
file 'config/locales/zh-CN.yml', <<~YAML
  zh-CN:
    grape:
      errors:
        format: ! '%{attributes} %{message}'
        messages:
          coerce: '无效'
          presence: '缺失'
          regexp: '无效'
          blank: '为空'
          values: '无效值'
          missing_vendor_option:
            problem: '缺少 :vendor 选项。'
            summary: '当版本号使用 Header 指定时，必须指定 :vendor 选项。'
            resolution: "如：version 'v1', using: :header, vendor: 'twitter'"
          missing_mime_type:
            problem: '缺少 %{new_format} 的 MIME 类型。'
            resolution:
              "你可以在 Grape::ContentTypes::CONTENT_TYPES 预置的 MIME 类型中选择一个，或者
              通过 content_type :%{new_format}, 'application/%{new_format}' 自定义一个
              "
          invalid_with_option_for_represent:
            problem: '你必须通过 :with 选项指定一个 entity 类。'
            resolution: '如：represent User, :with => Entity::User'
          missing_option: '你必须指定 :%{option} 选项。'
          invalid_formatter: '不能将类型 %{klass} 转换为 %{to_format}'
          invalid_versioner_option:
            problem: 'version 声明中 :using 选项的取值：%{strategy} 无法识别。'
            resolution: '有效的 :using 取值为 :path，:header 或 :param'
          unknown_validator: '未知的校验器: %{validator_type}'
          unknown_options: '未知选项：%{options}'
          unknown_parameter: '未知参数：%{param}'
          incompatible_option_values: '%{option1}: %{value1} 和 %{option2}: %{value2} 不兼容'
          mutual_exclusion: '是互斥的'
          at_least_one: '缺失, 至少需要提供一个参数'
          exactly_one: '缺失, 只能提供一个参数'
          all_or_none: '提供全部参数，或者都不提供'
          missing_group_type: '需要提够 group 类型'
          unsupported_group_type: 'group 类型只能是 Array 或 Hash'
          invalid_message_body:
            problem: "信息内容和指定的格式不相符"
            resolution:
              "当指定 %{body_format} 作为 content-type 时，你必须在请求的 'body' 中
              传递一个有效的 %{body_format}'
              "
          invalid_accept_header:
            problem: '无效的 Accept Header'
            resolution: '%{message}'
          invalid_version_header:
            problem: '无效的 version header'
            resolution: '%{message}'
    errors:
      messages:
        extension_white_list_error: '文件类型不允许'
    views:
      pagination:
        first: "&laquo;"
        last: "&raquo;"
        previous: "&lsaquo;"
        next: "&rsaquo;"
        truncate: "&hellip;"
    helpers:
      page_entries_info:
        one_page:
          display_entries:
            zero: "No %{entry_name} found"
            one: "Displaying <b>1</b> %{entry_name}"
            other: "Displaying <b>all %{count}</b> %{entry_name}"
        more_pages:
          display_entries: "Displaying %{entry_name} <b>%{first}&nbsp;-&nbsp;%{last}</b> of <b>%{total}</b> in total"
    flash:
      create:
        success: '创建成功'
      update:
        success: '更新成功'
    date:
      abbr_day_names:
      - 日
      - 一
      - 二
      - 三
      - 四
      - 五
      - 六
      abbr_month_names:
      -
      - 1月
      - 2月
      - 3月
      - 4月
      - 5月
      - 6月
      - 7月
      - 8月
      - 9月
      - 10月
      - 11月
      - 12月
      day_names:
      - 星期日
      - 星期一
      - 星期二
      - 星期三
      - 星期四
      - 星期五
      - 星期六
      formats:
        default: ! '%Y-%m-%d'
        long: ! '%Y年%b%d日'
        short: ! '%b%d日'
      month_names:
      -
      - 一月
      - 二月
      - 三月
      - 四月
      - 五月
      - 六月
      - 七月
      - 八月
      - 九月
      - 十月
      - 十一月
      - 十二月
      order:
      - :year
      - :month
      - :day
    datetime:
      distance_in_words:
        about_x_hours:
          one: 大约一小时
          other: 大约 %{count} 小时
        about_x_months:
          one: 大约一个月
          other: 大约 %{count} 个月
        about_x_years:
          one: 大约一年
          other: 大约 %{count} 年
        almost_x_years:
          one: 接近一年
          other: 接近 %{count} 年
        half_a_minute: 半分钟
        less_than_x_minutes:
          one: 不到一分钟
          other: 不到 %{count} 分钟
        less_than_x_seconds:
          one: 不到一秒
          other: 不到 %{count} 秒
        over_x_years:
          one: 一年多
          other: ! '%{count} 年多'
        x_days:
          one: 一天
          other: ! '%{count} 天'
        x_minutes:
          one: 一分钟
          other: ! '%{count} 分钟'
        x_months:
          one: 一个月
          other: ! '%{count} 个月'
        x_seconds:
          one: 一秒
          other: ! '%{count} 秒'
      prompts:
        day: 日
        hour: 时
        minute: 分
        month: 月
        second: 秒
        year: 年
    errors: &errors
      format: ! '%{attribute} %{message}'
      messages:
        accepted: 必须是可被接受的
        blank: 不能为空
        confirmation: 不一致
        empty: 不能留空
        equal_to: 必须等于 %{count}
        even: 必须为双数
        exclusion: 是保留关键字
        greater_than: 必须大于 %{count}
        greater_than_or_equal_to: 必须大于或等于 %{count}
        inclusion: 不包含于列表中
        invalid: 是无效的
        less_than: 必须小于 %{count}
        less_than_or_equal_to: 必须小于或等于 %{count}
        not_a_number: 不是数字
        not_an_integer: 必须是整数
        odd: 必须为单数
        record_invalid: ! '%{errors}'
        taken: 已经被使用
        too_long: 过长（最长为 %{count} 个字符）
        too_short: 过短（最短为 %{count} 个字符）
        wrong_length: 长度非法（必须为 %{count} 个字符）
        extension_white_list_error: 不允许的文件类型
      template:
        body: 如下字段出现错误：
        header:
          one: 有 1 个错误发生导致「%{model}」无法被保存。
          other: 有 %{count} 个错误发生导致「%{model}」无法被保存。
    helpers:
      select:
        prompt: 请选择
      submit:
        create: 新增%{model}
        submit: 保存%{model}
        update: 更新%{model}
    number:
      currency:
        format:
          delimiter: ! ','
          format: ! '%u %n'
          precision: 2
          separator: .
          significant: false
          strip_insignificant_zeros: false
          unit: CN¥
      format:
        delimiter: ! ','
        precision: 3
        separator: .
        significant: false
        strip_insignificant_zeros: false
      human:
        decimal_units:
          format: ! '%n %u'
          units:
            billion: 十亿
            million: 百万
            quadrillion: 千兆
            thousand: 千
            trillion: 兆
            unit: ''
        format:
          delimiter: ''
          precision: 1
          significant: false
          strip_insignificant_zeros: false
        storage_units:
          format: ! '%n %u'
          units:
            byte:
              one: Byte
              other: Bytes
            gb: GB
            kb: KB
            mb: MB
            tb: TB
      percentage:
        format:
          delimiter: ''
      precision:
        format:
          delimiter: ''
    support:
      array:
        last_word_connector: ! ', 和 '
        two_words_connector: ! ' 和 '
        words_connector: ! ', '
    time:
      am: 上午
      formats:
        default: ! '%Y年%b%d日 %H:%M:%S'
        full: ! '%Y年%b%d日 %A %H:%M:%S %Z'
        long: ! '%Y年%b%d日 %H:%M'
        short: ! '%b%d日 %H:%M'
      pm: 下午
    activerecord:
      models:
        invoice:
          status:
            canceled: 已取消
            closed: 关闭
            rejected: 审核不通过
            wait_audit: 等待审核
            passed: 审核通过
            invoiced: 已开票
          service_type:
            print: 印刷品
            service_fee: 服务费
          buyer_type:
            individual: 个人
            unit: 单位
        user_invoice_info:
          default:
            true: 是
            false: 否
      errors:
        <<: *errors
YAML
file 'README.md', <<~MD
  ### #{self.app_path}服务
  接口信息请下载本项目并在本地启动后查看 http://localhost:你的端口号/api/swagger#/
MD
```

