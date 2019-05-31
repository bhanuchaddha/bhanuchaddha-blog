---
title: "Spring Boot Devtools on Intellij"
date: 2019-05-31T01:11:42+02:00
author: "Bhanu Chaddha"
linktitle: Spring Boot Devtools with Intellij
next: /tutorials/github-pages-blog
prev: /tutorials/automated-deployments
title: Spring Boot Devtools with Intellij
weight: 10
authorAvatar: image/logo.svg
image: image/devtool.png
tags : [
    "backend",
    "java",
    "intellij",
    "springboot",
    "development",
]
categories : [
    "blog"
]
draft: true
---

Auto reloading feature does not work out of the box in Intellij. Below are the changes required to make it work correctly.

Spring Boot provide niche feature of hot reloading for a Spring Boot application.

It recompile your java application as soon as you make a change. But most important aspect is that it only recompile only those files in which changes are made. Thus process take quite a short time compared to complete application build. You can read more about DevTools here.

But this feature does not work out of the box in IntelliJ by just adding required dependancy in your application.

Below are all the steps required to make auto reload work in IntelliJ.

## Add Spring DevTools dependency
```
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-devtools</artifactId>
   <scope>runtime</scope>
</dependency>
```

## Changes in IntelliJ build configuration

Go to Preference -> build,execution,deployment. -> Compiler.

And Enable __build project automatically__

![Build Project Automatically](/image/buidProjectAutomatically.png)

## Changes in IntelliJ registory
Go to Cmd+Shift +A (Mac) or Ctrl+ Shift+A (Windows) -> Registry

Enable __compiler.automake.allow.when.app.running__

![Changes in IntelliJ registory](/image/registory.png)

Thats it. Enjoy the faster development!!!