---
title: spring-aop
date: 2018-05-08 14:16:51
tags: Spring aop
categories: Spring
---
#### 1.背景
> 分布于应用中多处的功能被称为<strong>横切关注点</strong>（cross-cutting concerns）。通常，这些横切关注点从概念上是于应用的业务逻辑相分离的（但是往往直接潜入到应用的业务逻辑之中）。将这些横切关注点于业务逻辑相分离正是面向切面编程（AOP）所要解决的。——《Spring实战》

#### 2.什么是面向切面编程
  继承与委托是最常见的实现重用通用功能的面向对象技术。但是我们不太会在应用中使用相同的基类，而使用委托需要对委托对象进行复杂的调用。
  切面提供了取代继承和委托的另一种选择，在很多场景下更清晰简洁。横切关注点可以被模块化为特殊的类，这些类被称为切面。
  这样做有两个好处：
  1. 每个关注点现在只集中于一处，而不是分散于多处代码中；
  2. 服务模块更简洁，因为它们只包含主要关注点（或核心功能）的代码，而次要关注点的代码被转移到切面中了。

#### 3.AOP术语
  与大多数技术一样，AOP已经形成了自己的术语。描述切面的常用术语有<strong>通知（advice）、切点（pointcut）和连接点（join point）</strong>
##### 3.1 通知（advice）
在AOP术语中，切面的工作被称为通知。Spring切面可以应用5种类型的通知：
* Before——在方法被调用之前调用通知。
* After——在方法完成之后调用通知，无论方法执行是否成功。
* After-returning——在方法成功执行之后调用通知。
* After-throwing——在方法抛出异常后调用通知。
* Around——通知包裹了被通知的方法，在被通知的方法调用之前和调用之后执行自定义的行为。

##### 3.2 连接点（Joinpoint）
* 我们的应用程序可能需要对数以千计的时机应用通知，这些时机被称为连接点。
* 连接点是在应用执行过程中能够插入切面的一个点，这个点可以是调用方法时、抛出异常时、甚至修改一个字段时。切面代码可以利用这些点插入到应用的正常流程之中，并添加新的行为。

##### 3.3 切点（Pointcut）
* 一个切面并不需要通知应用中所有的连接点，切点有助于缩小切面所通知的连接点的范围。
* 切点的定义会匹配通知所有要织入的一个或多个连接点。
* 通常使用明确的类和方法名称来指定这些切点，或是利用正则表达式定义匹配的类和方法名称模式来指定这些切点。

##### 3.4 切面（Aspect）
* 切面是通知和切点的结合。
* 通知和切点共同定义了关于切面的全部内容——它是什么，在何时完成其功能。

##### 3.5 引入（Introduction）
引入允许我们向现有的类添加新的方法或属性

##### 3.6 织入（Weaving）
织入是将切面应用到目标对象来创建新的代理对象的过程。切面在指定的连接点被织入到目标对象中。在目标对象的生命周期里有多个点可以进行织入：
1. 编译期——切面在目标类编译时被织入。这种方式需要特殊的编译器。AspectJ的织入编译器就是以这种方式织入切面的。
2. 类加载期——切面在目标类加载到JVM时被织入。这种方式需要特殊的类加载器（ClassLoader），它可以在目标类被引入之前增强该目标类的字节码。AspectJ 5的LTW就支持以这种方式织入切面。
3. 运行期——切面在应用运行的某个时刻被织入。一般情况下，在织入切面时，AOP容器回会目标对象动态创建一个代理对象。Spring AOP就是以这种方式织入切面的。

#### 4.Spring AOP
##### 4.1 Spring 提供了4种各具特色的AOP支持：
* 基于代理的经典AOP；
* @AspectJ 注解驱动的切面；
* 纯POJO切面；
* 注入式AspectJ切面（适合Spring各版本）

##### <font color="#dd0000">Spring 对AOP的支持局限于方法拦截，且只支持方法连接点（不支持字段和构造器接入点）。</font>
##### 4.2 使用切点选择连接点
* execution指示器是编写切点定义时最主要使用的指示器。
* execution()编写规则如：execution(* xxx.xxx.xxx.impl..*.*(..))

##### 4.3 在XML中声明切面
Spring的AOP配置元素简化了基于POJO切面的声明：
```xml
<aop:advisor> <!-- 定义AOP通知器 -->
<aop:after> <!-- 定义AOP后置通知（不管被通知的方法是否执行成功） -->
<aop:after-returning> <!-- 定义AOP after-returning 通知 -->
<aop:after-throwing> <!-- 定义AOP after-throwing 通知 -->
<aop:around> <!-- 定义AOP 环绕通知 -->
<aop:aspect> <!-- 定义切面 -->
<aop:aspectj-antoproxy> <!-- 启用@AspectJ 注解驱动的切面 -->
<aop:before> <!-- 定义AOP 前置通知 -->
<aop:config> <!-- 顶层的AOP配置元素。大多数的<aop:*>元素必须包含在<aop:config>元素内 -->
<aop:declare> <!-- 为被通知的对象引入额外的接口，并透明地实现 -->
<aop:pointcut> <!-- 定义切点 -->
```

##### 4.4 使用例子
```xml
<bean id="xxxHandler" class="xxx.BusinessExceptionAop">
  <property name="brave" value="#{brave}" />
</bean>
<aop:config>
  <aop:aspect ref="xxxHandler"> 
    <aop:pointcut id="exceptionService" expression="execution(* xxx.impl..*.*(..))" />
    <aop:after-throwing pointcut-ref="exceptionService" method="afterThrowing" throwing="e" />
  </aop:aspect>
</aop:config>
```