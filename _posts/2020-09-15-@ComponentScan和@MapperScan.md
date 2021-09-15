---
layout:     post
title:      @ComponentScan和@MapperScan
subtitle:   学习笔记-Spring注解
date:       2020-09-15
author:     Leeds
header-img: img/post-bg-miui6.jpg
catalog: 	  true
tags:
    - 学习资料
    - Spring注解
---

## springboot学习中关于@ComponentScan及 @MapperScan的一些总结



主程序加 **@componentScan** + dao层加@**Mapper** 访问不到

主程序加@**componentScan** + dao层加@**Component** 或者
@ **Component（“userDao”**）访问不到

主程序加@**componentScan** + dao层加@**Repository** 或者@**Repository（“userDao”）** 访问不到

主程序加上 @**MapperScan(“com.jay.dao”)** + dao层加@**Mapper** 访问 到

主程序加上 @**MapperScan(“com.jay.dao”)** + dao层加@**Component** 访问 到

主程序加上 @**MapperScan(“com.jay.dao”)** + dao层加@**Repository** 或者 @ **Repository(“userDao"）**访问到

总结 在**主程序只加**@**MapperScan(“com.jay.dao”) 即可** 自动扫描到dao层 并注入



1、项目中引入了，其他工程的mapper。BaseDao 有@Mapper注解。

![image-20210915155354554](D:\1studyword\xleeds.github.io\img-post\image-20210915155354554.png)



