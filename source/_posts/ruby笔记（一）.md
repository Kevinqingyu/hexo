---
title: ruby笔记（一）
date: 2017-11-14 11:48:30
tags: [ruby, rails]
---


![](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1511244769207&di=97fe709abd32e51b037c5bf3d37b8e4b&imgtype=0&src=http%3A%2F%2Ftc.sinaimg.cn%2Fmaxwidth.2048%2Ftc.service.weibo.com%2Fp3_pstatp_com%2Fe59d73299cfdc86f63ef07248bad7bd3.jpg)
<!--more-->

记录一下自己在工作当中会用到 的一些比较好用的方法

### 常用方法

#### step

`ruby2.X` 中我们这么用
```
pry(main)> range = 1..10
=> 1..10
pry(main)> range.step(2) {|x| puts x}
1
3
5
7
9
=> 1..10
```
- 创建一个1-10差为2的等差数列，有时也可以这么用

```
pry(main)> 0.step(10, 2) {|x| puts x}
0
2
4
6
8
10
=> 0
```
和上面的方法是一样的作用。

#### eval
- 将字符串当做代码执行
```
pry(main)> str = "hello"
=> "hello"
pry(main)> eval "str + ' Fred'"
=> "hello Fred"
```

#### camelize & underscore
蛇形和驼峰的互换
- camelize将蛇形转换为驼峰
- underscore将驼峰转换为蛇形

```
pry(main)> 'user_mailer'.camelize
=> "UserMailer"
pry(main)> "UserMailer".underscore
=> "user_mailer"
```
采用正则转换

#### constantize
将驼峰转换为类

```
pry(main)> "UserMailer".constantize
=> UserMailer
pry(main)> "UserMailer".constantize.class
=> Class
```

如果想要转换的类不存在，将会抛出异常
```
pry(main)> "AserMailer".constantize.class
NameError: uninitialized constant AserMailer
```
#### reduce & inject
官方示例
```
pry(main)> (5..10).reduce(:+)
=> 45
pry(main)> (5..10).inject {|sum, n| sum + n }
=> 45
pry(main)> (5..10).reduce(1, :*)
=> 151200
pry(main)> (5..10).inject(1) {|product, n| product * n }
=> 151200
```
这两个方法在使用上看起来不太一样，但是在结果上是一致的。
#### map(&:)

```
(5..10).map(&:to_s)
=> ["5", "6", "7", "8", "9", "10"]
(5..10).map {|i| i.to_s }
=> ["5", "6", "7", "8", "9", "10"]
```
#### hash.invert
互换hash 键和值的位置

```
pry(main)> {"1"=>"hello", "2"=>"dear"}.invert
=> {"hello"=>"1", "dear"=>"2"}
```

#### array.zip
将两个一维数组转换为二维数组

```
pry(main)> %w(供应商 供应商1 供应商2).zip([1, 2, 3])
=> [["供应商", 1], ["供应商1", 2], ["供应商2", 3]]
pry(main)> %w(供应商 供应商1 供应商2).zip([1, 2])
=> [["供应商", 1], ["供应商1", 2], ["供应商2", nil]]
pry(main)> %w(供应商 供应商1).zip([1, 2, 3])
=> [["供应商", 1], ["供应商1", 2]]
```
重新生成的数组长度以第一个数组为准，后者缺少的值以`nil`代替，后者多余的会直接丢弃。


