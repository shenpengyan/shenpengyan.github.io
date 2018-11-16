---
layout:     post
title:      "Java gc分析神器:gceasy"
subtitle:   "analyse-gc-log"
date:       2018-11-16 12:00:00
author:     "沈歌"
header-img: "img/home-bg-o.jpg"
tags:
        - java
        - gc
---


分析gc日志是高阶java工程师的必备技能，之前一直听别人说要分析gc日志，分析gc日志，但是真正上手分析，看着gc日志又无从下手。

直到，知道了有这样的一个工具，
[http://gceasy.io/](http://gceasy.io/)

真的是一个神器，把gc日志输入，就能出gc分析报表。

示例：这是线上一个有gc问题的工程跑几天后的gc分析结果：

[http://gceasy.io/my-gc-report.jsp?p=c2hhcmVkLzIwMTgvMTEvMTYvLS1nYy5sb2ctLTEyLTItMzg=](http://gceasy.io/my-gc-report.jsp?p=c2hhcmVkLzIwMTgvMTEvMTYvLS1nYy5sb2ctLTEyLTItMzg=)

关注重点：

1. 有没有连续gc
2. 有没有长的停顿
3. gc时间占比多大，一般要低于2%


看图：
![image](https://shenpengyan.github.io/img/in-post/janalyse-gc-log/1.png)

全占了,GC时间占了5.83%, 太高了。

其他各部分自己点击链接查看吧，确实蛮有用的。

