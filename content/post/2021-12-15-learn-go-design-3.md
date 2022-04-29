---
title: Go设计篇-3-设计实现最小化协程池
date: '2021-12-15'
slug: learn-go-design-3
categories:
  - programing
tags:
  - go
---
基于channel+select的方案，实现WorkPool的三个功能：
- pool 的创建与销毁；
- pool 中 worker（Goroutine）的管理；
- task 的提交与调度；