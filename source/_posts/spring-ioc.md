---
title: spring-ioc
date: 2018-05-07 14:16:26
tags: Spring ioc
categories: Spring
---

### 参考：[关于spring ioc](https://blog.csdn.net/javazejian/article/details/54561302)

#### （1）IOC(Inversion of Control)
> IOC，另外一种说法叫DI（Dependency Injection），即依赖注入。它并不是一种技术实现，而是一种设计思想。在任何一个有实际开发意义的程序项目中，我们会使用很多类来描述它们特有的功能，并且通过类与类之间的相互协作来完成特定的业务逻辑。这个时候，每个类都需要负责管理与自己有交互的类的引用和依赖，代码将会变的异常难以维护和极度的高耦合。而IOC的出现正是用来解决这个问题，我们通过IOC将这些相互依赖对象的创建、协调工作交给Spring容器去处理，每个对象只需要关注其自身的业务逻辑关系就可以了。在这样的角度上来看，<strong>获得依赖的对象的方式，进行了反转，变成了由spring容器控制对象如何获取外部资源（包括其他对象和文件资料等等）</strong>。

> Spring所倡导的开发方式就是如此，所有的类都会在spring容器中登记，告诉spring你是个什么东西，你需要什么东西，然后spring会在系统运行到适当的时候，把你要的东西主动给你，同时也把你交给其他需要你的东西。所有的类的创建、销毁都由 spring来控制，也就是说控制对象生存周期的不再是引用它的对象，而是spring。对于某个具体的对象而言，以前是它控制其他对象，现在是所有对象都被spring控制，所以这叫控制反转。

##### 总结：ioc 最主要的是改变了以往的编程思想，由原先通过 new 的方式获取依赖对象的方式（正转），变成了由spring容器来进行实例化（反转），应用程序由主动变成了被动接受。

#### （2）具体使用（自动装配与注解注入）
* Spring的自动装配有三种模式：<strong>byType(根据类型)，byName(根据名称)、constructor(根据构造函数)。</strong>

##### 2.1 byType
在byTpye模式中，Spring容器会基于反射查看bean定义的类，然后找到与依赖类型相同的bean注入到另外的bean中，<strong>这个过程需要借助setter注入来完成，因此必须存在set方法，否则注入失败。</strong>

``` java
//dao层
public class UserDaoImpl implements UserDao{
    //.......
    @Override
    public void doSomething(){
        System.out.println("UserDaoImpl doSomething ...");
    }
}
//service层
public class UserServiceImpl implements UserService {
    //需要注入的依赖
    private UserDao userDao;

    /**
     * set方法
     */
    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }
    @Override
    public void doSomething(){
        userDao.doSomething();
    }
}
```
基于xml的配置如下，通过使用＜bean＞中的autowire属性为"byType"，按类型自动装配 userService 
``` xml
<bean id="userDao" class="xxx.dao.impl.UserDaoImpl" />
<!-- byType 根据类型自动装配userDao-->
<bean id="userService" autowire="byType" class="xxx.service.impl.UserServiceImpl" />
```
byType模式可能存一种注入失败的情况：由于是基于类型的注入，因此当xml文件中存在多个相同类型名称不同的实例Bean时，Spring容器依赖注入仍然会失败，因为存在多种适合的选项，Spring容器无法知道该注入那种。
恰好spring为我们提供了解决方案（指定注入需要的Bean实例）：通过设置＜bean＞标签的autowire-candidate为false，来过滤那些不需要注入的实例Bean
``` xml
<bean id="userDao" class="xxx.dao.impl.UserDaoImpl" />
<!-- autowire-candidate="false" 过滤该类型 -->
<bean id="userDao2" autowire-candidate="false" class="xxx.dao.impl.UserDaoImpl" />
```

##### 2.2 byName(实际项目中用得比较多)
byName模式的自动装配：Spring只会尝试将属性名与bean名称进行匹配，如果找到则注入依赖bean。
``` xml
<bean id="userDao"  class="xxx.dao.impl.UserDaoImpl" />
<bean id="userDao2" class="xxx.dao.impl.UserDaoImpl" />
<!-- byName 根据名称自动装配，找到UserServiceImpl名为 userDao属性并注入-->
<bean id="userService" autowire="byName" class="xxx.service.impl.UserServiceImpl" />
```

##### 基于注解的自动装配(@Autowired&@Resource&@Value)
###### 1) @Autowired
* 上述的用法可以用在bean数量很少的情况下，如果数量过多，都需要手动配置xml的话，显然不太优雅，也不利于维护
* 好在Spring 2.5 中引入了 @Autowired 注释，它可以对类成员变量、方法及构造函数进行标注，完成自动装配的工作。 通过 @Autowired的使用标注到成员变量时不需要有set方法，@Autowired 默认按类型匹配的

在xml配置文件中加入以下配置信息：
``` xml
<context:annotation-config />
```
``` java
public class UserServiceImpl implements UserService {
    //标注成员变量
    @Autowired
    private UserDao userDao;

    @Override
    public void doSomething(){
        userDao.doSomething();
    }
}
```
在@Autowired中还传递了一个required=false的属性，false指明当userDao实例存在就注入不存在就忽略；如果为true，就必须注入，若userDao实例不存在，就抛出异常。
由于@Autowired默认情况下是按类型匹配的(byType)，如果想要按名称(byName)匹配的话，可以使用@Qualifier注解与@Autowired结合：
``` java
public class UserServiceImpl implements UserService {
    //标注成员变量
    @Autowired
    @Qualifier("userDao1")
    private UserDao userDao;   
 }
```

###### 2) @Resource
@Autowried具备相同功效，但是默认按 byName模式 自动注入,依赖包是 javax.annotation.Resource 可以标注在成员变量和set方法上，但无法标注构造函数。
@Resource有两个中重要的属性：name和type。Spring容器对于@Resource注解的name属性解析为bean的名字，type属性则解析为bean的类型。
若设置name属性，则按byName模式的自动注入，若设置type属性则按 byType模式自动注入。如果都不指定，Spring容器将通过反射技术默认按byName模式注入。
``` java
//@Autowired标注成员变量
@Autowired
@Qualifier("userDao")
private UserDao userDao;  
//上述代码等价于@Resource
@Resource(name="userDao")
private UserDao  userDao;//用于成员变量

//也可以用于set方法标注
@Resource(name="userDao")
public void setUserDao(UserDao userDao) {
   this.userDao= userDao;
}
```
###### 3) @Value
以上两种自动装配的依赖注入并不适合简单值类型，如int、boolean、long、String以及Enum等，对于这些类型，Spring容器提供了@Value注入的方式：
``` java
public class UserServiceImpl implements UserService {
    //标注成员变量
    @Autowired
    @Qualifier("userDao")
    private UserDao userDao;
    //占位符方式
    @Value("${jdbc.address}")
    private String address;
    //SpEL表达方式，其中代表xml配置文件中的id值configProperties
    @Value("#{configProperties['jdbc.username']}")
    private String userName;

    ...
}
```
#### （3）IOC管理 bean
##### 3.1 Bean的命名
每一个交给Spring IOC容器创建的对象必须被分配至少一个名称，如果开发者没有提供，Spring容器将会为其分配一个内部名称，通过Bean的名称，我们可以在其他类中查找该类并使用它，如前面的案例，也是通过Bean名称获取到实际对象并执行对应的操作。在基于xml的配置信息中，可以使用id属性来为一个Bean分配名称，在同一个xml配置文件中，<font color="#dd0000">id必须是唯一的，但不同的xml可以相同</font>，当然还可以使用name来为Bean分配名称，name属性可以分配多个名称，此时可使用空格、逗号、分号来分离给定Bean分配多个名称，而id属性则无法这样使用。
``` xml
<!-- name属性配置多个名称 -->
<bean name="userDao,userDao1" class="xxx.dao.impl.UserDaoImpl"/>
<!-- id属性配置唯一名称而且不能与name相同-->
<bean id="userDao2" class="xxx.dao.impl.UserDaoImpl"/>
```
上述的Bean对象声明使用都在xml内声明手动声明的方式，一旦Bean对象多起来，管理Bean就会变得复杂。
因此Spring提供了基于Java注解的配置方式，分别使用org.springframework.stereotype.Service（@Service）和org.springframework.stereotype.Repository（@Repository）声明XxxServiceImpl和XxxDaoImpl类，使用@Autowired注解注入xxxDao（需要在xml声明注解驱动），或者使用（@Component）。

> Spring的框架中提供了与@Component注解等效的三个注解，@Repository 用于对DAO实现类进行标注，@Service 用于对Service实现类进行标注，@Controller 用于对Controller实现类进行标注（web层控制器）

``` java
//@Component 相同效果
@Service
public class UserServiceImpl implements UserService {
  @Autowired
  private UserDao userDao;
}
//@Component 相同效果
@Repository
public class UserDaoImpl implements UserDao{
//......
}
```
有了注解声明，我们就不需要在xml中声明Bean了，但需要告诉Spring注解的Bean在那些包下，因此需要添加包扫描机制，此时需要启用Spring的context命名空间：
``` xml
<context:component-scan base-package="xxx.project" />
```

* 另外 @Component、@Service和@Repository可以输入一个String值的名称，如果没有提供名称，那么默认情况下就是一个简单的类名(第一个字符小写)变成Bean名称。

``` java
//@Component("userService") 相同效果
@Service("userService")
public class UserServiceImpl implements UserService {
  @Autowired
  private UserDao userDao;
}
//@Component("UserDao") 相同效果
@Repository("UserDao")
public class UserDaoImpl implements UserDao{
//......
}
```

#### （4）项目实践
##### 实际项目中，默认会按照byName方式进行依赖注入

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd"
       default-autowire="byName">

    <!--
        注意上面的default-autowire="byName"，如果没有这个声明不会被注入
    -->
    <description>Spring-配置</description>
    <!--
        自动扫描组件，这里要把web下面的
        controller去除，他们是在servlet-servlet.xml中配置的，如果不去除会影响事务管理的。
    -->
    <context:component-scan base-package="xxx"/>
    <!-- 用于持有ApplicationContext,可以使用SpringContextHolder.getBean('xxxx')的静态方法得到spring bean对象 -->
    <bean class="xxx.SpringContextHolder" lazy-init="false" />
</beans>
</xml>
```