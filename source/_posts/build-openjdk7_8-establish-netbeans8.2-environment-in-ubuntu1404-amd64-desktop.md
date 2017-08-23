---
title: build-jdk8-jdk7-establish-netbeans-environment
date: 2017-08-22 10:28:52
tags: build
categories: jdk8
---

Ubuntu1404-desktop-amd64 编译 open-jdk8 open-jdk7
在Ubuntu， netbeans8.2搭建openjdk7 的开发环境

<!-- more -->
>我在macos编译就没成功过，因为macos的gcc编译器是依赖于Xcode的，而Xcode在5.X以后就已经不支持gcc了而改为clang了。按照googke上的很多帖子试过，总是有无法解决致一系列莫名奇妙的问题发生
>最后只能退而就其次，在macos安装Ubuntu虚拟机，进行编译。刚开始编译，自己也是一脸懵逼，一步一步google过来，期间碰到了十几个容易出错的问题，记录一下，给自己后面做参考。
>我的笔记是记录在有道云笔记上的，因此这里挂出有道云笔记的链接
> <a href="http://note.youdao.com/noteshare?id=abba4c3dc2340b51e065fb56757dac0a"> Ubuntu1404-desktop-amd64 编译 JDk7/8 搭建开发环境过程 </a>
> 还是有挺多问题不明白，比如编译选项中的 fastdebug debug 和 product的区别，以及众多环境变量对编译产生的影响，编译出的各部件的连接关系，jdk 和hotspot的关系等等都有待去探究
> 另外很蛋疼的问题就是，每次编译都要花费半个小时！！！！！！！！！！！！！！！！！，感觉用虚拟机的小伙伴需要把内存和核数都给多一点。。。
