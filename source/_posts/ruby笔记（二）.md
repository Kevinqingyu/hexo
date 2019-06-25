---
title: ruby笔记（二）
date: 2017-11-20 11:20:36
tags: [ruby, rails]
---


![](https://ss3.bdstatic.com/70cFv8Sh_Q1YnxGkpoWK1HF6hhy/it/u=3942457143,1284096136&fm=27&gp=0.jpg)
<!--more-->

最近有朋友有问我`ActiveRecord`中 `scope` 和 `validate` 方法的实现机制，之前一直在使用它但是我还真的没有细细的了解过这个方法，于是决定深入探究一下。

#### scope(name, scope_options = {}) public

>添加一个用于检索和查询对象的类方法。

可以这样使用：

```ruby
class Shirt < ActiveRecord::Base
  scope :red, where(:color => 'red')
  scope :end_date, ->(date) { where(:end_date => date) }
  scope :dry_clean_only, joins(:washing_instructions).where('washing_instructions.dry_clean_only = ?', true)
end
```

>上面的调用`scope`定义了类方法`Shirt.red`和`Shirt.dry_clean_only`。 `Shirt.red`实际上代表查询`Shirt.where(:color =>'red')`。
请注意，这只是用于定义实际类方法的`语法糖`

实现的作用和类方法一样，但是类方法是在载入类的时候就会一起加载，而`scope` 定义的方法是在方法调用时才会加载
```ruby
class Shirt < ActiveRecord::Base
  def self.red
    where(color: 'red')
  end
end
```
不过需要注意的是，`scope`方法即使在什么也没查到的情况下依然会返回`Relation`对象，也就是说 `scope` 方法可以进行链式调用而不担心会抛出`nil:NilClass` 异常。

```ruby
class Article < ActiveRecord::Base
  scope :published, -> { where(published: true) }
  scope :featured, -> { where(featured: true) }

  def self.latest_article
    order('published_at desc').first
  end

  def self.titles
    pluck(:title)
  end
end
```

我们可以这样调用方法：
```bush
Article.published.featured.latest_article
Article.featured.titles
```

下面看一下具体的实现源码：

```ruby
# File activerecord/lib/active_record/scoping/named.rb, line 141
def scope(name, body, &block)
  unless body.respond_to?(:call)
    raise ArgumentError, 'The scope body needs to be callable.'
  end

  if dangerous_class_method?(name)
    raise ArgumentError, "You tried to define a scope named \"#{name}\" "                "on the model \"#{self.name}\", but Active Record already defined "                "a class method with the same name."
  end

  extension = Module.new(&block) if block

  singleton_class.send(:define_method, name) do |*args|
    scope = all.scoping { body.call(*args) }
    scope = scope.extending(extension) if extension

    scope || all
  end
end
```

除去数据验证以外，值得关注的方法就是
```ruby
singleton_class.send(:define_method, name)
```
这一句实现了将传入的方法名定义成一个类方法的过程，我们试着简易的实现以下`scope` 方法的原理

```ruby
def self.scope(name, body)
  singleton_class.send(:define_method, name, &body)
end
```

参数接受名字和代码块，然后将名字定义为方法名，代码块作为方法内部可执行代码。这样我们就简易的实现了`scope`这个方法。

这时候scope 的内部实现机制就算比较了解了，但是我对这个`singleton_class` 非常好奇，它是如何实现的呢？

#### singleton_class → class

>返回obj的单例类。 如果obj没有，则此方法创建一个新的单例类。
如果obj为nil，true或false，则分别返回NilClass，TrueClass或FalseClass。 如果obj是一个Fixnum或一个符号，它会引发一个TypeError。

我们可以这么用它：
```bush
Object.new.singleton_class  #=> #<Class:#<Object:0xb7ce1e24>>
String.singleton_class      #=> #<Class:String>
nil.singleton_class         #=> NilClass
```

那么`scope` 就算是告一段落，算是浅尝辄止，下次我们要仔细的研究一下`singleton_class`，下一步看看`validate`的实现机制。

#### validate(*methods, &block) public

>向类中添加验证方法或块。 当重写#validate实例方法变得过于强硬时，这是很有用的，而且您正在寻找更多关于验证的描述性声明。
这可以通过一个指向方法的符号来完成：

这个非常常用的校验方法，我们经常这样用：

```ruby
class Comment < ActiveRecord::Base
  validate :must_be_friends

  def must_be_friends
    errors.add_to_base("Must be friends to leave a comment") unless commenter.friend_of?(commentee)
  end
end
```

或者用一个传递当前记录的块进行验证：
```ruby
class Comment < ActiveRecord::Base
  validate do |comment|
    comment.must_be_friends
  end

  def must_be_friends
    errors.add_to_base("Must be friends to leave a comment") unless commenter.friend_of?(commentee)
  end
end
```
看得出这里 参数接受方法或者代码块，然后定义了一个实例方法，当实例不满足校验条件时，将会抛出异常。