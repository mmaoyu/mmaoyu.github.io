---
layout: post
title:  单元测试怎么写
categories:
- unit test
tags:
- 编码
--- 

**为什么要写单元测试**  
  >引用coolshell[单元测试要做多细](https://coolshell.cn/articles/8209.html)的一段话:  
  
  >1.开发过程中，单元测试应该来测试那些可能会出错的地方，或是那些边界情况。 
   
  >2.维护过程中，单元测试应该跟着我们的bug report来走，每一个bug都应该有个UT。
于是程序员就会对自己的代码变更有两个自信，一是bug 被 fixed，二是相同的bug不会再次出现。

**怎样写好单元测试**

	>了解单元测试常用的注解
	
	
  >善用mock
   @MockBean
   
  >集成依赖的外部应用测试
  spring boot 的应用
  
  >单元测试命名