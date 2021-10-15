# Mybatis

* 目录
  * [一、简介](#一简介)
    * [1\.1 持久化（动作）](#11-持久化动作)
    * [1\.2 持久层（概念）](#12-持久层概念)
    * [1\.3 为什么需要Mybatis？](#13-为什么需要mybatis)
  * [1\.4 配置解析](#14-配置解析)
    * [1\.4\.1 核心配置文件](#141-核心配置文件)
    * [1\.4\.2 环境配置（environments）](#142-环境配置environments)
    * [1\.4\.3 属性（properties）](#143-属性properties)
    * [1\.4\.4 类型别名（typeAliases）](#144-类型别名typealiases)
    * [1\.4\.5 设置（settings）](#145-设置settings)
    * [1\.4\.6 其他配置](#146-其他配置)
    * [1\.4\.7 映射器（mappers）](#147-映射器mappers)
    * [1\.4\.9 作用域（Scop）和生命周期](#149-作用域scop和生命周期)
  * [1\.5 解决属性名和字段名不一致的问题（ResultMap）](#15-解决属性名和字段名不一致的问题resultmap)
  * [1\.6 日志](#16-日志)
    * [1\.6\.1 日志工厂](#161-日志工厂)
    * [1\.6\.1 LOG4J](#161-log4j)
  * [1\.7 分页](#17-分页)
  * [1\.8 使用注解开发](#18-使用注解开发)
  * [1\.9 MyBatis的执行流程](#19-mybatis的执行流程)
  * [1\.10 多对一处理](#110-多对一处理)
  * [1\.11 一对多处理](#111-一对多处理)
    * [小结](#小结)
  * [1\.12动态 SQL](#112动态-sql)
    * [<strong>if</strong>](#if)
    * [choose（when、otherwise）](#choosewhenotherwise)
    * [where](#where)
    * [trim（where、set）](#trimwhereset)
    * [SQL片段](#sql片段)
    * [foreach](#foreach)
  * [1\.13 缓存](#113-缓存)
  * [1\.13\.1 简介](#1131-简介)
    * [1\.13\.2 MyBatis的缓存](#1132-mybatis的缓存)
      * [一级缓存](#一级缓存)
      * [二级缓存](#二级缓存)
    * [缓存原理](#缓存原理)
      * [缓存顺序](#缓存顺序)
    * [自定义缓存 ehcache](#自定义缓存-ehcache)

##  一、简介



###  1.1 持久化（动作）

数据持久化

- 持久化就是将程序的数据在持久状态和瞬时转台转换的过程
- 内存：**断电即失**
- 数据库（jdbc）、IO文件持久化
- 生活：冷藏、罐头

**为什么需要持久化？**

- 有一些对象，不能让他丢掉
- 内存太贵了

###  1.2 持久层（概念）

Dao层、Service层、Controller层...

- 完成持久化工作的代码块
- 层是界限十分明显

### 1.3 为什么需要Mybatis？

- 帮助程序员将数据存入到数据库中
- 方便
- 传统的JDBC代码太复杂了。简化。
- 优点
  - 简单易学
  - 灵活
  - sql和代码的分离，提高了可维护性。
  - 提供映射标签，支持对象与数据库的orm字段关系映射
  - 提供对象关系映射标签，支持对象关系组建维护提供xml标签，支持编写动态sql。

## 1.4 配置解析

### 1.4.1 核心配置文件

**MyBatis 的配置文件包含了会深深影响 MyBatis 行为的设置和属性信息。**

- configuration（配置）
  - properties（属性）
  - settings（设置）
  - typeAliases（类型别名）
  - typeHandlers（类型处理器）
  - objectFactory（对象工厂）
  - plugins（插件）
  - environments（环境配置）
    - environment（环境变量）
      - transactionManager（事务管理器）
      - dataSource（数据源）
  - databaseIdProvider（数据库厂商标识）
  - mappers（映射器）

### 1.4.2 环境配置（environments）

MyBatis 可以配置成适应多种环境

**尽管可以配置多个环境，但每个 SqlSessionFactory 实例只能选择一种环境。**

Mybatis默认的事务管理器就是JDBC/MANAGED （认识就行），连接池：POOLED（默认）/UNPPPLED/JNDI

### 1.4.3 属性（properties）

我们可以通过properties属性来实现引用配置文件

这些属性都是可外部配置且可动态替换的，既可以在典型的Java属性文件中配置，亦可通过properties元素的子元素来传递。【db.properties】

编写配置文件（db.properties）

```properties
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/spring_test?useSSL=true&useUnicode=true&characterEncoding=utf-8
userName=root
password=123456
```

在核心配置文件中引入

```xml
    <!-- 引入外部配置文件 -->
    <properties resource="db.properties">
        <property name="user" value="root"/>
    </properties>
```

- 可以直接引入外部文件
- 可以增加一些属性配置
- 如果两个文件有 同一个字段，有限使用外部配置文件的

### 1.4.4 类型别名（typeAliases）

- 类型别名是为Java类型设置一个短的名字。
- 存在的意义仅在于用来减少类完全限定名的冗余。

```xml
<!-- 实体类起别名 -->
<typeAliases>
    <typeAlias type="com.zjp.pojo.User" alias="User"/>
</typeAliases>
```

也可以指定一个包名，MyBatis会在包名下面搜索需要的Java Bean，比如：
扫描实体类的包，它的默认别名就为这个类的类名，首字母小写！

```xml
<!-- 实体类起别名 -->
<typeAliases>
    <package name="com.zjp.pojo"/>
</typeAliases>
```

在实体类比较少的时候，使用第一种方式。

如果实体类十分多，建议使用第二种。

第一种可以DIY别名，第二种则·不行·，如果非要改，需要在实体上增加注解

```java
@Alias("user")
public class User {
```

### 1.4.5 设置（settings）

| 设置名             | 描述                                                         | 有效值      | 默认值 |
| :----------------- | :----------------------------------------------------------- | :---------- | :----- |
| cacheEnabled       | 全局性地开启或关闭所有映射器配置文件中已配置的任何缓存。     | true\|false | true   |
| lazyLoadingEnabled | 延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。 特定关联关系中可通过设置 `fetchType` 属性来覆盖该项的开关状态。 | true\|false | false  |
| logImpl            | 指定 MyBatis 所用日志的具体实现，未指定时将自动查找。        |             | 未设置 |

![image-20201029163312719](C:\Users\ZJP\AppData\Roaming\Typora\typora-user-images\image-20201029163312719.png)

### 1.4.6 其他配置

- plugins插件
  - mybatis-generator-core
  - mybatis-plus
  - 通用mapper

### 1.4.7 映射器（mappers）

MapperRegistry：注册绑定我们的Mapper文件；

方式一：

```xml
<mappers>
        <mapper resource="com/zjp/dao/UserMapper.xml"/>
</mappers>
```

方式二：

```xml
<mappers>
    <mapper class="com.zjp.dao.UserMapper"/>
</mappers>
```

注意：

- 接口和他的Mapper配置文件必须同名！
- 接口和他的Mapper配置文件必须在同一个包下！

方式三：使用扫描包进行注入

```xml
<mappers>
    <package name="com.zjp.dao"/>
</mappers>
```

### 1.4.9 作用域（Scop）和生命周期

生命周期，和作用域，是至关重要的，因为错误的使用会导致非常严重的**并发问题**。

![image-20201029170727533](C:\Users\ZJP\AppData\Roaming\Typora\typora-user-images\image-20201029170727533.png)

**SqlSessionFactoryBuilder：**

- 一旦创建了SqlSessionFactory，就不再需要它了
- 局部变量

**SqlSessionFactory：**

- 可以现象成为：数据库连接池
- SqlSessionFactory 一旦被创建就应该在应用的运行期间一直存在，**没有任何理由丢弃它或重新创建另一个实例。**
- 因此SqlSessionFactory的最佳作用域是应用作用域。
- 最简单的就是使用**单例模式**或者静态单例模式。

**SqlSession：**

- 连接到连接池的一个请求
- SqlSession 的实例不是线程安全的，因此是不能被共享的，所以它的最佳的作用域是请求或方法作用
- 需要开启和及时关闭，否则资源被占用

 

![image-20201029171547689](C:\Users\ZJP\AppData\Roaming\Typora\typora-user-images\image-20201029171547689.png)

## 1.5 解决属性名和字段名不一致的问题（ResultMap）

User.java

```java
public class User {
    private Integer id;
    private String name;
    private String password;
}
```

![image-20201029172503604](C:\Users\ZJP\AppData\Roaming\Typora\typora-user-images\image-20201029172503604.png)

解决方法：

- 起别名

  ```xml
  <select id="getuserById"resultrype="com. kuang. pojo. user">
  select id, name, pwd as password from mybatis. user where id=#{ id}
  </select>
  ```

- **ResultMap 结果集映射**

  ```xml
  <resultMap id="user" type="user">
          <id property="id" column="id" javaType="java.lang.Integer" jdbcType="INTEGER"/>
          <result property="name" column="name" javaType="java.lang.String" jdbcType="VARCHAR"/>
          <result property="password" column="pwd" javaType="java.lang.String" jdbcType="VARCHAR"/>
      </resultMap>
  ```

  - `resultMap` 元素是 MyBatis 中最重要最强大的元素。

  - ResultMap的设计思想是，对于简单的语句根本不需要配置显式的结果映射，而对于复杂一点的语句只需要描述它们的关系就行了。

  - `ResultMap`最优秀的地方在于，虽然你已经对它相当了解了，但是根本就不需要显式地用到他们。

    ```xml
    <resultMap id="user" type="user">
            <result property="password" column="pwd" javaType="java.lang.String" jdbcType="VARCHAR"/>
        </resultMap>
    ```


## 1.6 日志

### 1.6.1 日志工厂

如果一个数据库操作，出现了异常，我们需要排错。日志就是最好的助手！

曾经：sout、debug

现在：日志工厂！

![image-20201030093846665](C:\Users\ZJP\AppData\Roaming\Typora\typora-user-images\image-20201030093846665.png)

- SLF4J 
-  LOG4J 【掌握】
-  LOG4J2 
-  JDK_LOGGING 
-  COMMONS_LOGGING 
-  STDOUT_LOGGING 【掌握】
-  NO_LOGGING

在MyBatis中具体使用那个日志实现，在设置中设定

**STDOUT_LOGGING 标准日志**

![image-20201030094843031](C:\Users\ZJP\AppData\Roaming\Typora\typora-user-images\image-20201030094843031.png)

### 1.6.1 LOG4J

什么是LOG4J 

- Logj是Apache的一个开源项目，通过使用Logj，我们可以控制日志信息输送的目的地是控制台、文件、GUl组件
- 我们也可以控制每一条日志的输出格式；
- 通过定义每一条日志信息的级别，我们能够更加细致地控制日志的生成过程。
- 通过一个配置文件来灵活地进行配置，而不需要修改应用的代码。

1.先导入LOG4J的包

```xml
<!-- https://mvnrepository.com/artifact/log4j/log4j -->
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```



2.log4j.properties

```properties
#将等级为加EBUG的日志信息输出到consolefile这两个目的地，consolefile的定义在下面的代码
1og4j.rootLogger=DEBUG,console,file
#控制台输出的相关设置
1og4j.appender.console=org.apache.1og4j.ConsoleAppender
1og4j.appender.console.Target=System.out
1og4j.appender.console.Threshold=DEBUG
1og4j.appender.console.layout=org.apache.log4j.PatternLayout
1og4j.appender.console.layout.ConversionPattern=[%c]-%m%n
#文件输出的相关设置
1og4j.appender.file=org.apache.1og4j.RollingFileAppender
1og4j.appender.file.File=./1og/kuang.1og
1og4j.appender.file.MaxFileSize=10mb
1og4j.appender.file.Threshold=DEBUG
1og4j.appender.file.layout=org.apache.1og4j.PatternLayout
1og4j.appender.file.1ayout.ConversionPattern=[%p][%d{yy-MM-dd}][%c]%m%n
1og4j.logger.org.mybatis=DEBUG
1og4j.logger.java.sq1=DEBUG
1og4j.logger.java.sq1.Statement=DEBUG1og4j.logger.java.sql.ResultSet=DEBUG
1og4j.logger.java.sql.PreparedStatement=DEBUG
```

3.配置log4j为日志实现

```xml
<settings>
  	<setting name="logImpl" value="LOG4J2"/>
</settings>
```

## 1.7 分页

1. 使用limit分页

   ```sql
   语法 select * from user limit startIndex,pageSize;
   select * from user limit 3;	#[0,n]
   ```

   

2. Mybatis分页

   map传入startIndex和pageSize给Mapper接口

3. RowBounds分页

   ```java
   @Test
   public void getuserByRowBounds）{
   Sqlsession sqlsession=Mybatisutils.getsqlsession（）；
   //RowBounds实现
   RowBounds rowBounds =new RowBounds（1，2）；
   //通过Java代码层面实现分页
   List<user> userList=
   sqlsession.selectList（"com.kuang.dao.UserMapper.getUserByRowBounds"，nu11，rowBounds
   ）；for（User user:userList）{system.out.println（user）；sqlsession.close（）；
   }
   ```

   

4. 分页插件

   MyBatis分页插件PageHelper

##  1.8 使用注解开发

@Select("select * from user")

## 1.9 MyBatis的执行流程

![image-20201030113638195](C:\Users\ZJP\AppData\Roaming\Typora\typora-user-images\image-20201030113638195.png)

## 1.10 多对一处理

1. 按照查询嵌套处理

```xml
<resultMap id="studentTeacher" type="com.zjp.pojo.Student">
  <id property="id" column="id"/>
  <result property="name" column="name"/>
  <!-- 复杂的属性，外吗需要单独处理     对象： association 集合：collection-->
  <association property="teacher" column="tid" javaType="com.zjp.pojo.Teacher" select="getTeacher"/>
</resultMap>

<select id="getAllStudents" resultMap="studentTeacher">
  SELECT * from spring_test.student
</select>

<select id="getTeacher" resultType="com.zjp.pojo.Teacher">
  SELECT * from spring_test.teacher where id = #{id}
</select>
```

1. **按照结果嵌套处理**

```xml
<resultMap id="studentTeacher2" type="com.zjp.pojo.Student">
  <id property="id" column="sid"/>
  <result property="name" column="sname"/>
  <association property="teacher" javaType="com.zjp.pojo.Teacher">
    <result property="name" column="tname"/>
  </association>
</resultMap>

<!-- 按照结果嵌套处理 -->
<select id="getAllStudents2" resultMap="studentTeacher2">
  SELECT s.id sid,s.name sname,t.name tname from spring_test.student s,spring_test.teacher t
  WHERE s.tid = t.id
</select>
```

## 1.11 一对多处理

1. 按照查询嵌套处理

```xml
<resultMap id="teacherStudent" type="com.zjp.pojo.Teacher">
  <id property="id" column="id"/>
  <result property="name" column="name"/>
  <collection property="students" ofType="Student" column="id" javaType="ArrayList" select="getStudents"/>
</resultMap>

<select id="getTeacherById" resultMap="teacherStudent">
  SELECT * FROM spring_test.teacher WHERE id = #{id};
</select>

<select id="getStudents" resultType="Student">
  SELECT * FROM spring_test.student WHERE tid = #{tid}
</select>
```

1. 2 **按照结果嵌套处理**

```xml
<select id="getTeacherById2" resultMap="teacherStudent2">
  SELECT s.id sid,s.name sname,t.name tname,t.id tid
  FROM spring_test.student s,spring_test.teacher t
  WHERE s.tid = t.id AND t.id = #{id}
</select>

<resultMap id="teacherStudent2" type="com.zjp.pojo.Teacher">
  <id property="id" column="tid"/>
  <result property="name" column="tname"/>
  <collection property="students" javaType="ArrayList" ofType="com.zjp.pojo.Student">
    <id property="id" column="sid"/>
    <result property="name" column="sname"/>
    <result property="tid" column="tid"/>
  </collection>
</resultMap>
```

### 小结

1. 关联 - association	【多对一】

2. 集合 - collection       【一对多】

3. javaType  & ofType

   1.javaType   指定实体类中属性的类型

   2.ofType   指定映射到List会哦在集合中的pojo类，泛型中的约束类型

注意点：

- 保证SQL的可读性，尽量保证通俗易懂
- 注意一对多和多对一中，属性名和字段的问题！
- 如果问题不好排查错误，可以使用日志，建议使用Log4j

==面试高频==

- Mysq引擎
- InnoDB底层原理
- 索引
- 索引优化！

##  1.12动态 SQL

###   **if**

```xml
<select id="findActiveBlogWithTitleLike"
     resultType="Blog">
  SELECT * FROM BLOG
  WHERE state = ‘ACTIVE’
  <if test="title != null">
    AND title like #{title}
  </if>
</select>
```

### choose（when、otherwise）

类似 Java 的 switch case

```xml
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG WHERE 1 = 1
  <choose>
    <when test="title != null">
      AND title like #{title}
    </when>
    <when test="author != null and author != ‘’">
      AND author_name like #{author}
    </when>
    <otherwise>
      AND views = #{views}
    </otherwise>
  </choose>
</select>
```

### where

```xml
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG
  <where>
  <choose>
    <when test="title != null">
      AND title like #{title}
    </when>
    <when test="author != null and author != ‘’">
      AND author_name like #{author}
    </when>
    <otherwise>
      AND views = #{views}
    </otherwise>
  </choose>
  </where>
</select>
```

### trim（where、set）

```xml
<update id="updateAuthorIfNecessary">
  update Author
    <set>
      <if test="username != null">username=#{username},</if>
      <if test="password != null">password=#{password},</if>
      <if test="email != null">email=#{email},</if>
      <if test="bio != null">bio=#{bio}</if>
    </set>
  where id=#{id}
</update>
```

where

```xml
<trim prefix="WHERE" prefixOverrides="AND |OR ">
  ...
</trim>
```

set

```xml
<trim prefix="SET" suffixOverrides=",">
  ...
</trim>
```

### SQL片段

```xml
<sql id="baseSql">
  <if test="username != null">username=#{username},</if>
   <if test="password != null">and password=#{password}</if>
</sql>
```

### foreach

```xml
<select id="selectPostIn" resultType="domain.blog.Post">
  SELECT *
  FROM POST P
  WHERE ID in
  <foreach item="item" index="index" collection="list"
      open="(" separator="," close=")">
        #{item}
  </foreach>
</select>
```

==动态SQL就是在拼接SQL语句，保证SQL的正确性，按照SQL的格式，去排列组合就可以了==

- 在MySQL中写出完整的SQL，再对应的修改成为我们的动态SQL实现通用即可

##  1.13 缓存

##  1.13.1 简介

1. 什么是缓存[Cache]
   - 存在内存中的临时数据
   - 将用户据经常查询的数据放在缓存（内存）中，用户去查询数据就不用从磁盘上（关系型数据库数据文件）查询，从缓存中查询，从而提高查询效率，解决了高并发系统的性能问题
2. 为什么使用缓存
   - 减少和数据库的交互次数，减少系统开销，提高系统效率
3. 什么样的数据能使用缓存？
   - 经常查询并且不经常改变的数据

### 1.13.2 MyBatis的缓存

- Mybatis包含一个非常签到的查询缓存特性，它可以非常方便的定制和配置缓存。缓存可以极大的提升查询效率
- MyBatis系统中默认定义了两级缓存：**一级缓存**和**二级缓存**
  - 默认情况下，只有一级缓存开启（SqlSession级别的缓存，也称为本地缓存）
  - 二级缓存需要手动开启和配置，是基于namespace级别的缓存
  - 为了提高扩展性，MyBatis定义了缓存接口Cache。我们可以通过实现Cache接口来自定义二级缓存

#### 一级缓存

- 一级缓存也叫本地缓存：SqlSession
  - 与数据库同义词会话期间查询到的数据会放在本地缓存中
  - 以后如果需要获取相同的数据，直接从缓存中拿，没有必要再去查询数据库

测试步骤：

1. 开启日志

2. 测试在一个Session中查询两次相同的记录

3. 查看日志输出

   ![image-20201031141510410](C:\Users\ZJP\AppData\Roaming\Typora\typora-user-images\image-20201031141510410.png)

缓存失效的情况：

1. 增删改操作可能会改变愿你的数据，所有必定会刷新缓存！
2. 查询不同的东西
3. 查询不同的mapper.xml
4. 手动清理缓存

![image-20201031143832416](C:\Users\ZJP\AppData\Roaming\Typora\typora-user-images\image-20201031143832416.png)

小结：一级缓存默认是开启的，只在一次SqlSession中有效，也就是拿到连接到关闭连接这个区间段

一级缓存就是一个Map

#### 二级缓存

- 二级缓存也叫全局缓存，一级缓存作用域太低了，所以诞生了二级缓存
- 基于namespace级别的缓存，一个命名空间，对应一个二级缓存
- 工作机制
  - 一个会话查询一条数据，你这个数据就会被放在当前会话的一级缓存中
  - 如果当前会话关闭了，这个会话对应的一级缓存就没了；但是我们想要的是，会话关闭了，一级缓存中的数据就被保存二级缓存中；
  - 新的会话查询信息，就可以从二级缓存中获取内容；
  - 不同的mapper查出的数据会放在自己对应的缓存（map）中

步骤：

1.开启全局缓存

```xml
<!-- 显示开启全局缓存 -->
<setting name="cacheEnabled" value="true"/>
```

2.在要使用二级缓存的mapper.xml文件中开启

```xml
<!-- 在当前mapper.xml文件中使用二级缓存 -->
<cache/>
```

也可以自定义参数

```xml
<!-- 在当前mapper.xml文件中使用二级缓存 -->
<cache eviction="FIFO"
       flushInterval="60000"
       size="512"
       readOnly="true"/>
```

3.测试

​	1.问题：我们需要将实体类序列化！否则就会报错

小结：

- 只要开启了二级缓存，在同一个Mapper下就有效
- 所有的数据都会想放在一级缓存中，只有当会话提交或者关闭的时候，才会提交到二级缓存中

### 缓存原理

![img](file:///C:\Users\ZJP\Documents\Tencent Files\906030149\Image\C2C\{6F9CC21B-584F-7057-AA53-460AD955B843}.png)

#### 缓存顺序

1. 先看二级缓存中有没有
2. 再看一级缓存中有没有
3. 查询数据库

### 自定义缓存 ehcache

```
Ehcache是一种广泛使用的开源Java分布式缓存。主要面向通用缓存
```

步骤：

1. 导包

   ```xml
   <!-- https://mvnrepository.com/artifact/org.mybatis.caches/mybatis-ehcache -->
   <dependency>
     <groupId>org.mybatis.caches</groupId>
     <artifactId>mybatis-ehcache</artifactId>
     <version>1.1.0</version>
   </dependency>
   ```

2. 在mapper中指定ehcache缓存实现

   ```xml
   <cache type="org.mybatis.caches.ehcache.EhcacheCache"/>
   ```

   

3. ehcache.xml

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:noNamespaceSchemaLocation="http://ehcache.org/ehcache.xsd"
            updateCheck="false">
   
       <!-- diskStore: 为缓存路径，ehcache分为内存和磁盘两级，此属性定义磁盘的缓存位置。参数介绍如下：
        user.home - 用户住目录
        user.dir  - 用户当前工作目录
        java.io.tmpdir - 默认临时文件路径 -->
   
       <diskStore path="./tmpdir/Tmp_EhCache"/>
   
       <defaultCache
               eternal="false"
               maxElementsInMemory="10000"
               overflowToDisk="false"
               diskPersistent="false"
               timeToIdleSeconds="1800"
               timeToLiveSeconds="259200"
               memoryStoreEvictionPolicy="LRU"/>
   
       <cache
               name="cloud_user"
               eternal="false"
               maxElementsInMemory="5000"
               overflowToDisk="false"
               diskPersistent="false"
               timeToIdleSeconds="1800"
               timeToLiveSeconds="1800"
               memoryStoreEvictionPolicy="LRU"/>
   </ehcache>
   ```

   

Redis数据库来做缓存