---
layout:     post
title:      "猎聘招聘网站数据抓取（爬虫案例）"
---

## 简介
![web scraper](/img/posts/scraper/logo-liepin.png)
爬虫软件 - 简单来说就是可以一个可以自动抓取网站特定内容的一种工具。其最核心的内容是把网站结构数据化，使我们更加容易的筛选和提取内容。如果你对如何将网站内容数据化感到兴趣的话。强烈建议阅读我在Github上写的另一个小程序 - [Domparser](https://github.com/BranLiang/domparser). 但是今天的主题是爬虫的操作，因此如何数据化网站内容就不做过多的介绍了。需要指出的一点事，由于每个网站都有其独一无二的结构系统，因此对于每个进行内容抓取都需要进行单独案例剖析分解，因此暂时还没有诞生一个全能的网站傻瓜爬虫软件，每一次的抓取都是一次对网站结构的研究。可以说，每一次的抓取实验都是独一无二的。我们这次也不例外。

首先确定一下这个程序最终需要达到的效果： 我希望能够抓取猎聘网所有相关搜索的招聘标题，公司，薪水等相关内容。并且最终以表格的形式整理出来。
[Github](https://github.com/BranLiang/assignment_web_scraper)
如果你只想使用这个工具，直接进入我的Github点击然后拿走即可。

## 概述
整理思路及所需工具概述

 - [Mechanize](https://github.com/sparklemotion/mechanize) 工具包: 这是本次爬虫实验的核心工具， 主要用来结构化网站数据。
 - CSV 组件： 用来对最终结果进行写入。
 - Chrome 开发组件： 用于网站结构分析，并辅助其后的css selector

## 开始

首先，插入所需的mechanize 和 csv的Ruby组件。下面绝大部分的方程都来自于此。

```ruby
require 'mechanize'
require 'csv'
```

接下来，定义你最终数据的结构形式。

```ruby
Job = Struct.new :title, :company, :salary, :place, :description
```

这里我把接下的网站地址保存在了一个变量名上。为什么这里我没有用`https://www.liepin.com/`，而是再此基础上添加了`zhaopin`，其实两者都可以，区别仅仅在于返回的`form`数量不同，如果只是使用`https://www.liepin.com/`，你还需要筛选掉登录表格。

```ruby
search_page = 'https://www.liepin.com/zhaopin/'
```

这里我新建了一个`agent`，它可以在之后读取网站。同时还给他添加了抬头`Windows Chrome`，为了防止不必要的被网站禁止，必要的延迟必须添加，否则网站会视你为DDos之类的攻击，这里我给他添加的延迟是每个`request`之间添加0.5秒的间隔。

```ruby
agent = Mechanize.new do |agent|
  agent.user_agent_alias = 'Windows Chrome'
  agent.history_added = Proc.new { sleep 0.5 }
end
```

这个新变量使用来存储结果的

```ruby
found = []
```

抓取第一步必然是读取网页

```ruby
agent.get(search_page) do |page|
end
```

抓取之后，紧接着就应该是内容搜索，随机抓取内容而不进行筛选是没有任何意义的。这里的`key`和`dqs`来自网站的`form`表格.

```ruby
results = page.form do |search|
  search.key = '前端' # Key words
  search.dqs = '010' # City Beijing
end.submit
```

经过上一步的搜索之后，就能够看到我们所需的部分结果了，但是很明显页面只显示部分，还有绝大部分结果在第二页，第三页等等的后面，因此自然而然的一个循环的自动点击系统就显得非常有必要。这段代码非常长，但很多内容都是重复的，除去重复部分，这段代码所做的就是，循环所有页面，对每个页面的招聘链接进行点击，进入详细招聘页面后，使用css selector定位内容所在位置。对内容进行初步处理之后，存储进相应的节点。

```ruby
link_number = 2
loop do
  lien = results.link_with(:text=> link_number.to_s)
  link_number += 1

  results.links_with(:href => /job.liepin.com\/(\d){3}_(\d){7}/ ).each do |link|
    current_job = Job.new
    # Go to the job describtion page
    description_page = link.click

    description_page.search("div.title-info h1").each do |node|
      current_job.title = node.text.strip
    end

    description_page.search("div.title-info h3 a").each do |node|
      current_job.company = node.text.strip
    end

    description_page.search("div.job-item p.job-item-title").each do |node|
      current_job.salary = node.text.gsub(/\s+/, "")
    end

    description_page.search("div.job-item p.basic-infor span a").each do |node|
      current_job.place = node.text.strip
    end

    description_page.search("div.job-item.main-message div.content.content-word").each do |node|
      current_job.description = node.text.strip
    end
    found << current_job
  end

  break unless lien
  results = lien.click
end
```

最后，把所得的结果存储为相应的CSV结构即可。

```ruby
CSV.open("results_qianduan.csv", "w+") do |csv_file|
  found.each do |row|
    csv_file << row
  end
end
```

## 结果展示
如果一切顺利的话，你应该能够得到我Github上面的csv文件。
