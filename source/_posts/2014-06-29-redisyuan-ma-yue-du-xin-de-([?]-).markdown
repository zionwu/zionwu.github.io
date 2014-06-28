---
layout: post
title: "redis源码阅读心得（一）"
date: 2014-06-28 23:51
comments: true
categories: 
---

###前言
最近这一个月的业余时间阅读了redis 2.8.9版本的源代码。之所以会阅读源代码，一开始是在去年的时候看到了[@huangz](http://weibo.com/huangz1990)在网上发布的书[redis设计与实现旧版](http://origin.redisbook.com/en/latest/)。huangz是一个和我同龄的人，但却已经通读了redis代码，并且写出了一本那么优秀的书以及翻译了众多[文章](https://redis.readthedocs.org/en/latest/)，让我受到很大震动。那时就开始参照他的书阅读redis代码，无奈当时只是将底层数据结构的代码读完后就不了了之了。今年5月，决定重新开始，每晚抽出时间阅读，并且加上中文注释上传到github的[repos](https://github.com/zionwu/redis-2.8.9-annotation)，以此来记录并鞭策自己。    

阅读redisd代码的过程充满了乐趣。印象最深刻便是我在深夜时发现了一个bug，提出了issue和[pull request](https://github.com/antirez/redis/pull/1788), 之后就兴奋得睡不着觉。 而最终成功merge到redis的[repos](https://github.com/antirez/redis)时，十分高兴，这是自己第一次对开源项目的贡献。想到以后redis的新版本，包含了我的小小的贡献，被广泛使用，就很有成就感。难怪那么多人痴迷于开源软件了！      


虽然已经有了huangz的书了，但是他书中是对redis的设计和原理进行客观的剖析。而我这个系列的文章，是从我的角度，分析和评价redis代码，论述认为值得学习的设计或者代码技巧，同时也对不合理的地方进行评价。
