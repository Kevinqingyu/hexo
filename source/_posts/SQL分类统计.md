---
title: SQL分类统计
date: 2018-11-21 11:21:07
tags: [SQL, ruby, rails]
---

## 用SQL分类统计来避免循环

>假设有一张表 users, 其中category字段表示用户的类型，假设 99 代表管理员，1代表门店，2代表工厂，当我们要统计 每个类型的人数的时候可以这样统计

```SQL
select
  sum(case when category=99 then 1 end) as 'admin',
  sum(case when category=1 then 1 end) as 'formal',
  sum(case when category=2 then 1 end) as 'factory'
from users
```

>如果不使用分类统计的话可能要将用户查询出来进行遍历叠加的方式算出每种类型的人有多少个


```SQL
admin   formal   factory
8        1846     28
```

>用分类统计一条SQL 就可以统计出结果。