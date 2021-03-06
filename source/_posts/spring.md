---
title: spring
date: 2018-05-07 10:39:16
tags: Spring
categories: Spring
---
#### 什么是Spring
> Spring是一个开源框架，Spring是于2003年兴起的一个轻量级的Java开发框架，由Rod Johnson在其著作Expert One-On-One J2EE Development and Design中阐述的部分理念和原型衍生而来。它是为了解决企业应用开发的复杂性而创建的。框架的主要优势之一就是其分层架构，分层架构允许使用者选择使用哪一个组件，同时为J2EE应用程序开发提供集成的框架。Spring使用基本的JavaBean来完成以前只可能由EJB完成的事情。然而，Spring的用途不仅限于服务器端的开发。从简单性、可测试性和松耦合的角度而言，任何Java应用都可以从Spring中受益。<Strong>Spring的核心是控制反转(IoC)和面向切面(AOP)。</Strong>简单来说，Spring是一个分层的JavaSE/EEfull-stack(一站式)轻量级开源框架。

EE开发可分成三层架构，针对JavaEE的三层结构，每一层Spring都提供了不同的解决技术：
* WEB层：[SpringMVC]()
* 业务层：[Spring的IoC](/2018/05/07/spring-ioc/)
* 持久层：Spring的JDBCTemplate(Spring的JDBC模板，ORM模板用于整合其他的持久层框架)

Spring的核心有两部分：
* [IoC：控制反转](/2018/05/07/spring-ioc/)
* [AOP：面向切面编程](/2018/05/08/spring-aop/) 

#### Spring优势
* 方便解耦，简化开发
Spring就是一个大工厂，可以将所有对象的创建和依赖关系的维护，交给Spring管理。
* AOP编程的支持
Spring提供面向切面编程，可以方便的实现对程序进行权限拦截、运行监控等功能。
* 声明式事务的支持
只需要通过配置就可以完成对事务的管理，而无须手动编程。
* 方便程序的测试 
Spring对Junit4支持，可以通过注解方便的测试Spring程序。
* 方便集成各种优秀的框架
Spring不排斥各种优秀的开源框架，其内部提供了对各种优秀框架(如：Struts2、Hibernate、MyBatis、Quartz等)的直接支持。
* 降低JavaEE API的使用难度 
Spring对JavaEE开发中非常难用的一些API(JDBC、JavaMail、远程调用等)，都提供了封装，使这些API应用难度大大降低。