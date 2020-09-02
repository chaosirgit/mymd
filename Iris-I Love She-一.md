---
title: Iris I Love She(一)
tags:
  - go
  - iris
keywords:
  - go
  - iris
categories:
  - go
abbrlink: 7f9f2030
date: 2020-03-01 14:17:47
---

## 前言

​	GO 语言风靡全球，国外如 Google、AWS、Cloudflare、CoreOS 等，国内如七牛、阿里等都已经开始大规模使用 Golang 开发其云计算相关产品。 在学习 GO 的过程中，我也深深的被 GO 折服。且 GO 语言原生支持高并发，性能强劲，如果熟练的话，开发效率不差于 Python 等语言。

​	在疫情肆虐的今天，终于可以静下心来认真体验一下 GO 的魅力了。

​	由于目前业务主要为 Web 应用，所以首先学习一种 Web 框架以便快速提供生产力。

## 框架的选择

Google 了一下 GO 的 Web 框架，有诸如：

* Beego：[开源的高性能 Go 语言 Web 框架](https://github.com/astaxie/beego)。
* Buffalo：[使用 Go 语言快速构建 Web 应用](https://github.com/gobuffalo/buffalo)。
* Echo：[简约的高性能 Go 语言 Web 框架](https://github.com/labstack/echo)。
* Gin：[Go 语言编写的 Web 框架，以更好的性能实现类似 Martini 框架的 Api](https://github.com/gin-gonic/gin)。
* Iris：[全宇宙最快的 Go 语言 Web 框架。完备 MVC 支持，未来尽在掌握](https://github.com/kataras/iris)。
* Revel：[Go 语言的高效、全栈 Web 框架](https://github.com/revel/revel)。

其中，我一眼就相中了 Iris 不只是因为她号称全宇宙最快、完备支持 MVC 吸引了我，更因为她的名字： **爱丽丝**

## 关于性能

这里读了一篇文章 [《Go、Nginx、Php、Nodejs谁能胜出紫禁之巅》](https://cloud.tencent.com/developer/article/1408523)。

看来用 Iris 框架搭建 Web 应用果然可以满足需求。:)

