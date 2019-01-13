---
layout: post
title:  maven-build插件之spring-boot-maven-plugin
excerpt: "maven虽然很流行，但是很多的问题常常无可避免，有句话说的好既然我们无法改变（当前如果是大神也可以重写一下maven的框架），就想办法适应它，发现就记录..."
categories: [java]
tags: [maven]
comments: true
---

[**上一篇**](https://silentself.github.io/articles/2018-11/maven-build-1)

##### 前言：上次说到的解决方案，（其实并不止这一种，但是我成功的就这一种）后来简单做了几次测试，发现了真正成功的原因是下面的标签

```xml
<executions>
    <execution>
        <goals>
            <goal>repackage</goal>
        </goals>
    </execution>
</executions>
```

引入当前标签，package之后会在targe下出现俩包，其中jar文件

摘自：[Spring cloud的Maven插件（一）：repackage目标](http://www.cnblogs.com/zhouqinxiong/p/repackage.html)