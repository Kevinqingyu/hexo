---
title: rails导出电子表格
date: 2018-01-10 15:05:24
tags: [ruby, rails, excel]
---

#### 如何在`rails`应用中导出电子表格

![](https://www.edx.org/sites/default/files/course/image/promoted/analyzing_and_visualizing_data_with_excel_378x225.jpg)
<!--more-->
在 `Gemfile` 中添加
```ruby
gem 'spreadsheet'
```

```bush
bundle install
```

假设我们要导出 项目数据：

`controller` 中添加导出：
```ruby
def exprot
  projects = Project.limit(10)
  send_data(Project.exprot_projects(projects), :type => "text/excel;charset=utf-8; header=present", :filename => "项目统计表.xls")
end
```
`model` 中添加想要导出的数据格式
```ruby
def self.exprot_pojects(projects)
  xls_report = StringIO.new
  Spreadsheet.client_encoding = "UTF-8"
  book = Spreadsheet::Workbook.new
  style0 = Spreadsheet::Format.new :weight => :bold, :size => 16, :align => :center
  style = Spreadsheet::Format.new :weight => :bold, :size => 10
  sheet1 = book.create_worksheet :name => "项目统计表.xls"
  # 设置单元格高度
  sheet1.row(0).height = 18
  sheet1.row(0).default_format = style
  # 设置单元格宽度
  sheet1.column(0).width = 15
  sheet1.column(1).width = 15
  sheet1.row(0).concat ["项目名称","项目编号"]

  projects.each_with_index do |project, index|
    sheet1.row(index+1).concat [project.name, project.number]
  end

  book.write xls_report
  xls_report.string
end
```

这样在controller中调用此方法，或者传入想要导出的集合就可以导出了

