---
title: JavaEE(SSM) MyBatis基础
top: false
cover: false
img: 'http://q7qon7hdm.bkt.clouddn.com/medias/featureimages/ssm.jpg'
toc: true
mathjax: true
date: 2020-03-26 15:20:14
password:
summary: Java EE (SSM) 企业应用实践 第一章 MyBatis基础
tags:
    - J2EE
    - SSM
    - MyBatis
categories:
    - J2EE课程
---
# Java EE (SSM) 企业应用实践 
> 课程亮点  
本课程全面介绍了Java EE中MyBatis、Spring和Spring MVC三大框架的基本知识和使用方法。课程中对知识点的讲解由浅入深、通俗易懂，同时配备大量的操作案例，通过案例的演示帮助读者理解技术原理并提高实际操作能力。 全课程主要讲解了MyBatis、Spring、Spring MVC的相关知识，最后是一个项目案例，通过项目案例帮助读者掌握SSM框架整合的技术，让读者适应企业级开发的技术需要，为大型项目开发奠定基础。 本课程适合作为高等院校计算机类相关课程的教材，同时也可作为编程人员的学习指南。  

# 第1章 MyBatis基础

## 学习目标：  
1. 了解ORM框架的概念  
2. 理解MyBatis的功能架构和工作流程
3. 掌握MyBatis入门程序的编写
4. 掌握MyBatis的基本操作

## MyBatis概述
### 传统JDBC的劣势
- 代码繁琐
- 表关系维护复杂
- 硬编码
- 性能问题  

由于JDBC存在的缺陷，企业中通常使用ORM框架来完成数据库的操作
### ORM简介  
ORM的全称是Object-Relation Mapping，即对象-关系映射。ORM是一种规范，它是将简单Java对象（POJO）和数据库表进行映射，使数据库表中的记录和POJO对象一一对应。  
目前Java应用领域流行的ORM框架有Hibernate、MyBatis等  
- Hibernate  
    - 优点：Hibernate封装性较高，开发人员通过XML映射文件或者注解定义映射规则。Hibernate会根据映射规则自动生成SQL语句并调用JDBC的API执行，大大提升了开发效率。
    - 局限：无法根据不同的条件组装不同的SQL语句，对多表关联和复杂SQL查询支持比较差。
- MyBatis
    - **半自动化的**ORM框架，需要手动提供POJO、SQL语句并匹配映射关系。
    - 优点：高度灵活、可优化、易维护
    - 缺点：与Hibernate相比，编码量较大  

### MyBatis简介  
- Apache组织的开源项目iBatis  
    2010年迁移到Google Code并改名为MyBatis  
    2013年迁移至GitHub，目前由GitHub维护    
- 使用XML文件或注解进行配置和映射，将接口和Java的POJO映射成数据库中的记录
- 核心思想：剥离程序中的大量SQL语句并将这些SQL语句配置映射到文件中。  
    - 可以根据开发需要灵活编写SQL语句并指定映射规则
    - 程序和SQL语句分离，提升了程序的扩展性
- MyBatis支持动态列、动态表名、存储过程，同时提供了简易的日志、缓存和级联功能  

### MyBatis的功能架构  
- MyBatis的功能架构由三层组成  
    - API接口层：开发人员操纵
    - 数据处理层：负责具体参数的映射、SQL解析、SQL执行、结果映射
    - 基础支撑层：提供链接管理、事务管理、配置加载、缓存处理等，作为基础组件支持上层的数据处理层  

### MyBatis的工作流程  
- MyBatis读取配置文件和映射文件
    - 配置文件：设置了数据源、事务等信息
    - 映射文件：设置了SQL执行相关的信息
    - 映射文件要引入到配置文件中才能被执行
- MyBatis给句配置信息和映射信息生成SqlSessionFactory对象
    - SqlSessionFactory对象的重要功能是创建MyBatis的核心类对象SqlSession
    - SqlSession对象封装了操作数据库的所有方法，开发人员调用SqlSession的方法完成对数据库的操作。  
    注：SqlSession是一个接口，有2个实现类，分别是DefaultSqlSession(默认使用)以及SqlSessionManager
    - SqlSession通过内部存放的执行器（Executor）来对数据进行CRUD  
        注：Executor（执行器）接口有两个实现类。  
        - BaseExecutor  
        注：BaseExecutor有三个继承类分别是：  
            BatchExecutor（重用语句并执行批量更新）  
            ReuseExecutor（重用预处理语句prepared statements）  
            SimpleExecutor（普通的执行器）      
        - CachingExecutor（缓存执行器）  
            Mybatis在Executor的设计上面使用了装饰者模式，我们可以用CachingExecutor来装饰前面的三个执行器目的就是用来实现缓存。
    - Executor（执行器）将要执行的SQL语句信息封装成MappedStatement对象  
    - 在执行SQL语句之前，Executor通过MappedStatement对象将输入的Java数据映射到SQL语句上
    - 在执行SQL语句之后，Executor通过MappedStatement对象将SQL语句的执行结果映射为Java数据  

## MyBatis的重要API  

### SqlSessionFactory  
- 创建SqlSession对象
- 每一个Mybatis应用程序都以SqlSessionFactory为基础，**SqlSessionFactory存在于Mybatis应用的整个生命周期**。
- 一般使用单例模式创建SqlSessionFactory对象，即一个数据库对应一个   SqlSessionFactory对象
- SqlSessionFactory对象由SqlSessionFactoryBuilder对象创建
    - SqlSessionFactoryBuilder.build()有5个重载：
        ```java
        SqlSessionFactory build(InputStream inputStream)
        // inputStream封装了XML文件的配置信息
        SqlSessionFactory build(InputStream inputStream, String environment)
        // environment决定将要加载的环境，包括数据源和事务管理器
        SqlSessionFactory build(InputStream inputStream, Properties properties)
        // properties决定将要加载的properties文件
        SqlSessionFactory build(InputStream inputStream, String env, Properties props)
        SqlSessionFactory build(Configuration config)
        // config需要预定义，它封装了绝大部分的配置信息
        ```
- 创建好SqlSessionFactory对象之后，接下来调用SqlSessionFactory对象的openSession() 方法创建SqlSession对象
    - SqlSessionFactory.openSession()有8个重载：
        ```java
        SqlSession openSession()
        SqlSession openSession(boolean autoCommit)
        SqlSession openSession(Connection connection)
        SqlSession openSession(TransactionIsolationLevel level)
        SqlSession openSession(ExecutorType execType, TransactionIsolationLevel level)
        SqlSession openSession(ExecutorType execType)
        SqlSession openSession(ExecutorType execType, boolean autoCommit)
        SqlSession openSession(ExecutorType execType, Connection connection)
        ```  

### SqlSession  
- SqlSession对象是MyBatis的核心类对象，他类似于JDBC中的Connection对象，其首要作用是执行持久化操作，开发中最为常用
- SqlSession对象的**生命周期贯穿数据库处理事务的过程**，一定时间没有使用对象要及时关闭，以免影响系统性能。
- SqlSession对象是线程不安全的，也不能被共享。
- SqlSession提供的方法，具体见MyBatis提供的[文档](https://mybatis.org/mybatis-3/zh/java-api.html#sqlSessions)  


## MyBatis的下载和使用  
> 由于MyBatis由第三方提供，使用MyBatis之前首先要下载jar包  
  本课程基于MyBatis 3.4.6
  这里下载的是MyBatis 3.5.4

### 下载MyBatis  
- 在GitHub下载相应的[Release](https://github.com/mybatis/mybatis-3/releases)
    - 核心jar包：/mybatis-3.5.4.jar
    - 依赖jar包：/lib/*
    - Docs：/mybatis-3.5.4.pdf  
  
### 下载配置MySQL
- 安装 略
- 启动MySQL
    ```
    net start mysql80
    ```
- 登录MySQL
    ```
    mysql -u root -p
    ```  

### 安装 Navicat for MySQL
- 略  
    
## MyBatis的简单应用
### 搭建开发环境  
- 数据准备
    - 在MySQL中创建数据库chapter01和数据表student
        ```sql
        DROP DATABASE IF EXISTS chapter01;
        CREATE DATABASE chapter01;
        USE chapter01;
        CREATE TABLE student (
            sid INT PRIMARY KEY AUTO_INCREMENT,#ID
            sname VARCHAR(20),#学生姓名
            age VARCHAR(20),#学生年龄
            course VARCHAR(20) #专业
        );
        ```
    - 向数据表中插入数据
        ```sql
        INSERT INTO student VALUES (1,'ZhangSan','20','Java');
        INSERT INTO student VALUES (2,'LiSi','21','Java');
        INSERT INTO student VALUES (3,'WangWu','22','Java');
        INSERT INTO student VALUES (4,'ZhaoLiu','22','Python');
        INSERT INTO student VALUES (5,'SunQi','22','PHP');
        INSERT INTO student VALUES (6,'ZhangSanSan','22','PHP');
        ```
- 创建工程  
    需要的jar包：
    - MyBatis核心jar包：mybatis-3.5.4.jar
    - MyBatis依赖jar包：lib/*
    - MySQL Connector jar包：mysql-connector-java-5.1.48.jar  

### 创建POJO类  

- 在工程chapter01的src目录下新建com.qfedu.pojo包，在com.qfedu.pojo包下新建Java类Student    

    ```java
    package com.qfedu.pojo;

    public class Student {
        private int sid;
        private String name;
        private String age;
        private String course;
        
        public Student() {}

        public Student(int sid, String name, String age, String course) {
            super();
            this.sid = sid;
            this.name = name;
            this.age = age;
            this.course = course;
        }

        public int getSid() {
            return sid;
        }

        public void setSid(int sid) {
            this.sid = sid;
        }

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        public String getAge() {
            return age;
        }

        public void setAge(String age) {
            this.age = age;
        }

        public String getCourse() {
            return course;
        }

        public void setCourse(String course) {
            this.course = course;
        }

        @Override
        public String toString() {
            return "Student [sid=" + sid + ", name=" + name + ", age=" + age + ", course=" + course + "]";
        }
        
    }
    ```  

### 创建配置文件  

- 在src目录下创建配置文件mybatis-config.xml
    ```xml
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE configuration
    PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <configuration>
        <environments default="mysql">
            <environment id="mysql">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/chapter01"/>
                <property name="username" value="root"/>
                <property name="password" value="jiabi"/>
            </dataSource>
            </environment>
        </environments>
        <mappers>
            <mapper resource="com/qfedu/mapper/StudentMapper.xml"/>
        </mappers>
    </configuration>
    ```
- 在com.qfedu.mapper下创建映射文件StudentMapper.xml
    ```xml
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
    PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="student">
        <select id="findStudentBySid" parameterType="Integer" resultType="com.qfedu.pojo.Student">
            select * from Student where sid = #{sid}
        </select>
    </mapper>
    ```  

### 编写测试类
#### 测试通过sid字段查询学生信息  
- 在com.qfedu.test包下创建类TestFindBySid
    ```java
    package com.qfedu.test;

    import java.io.IOException;
    import java.io.InputStream;

    import org.apache.ibatis.io.Resources;
    import org.apache.ibatis.session.SqlSession;
    import org.apache.ibatis.session.SqlSessionFactory;
    import org.apache.ibatis.session.SqlSessionFactoryBuilder;

    import com.qfedu.pojo.Student;

    public class TestFindBySid {
        public static void main(String[] args) {
            try {
                // 根据配置文件获得SqlSessionFactory对象
                String filename = "mybatis-config.xml";
                InputStream in = Resources.getResourceAsStream(filename);
                SqlSessionFactoryBuilder bu = new SqlSessionFactoryBuilder();
                SqlSessionFactory fac = bu.build(in);
                // 创建SqlSession对象
                SqlSession session = fac.openSession();

                // 调用selectOne()ִ 执行映射文件中的一条sql语句
                Student stu = session.selectOne("student.findStudentBySid",1);
                // 打印查询结果
                System.out.println(stu.toString());

                // 关闭session
                session.close();
            } catch (IOException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
        }
    }

    ```  

#### 测试通过sname字段模糊查询学生信息  
- 在StudentMapper.xml文件的<mapper>元素中加入映射信息  
    具体代码如下：  
    注：SQL语句中 $ 具有拼接符的功能，{value}表示传入的参数  
    ```xml
    <select id="findStudentBySname" parameterType="String" resultType="com.qfedu.pojo.Student">
    	select * from Student where sname like '%${value}%'
    </select>
    ```
- 在com.qfedu.test包下创建类TestFindBySname
    ```java
    package com.qfedu.test;

    import java.io.IOException;
    import java.io.InputStream;
    import java.util.List;

    import org.apache.ibatis.io.Resources;
    import org.apache.ibatis.session.SqlSession;
    import org.apache.ibatis.session.SqlSessionFactory;
    import org.apache.ibatis.session.SqlSessionFactoryBuilder;

    import com.qfedu.pojo.Student;

    public class TestFindBySname {
        public static void main(String[] args) {
            try {
                // 根据配置文件获得SqlSessionFactory对象
                String filename = "mybatis-config.xml";
                InputStream in = Resources.getResourceAsStream(filename);
                SqlSessionFactoryBuilder bu = new SqlSessionFactoryBuilder();
                SqlSessionFactory fac = bu.build(in);
                // 创建SqlSession对象
                SqlSession session = fac.openSession();

                // 调用selectList()ִ 执行映射文件中的一条sql语句
                List<Student> list = session.selectList("student.findStudentBySname","Zhang");
                if(list.size()==0) {
                    System.out.println("无符合条件的查询结果");
                }else {
                    // 打印查询结果
                    for (Student stu: list) {
                        System.out.println(stu.toString());
                    }
                }
                
                // 关闭session
                session.close();
            } catch (IOException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
        }
    }
    ```  

#### 测试添加学生信息  
- 在StudentMapper.xml文件的<mapper>元素中加入映射信息  
    具体代码如下：
    ```xml
    <insert id="addStudent" parameterType="com.qfedu.pojo.Student">
    	insert into Student(sname,age,course) values(#{sname},#{age},#{course})
    </insert>
    ```
- 在com.qfedu.test包下创建类TestAddStudent
    ```java
    package com.qfedu.test;

    import java.io.IOException;
    import java.io.InputStream;
    import java.util.List;

    import org.apache.ibatis.io.Resources;
    import org.apache.ibatis.session.SqlSession;
    import org.apache.ibatis.session.SqlSessionFactory;
    import org.apache.ibatis.session.SqlSessionFactoryBuilder;

    import com.qfedu.pojo.Student;

    public class TestAddStudent {
        public static void main(String[] args) {
            try {
                // 根据配置文件获得SqlSessionFactory对象
                String filename = "mybatis-config.xml";
                InputStream in = Resources.getResourceAsStream(filename);
                SqlSessionFactoryBuilder bu = new SqlSessionFactoryBuilder();
                SqlSessionFactory fac = bu.build(in);
                // 创建SqlSession对象
                SqlSession session = fac.openSession();
                
                Student stu = new Student(0, "xiaoming", "25", "Java");
                // 调用insert(),ִ 返回受影响行数
                int count = session.insert("student.addStudent", stu);
                if(count > 0) {
                    System.out.println("成功插入"+count+"条数据");
                }else {
                    System.out.println("插入数据失败");
                }

                // 调用commit()提交事务
                session.commit();
                // 关闭session
                session.close();
            } catch (IOException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
        }
    }
    ```  

#### 测试更新学生信息  
- 在StudentMapper.xml文件的<mapper>元素中加入映射信息  
    具体代码如下：
    ```xml
    <update id="updateStudent" parameterType="com.qfedu.pojo.Student">
    	update student set sname=#{sname},age=#{age},course=#{course}
    		where sid=#{sid}
    </update>
    ```
- 在com.qfedu.test包下创建类TestUpdateStudent
    ```java
    package com.qfedu.test;

    import java.io.IOException;
    import java.io.InputStream;
    import java.util.List;

    import org.apache.ibatis.io.Resources;
    import org.apache.ibatis.session.SqlSession;
    import org.apache.ibatis.session.SqlSessionFactory;
    import org.apache.ibatis.session.SqlSessionFactoryBuilder;

    import com.qfedu.pojo.Student;

    public class TestUpdateStudent {
        public static void main(String[] args) {
            try {
                // 根据配置文件获得SqlSessionFactory对象
                String filename = "mybatis-config.xml";
                InputStream in = Resources.getResourceAsStream(filename);
                SqlSessionFactoryBuilder bu = new SqlSessionFactoryBuilder();
                SqlSessionFactory fac = bu.build(in);
                // 创建SqlSession对象
                SqlSession session = fac.openSession();
                
                Student stu = new Student(8, "xiaohong", "18", "HTML");
                // 调用update(),ִ返回受影响行数
                int count = session.update("student.updateStudent", stu);
                System.out.println("更新时"+count+"条记录受影响");

                // 调用commit()提交事务
                session.commit();
                // 关闭session
                session.close();
            } catch (IOException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
        }
    }
    ```  

#### 测试删除学生信息  
- 在StudentMapper.xml文件的<mapper>元素中加入映射信息  
    具体代码如下：
    ```xml
    <delete id="delStudent" parameterType="Integer">
    	delete from student where sid = #{sid}
    </delete>
    ```
- 在com.qfedu.test包下创建类TestDeleteStudent
    ```java
    package com.qfedu.test;

    import java.io.IOException;
    import java.io.InputStream;
    import java.util.List;

    import org.apache.ibatis.io.Resources;
    import org.apache.ibatis.session.SqlSession;
    import org.apache.ibatis.session.SqlSessionFactory;
    import org.apache.ibatis.session.SqlSessionFactoryBuilder;

    import com.qfedu.pojo.Student;

    public class TestDeleteStudent {
        public static void main(String[] args) {
            try {
                // 根据配置文件获得SqlSessionFactory对象
                String filename = "mybatis-config.xml";
                InputStream in = Resources.getResourceAsStream(filename);
                SqlSessionFactoryBuilder bu = new SqlSessionFactoryBuilder();
                SqlSessionFactory fac = bu.build(in);
                // 创建SqlSession对象
                SqlSession session = fac.openSession();

                // 调用delete(),ִ返回受影响行数
                int count = session.delete("student.delStudent", 6);
                System.out.println(count+"条记录被删除");

                // 调用commit()提交事务
                session.commit();
                // 关闭session
                session.close();
            } catch (IOException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
        }
    }
    ```  

## 本章小结  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;本章主要介绍了MyBatis的基础知识，包括MyBatis的概念、功能结构、工作流程以及重要API等，最后通过一个案例详细讲解了使用MyBatis操作数据库的方法。通过本音知识的学习，大家应该理解MyBatis的基础概念和原理，掌握MyBatis的开发流程并能够开发简单的MyBatis程序。