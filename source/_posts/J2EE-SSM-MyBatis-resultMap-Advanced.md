---
title: JavaEE(SSM) MyBatis关联映射
top: false
cover: false
img: 'http://q7qon7hdm.bkt.clouddn.com/medias/featureimages/ssm.jpg'
toc: true
mathjax: true
date: 2020-03-27 01:30:33
password:
summary: Java EE (SSM) 企业应用实践 第3章 MyBatis关联映射
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

# 第3章 MyBatis关联映射

## 学习目标

1. 理解表与表之间的关系
2. 掌握一对一关系的映射方法
3. 掌握一对多关系的映射方法
4. 掌握多对多关系的映射方法

## 表与表之间的关系

- 在学习MyBatis的关联映射前，首先要了解表与表之间的关系。表与表之间的关系主要包括一对一、一对多和多对多等
- 在一对一关系中，一方数据表中的一条记录最多可以和另一方数据表中的一条记录相关  
    例如，学生与校园卡就属于一对一的关系，一个学生只能拥有一张有效的校园卡，一张有效的校园卡只能属于一个学生  
- 在一对多关系中，主键数据表中的一条记录可以和另外一张数据表的多条记录相关  
    例如，班级与学生的关系就属于一对多的关系，一个班级可以有很多学生，一个学生只能属于一个班级
- 在多对多关系中，两个数据表里的每条记录都可以和另一方数据表里任意数量的记录相关  
    例如，学生与教师就属于多对多的关系，一名学生可以由多名教师授课，一名教师可以为多名学生授课  
- 如果直接通过SQL语句维护数据表
    - 在维护一对一的表关系时，通常采用任意一方引入对方的主键作为外键的方式  
    - 在维护一对多的表关系时，通常采用在"多"方引入"一"方的主键作为外键的方式  
    - 在维护多对多的表关系时，通常采用中间表的方式


## 一对一

### 基础知识  

- 先了解一下&lt;resultMap&gt;元素  
    ResultMap 的设计思想是，对简单的语句做到零配置，对于复杂一点的语句，只需要描述语句之间的关系就行了。  
- 可以不用显式地配置&lt;resultMap&gt;  
    也可以显式使用外部的&lt;resultMap&gt;，这也是解决列名不匹配的另外一种方式。
    - MyBatis 会在幕后自动创建一个&lt;resultMap&gt;，再根据属性名来映射列到JavaBean的属性上。  
    如果列名和属性名不能匹配上，可以在 SELECT 语句中设置列别名来完成匹配  
    比如：
        ```xml
        <select id="selectUsers" resultType="User">
            select
                user_id             as "id",
                user_name           as "userName",
                hashed_password     as "hashedPassword"
            from some_table
            where id = #{id}
        </select>
        ```
    - 也可以显式使用外部的&lt;resultMap&gt;  
    比如：  
        ```xml
        <resultMap id="userResultMap" type="User">
            <id property="id" column="user_id" />
            <result property="username" column="user_name"/>
            <result property="password" column="hashed_password"/>
        </resultMap>
        ```
        然后在引用它的语句中设置 resultMap 属性就行了  
        ```xml
        <select id="selectUsers" resultMap="userResultMap">
            select user_id, user_name, hashed_password
            from some_table
            where id = #{id}
        </select>
        ```
- 下面列出&lt;resultMap&gt;元素的概念视图  
    - constructor - 用于在实例化类时，注入结果到构造方法中  
        - idArg - ID 参数；标记出作为 ID 的结果可以帮助提高整体性能
        - arg - 将被注入到构造方法的一个普通结果
    - id – 一个 ID 结果；标记出作为 ID 的结果可以帮助提高整体性能
    - result – 注入到字段或 JavaBean 属性的普通结果
    - association – 一个复杂类型的关联；许多结果将包装成这种类型
        - 嵌套结果映射 – 关联可以是 resultMap 元素，或是对其它结果映射的引用
    - collection – 一个复杂类型的集合
        - 嵌套结果映射 – 集合可以是 resultMap 元素，或是对其它结果映射的引用
    - discriminator – 使用结果值来决定使用哪个 resultMap
        - case – 基于某些值的结果映射
            - 嵌套结果映射 – case 也是一个结果映射，因此具有相同的结构和元素；或者引用其它的结果映射
- 下面列出&lt;resultMap&gt;元素的属性列表  

    属性    |   描述
     :----  | :----  
    id	|   当前命名空间中的一个唯一标识，用于标识一个结果映射。
    type    |	类的完全限定名, 或者一个类型别名
    autoMapping	    |   如果设置这个属性，MyBatis 将会为本结果映射开启或者关闭自动映射。 这个属性会覆盖全局的属性 autoMappingBehavior。默认值：未设置（unset）。  
- &lt;association&gt;元素
    - 关联（association）元素处理**一对一**类型的关系。   
    比如，在我们的示例中，一个学生有一张有效的校园卡。
    - MyBatis 有两种不同的方式加载关联
        - 嵌套 Select 查询：通过执行另外一个 SQL 映射语句来加载期望的复杂类型。
        - 嵌套结果映射：使用嵌套的结果映射来处理连接结果的重复子集。
- 更详细的内容，参见[MyBatis官方文档](https://mybatis.org/mybatis-3/zh/sqlmap-xml.html#Result_Maps)  

### 实践  

- 数据准备
    - 在MySQL中创建数据库chapter03，在chapter03中创建数据表stu_card、stu并插入数据。  
    其中stu_card用于映射校园卡信息，stu用于映射学生信息     
    SQL语句如下  
    ```sql
    DROP DATABASE IF EXISTS chapter03;
    CREATE DATABASE chapter03;
    USE chapter03;

    CREATE TABLE stu_card(
        cid INT PRIMARY KEY AUTO_INCREMENT,#ID
        balance DOUBLE #卡内余额
    );

    CREATE TABLE stu(
        sid INT PRIMARY KEY AUTO_INCREMENT,#ID
        sname VARCHAR(20),#姓名
        age VARCHAR(20),#年龄
        course VARCHAR(20),#科目
        cardid INT UNIQUE,#校园卡ID
        FOREIGN KEY(cardid) REFERENCES stu_card(cid) #外键
    );

    INSERT INTO stu_card (balance) VALUES (1000.5);
    INSERT INTO stu_card (balance) VALUES (5000.5);
    INSERT INTO stu (sname,age,course,cardid) VALUES ('ZhangSan','20','Java',1);
    INSERT INTO stu (sname,age,course,cardid) VALUES ('LiSi','21','Java',2);
    ```
    - 查询数据是否插入成功
- 创建工程  
    新建工程chapter03，导入相应jar包
    - MyBatis核心jar包：mybatis-3.5.4.jar
    - MyBatis依赖jar包：lib/*
    - MySQL Connector jar包：mysql-connector-java-5.1.48.jar    
- 创建POJO类  
    在工程chapter03的src目录下创建com.qfedu.pojo包  
    在com.qfedu.pojo包下分别创建两个POJO类：Stu和StuCard
    ```java
    package com.qfedu.pojo;

    public class Stu {
        private int sid;
        private String sname;
        private String age;
        private String course;
        private StuCard sc;

        public int getSid() {
            return sid;
        }

        public void setSid(int sid) {
            this.sid = sid;
        }

        public String getSname() {
            return sname;
        }

        public void setSname(String sname) {
            this.sname = sname;
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

        public StuCard getSc() {
            return sc;
        }

        public void setSc(StuCard sc) {
            this.sc = sc;
        }

        @Override
        public String toString() {
            return "Stu [sid=" + sid + ", sname=" + sname + ", age=" + age + ", course=" + course + ", sc=" + sc + "]";
        }

    }
    ```
    ```java
    package com.qfedu.pojo;

    public class StuCard {
        private int cid;
        private double balance;
        public StuCard() {
            super();
            // TODO Auto-generated constructor stub
        }
        public StuCard(int cid, double balance) {
            super();
            this.cid = cid;
            this.balance = balance;
        }
        public int getCid() {
            return cid;
        }
        public void setCid(int cid) {
            this.cid = cid;
        }
        public double getBalance() {
            return balance;
        }
        public void setBalance(double balance) {
            this.balance = balance;
        }
        @Override
        public String toString() {
            return "StuCard [cid=" + cid + ", balance=" + balance + "]";
        }
    }
    ```
- 创建映射文件  
    在工程chapter03的src目录下创建com.qfedu.mapper包  
    在com.qfedu.mapper包下创建两个映射文件：StuCardMapper.xml和StuMapper.xml  
    ```xml
    <!--StuCardMapper.xml-->
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
    PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="stucard">
        <select id="findStuCardBycid" parameterType="Integer" resultType="stuCard">
            select * from stu_card where cid = #{cid}
        </select>
    </mapper>
    ```
    ```xml
    <!--StuMapper.xml-->
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
    PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="stu">
        <select id="findStudentBySid" parameterType="Integer" resultMap="stuResultsMap">
            select s.*, c.balance from stu s inner join  stu_card c
            on s.cardid = c.cid
            where s.sid = #{sid}
        </select>
        <resultMap type="stu" id="stuResultsMap">
            <id column="sid" property="sid" />
            <result column="sname" property="sname" />
            <result column="age" property="age" />
            <result column="course" property="course" />
            <association property="sc" javaType="stuCard">
                <id column="sid" property="cid" />
                <result column="balance" property="balance" />
            </association>
        </resultMap>
    </mapper>
    ```
- 创建配置文件  
    在工程chapter03的src目录下创建mybatis-config.xml
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
                        value="jdbc:mysql://localhost:3306/chapter03" />
                    <property name="username" value="root" />
                    <property name="password" value="jiabi" />
                </dataSource>
            </environment>
        </environments>
        <!--引入映射文件 -->
        <mappers>
            <mapper resource="com/qfedu/mapper/StuMapper.xml"/>
            <mapper resource="com/qfedu/mapper/StuCardMapper.xml"/>
        </mappers>
    </configuration>
    ```
- 创建测试类  
    在工程chapter03的src目录下创建com.qfedu.test包  
    在com.qfedu.test包下创建测试类TestOneToOne
    ```java
    package com.qfedu.test;

    import java.io.InputStream;
    import org.apache.ibatis.io.Resources;
    import org.apache.ibatis.session.*;
    import com.qfedu.pojo.Stu;

    public class TestOneToOne {
        public static void main(String[] args) throws Exception {
            InputStream in = Resources.getResourceAsStream("mybatis-config.xml");
            SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(in);
            SqlSession session = sessionFactory.openSession();

            // 使用MyBatis查询结果sid为1的学生信息（包括学生的校园卡信息）
            Stu stu = session.selectOne("stu.findStudentBySid", 1);
            System.out.println(stu);

            session.close();
        }
    }
    ```
- 测试结果
    ```
    Stu [sid=1, sname=ZhangSan, age=20, course=Java, sc=StuCard [cid=1, balance=1000.5]]
    ```

## 一对多  

### 基础知识
- 你已经在上面看到了如何处理**一对一**类型的关联。但是该怎么处理**多对多**类型的关联呢？这就是我们接下来要介绍的。
- &lt;collection&gt;元素  
    集合元素和关联元素几乎是一样的，它们相似的程度之高，以致于没有必要再介绍集合元素的相似部分。 所以让我们来关注它们的不同之处：  
    新的 “ofType” 属性。这个属性非常重要，它用来将 JavaBean（或字段）属性的类型和集合存储的类型区分开来。    
    现在你可能已经猜到了集合的嵌套结果映射是怎样工作的——除了新增的 “ofType” 属性，它和关联的完全相同！

### 实践  

- 集合的嵌套结果映射(一对多)
- 数据准备  
    在数据库chapter03中创建数据表stu_class、stu_info  
    其中stu_class用于映射班级信息stu_info用于映射学生信息  
    SQL语句略
  
- 创建POJO类  
    在com.qfedu.pojo包下分别创建两个POJO类：ClassInfo和StuInfo
    ```java
    package com.qfedu.pojo;

    import java.util.List;

    public class ClassInfo {
        private Integer cid;
        private String cname;
        private Integer sum;
        private List<TeachInfo> teachInfoList;

        public Integer getCid() {
            return cid;
        }

        public void setCid(Integer cid) {
            this.cid = cid;
        }

        public String getCname() {
            return cname;
        }

        public void setCname(String cname) {
            this.cname = cname;
        }

        public Integer getSum() {
            return sum;
        }

        public void setSum(Integer sum) {
            this.sum = sum;
        }

        public List<TeachInfo> getTeachInfoList() {
            return teachInfoList;
        }

        public void setTeachInfoList(List<TeachInfo> teachInfoList) {
            this.teachInfoList = teachInfoList;
        }

        @Override
        public String toString() {
            return "ClassInfo [cid=" + cid + ", cname=" + cname + ", sum=" + sum + ", teachInfoList=" + teachInfoList + "]";
        }
    }
    ```
    ```java
    package com.qfedu.pojo;

    public class StuInfo {
        private int sid;
        private String sname;
        private String age;
        private String course;

        public int getSid() {
            return sid;
        }

        public void setSid(int sid) {
            this.sid = sid;
        }

        public String getSname() {
            return sname;
        }

        public void setSname(String sname) {
            this.sname = sname;
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
            return "Stu [sid=" + sid + ", sname=" + sname + ", age=" + age + ", " + " course=" + course + "]";
        }
    }
    ```
- 创建映射文件   
    在com.qfedu.mapper包下创建映射文件ClassInfoMapper.xml
    - ClassInfoMapper.xml对应的 SQL 语句：
        ```xml
        <select id="findClassInfoByCid" parameterType="Integer" resultMap="classInfoResultsMap">
            select c.*,t.* from class_info c ,teach_info t, class_teach
            ct where
            c.cid=ct.class_id
            and ct.teach_id = t.tid and c.cid = #{cid}
	    </select>
        ```
    - 这里连接了班级信息和学生信息，将结果映射到POJO类ClassInfo  
    就这么简单：
        ```xml
        <resultMap type="classInfo" id="classInfoResultsMap">
            <id column="cid" property="cid" />
            <result column="cname" property="cname" />
            <result column="sum" property="sum" />
            <collection property="teachInfoList" ofType="teachInfo">
                <id column="tid" property="tid" />
                <result column="tname" property="tname" />
                <result column="age" property="age" />
                <result column="course" property="course" />
            </collection>
        </resultMap>
        ```
- 更新配置文件  
    在mybatis-config.xml中引入新的映射文件
    ```xml
    <mapper resource="com/qfedu/mapper/ClassInfoMapper.xml"/>
    ```
- 创建测试类    
    在com.qfedu.test包下创建测试类TestOneToMany
    ```java
    package com.qfedu.test;

    import java.io.InputStream;
    import java.util.List;
    import org.apache.ibatis.io.Resources;
    import org.apache.ibatis.session.*;
    import com.qfedu.pojo.*;

    public class TestOneToMany {
        public static void main(String[] args) throws Exception {
            InputStream in = Resources.getResourceAsStream("mybatis-config.xml");
            SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(in);
            SqlSession session = sessionFactory.openSession();

            // 使用MyBatis查询cid为1的班级信息（包括该班级的学生信息）
            StuClass stuClass = session.selectOne("stuClass.findStuClassByCid", 1);
            System.out.println("班级ID：" + stuClass.getCid() + "\n班级名称：" + stuClass.getCname() 
                + "\n班级人数：" + stuClass.getSum()+ "\n学生信息：");
            List<StuInfo> stuInfoList = stuClass.getStuInfoList();
            for (StuInfo stuInfo : stuInfoList) {
                System.out.println(stuInfo);
            }

            session.close();
        }
    }
    ```

## 多对多

- 通常情况下，多对多关系要转化为一对多的形式进行处理，这种转化是通过一张中间表实现的。  
    在使用MyBatis处理多对多关系时，需要先将多对多关系转化为一对多关系，然后使用&lt;collection&gt;元素完成映射
- 操作与处理一对多关系时相似，示例略

## 主键映射

- 数据表的主键用于标识该表中每条记录的唯一性，使用MyBatis操作数据库应考虑到多表关联下的主键映射问题
- 在实际开发中，当往数据表中插入数据时，如果对于插入数据的主键没有特殊要求，那么可以采用不返回主键值的方式配置插入语句，这样能够避免额外的SQL开销。  
但是实际开发中常遇到一些多表关联下的操作。  
例如，在一次操作中插入具有一对多关系的数据表，当插入多方的数据时，需要获取刚刚插入的一方的主键值，此时就要采用插入后获取主键值的方式配置。
- 第一，如果你的数据库支持自动生成主键的字段（比如 MySQL 和 SQL Server），那么你可以设置 useGeneratedKeys=”true”，然后再把 keyProperty 设置为目标属性就 OK 了。例如，如果下面的Author表已经在 id 列上使用了自动生成，那么语句可以编写为：
    ```xml
    <insert id="insertAuthor" useGeneratedKeys="true" keyProperty="id">
        insert into Author (username,password,email,bio)
        values (#{username},#{password},#{email},#{bio})
    </insert>
    ```
    - 如果数据库还支持多行插入, 也可以传入一个 Author 数组或集合，并返回自动生成的主键。
- 第二，对于不支持自动生成主键列的数据库和可能不支持自动生成主键的 JDBC 驱动，MyBatis 有另外一种方法来生成主键。
    - 这个示例可以生成一个随机 ID（不建议实际使用）
        ```xml
        <insert id="insertAuthor">
            <selectKey keyProperty="id" resultType="int" order="BEFORE">
                select CAST(RANDOM()*1000000 as INTEGER) a from SYSIBM.SYSDUMMY1
            </selectKey>
            insert into Author
                (id, username, password, email,bio, favourite_section)
            values
                (#{id}, #{username}, #{password}, #{email}, #{bio}, #{favouriteSection,jdbcType=VARCHAR})
        </insert>
        ```
    - 在上面的示例中，首先会运行 selectKey 元素中的语句，并设置 Author 的 id，然后才会调用插入语句。这样就实现了数据库自动生成主键类似的行为，同时保持了 Java 代码的简洁。
    - selectKey 元素描述如下：
        ```xml
            <selectKey
                keyProperty="id"
                resultType="int"
                order="BEFORE"
                statementType="PREPARED">
        ```
    - selectKey 元素的属性  

    属性    |    描述
    :---- |:----   
    keyProperty	|   selectKey 语句结果应该被设置到的目标属性。如果生成列不止一个，可以用逗号分隔多个属性名称。
    keyColumn	|   返回结果集中生成列属性的列名。如果生成列不止一个，可以用逗号分隔多个属性名称。
    resultType	|   结果的类型。通常 MyBatis 可以推断出来，但是为了更加准确，写上也不会有什么问题。MyBatis 允许将任何简单类型用作主键的类型，包括字符串。如果生成列不止一个，则可以使用包含期望属性的 Object 或 Map。
    order	|   可以设置为 BEFORE 或 AFTER。如果设置为 BEFORE，那么它首先会生成主键，设置 keyProperty 再执行插入语句。如果设置为 AFTER，那么先执行插入语句，然后是 selectKey 中的语句 - 这和 Oracle 数据库的行为相似，在插入语句内部可能有嵌入索引调用。
    statementType	|   和前面一样，MyBatis 支持 STATEMENT，PREPARED 和 CALLABLE 类型的映射语句，分别代表 Statement, PreparedStatement 和 CallableStatement 类型。

## 本章小结

- 本章首先介绍了表与表之间的关系，其中包括一对一、一对多、多对多等
- 然后分别讲解了如何使用MyBatis实现不同表关系的关联映射
- 最后详细讲解了如何使用MyBatis实现主键映射
- 通过本章知识的学习应该理解表与表之间的关联关系，掌握处理表关系的思路和方法，能够使用MyBatis框架完成关联状态下的多表操作 