---
layout:     post
title:      "Java开发团队管理细则"
subtitle:   "Java development team management rules"
date:       2018-09-26 12:00:00
author:     "沈歌"
header-img: "img/home-bg-o.jpg"
tags:
        - 软件工程
---


> 软件开发是团队协作，多人开发很容易造成协调问题，因此，做一些必要的开发规范，有助于帮助新员工成长，也有助于提高开发效率，防止各种问题影响开发进度。

#### 1. 代码规范

建议每位java开发人员都读一下[《阿里巴巴Java开发手册》](https://github.com/alibaba/p3c/blob/master/%E9%98%BF%E9%87%8C%E5%B7%B4%E5%B7%B4Java%E5%BC%80%E5%8F%91%E6%89%8B%E5%86%8C%EF%BC%88%E8%AF%A6%E5%B0%BD%E7%89%88%EF%BC%89.pdf)
阿里作为中国最大规模使用Java的公司，也是Java技术实力最强的公司。这个手册在业界影响很大，已经成为了很多团队的开发标准，更加方便的是，开发了IntelliJ Idea插件，使用方式见官方说明文档：<https://github.com/alibaba/p3c/blob/master/idea-plugin/README_cn.md> 可以在写代码时实时对常见的代码书写错误或者可能留坑的地方进行提示，非常有用。

如图：插件利用Inspections设置了很多规则进行检验，包含的都是代码规范，如果有错误或者不规范的地方，会标出来，有些还会给出修正建议，非常方便。
![](https://shenpengyan.github.io/img/in-post/java-team-coding-rule-1/1.png)

扫描生产环境一个老项目，结果如下，注意这些问题，有助于提高员工能力。
![](https://shenpengyan.github.io/img/in-post/java-team-coding-rule-1/2.png)

PS：推荐FindBugs-IDEA，能够帮助我们找出一些代码中的潜在问题，建议配合Alibaba Java Coding Guidelines一起使用。

#### 2. 项目行结束符统一

当一个开发同一个项目的开发人员，有些使用mac/lunix, 有些使用windows时，很容易因为行结束符的不同导致做code review比较diff的时候出现整个文件不一样的情形，其实不是文本不一样，只是行结束符不一样。
windows的行结束符是CRLF(\r\n), 而Unix and OS X的是LF(\n)
因此，最好将行结束符统一设为LF。

设置方式：intellij idea → file → setting → code style → line separator 设为Unix and OS X(\n)
![](https://shenpengyan.github.io/img/in-post/java-team-coding-rule-1/3.png)

然后开启本地行结束符提示，当有文件行结束符与设置不一致时，文件上边缘会出现提示，并支持一键修复，如果是老代码进行修改，也有对整个project进行扫描，并一键全部替换，非常方便。
![](https://shenpengyan.github.io/img/in-post/java-team-coding-rule-1/4.png)

#### 3. code style

大括号应不应该换行，== 两边应不应该空格，一行代码最长写多少，这些都是代码格式规范，
在 intellij idea → file → setting → code style 中都有设置，同一个项目甚至同一个公司的开发人员，最好都使用同一份模板，保证大家代码的一致性，在写完一段代码后，执行Reformat Code(windows下Crtl + Alt + L), 可以一键将不符合格式规范的代码进行修正。

如果公司没有统一的规范的话，建议使用Google的规范，https://github.com/google/styleguide/blob/gh-pages/intellij-java-google-style.xml

#### 4. git使用规范

多人开发，很容易出现merge conflict，一般来说都有test分支和master分支，在每次合test分支前，先merge master，保证当前分支与master同步，避免在test分支合自己分支时，出现由于自己代码分支版本落后而导致不应出现的conflict。

如果可以，建议使用gitflow框架，条理清楚，操作方便。