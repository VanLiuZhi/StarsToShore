---
title: Maven
date: 2019-04-05 00:00:00
tags: [java, note]
categories: Java
---

Maven的使用与学习总结

<!-- more -->

## base

POM 项目对象模型

构建生命周期：包含三个标准生命周期，clean default site，每个生命周期都包含若干个阶段，执行某个阶段的时候，该阶段之前的其它阶段都会被执行

仓库：在 Maven 中，任何一个依赖、插件或者项目构建的输出，都可以称之为构件。
仓库安装本地，中央，远程的顺序来查找，中央是官方仓库，远程是用户自定义远程仓库

插件：生命周期的执行都是由对应插件来完成的，可以配置相关插件的具体操作

项目模板，项目文档

快照：通过快照可以固化某个pom配置

自动化构建：构建流程控制，比如依赖的项目构建完成，才开始构建本项目

依赖管理

自动构建，通过对应的插件Release实现自动构建

## 打包

通过IDEA就可以打包了，前提是要配置好pom文件，项目是maven工程

目前已知打包会扫描测试部分的代码，测试不通过也不行。打包后得到jar包，用`java -jar api-1.0.jar`命名就可以运行，`api-1.0.jar.original`后缀的文件是给其它项目依赖用的(暂时不管，这个不能用来部署)

## 导入包到当前仓库

mvn install:install-file -DgroupId=com.ctg.itrdc.cache -DartifactId=ctg-cache-nclient -Dversion=2.4.0 -Dpackaging=jar -Dfile=/Users/liuzhi/Downloads/ctg-cache-nclient-2.4.0.jar

修改对应的参数