---
title: JavaEE(SSM) MyBatis动态SQL和注解
top: false
cover: false
img: 'http://q9apk9x38.bkt.clouddn.com/medias/featureimages/ssm.jpg'
toc: true
mathjax: true
date: 2020-03-27 22:42:13
password:
summary: Java EE (SSM) 企业应用实践 第4章 动态SQL和注解
tags:
    - J2EE
    - SSM
    - MyBatis
    - DynamicSQL
categories:
    - J2EE课程
---
# Java EE (SSM) 企业应用实践 
> 课程亮点  
本课程全面介绍了Java EE中MyBatis、Spring和Spring MVC三大框架的基本知识和使用方法。课程中对知识点的讲解由浅入深、通俗易懂，同时配备大量的操作案例，通过案例的演示帮助读者理解技术原理并提高实际操作能力。 全课程主要讲解了MyBatis、Spring、Spring MVC的相关知识，最后是一个项目案例，通过项目案例帮助读者掌握SSM框架整合的技术，让读者适应企业级开发的技术需要，为大型项目开发奠定基础。 本课程适合作为高等院校计算机类相关课程的教材，同时也可作为编程人员的学习指南。  

# 第4章 动态SQL和注解

## 学习目标

1. 理解动态SQL的功能
2. 掌握动态SQL中常用元素的使用
3. 理解MyBatis中的常用注解
4. 掌握MyBatis中常用注解的使用

## 动态SQL

### 动态SQL简介

- 动态 SQL 是 MyBatis 的强大特性之一。  
如果你使用过 JDBC 或其它类似的框架，你应该能理解根据不同条件拼接 SQL 语句有多痛苦，例如拼接时要确保不能忘记添加必要的空格，还要注意去掉列表最后一个列名的逗号。利用动态 SQL，可以彻底摆脱这种痛苦。
- 动态SQL是MyBatis提供的拼接SQL语句的强大机制。在MyBatis的映射文件中，开发人员可通过动态SQL元素灵活组装SQL语句，这在很大程度上避免了单一SQL语句的反复堆砌，提高了SQL语句的复用性。
- MyBatis提供了一系列的动态SQL元素，其中常用的元素如表所示。

    设置参数 | 概述
    :----   | :----
    if      | 相当于判断语句，常用于单条件分支判断
    choose (when, otherwise)    | 相当于判断语句，常用于多条件分支判断
    trim (where, set)   |   辅助元素，用于处理一些拼装问题
    foreach  |  相当于循环语句，在in语句等列举条件时常用

### &lt;if&gt;元素

- &lt;if&gt;元素主要用于条件判断，它类似于Java代码中的if语句，通常与test属性联合使用。
- 在实际开发中，&lt;if&gt;元素的任务是根据需求动态控制where 子句的内容。
- 例如，现有一张学生表，程序可通过学生姓名、年龄、科目等字段查询学生信息，如果将科目作为非必选的查询条件，满足既可以通过姓名和科目查询学生信息，又可以直接通过姓名查询学生信息，此时需要使用到&lt;if&gt;元素。
- 接下来，本节通过以上实例演示&lt;if&gt;元素的使用。
- 数据准备
    - 在数据库chapter04中创建数据表student，该表用于映射学生信息  
        SQL语句如下所示
        ```sql
        DROP DATABASE IF EXISTS chapter04;
        CREATE DATABASE chapter04;
        USE chapter04;
        CREATE TABLE student (
            sid INT PRIMARY KEY AUTO_INCREMENT,#ID
            sname VARCHAR(20),#学生姓名
            age VARCHAR(20),#学生年龄
            course VARCHAR(20) #科目
        );
        ```
    - 向数据表student中插入数据  
        SQL语句如下所示
        ```sql
        INSERT INTO student VALUES ('ZhangSan','20','Java');
        INSERT INTO student VALUES ('LiSi','21','Java');
        INSERT INTO student VALUES ('LiSi','21','Python');
        INSERT INTO student VALUES ('WangWu','19','Java');
        ```
    - 通过bash测试是否插入成功
        ```bash
        mysql> use chapter04
        Database changed
        mysql>  select * from student;
        +-----+----------+------+--------+
        | sid | sname    | age  | course |
        +-----+----------+------+--------+
        |   1 | ZhangSan | 20   | Java   |
        |   2 | LiSi     | 21   | Java   |
        |   3 | LiSi     | 21   | Python |
        |   4 | WangWu   | 19   | Java   |
        +-----+----------+------+--------+
        4 rows in set (0.00 sec)
        ```
- 创建工程
    - 与之前相似，略
- 创建POJO类  
    在工程chapter04的src目录下创建com.qfedu.pojo包，在com.qfedu.pojo包下创建POJO类Student，该类的代码与[第1章](https://jiabi.tech/2020/03/26/j2ee-ssm-mybatis-basis/#%E5%88%9B%E5%BB%BAPOJO%E7%B1%BB)相同，此处不再重复列出。
- 创建配置文件  
    在src目录下新建配置文件mybatis-config.xml
    ```xml
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE configuration
    PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <configuration>
        <!--配置别名 -->
        <typeAliases>
            <package name="com.qfedu.pojo" />
        </typeAliases>
        <!--配置环境 -->
        <environments default="mysql">
            <environment id="mysql">
                <transactionManager type="JDBC" />
                <dataSource type="POOLED">
                    <property name="driver" value="com.mysql.jdbc.Driver" />
                    <property name="url"
                        value="jdbc:mysql://localhost:3306/chapter04" />
                    <property name="username" value="root" />
                    <property name="password" value="jiabi" />
                </dataSource>
            </environment>
        </environments>
        <!--引入映射文件 -->
        <mappers>
            <mapper resource="com/qfedu/mapper/StudentMapper.xml"/>	
        </mappers>
    </configuration>
    ```
- 创建映射文件  
    在工程chapter04的src目录下创建com.qfedu.mapper包，在com.qfedu.mapper包下创建StuMapper.xml文件  
    由于要满足科目作为非必选的查询条件，映射文件中使用&lt;if&gt;元素动态拼装SQL语句。
    ```xml
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
    PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="student">
        <select id="findStudentBySnameAndCourse" parameterType="student"
            resultType="student">
            select * from student where sname = #{sname}
            <!--根据条件动态拼装SQL语句 -->
            <if test=" null!=course and ''!=course">
                and course =#{course}
            </if>
        </select>
    </mapper>
    ```
- 编写测试类
    在src目录下新建包com.qfedu.test  
    在该包下新建类TestIf
    ```java
    package com.qfedu.test;

    import java.io.*;
    import java.util.List;
    import org.apache.ibatis.io.Resources;
    import org.apache.ibatis.session.*;
    import com.qfedu.pojo.Student;

    public class TestIf {
        public static void main(String[] args) {
            String resource = "mybatis-config.xml";
            try {
                InputStream in = Resources.getResourceAsStream(resource);
                SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(in);
                SqlSession sqlSession = factory.openSession();
                Student student = new Student();

                student.setSname("LiSi");
                List<Student> selectList = sqlSession.selectList("student.findStudentBySnameAndCourse", student);
                for (Student stu : selectList) {
                    System.out.println(stu.toString());
                }
                System.out.println("------------------------------");
                student.setCourse("Java");
                selectList = sqlSession.selectList("student.findStudentBySnameAndCourse", student);
                for (Student stu : selectList) {
                    System.out.println(stu.toString());
                }
                sqlSession.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
    ```
- 查询结果  
    第一次查询时，仅通过姓名查询，而第二次通过姓名和课程查询
    ```
    Student [sid=2, sname=LiSi, age=21, course=Java]
    Student [sid=3, sname=LiSi, age=21, course=Python]
    ------------------------------
    Student [sid=2, sname=LiSi, age=21, course=Java]
    ```

### &lt;choose&gt;、&lt;when&gt;和&lt;otherwise&gt;元素
- 与&lt;if&gt;元素的功能类似，&lt;choose&gt;元素同样用于条件判断，但不同的是，&lt;choose&gt;元素适用于多个判断条件的场景，它类似于Java代码中的switch语句。
- &lt;choose&gt;元素包含&lt;when&gt;和&lt;otherwise&gt;两个子元素   
    - 其中，一个&lt;choose&gt;元素中至少要包含1个&lt;when&gt;子元素、0个或1个&lt;otherwise&gt;子元素。  
    - 当程序中的业务关系相对复杂时，MyBatis可通过&lt;choose&gt;元素动态控制SQL语句中的内容。  
    - 例如，现在要从数据表student中查询学生信息  
        如果sid不为空，则根据sid查询学生信息；  
        如果sid为空、sname不为空，则根据sname模糊查询学生信息；  
        如果sid为空、sname为空，则查询所有course值为Java的学生信息。  
        接下来，本节通过以上实例演示&lt;choose&gt;元素的使用。
- 修改映射文件  
    在StudentMapper.xml文件的&lt;mapper&gt;元素中加入映射信息  
    具体代码如下
    ```xml
    <select id="findStudentByChoose" parameterType="student" resultType="student">
		select * from student where 1=1
		<choose>
			<when test="null!=sid and 0!=sid">
				and sid = #{sid}
			</when>
			<when test="null!=sname and ''!=sname">
				and sname like '%${sname}%'
			</when>
			<otherwise>
				and course = 'Java'
			</otherwise>
		</choose>
	</select>
    ```
    在以上代码中，&lt;choose&gt;元素用于根据条件组装SQL语句  
    第一个&lt;when&gt;元素的test属性为True时，则追加该&lt;when&gt;元素包含的SQL语句  
    否则，进入第二个&lt;when&gt;元素的条件判断  
    当前两个&lt;when&gt;元素的test属性为False时，则直接追加&lt;otherwise&gt;元素所包含的SQL语句。
- 编写测试类  
    在com.qfedu.test包下新建类TestChoose  
    ```java
    package com.qfedu.test;
    import java.io.*;
    import java.util.List;
    import org.apache.ibatis.io.Resources;
    import org.apache.ibatis.session.*;
    import com.qfedu.pojo.Student;
    public class TestChoose {
        public static void main(String[] args) {
            String resource = "mybatis-config.xml";
            try {
                InputStream in = Resources.getResourceAsStream(resource);
                SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(in);
                SqlSession sqlSession = factory.openSession();
                Student student = new Student();

                student.setSid(2);
                List<Student> selectList = sqlSession.selectList("student.findStudentByChoose",student);
                for (Student stu : selectList) {
                    System.out.println(stu.toString());
                }
                System.out.println("------------------------------1");
                student.setSid(0);
                student.setSname("Li");
                selectList = sqlSession.selectList("student.findStudentByChoose",student);
                for (Student stu : selectList) {
                    System.out.println(stu.toString());
                }
                System.out.println("------------------------------2");
                student.setSid(0);
                student.setSname("");
                selectList = sqlSession.selectList("student.findStudentByChoose",student);
                for (Student stu : selectList) {
                    System.out.println(stu.toString());
                }

                sqlSession.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
    ```
- 查询结果  
    ```
    Student [sid=2, sname=LiSi, age=21, course=Java]
    ------------------------------1
    Student [sid=2, sname=LiSi, age=21, course=Java]
    Student [sid=3, sname=LiSi, age=21, course=Python]
    ------------------------------2
    Student [sid=1, sname=ZhangSan, age=20, course=Java]
    Student [sid=2, sname=LiSi, age=21, course=Java]
    Student [sid=4, sname=WangWu, age=19, course=Java]
    ```

### &lt;where&gt;元素、&lt;set&gt;元素和&lt;trim&gt;元素  
- 前面几个例子已经合宜地解决了一个臭名昭著的动态 SQL 问题。  
    现在回到之前的 “if” 示例，这次我们将 “sname = #{sname}” 设置成动态条件，看看会发生什么。
    ```xml
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
    PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="student">
        <select id="findStudentBySnameAndCourse" parameterType="student" resultType="student">
            select * from student where
            <if test=" null!=course and ''!=course">
                sname = #{sname}
            </if>
            <if test=" null!=course and ''!=course">
                and course = #{course}
            </if>
        </select>
    </mapper>
    ```
    - 如果没有匹配的条件会怎么样？最终这条 SQL 会变成这样：
        ```sql  
        select * from student where  
        ```
        这会导致查询失败。  
    - 如果匹配的只是第二个条件又会怎样？这条 SQL 会是这样:
        ```sql
        select * from student where
            and course = #{course}
        ```
        这个查询也会失败。
- 这个问题不能简单地用条件元素来解决。这个问题是如此的难以解决，以至于解决过的人不会再想碰到这种问题。
- MyBatis 有一个简单且适合大多数场景的解决办法。而在其他场景中，可以对其进行自定义以符合需求。而这，只需要一处简单的改动：
    ```xml
    <select id="findStudentBySnameAndCourse" parameterType="student" resultType="student">
        select * from student
        <where>
            <if test=" null!=course and ''!=course">
                sname = #{sname}
            </if>
            <if test=" null!=course and ''!=course">
                and course = #{course}
            </if>
        </where>
    </select>
    ```
- where 元素只会在子元素返回任何内容的情况下才插入 “WHERE” 子句。而且，若子句的开头为 “AND” 或 “OR”，where 元素也会将它们去除。
- 如果 where 元素与你期望的不太一样，你也可以通过自定义 trim 元素来定制 where 元素的功能。比如，和 where 元素等价的自定义 trim 元素为：
    ```xml
    <trim prefix="WHERE" prefixOverrides="AND | OR ">
    ...
    </trim>
    ```
    - prefixOverrides 属性会忽略通过管道符分隔的文本序列（注意此例中的空格是必要的）。   
    上述例子会移除所有 prefixOverrides 属性中指定的内容，并且插入 prefix 属性中指定的内容。
- 用于动态更新语句的类似解决方案叫做 set。set 元素可以用于动态包含需要更新的列，忽略其它不更新的列。比如：
    ```xml
    <update id="updateStudent" parameterType="student">
		update student
            <set>
                <if test="null!=sname and ''!=sname">
                    sname = #{sname},
                </if>
                <if test="null!=age and ''!=age">
                    age  = #{age}, 
                </if>
            </set>
		where sid = #{sid}
	</update>
    ```
    这个例子中，更新指定sid学生的姓名或者年龄或者同时更新   
    set 元素会动态地在行首插入 set 关键字，并会删掉额外的逗号。
- 来看看与 set 元素等价的自定义 trim 元素吧：
    ```xml
    <trim prefix="SET" suffixOverrides=",">
    ...
    </trim>
    ```
    这里覆盖了后缀值设置，并且自定义了前缀值。

### &lt;foreach&gt;元素 
- 动态 SQL 的另一个常见使用场景是对集合进行遍历（尤其是在构建 IN 条件语句的时候）。  
    比如：
    ```xml
    <select id="findStudentByForeach"    resultType="student">
		select  * from student where sid in
		<foreach item="sid"  index="index" collection="list" open="(" separator="," close=")" >
			#{sid}
		</foreach>
	</select>
    ```
- 你可以将任何可迭代对象（如 List、Set 等）、Map 对象或者数组对象作为集合参数传递给 foreach。  
- 当使用可迭代对象或者数组时，index 是当前迭代的序号，item 的值是本次迭代获取到的元素。  
- 当使用 Map 对象（或者 Map.Entry 对象的集合）时，index 是键，item 是值。

### &lt;bind&gt;元素

- 在实际开发中，由于不同数据库支持的SQL语法略有不同，如果需要更换数据库，那么程序中的相应SQL语句就需要重写，这就会带来维护成本的上升。  
- 此时，可以通过&lt;bind&gt;元素来解决此类问题。
- &lt;bind&gt;元素允许在 OGNL 表达式以外创建一个变量，并将其绑定到当前的上下文。，通过&lt;bind&gt;元素，对变量的值的引用变得简单，而且，MyBatis程序的可移植性得到提升。  
    例如
    ```xml
    <select id="findStudentByBind"  parameterType="student"  resultType="student">
		select * from student  where 
		<bind name="sname_pattern" value="'%'+sname+'%'"/>
		<if test="null!=sname_pattern and ''!=sname_pattern">
				sname like  #{sname_pattern}
		</if>
	</select>
    ```

## 注解

### 简介
- 注解提供了一种简单且低成本的方式来实现简单的映射语句。
- 在MyBatis中，除了XML的映射方式，MyBatis还支持通过注解实现POJO对象和数据表之间的关系映射。- 使用注解时，一般将SQL语句直接写在接口上。与XML的映射方式相比，注解相对简单并且不会造成大量的开销。
- MyBatis提供了若干注解，其中常用的注解如表所示。
    更多内容参加[MyBatis官方文档](https://mybatis.org/mybatis-3/zh/java-api.html)
    注解 | XML等价形式 | 描述
    :--  | :-- | :--
    @Select注解 | &lt;select&gt; |@Select注解用于映射查询语句，其作用等同于xml文件中的&lt;select&gt;元素。
    @Insert注解 | &lt;insert&gt; |@Insert注解用于映射插入语句，其作用等同于xml文件中的&lt;insert&gt;元素。
    @Update注解 | &lt;update&gt; |@Update注解用于映射更新语句，其作用等同于xml文件中的&lt;update&gt;元素。
    @Delete注解 | &lt;delete&gt; |@Delete注解用于映射删除语句，其作用等同于xml文件中的&lt;delete&gt;元素。
    @Param注解  | 无 |@Param注解的功能是指定参数，通常用于SQL语句中参数比较多的情况。
- 接下来，本节将通过一个实例演示这几个注解的使用。

### 实践
- 创建Mapper接口
    在com.qfedu.mapper包下创建接口StudentMapper。
    ```java
    package com.qfedu.mapper;

    import org.apache.ibatis.annotations.Delete;
    import org.apache.ibatis.annotations.Insert;
    import org.apache.ibatis.annotations.Param;
    import org.apache.ibatis.annotations.Select;
    import org.apache.ibatis.annotations.Update;

    import com.qfedu.pojo.Student;

    public interface StudentMapper {
        @Select("select * from student where sid = #{sid}")
        Student selectStudent(int sid);
        @Insert("insert into student(sname,age,course) "
                + " values(#{sname},#{age},#{course})")
        int insertStudent(Student student);
        @Update("update student "
                + "set sname = #{sname},age = #{age} where sid = #{sid}")
        int updateStudent(Student student);
        @Delete("delete from student where sid = #{sid}")
        int deleteStudent(int sid);
        
        @Select("select * from student where sname = #{param01} "
                + "and course = #{param02}")
        Student selectBySnameAndCourse(@Param("param01") String sname,
                                    @Param("param02") String course);
    }
    ```
- 在配置文件mybatis-config.xml的&lt;mappers&gt;元素加入相应的配置信息
    ```xml
    <mappers>
		<mapper class="com.qfedu.mapper.StudentMapper"/>
	</mappers>
    ```
- 创建测试类
    在com.qfedu.test包下创建测试类TestAnnotation
    ```java
    package com.qfedu.test;

    import com.qfedu.mapper.StudentMapper;
    import com.qfedu.pojo.Student;
    import org.apache.ibatis.io.Resources;
    import org.apache.ibatis.session.SqlSession;
    import org.apache.ibatis.session.SqlSessionFactory;
    import org.apache.ibatis.session.SqlSessionFactoryBuilder;

    import java.io.IOException;
    import java.io.InputStream;

    public class TestAnnotation {
        public static void main(String[] args) {
            String resource = "mybatis-config.xml";
            try {
                InputStream in = Resources.getResourceAsStream(resource);
                SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(in);
                SqlSession sqlSession = factory.openSession();

                StudentMapper mapper;
                Student student;
                int result;

                System.out.println("-----------test @Select-----------");
                mapper = sqlSession.getMapper(StudentMapper.class);
                student = mapper.selectStudent(1);
                System.out.println(student.toString());

                System.out.println("-----------test @Insert-----------");
                student.setSname("ZhouBa");
                student.setAge("21");
                student.setCourse("Java");
                result = mapper.insertStudent(student);
                if (result > 0) {
                    System.out.println("成功插入" + result + "条数据");
                } else {
                    System.out.println("插入操作失败");
                }
                sqlSession.commit();

                System.out.println("-----------test @Update-----------");
                student.setSid(5);
                student.setSname("WuJiu");
                student.setCourse("Python");
                result = mapper.updateStudent(student);
                if (result > 0) {
                    System.out.println("成功更新" + result + "条数据");
                } else {
                    System.out.println("更新操作失败");
                }
                sqlSession.commit();

                System.out.println("-----------test @Delete-----------");
                result = mapper.deleteStudent(5);
                if (result > 0) {
                    System.out.println("成功删除" + result + "条数据");
                } else {
                    System.out.println("删除操作失败");
                }
                sqlSession.commit();

                System.out.println("-----------test @Param-----------");
                student = mapper.selectBySnameAndCourse("LiSi", "Java");
                System.out.println(student.toString());

                sqlSession.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

    }
    ```
- 测试结果
    ```
    -----------test @Select-----------
    Student [sid=1, sname=ZhangSan, age=20, course=Java]
    -----------test @Insert-----------
    成功插入1条数据
    -----------test @Update-----------
    成功更新1条数据
    -----------test @Delete-----------
    成功删除1条数据
    -----------test @Param-----------
    Student [sid=2, sname=LiSi, age=21, course=Java]
    ```

## 本章小结
- 本章首先介绍了MyBatis中动态SQL的相关知识，详细演示了常见动态SQL元素的使用方法
- 然后介绍了MyBatis中注解的概念及具体使用方法
- 通过本章知识的学习，应该理解动态SQL和注解的概念，掌握动态SQL和注解的使用方法，能够根据不同的需求场景灵活实现表与POJO 对象的映射关系。
