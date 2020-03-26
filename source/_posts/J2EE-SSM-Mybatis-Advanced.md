---
title: JavaEE(SSM) MyBatis进阶
top: false
cover: false
img: 'http://q7qon7hdm.bkt.clouddn.com/medias/featureimages/ssm.jpg'
toc: true
mathjax: true
date: 2020-03-26 16:41:17
password:
summary: Java EE (SSM) 企业应用实践 第2章 MyBatis进阶
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

# 第2章 MyBatis进阶

## 学习目标
1. 理解MyBatis的配置
2. 理解MyBatis的映射
3. 掌握MyBatis配置文件的编写方法
4. 掌握MyBatis映射文件的编写方法

## MyBatis的配置文件

### 配置文件的结构
- 配置文件对MyBatis的整个运行体系产生影响，包含了很多控制MyBatis功能的重要信息，是MyBatis实现功能的重要保证。
- 在开发过程中，当需要更改MyBatis的配置信息时，只需更改配置文件中的相关元素即可。
- MyBatis规定了其配置文件的层次结构，配置文档的顶层结构如下：
    ```
    configuration（配置）
        properties（属性）
        settings（设置）
        typeAliases（类型别名）
        typeHandlers（类型处理器）
        objectFactory（对象工厂）
        plugins（插件）
        environments（环境配置）
            environment（环境变量）
                transactionManager（事务管理器）
                dataSource（数据源）
        databaseIdProvider（数据库厂商标识）
        mappers（映射器）
    ```
- **MyBatis配置文件的元素在文件中的先后顺序是固定的**，打乱顺序会造成MyBatis解析配置文件时报错
- 在IDE中，在相应元素的位置上按F2会给出顺序的提示

### &lt;properties&gt;元素
- &lt;properties&gt;是一个用于配置属性的元素，MyBatis支持&lt;properties&gt;元素的两种配置方式：
    - &lt;property&gt;子元素
    - properties文件
1. 通过&lt;property&gt;子元素
    - &lt;properties&gt;通过其子元素&lt;property&gt;完成属性传递，在MyBatis的配置文件中添加&lt;properties&gt;元素，具体代码如下：
    ```xml
    <properties>
        <property name="driver" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/chapter02"/>
        <property name="username" value="root"/>
        <property name="password" value="jiabi"/>
    </properties>
    ```
    - 在完成上述配置之后，&lt;dataSource&gt;元素的代码可直接引用&lt;properties&gt;元素中的信息，具体代码如下：
    ```xml
    <dataSource type="POOLED">
        <property name="driver" value="${driver}"/>
        <property name="url" value="${url}"/>
        <property name="username" value="${username}"/>
        <property name="password" value="${password}"/>
    </dataSource>
    ```
2. properties文件
    - 在src目录下新建一个db.properties文件，具体代码如下：
    ```
    jdbc.driver = com.mysql.jdbc.Driver
    jdbc.url = jdbc:mysql://localhost:3306/chapter01
    jdbc.username = root
    jdbc.password = jiabi
    ```
    - 在MyBatis的配置文件中加入&lt;properties&gt;元素，具体代码如下：
    ```xml
    <properties resource="db.properties">

    </properties>
    ```
    此时，&lt;dataSource&gt;元素的代码可直接引用&lt;properties&gt;元素中的信息，如此一来&lt;properties&gt;元素通过properties文件实现参数配置。具体代码如下：
    ```xml
    <dataSource type="POOLED">
        <property name="driver" value="${driver}"/>
        <property name="url" value="${url}"/>
        <property name="username" value="${username}"/>
        <property name="password" value="${password}"/>
    </dataSource>
    ```
    - 如果一个属性在不只一个地方进行了配置，那么，MyBatis 将按照下面的顺序来加载：
        - 首先读取在 properties 元素体内指定的属性。
        - 然后根据 properties 元素中的 resource 属性读取类路径下属性文件，或根据 url 属性指定的路径读取属性文件，并覆盖之前读取过的同名属性。
        - 最后读取作为方法参数传递的属性，并覆盖之前读取过的同名属性。  

        因此，**通过方法参数传递的属性具有最高优先级，resource/url 属性中指定的配置文件次之，最低优先级的则是 properties 元素中指定的属性。**  

### &lt;settings&gt;元素

- &lt;settings&gt;元素是 MyBatis 中极为重要的调整设置，它们会改变 MyBatis 的运行时行为。  
 详表见[MyBatis官方文档](https://mybatis.org/mybatis-3/zh/configuration.html)

### &lt;typeAliases&gt;元素

- 由于类的完全限定名较长，为了简化开发、降低编码的烦琐度，MyBatis支持使用别名。别名就是为类设置一个简短的名字，别名存在的意义是为了减少类完全限定名造成的冗余,方便开发人员编程。
- 别名的设置一般通过配置文件中的&lt;typeAliases&gt;元素元素进行,具体示例代码如下：
    ```XML
    <typeAliases>
        <typeAlias type="com.qfedu.pojo.Student" alias="stu" />
    </typeAliases>
    ```
- 在以上示例代码中，&lt;typeAliases&gt;元素包含一个子元素&lt;typeAlias&gt;,&lt;typeAlias&gt;元素的type属性用于指定被设走别名类的完全限定名，alias属性用于指定类的别名。完成配置后，可以在很多场景下以别名stu代替com.qfedu.pojo.Student,由此一来,开发人员的工作量大大降低，也提升了程序的可读性和维护性。  
- 除此之外，MyBatis还支持通过扫描包的形式设置别名，每一个在包com.qfedu.pojo中的 Java Bean，在没有注解的情况下，会使用 Bean 的首字母小写的非限定类名来作为它的别名。具体代码如下：
    ```xml
    <typeAliases>
        <package name="com.qfedu.pojo"/>
    </typeAliases>
    ```
    在本例中，com.qfedu.pojo.Student类的别名是student。
- 若有注解，则别名为其注解值。注解形式为：
    ```java
    @Alias(value = "stu")
    public class Student {

    }
    ```
    在本例中，com.qfedu.pojo.Student类的别名是stu。
- 为了开发方便，MyBatis为一些常见的 Java 类型内建了类型别名  
详表见[MyBatis官方文档](https://mybatis.org/mybatis-3/zh/configuration.html)  

### &lt;typeHandlers&gt;元素

- MyBatis在设置预处理语句（PreparedStatement）中的参数或从结果集中取出一个值时，都会用typeHandlers(类型处理器)将获取到的值以合适的方式转换成Java类型。
- MyBatis内建了一些默认的类型处理器
- 可以重写已有的类型处理器或创建新的类型处理器来处理不支持的或非标准的类型。 
    - 自定义的类型处理器需要实现 org.apache.ibatis.type.TypeHandler 接口   
    或者继承一个基类 org.apache.ibatis.type.BaseTypeHandler
    - 可以通过两种方式查找类型处理器
        ```xml
        <typeHandlers>
            <typeHandler javaType = "String" jdbcType = "VARCHAR" handler="com.qfedu.handler.StudentTypeHandler"/>
            <package name="com.qfedu.handler"/>
        </typeHandlers>
        ```
    - 可以通过两种方式来指定关联的javaType：
        - 在类型处理器的配置元素（typeHandler 元素）上增加一个 javaType 属性  
        比如：javaType="String"
        - 在类型处理器的类上增加一个 @MappedTypes 注解指定与其关联的 Java 类型列表。 
        - **如果在 javaType 属性中也同时指定，则注解上的配置将被忽略。**
    - 可以通过两种方式来指定关联的jdbcType：
        - 在类型处理器的配置元素上增加一个 jdbcType 属性  
        比如：jdbcType="VARCHAR"
        - 在类型处理器的类上增加一个 @MappedJdbcTypes 注解指定与其关联的 JDBC 类型列表。 
        - **如果在 jdbcType 属性中也同时指定，则注解上的配置将被忽略。**

### &lt;objectFactory&gt;元素

- 每次 MyBatis 创建结果对象的新实例时，它都会使用一个对象工厂（ObjectFactory）实例来完成实例化工作。 默认的对象工厂需要做的仅仅是实例化目标类，要么通过默认无参构造方法，要么通过存在的参数映射来调用带有参数的构造方法。 
- 如果想覆盖对象工厂的默认行为，可以通过创建自己的对象工厂来实现。
- 自定义的objectFactory需要实现 ObjectFactory 接口   
    或者继承一个基类 DefaultObjectFactory
- MyBatis通过&lt;objectFactory&gt;元素配置自定义的objectFactory，如：
    ```xml
    <objectFactory type = "com.qfedu.factory.StudentObjectFactory"/>
    ```  

### &lt;environments&gt;元素

- MyBatis 可以配置成适应多种环境，这种机制有助于将 SQL 映射应用于多种数据库之中， 现实情况下有多种理由需要这么做。  
例如，开发、测试和生产环境需要有不同的配置；或者想在具有相同 Schema 的多个生产数据库中使用相同的 SQL 映射。还有许多类似的使用场景。
- **尽管可以配置多个环境，但每个 SqlSessionFactory 实例只能选择一种环境。**
- &lt;environments&gt;元素定义了如何配置环境
    ```xml
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC">
                <property name="..." value="..."/>
            </transactionManager>

            <dataSource type="POOLED">
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>
    ```
    - 注意一些关键点:
        - 默认使用的环境 ID（比如：default="development"）。
        - 每个 environment 元素定义的环境 ID（比如：id="development"）。
        - 事务管理器的配置（比如：type="JDBC"）。
        - 数据源的配置（比如：type="POOLED"）。
    - 默认环境和环境 ID 顾名思义。 环境可以随意命名，但务必保证默认的环境 ID 要匹配其中一个环境 ID。

#### 事务管理器（transactionManager）
- 在 MyBatis 中有两种类型的事务管理器（也就是 type="[JDBC|MANAGED]"）
- JDBC   
这个配置直接使用了 JDBC 的提交和回滚设施，它依赖从数据源获得的连接来管理事务作用域。
- MANAGED  
这个配置几乎没做什么。它从不提交或回滚一个连接，而是让容器来管理事务的整个生命周期（比如 JEE 应用服务器的上下文）, 默认情况下它会关闭连接。

#### 数据源（dataSource）
- dataSource 元素使用标准的 JDBC 数据源接口来配置 JDBC 连接对象的资源。
- 有三种内建的数据源类型（也就是 type="[UNPOOLED|POOLED|JNDI]"）
- UNPOOLED  
    这个数据源的实现会每次请求时打开和关闭连接。虽然有点慢，但对那些数据库连接可用性要求不高的简单应用程序来说，是一个很好的选择。   
    性能表现则依赖于使用的数据库，对某些数据库来说，使用连接池并不重要，这个配置就很适合这种情形。  
    UNPOOLED 类型的数据源需要配置以下 5 种属性：
    - driver  
    这是 JDBC 驱动的 Java 类全限定名（并不是 JDBC 驱动中可能包含的数据源类）。
    - url  
    这是数据库的 JDBC URL 地址。
    - username    
    登录数据库的用户名。
    - password   
    登录数据库的密码。
    - defaultTransactionIsolationLevel  
    默认的连接事务隔离级别。
    - defaultNetworkTimeout  
    等待数据库操作完成的默认网络超时时间（单位：毫秒）。
- POOLED  
    这种数据源的实现利用“池”的概念将 JDBC 连接对象组织起来，避免了创建新的连接实例时所必需的初始化和认证时间。 这种处理方式很流行，能使并发 Web 应用快速响应请求。  
    除了上述提到 UNPOOLED 下的属性外，还有更多属性用来配置 POOLED 的数据源
    - poolMaximumActiveConnections   
    在任意时间可存在的活动（正在使用）连接数量，默认值：10
    - poolMaximumIdleConnections   
    任意时间可能存在的空闲连接数。
    - poolMaximumCheckoutTime  
    在被强制返回之前，池中连接被检出（checked out）时间  
    默认值：20000 毫秒（即 20 秒）
    - poolTimeToWait  
    这是一个底层设置，如果获取连接花费了相当长的时间，连接池会打印状态日志并重新尝试获取一个连接（避免在误配置的情况下一直失败且不打印日志）  
    默认值：20000 毫秒（即 20 秒）。
    - poolMaximumLocalBadConnectionTolerance   
    这是一个关于坏连接容忍度的底层设置， 作用于每一个尝试从缓存池获取连接的线程。 如果这个线程获取到的是一个坏的连接，那么这个数据源允许这个线程尝试重新获取一个新的连接，但是这个重新尝试的次数不应该超过 poolMaximumIdleConnections 与 poolMaximumLocalBadConnectionTolerance 之和。   
    默认值：3（新增于 3.4.5）
    - poolPingQuery  
    发送到数据库的侦测查询，用来检验连接是否正常工作并准备接受请求。默认是“NO PING QUERY SET”，这会导致多数数据库驱动出错时返回恰当的错误消息。
    - poolPingEnabled  
    是否启用侦测查询。  
    若开启，需要设置 poolPingQuery 属性为一个可执行的 SQL 语句（最好是一个速度非常快的 SQL 语句）  
    默认值：false。
    - poolPingConnectionsNotUsedFor  
    配置 poolPingQuery 的频率。  
    可以被设置为和数据库连接超时时间一样，来避免不必要的侦测   
    默认值：0（即所有连接每一时刻都被侦测 — 当然仅当 poolPingEnabled 为 true 时适用）。
- JNDI   
    这个数据源实现是为了能在如 EJB 或应用服务器这类容器中使用，容器可以集中或在外部配置数据源，然后放置一个 JNDI 上下文的数据源引用。  
    这种数据源配置只需要两个属性：
    - initial_context   
    这个属性用来在 InitialContext 中寻找上下文（即，initialContext.lookup(initial_context)）。  
    这是个可选属性，如果忽略，那么将会直接从 InitialContext 中寻找 data_source 属性。
    - data_source   
    这是引用数据源实例位置的上下文路径。  
    提供了 initial_context 配置时会在其返回的上下文中进行查找，没有提供时则直接在 InitialContext 中查找。

### &lt;mappers&gt;元素 

- 既然 MyBatis 的行为已经由上述元素配置完了，我们现在就要来定义 SQL 映射语句了。   
但首先，我们需要告诉 MyBatis 到哪里去找到这些语句。  
可以使用相对于类路径的资源引用，或完全限定资源定位符（包括 file:/// 形式的 URL），或类名和包名等。      
例如：
    ```xml
    <!-- 使用相对于类路径的资源引用 -->
    <mappers>
        <mapper resource="org/mybatis/builder/AuthorMapper.xml"/>
    </mappers>
    ```
    ```xml
    <!-- 使用完全限定资源定位符（URL） -->
    <mappers>
        <mapper url="file:///var/mappers/AuthorMapper.xml"/>
    </mappers>
    ```
    ```xml
    <!-- 使用映射器接口实现类的完全限定类名 -->
    <mappers>
        <mapper class="org.mybatis.builder.AuthorMapper"/>
    </mappers>
    ```
    ```xml
    <!-- 将包内的映射器接口实现全部注册为映射器 -->
    <mappers>
        <package name="org.mybatis.builder"/>
    </mappers>
    ```
- 这些配置会告诉 MyBatis 去哪里找映射文件，剩下的细节就应该是每个 SQL 映射文件了，也就是接下来我们要讨论的。

## MyBatis的映射文件

### 映射文件的结构

- SQL 映射文件只有很少的几个顶级元素（按照应被定义的顺序列出）：
    - cache  
    该命名空间的缓存配置。
    - cache-ref  
    引用其它命名空间的缓存配置。
    - resultMap  
    描述如何从数据库结果集中加载对象，是最复杂也是最强大的元素。
    - ~~parameterMap~~  
    ~~老式风格的参数映射。此元素已被废弃，并可能在将来被移除！请使用行内参数映射。~~
    - sql  
    可被其它语句引用的可重用语句块。
    - insert  
    映射插入语句。
    - update  
    映射更新语句。
    - delete  
    映射删除语句。
    - select  
    映射查询语句。

### &lt;select&gt;元素 

查询语句是 MyBatis 中最常用的元素之一  
光能把数据存到数据库中价值并不大，还要能重新取出来才有用，多数应用也都是查询比修改要频繁。   
MyBatis 的基本原则之一是：在每个插入、更新或删除操作之间，通常会执行多个查询操作。因此，MyBatis 在查询和结果映射做了相当多的改进。  
- 一个简单查询的 select 元素是非常简单的。比如：
    ```sql
    <select id="selectPerson" parameterType="int" resultType="hashmap">
        SELECT * FROM PERSON WHERE ID = #{id}
    </select>
    ```
    这个语句名为 selectPerson，接受一个 int（或 Integer）类型的参数，并返回一个 HashMap 类型的对象，其中的键是列名，值便是结果行中的对应值。
- #{id}  
    这就告诉 MyBatis 创建一个预处理语句（PreparedStatement）参数  
    在 JDBC 中，这样的一个参数在 SQL 中会由一个“?”来标识，并被传递到一个新的预处理语句中，就像这样：
    ```java
    // 近似的 JDBC 代码，非 MyBatis 代码...
    String selectPerson = "SELECT * FROM PERSON WHERE ID=?";
    PreparedStatement ps = conn.prepareStatement(selectPerson);
    ps.setInt(1,id);
    ```
当然，使用 JDBC 就意味着使用更多的代码，以便提取结果并将它们映射到对象实例中，而这就是 MyBatis 的拿手好戏。  
select 元素允许你配置很多属性来配置每条语句的行为细节。详表见[MyBatis官方文档](https://mybatis.org/mybatis-3/zh/sqlmap-xml.html#select)  

### &lt;insert&gt;元素、&lt;update&gt;元素和&lt;delete&gt;元素

- 数据变更语句 insert，update 和 delete 的实现非常接近：
    ```xml
    <insert
    id="insertAuthor"
    parameterType="domain.blog.Author"
    flushCache="true"
    statementType="PREPARED"
    keyProperty=""
    keyColumn=""
    useGeneratedKeys=""
    timeout="20">

    <update
    id="updateAuthor"
    parameterType="domain.blog.Author"
    flushCache="true"
    statementType="PREPARED"
    timeout="20">

    <delete
    id="deleteAuthor"
    parameterType="domain.blog.Author"
    flushCache="true"
    statementType="PREPARED"
    timeout="20">
    ```
- Insert, Update, Delete 元素的属性  

|属性  | 描述 |
|:----|:----|
id |	在命名空间中唯一的标识符，可以被用来引用这条语句。
parameterType |	将会传入这条语句的参数的类全限定名或别名。这个属性是可选的，因为 MyBatis |可以通过类型处理器（TypeHandler）推断出具体传入语句的参数，默认值为未设置（unset）。
~~parameterMap~~ |	~~用于引用外部 parameterMap 的属性，目前已被废弃。请使用行内参数映射和 parameterType 属性。~~
flushCache |	将其设置为 true 后，只要语句被调用，都会导致本地缓存和二级缓存被清空，默认值：（对 insert、update 和 delete 语句）true。
timeout	 | 这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认值为未设置（unset）（依赖数据库驱动）。
statementType |	可选 STATEMENT，PREPARED 或 CALLABLE。这会让 MyBatis 分别使用 Statement，PreparedStatement 或 CallableStatement，默认值：PREPARED。
useGeneratedKeys |	**仅适用于 insert 和 update**这会令 MyBatis 使用 JDBC 的 getGeneratedKeys  | 方法来取出由数据库内部生成的主键（比如：像 MySQL 和 SQL Server 这样的关系型数据库管理系统的自动递增字段），默认值：false。
keyProperty	| **仅适用于 insert 和 update**指定能够唯一识别对象的属性，MyBatis 会使用 getGeneratedKeys 的返回值或 insert 语句的 selectKey 子元素设置它的值，默认值：未设置（unset）。如果生成列不止一个，可以用逗号分隔多个属性名称。
keyColumn |	**仅适用于 insert 和 update**设置生成键值在表中的列名，在某些数据库（像 PostgreSQL）中，当主键列不是表中的第一列的时候，是必须设置的。如果生成列不止一个，可以用逗号分隔多个属性名称。
databaseId |	如果配置了数据库厂商标识（databaseIdProvider），MyBatis 会加载所有不带 databaseId 或匹配当前 databaseId 的语句；如果带和不带的语句都有，则不带的会被忽略。

- 下面是 insert，update 和 delete 语句的示例
    ```xml
    <insert id="insertAuthor">
        insert into Author (id,username,password,email,bio)
        values (#{id},#{username},#{password},#{email},#{bio})
    </insert>

    <update id="updateAuthor">
        update Author set
            username = #{username},
            password = #{password},
            email = #{email},
            bio = #{bio}
        where id = #{id}
    </update>

    <delete id="deleteAuthor">
        delete from Author where id = #{id}
    </delete>
    ```
- 如前所述，插入语句的配置规则更加丰富，在插入语句里面有一些额外的属性和子元素用来处理主键的生成，并且提供了多种生成方式。

    - 首先，如果你的数据库支持自动生成主键的字段（比如 MySQL 和 SQL Server），那么你可以设置 useGeneratedKeys=”true”，然后再把 keyProperty 设置为目标属性就 OK 了。     
    例如，如果上面的 Author 表已经在 id 列上使用了自动生成，那么语句可以修改为：
        ```xml
        <insert id="insertAuthor" useGeneratedKeys="true"
            keyProperty="id">
            insert into Author (username,password,email,bio)
                values (#{username},#{password},#{email},#{bio})
        </insert>
        ```
    - 如果你的数据库还支持多行插入, 你也可以传入一个 Author 数组或集合，并返回自动生成的主键。
        ```xml
        <insert id="insertAuthor" useGeneratedKeys="true"
            keyProperty="id">
            insert into Author (username, password, email, bio) values
            <foreach item="item" collection="list" separator=",">
                (#{item.username}, #{item.password}, #{item.email}, #{item.bio})
            </foreach>
        </insert>
        ```

### &lt;sql&gt;元素

这个元素可以用来定义可重用的 SQL 代码片段，以便在其它语句中使用。 参数可以静态地（在加载的时候）确定下来，并且可以在不同的 include 元素中定义不同的参数值。  
比如：
```xml
<sql id="userColumns"> 
    ${alias}.id,${alias}.username,${alias}.password 
</sql>
```
这个 SQL 片段可以在其它语句中使用，例如：
```xml
<select id="selectUsers" resultType="map">
    select
        <include refid="userColumns">
            <property name="alias" value="t1"/>
        </include>,
        <include refid="userColumns">
            <property name="alias" value="t2"/>
        </include>
    from some_table t1
        cross join some_table t2
</select>
```
也可以在 include 元素的 refid 属性或内部语句中使用属性值，例如：
```xml
<sql id="sometable">
    ${prefix}Table
</sql>

<sql id="someinclude">
    from
        <include refid="${include_target}"/>
</sql>

<select id="select" resultType="map">
    select
        field1, field2, field3
    <include refid="someinclude">
        <property name="prefix" value="Some"/>
        <property name="include_target" value="sometable"/>
    </include>
</select>
```

### &lt;resultMap&gt;元素

resultMap 元素是MyBatis中最重要最强大的元素。更具体地内容将在下一章 MyBatis的关联映射 学习。

## 本章小结

- 本章首先讲解了MyBatis中配置文件的作用、结构以及重要元素  
- 其中，重点讲解了各个元素的功能和用法；然后讲解了 MyBati的映射文件,包括映射文件的结构、映射文件中重要元素的功能和用法等。  
- 通过本章知识的学习，应该理解配置文件和映射文件在MyBatis体系中的角色和地位,掌握配置文件和映射文件的编写和使用,能够灵活利用配置文件和映射文件编写一些MyBatis程序。
