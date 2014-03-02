---
layout: post
title: "java-struts-spring-mybatis"
date: 2013-04-21 13:57
comments: true
categories: 
---
Struts2 is a popular Java MVC framework, it is an action-based presentation framework;
well, how to use struts2 in your java web project? In simple terms, just add filter and 
filter-mapping in your web.xml file, as followings:
<!-- more -->
```
    <filter>
        <filter-name>struts2</filter-name>
        <filter-class>
            org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter
        </filter-class>
    </filter>
    <filter-mapping>
        <filter-name>struts2</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```
then add your struts.xml file in your classes directory to config
your action to your pages.

OK, then you want to use spring framework to help you manage different components,
Spring is a dependency injection framework and it has to know which beans it must create
and how to bind them together and that is what applicationContext.xml(default) file is for.
You can define your struts2 action class in your applicationContext.xml file.
Intergrate into your web project is also very simple, just add following contents to your web.xml:
```
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:spring-beans.xml</param-value>
</context-param>
<listener>
    <listener-class>
        org.springframework.web.context.ContextLoaderListener
    </listener-class>
</listener>
```
As you see above, context-params label just load spring configuration file,
and listener is used to start up spring container through listener.
And then you should config your bean in your spring configuration file 
`spring-beans.xml`above example.

At last you also want a framework to help you comunicate with dadabases.
maybe MyBatis is also a good one, MyBatis is a framework which automate the 
mapping between SQL and objects. It is very powerful SQL mappings framework
with automatic databases access class generation using proxy implementation
of the services defined. It will reduce many database work codes.
If you use spring to manage your mybatis, It will be convenient.
Just add following contents to your spring config files:
```
<bean id="DBPoolDataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource" destroy-method="close">
    <property name="driverClass" value="com.mysql.jdbc.Driver" />
    <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/test" />
    <property name="user" value="root" />
    <property name="password" value="******" />
</bean>
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="DBPoolDataSource" />
</bean>
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="configLocation" value="classpath:mybatis-config.xml"></property>
    <property name="dataSource" ref="DBPoolDataSource"></property>
</bean>
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <property name="basePackage" value="com.baijian.web.helloworld.mapper" />
</bean>
```
First bean is to config your database pool, 
second bean is to config your transaction manager to decide and control your transaction,
then you should config your SqlSessionFactory ,this bean will provide SessionFactory instance 
of MyBatis, it will create sqlSession and close it automatic. 
Last bean is used to scan for mappers and let them autowired, thanks to MapperScannerConfigurer.
MapperScannerConfigurer is used to publish the data service
interfaces in defined for MyBatis to configure as Spring Beans, then you can get the instances 
of UserMapper using @Autowired annotation, you do not have to implement the interface
as mybatis will provide porxy implementation for this, so conventient~

<a href="https://www.github.com/baijian/java-ssi-demo/">MyDemoProject</a>

