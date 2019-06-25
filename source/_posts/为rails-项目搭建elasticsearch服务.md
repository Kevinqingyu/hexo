---
title: 为rails 项目搭建elasticsearch服务
date: 2017-09-06 22:11:57
tags: [elasticsearch, ruby, rails]
---

## 为 rails 本地项目搭建  elasticsearch  服务

### 首先安装  elasticsearch  服务
OSX  系统
```bash
brew install elasticsearch@2.4
brew services start elasticsearch
```
<!--more-->
测试服务是否启动
浏览器输入 localhost:9200
```
{
  "name" : "Lynx",
  "cluster_name" : "elasticsearch_marin",
  "cluster_uuid" : "acE95aJmQxuMz0cx47b2WQ",
  "version" : {
    "number" : "2.4.6",
    "build_hash" : "5376dca9f70f3abef96a77f4bb22720ace8240fd",
    "build_timestamp" : "2017-07-18T12:17:44Z",
    "build_snapshot" : false,
    "lucene_version" : "5.5.4"
  },
  "tagline" : "You Know, for Search"
}
```
出现类似上述信息 number 表示当前  elasticsearch  的版本号，需要注意的是  elasticsearch  现在分为 v2+ 和 v5+  两个版本，要根据自己的版本来选择  searchkick  对应的版本是否合适

### gemfile中  引用  searchkick
这里我们是使用了 1.3.3 版本的
```ruby
gem 'searchkick', '1.3.3'
```
### model 中引用searchkick

现在我们已经有搜索服务了，现在要配置需要搜索的  model
在  model  中引用  searchkick

```ruby
# 全文检索  searchkick
searchkick
```
给  products  表重建索引
```ruby
Product.reindex
```
进行搜索
```ruby
products = Product.search "apples"
```
这时就会得到结果集。
如果是简单的应用到这里就可以满足要求 ，当然我们有时候需要一些个性化的配置。
### 给部分字段建立索引
reindex 方法会默认给所有的字段建立索引，但是由于字段过长，或者性能原因我们只需要部分字段有索引 可以这样：
```ruby
def search_data
  { name: name }
end
```
重写  search_data 方法加入name 这样就只给  name  字段打索引了

### 关联表建索引
```ruby
class Catalog < ActiveRecord::Base
  has_many :products
end
```
```ruby
class Product < ActiveRecord::Base
  belongs_to :catalog
end
```
```ruby
def search_data
  { name: name }.merge{ catalog_name: catalog.name }
end
```
这里 我们给  product  表添加一个索引叫做 分类名称  catalog_name
这样搜索分类名称就可以搜索出 同一个分类的 商品列表了

### 指定查询字段

```ruby
Product.search key
```

这个方法会默认搜索 所有的字段 并返回 所有包含  key  的结果集。
如果我们想搜索指定的字段该如何设置呢？

```ruby
def self.elasticsearch(params = {}, options = {})
  key = params[:key].blank? ? "*" : params[:key] # 关键字
  params[:page] ||= 1 # 分页
  params[:per_page] ||= 20 # 每页条数
  where_hash = {
    status: 'success', # 固定筛选值
  }
  conditions = {where: where_hash}
  conditions[:page] = params[:page]
  conditions[:per_page] = params[:per_page]
  search key, conditions
end
```
这样 就相当于给结果集添加  scope，和分页效果，前端配合  kaminari  就可以实现分页效果了
```ruby
@articles = Product.elasticsearch(params)
```
action  中将设定好的参数传入就可以得到结果集了

 QWQ~!!
