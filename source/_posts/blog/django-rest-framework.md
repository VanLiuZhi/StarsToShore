---
title: Django 整合 rest-framework
date: 2019-02-26 14:45:00
tags: [python, web, technology]
categories: technology 技术
---

Django 整合 rest-framework

<!-- more -->

## viewsets

通过route注册视图后，可以实现自定义接口名称的路由访问，常规的做法是对一url使用一个视图，然后视图实现get，post方法，使用viewsets后，就可以通过url+视图方法名设计接口，每个方法名就是一个接口。