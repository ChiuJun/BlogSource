---
title: JavaEE(SSM) MyBatis缓存处理
top: false
cover: false
img: 'http://q9apk9x38.bkt.clouddn.com/medias/featureimages/ssm.jpg'
toc: true
mathjax: true
date: 2020-03-28 23:13:31
password:
summary: Java EE (SSM) 企业应用实践 第5章 MyBatis缓存处理
tags:
    - J2EE
    - SSM
    - MyBatis
    - EhCache
categories:
    - J2EE课程
---
# Java EE (SSM) 企业应用实践 
> 课程亮点  
本课程全面介绍了Java EE中MyBatis、Spring和Spring MVC三大框架的基本知识和使用方法。课程中对知识点的讲解由浅入深、通俗易懂，同时配备大量的操作案例，通过案例的演示帮助读者理解技术原理并提高实际操作能力。 全课程主要讲解了MyBatis、Spring、Spring MVC的相关知识，最后是一个项目案例，通过项目案例帮助读者掌握SSM框架整合的技术，让读者适应企业级开发的技术需要，为大型项目开发奠定基础。 本课程适合作为高等院校计算机类相关课程的教材，同时也可作为编程人员的学习指南。  

# 第5章 MyBatis缓存处理

## 学习目标
1. 理解MyBatis的一级缓存机制和二级缓存机制
2. 掌握MyBatis处理一级缓存的方法
3. 掌握MyBatis处理二级缓存的方法
4. 掌握MyBatis集成EhCache缓存的方法

## MyBatis的缓存机制
- 通常情况下，缓存将程序使用频率较高的数据存储在内存中，它具有可被快速读取和使用的特点。
- MyBatis支持缓存，它在内存中开辟一块区域，该区域用于保存程序对数据库的操作信息和数据库返回的数据，如果下一次程序再执行相同的操作，那么直接从缓存中读取数据而不是从数据库读取，如此一来，大大提升数据库性能并减轻系统压力。
- Mybatis使用到了两种缓存：一级缓存(本地缓存 local cache)和二级缓存（second level cache）。
- 其中，一级缓存是SqlSession级别的缓存。当创建SqlSession对象操作数据库时，MyBatis在SqlSession对象中引入一个HashMap对象作为存储数据的区域，HashMap对象的key是由SQL语句、条件、statement等信息组成一个唯一值。HashMap对象的value是查询出的结果对象。不同SqlSession对象之间的HashMap对象互不影响。
    - 默认情况下，一级缓存数据的生命周期等同于整个 session 的周期。
        由于缓存会被用来解决循环引用问题和加快重复嵌套查询的速度，所以无法将其完全禁用。
        但是可以通过设置 localCacheScope=STATEMENT 来只在语句执行时使用缓存。
    - 在MyBatis中，当同一个SqlSession对象多次执行完全相同的SQL语句时  
        在第一次执行完成后，MyBatis会将查询结果写入到一级缓存，此后如果程序没有执行插入、更新、删除操作   
        当第二次执行相同的查询语句时，MyBatis会直接读取一级缓存中的数据。

- 二级缓存是Mapper级别的缓存，多个SqlSession去操作同一个Mapper的SQL语句，二级缓存是跨SqlSession的，多个SqlSession可以共用二级缓存。与一级缓存的HashMap对象相同，二级缓存的HashMap对象的key由SQL语句、条件、statement等信息组成，HashMap对象的value是查询出的结果对象。
    - 默认情况下，只启用了一级缓存，它仅仅对一个会话中的数据进行缓存。二级缓存需要手动开启。
    - 当开启二级缓存时，MyBatis以namespace区分Mapper  
        如果多个SqlSession对象使用同一个Mapper的相同查询语句去操作数据库，在第一个SqlSession对象执行完成后，MyBatis会将查询结果写入到二级缓存，此后，如果程序没有执行插入、更新、删除操作，当第二个SqlSession对象执行相同的查询语句时，MyBatis会直接读取二级缓存中的数据。
    -  要启用全局的二级缓存，只需要在相应的的SQL映射文件中添加一行 &lt;cache/&gt;
        该语句的效果为：
        - 映射语句文件中的所有 select 语句的结果将会被缓存。
        - 映射语句文件中的所有 insert、update 和 delete 语句会刷新缓存。
        - 缓存会使用最近最少使用算法（LRU, Least Recently Used）算法来清除不需要的缓存。
        - 缓存不会定时进行刷新（也就是说，没有刷新间隔）。
        - 缓存会保存列表或对象（无论查询方法返回哪种）的 1024 个引用。
        - 缓存会被视为读/写缓存，这意味着获取到的对象并不是共享的，可以安全地被调用者修改，而不干扰其他调用者或线程所做的潜在修改。
- Mybatis 通过使用内置的日志工厂提供日志功能。内置日志工厂将会把日志工作委托给下面的实现之一
    - SLF4J
    - Apache Commons Logging
    - Log4j 2
    - Log4j
    - JDK logging
- 查看MyBatis一级缓存的工作状态，可以通过log4j日志组件    
    - 在src目录下创建log4j.properties文件
    - 引入相应的jar包即可

## EhCache缓存 
- 在实际开发中，很多项目采用分布式的系统架构。  
    分布式系统架构是将项目拆分成若干个子项目并使它们协同发挥功能的解决方案。在分布式系统架构下，为了提升系统性能，通常会采用分布式缓存对缓存数据进行集中管理。  
    由于MyBatis自身无法实现分布式缓存，需要整合其他分布式缓存框架，如EhCache缓存。
- EhCache是一种获得广泛应用的开源分布式缓存框架，具有快速、简单、能够提供多种缓存策略等特点，主要面向通用缓存、	和轻量级容器等。
- EhCache通常采用名称为ehcache.xml的配置文件实现功能配置，EhCache的配置文件有其自身特有的层次结构，具体参见[官方文档](http://mybatis.org/ehcache-cache/)
- 整合EhCache缓存
    - 在[GitHub仓库](https://github.com/mybatis/ehcache-cache)下载最新的release并将jar包引入工程
    - 在映射文件中，配置&lt;cache&gt;标签的type属性为EhCache对cache接口的实现类的完全限定名即可

## 本章小结
- 本章首先介绍了MyBatis的缓存机制
- 接下来讲解了MyBatis中的一级缓存，包括一级缓存的原理
- 然后讲解了MyBatis的二级缓存，包括二级缓存的原理、配置
- 最后讲解了MyBatis与EhCache缓存的整合
- 通过本章知识的学习，大家应该理解MyBatis的缓存机制，重点掌握二级缓存的应用和MyBatis整合EhCache缓存的方法，能够使用缓存提升MyBatis程序的查询效率、优化系统整体性能。

## 尾声
- 其实这门课程是学校开设的生产实习的内容，博主本身对J2EE并不感兴趣
- 课程内容来自千锋教育，几节课看下来，深感课程内容之浅，缺乏诚意。观看体验很糟糕，也浪费时间。
- 可能不再更新这个系列的Blog