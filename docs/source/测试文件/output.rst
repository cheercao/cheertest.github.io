1.简介
======

1.1什么是Mybatis
----------------

-  MyBatis 是一款优秀的\ **持久层框架**

-  它支持自定义 SQL、存储过程以及高级映射。

-  MyBatis0 免除了几乎所有的 JDBC 代码以及设置参数和获取结果集

-  MyBatis 可以通过简单的 XML 或注解来配置和映射原始类型、接口和 Java
   POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录

1.2持久化
---------

数据持久化

-  持久化就是将程序的数据在持久状态和瞬时状态转化的过程
-  内存：断电即失
-  数据库（jdbc），io文件持久化

1.3持久层
---------

Dao层，Service层，Controller层…

-  完成持久化工作的代码块
-  层界限十分明显

2.Mybatis实例
=============

.. figure:: assets/image-20220830113753226-16618306776321.png
   :alt: image-20220830113753226

   image-20220830113753226

mybatis依赖

.. container:: sourceCode
   :name: cb1

   .. code:: xml

      <dependency>
                  <groupId>org.mybatis</groupId>
                  <artifactId>mybatis</artifactId>
                  <version>3.5.2</version>
              </dependency>

mybatis-config.xml配置

.. container:: sourceCode
   :name: cb2

   .. code:: xml

      <?xml version="1.0" encoding="UTF-8" ?>
      <!DOCTYPE configuration
              PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
              "http://mybatis.org/dtd/mybatis-3-config.dtd">
      <configuration>
          <environments default="development">
              <environment id="development">
                  <transactionManager type="JDBC"/>
                  <dataSource type="POOLED">
                      <property name="driver" value="com.mysql.jdbc.Driver"/>
                      <property name="url" value="jdbc:mysql://localhost:3306/school?useSSL=true&amp;useUnicode=true&amp;characterEncoding=UTF-8"/>
                      <property name="username" value="root"/>
                      <property name="password" value="123456"/>
                  </dataSource>
              </environment>
          </environments>
          <!--每一个Mapper.XML都需要在Mybatis核心配置文件中注册！-->
          <mappers>
              <mapper resource="test/dao/StudentMapper.xml"/>
          </mappers>
      </configuration>

StudentMapper.xml配置

.. container:: sourceCode
   :name: cb3

   .. code:: xml

      <?xml version="1.0" encoding="UTF-8" ?>
      <!DOCTYPE mapper
              PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
              "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
      <!-- 注意配置namespace的路径 注明绑定的是哪个接口   -->
      <mapper namespace="test.dao.StudentDao">
          <!-- select查询语句  -->
          <!-- id 是 注明绑定的是接口的哪个方法 resultType注明返回数据的类，这里是student类 -->
          <select id="getStudentList" resultType="test.pojo.student">
              select * from school.student
          </select>
      </mapper>

==发生此种情况解决方法如下，以后每次使用Mybatis都需要在pom.xml中添加此配置来防止找不到资源的问题==

.. container:: sourceCode
   :name: cb4

   .. code:: java

      //注意此错在之后可能会经常遇到
      org.apache.ibatis.binding.BindingException: Type interface test.dao.StudentDao is not known to the MapperRegistry. //类型接口在mapper里面没有注册
      //每一个Mapper.XML都需要在Mybatis核心配置文件中注册！
      //找不到资源文件，由于test下的用例不和配置文件在一起
      Could not find resource test/dao/StudentMapper.xml
      //发生此种情况解决方法如下，以后每次使用Mybatis都需要在pom.xml中添加此配置来防止找不到资源的问题
          <!--在Build中配置resources，来防止资源导出失败的问题 -->
          <build>
              <resources>
                  <resource>
                      <directory>src/main/java</directory>
                      <includes>
                          <include>**/*.xml</include>
                          <include>**/*.properties</include>
                      </includes>
                  </resource>

                  <resource>
                      <directory>src/main/resources</directory>
                      <includes>
                          <include>**/*.xml</include>
                          <include>**/*.properties</include>
                      </includes>
                  </resource>
              </resources>
          </build>

**1.先写Mybatis的工具类**

**2.写Mybatis的配置文件**

**3.写实体类**

**4.写实体类的接口和xml配置文件**

**5.写test测试配置文件是否正确**

**切记接口文件的命名和xml文件的命名要一致，否则可能会发送找不到文件的情况**

3.CRUD
======

1.namespace
-----------

namespace中的包名要和Dao/Mapper接口的包名一致

2.select
--------

选择查询语句

-  id：就是对应的namespace中的方法名
-  resultType：Sql语句执行的返回值
-  paramenterType：参数类型

1.编写接口

.. container:: sourceCode
   :name: cb5

   .. code:: java

      //插入一个学生
      int InsertStudent(student stu);

2.编写对应的mapper中的sql语句

.. container:: sourceCode
   :name: cb6

   .. code:: xml

      <!--根据id查询学生-->
      <select id="getStudentById" parameterType="int" resultType="test.pojo.student">
          select * from school.student where id = #{id}
      </select>

3.测试

.. container:: sourceCode
   :name: cb7

   .. code:: java

      @Test
          public void test(){
              //测试查询所有学生
              //第一步：获得SqlSession对象
              SqlSession sqlSession = MybatisUtils.getSqlSession();
              //方式一：getMapper 推荐使用
              // 通过mapper获得接口的实体类，然后就可以直接调用方法
              StudentDao studentDao = sqlSession.getMapper(StudentDao.class);
              List<student> studentList = studentDao.getStudentList();
              //方式二 不推荐使用
             // List<student> studentList = sqlSession.selectList("test.dao.StudentDao.getStudentList");
              for (student student : studentList) {
                  System.out.println(student);
              }
              sqlSession.close();
          }

3.insert
--------

.. container:: sourceCode
   :name: cb8

   .. code:: xml

      <!--插入一个学生-->
          <!--对象中的属性可以直接取出来-->
          <insert id="InsertStudent" parameterType="test.pojo.student" >
              insert into school.student (id,`name`,password,sex,birthday,address,email)
              values (#{id},#{name},#{password},#{sex},#{birthday},#{address},#{email});
          </insert>

4.update
--------

.. container:: sourceCode
   :name: cb9

   .. code:: xml

      <!--修改一个学生信息-->
          <!--对象中的属性可以直接取出来-->
          <update id="updateStudent" parameterType="test.pojo.student">
              update school.student
              set `name` = #{name},
                  password=#{password},
                  sex = #{sex}
              where id = #{id};
          </update>

5.delete
--------

.. container:: sourceCode
   :name: cb10

   .. code:: xml

      <!--删除一个学生的信息-->
      <delete id="deleteStudent" parameterType="int">
          delete from school.student where id=#{id};
      </delete>

注意点：

-  增删改需要提交事务
-  sqlSession需要close关闭，不要忘记

4.配置解析
==========

1.核心配置文件
--------------

-  mybatis-config.xml
-  MyBatis的配置文件包含了会深深影响MyBatis行为的设置和属性信息

.. container:: sourceCode
   :name: cb11

   .. code:: java

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

2.环境配置
----------

MyBatis可以配置多种环境

MyBatis默认的事务管理器就是JDBC，连接池：POOLED

3.属性(properties)
------------------

我们可以通过properties属性来实现引用配置文件

这些属性可以在外部进行配置，并可以进行动态替换。你既可以在典型的 Java
属性文件中配置这些属性，也可以在 properties 元素的子元素中设置

.. figure:: assets/image-20220830165758353.png
   :alt: image-20220830165758353

   image-20220830165758353

4.类型别名（typeAliases）
-------------------------

-  类型别名可为 Java 类型设置一个缩写名字
-  它仅用于 XML 配置，意在降低冗余的全限定类名书写

==在XML中写配置时，各个标签之间由严格的顺序关系==

.. container:: sourceCode
   :name: cb12

   .. code:: xml

      <!--这个别名是为某一个具体的类起别名 -->
      <typeAliases>
              <typeAlias type="test.pojo.student" alias="student"></typeAlias>
      </typeAliases>
      <!--这个别名是将某一个包里面的所有类起别名，别名为将该类的类名第一个字母小写，如果该类第一个字母本身就是小写则别名就是该类自身 如果该类有@Alias注解，则别名为注解名-->
      <typeAliases>
              <package name="test.pojo"/>
          </typeAliases>

在实体类较少时使用第一种，在实体类较多时使用第二种

第一种可以自己取别名，而第二种的别名基本就是类名，如果有注解时则别名为注解名

5.设置
------

.. figure:: assets/image-20220830171243001.png
   :alt: image-20220830171243001

   image-20220830171243001

.. figure:: assets/image-20220830171258160.png
   :alt: image-20220830171258160

   image-20220830171258160

6.映射器（mappers）
-------------------

MapperRegistry：注册绑定我们的接口文件

方式一：【推荐使用】

.. container:: sourceCode
   :name: cb13

   .. code:: xml

      <mappers>
          <mapper resource="test/dao/StudentMapper.xml"/>
      <!--        <mapper class="test.dao.StudentDao"></mapper>-->
      </mappers>

方式二：

.. container:: sourceCode
   :name: cb14

   .. code:: xml

      <mappers>
      <!--        <mapper resource="test/dao/StudentMapper.xml"/>-->
          <mapper class="test.dao.StudentMapper"></mapper>
      </mappers>

注意点：

-  接口和它的mapper配置文件必须同名
-  接口和它的mapper配置文件必须在同一个包下

方式三：使用扫描包注入绑定

.. container:: sourceCode
   :name: cb15

   .. code:: xml

      <mappers>
      <!--        <mapper resource="test/dao/StudentMapper.xml"/>-->
      <!--        <mapper class="test.dao.StudentMapper"></mapper>-->
              <package name="test.dao.StudentMapper"/>
          </mappers>

5.解决属性名和字段名不一致的问题
================================

**通过结果集映射解决问题**

.. figure:: assets/image-20220830175147396.png
   :alt: image-20220830175147396

   image-20220830175147396

.. container:: sourceCode
   :name: cb16

   .. code:: xml

      <!--这里的id可以自定义，这里的type一定是实体类的类名-->
      <resultMap id="sMapper" type="student">
              <result column="password" property="psw"></result>
          </resultMap>
          
          <!--根据id查询学生-->
          <select id="getStudentById" parameterType="int" resultMap="sMapper">
              select * from school.student where id = #{id}
          </select>

6.日志
======

1.日志工厂
----------

Mybatis内置了有日志工厂，可以直接配置使用

在Mybatis核心配置文件中配置日志

.. container:: sourceCode
   :name: cb17

   .. code:: xml

      <settings>
              <setting name="logImpl" value="STDOUT_LOGGING"/>
          </settings>

.. figure:: assets/image-20220830182835192.png
   :alt: image-20220830182835192

   image-20220830182835192

2.log4j配置
-----------

.. container:: sourceCode
   :name: cb18

   .. code:: xml

      #将等级为DEBUG的日志信息输出到console和file这两个目的地，console和file的定义在下面的代码
      log4j.rootLogger=DEBUG,console,file

      #控制台输出的相关设置
      log4j.appender.console = org.apache.log4j.ConsoleAppender
      log4j.appender.console.Target = System.out
      log4j.appender.console.Threshold=DEBUG
      log4j.appender.console.layout = org.apache.log4j.PatternLayout
      log4j.appender.console.layout.ConversionPattern=【%c】-%m%n

      #文件输出的相关设置
      log4j.appender.file = org.apache.log4j.RollingFileAppender
      log4j.appender.file.File=./log/kuang.log
      log4j.appender.file.MaxFileSize=10mb
      log4j.appender.file.Threshold=DEBUG
      log4j.appender.file.layout=org.apache.log4j.PatternLayout
      log4j.appender.file.layout.ConversionPattern=【%p】【%d{yy-MM-dd}】【%c】%m%n

      #日志输出级别
      log4j.logger.org.mybatis=DEBUG
      log4j.logger.java.sql=DEBUG
      log4j.logger.java.sql.Statement=DEBUG
      log4j.logger.java.sql.ResultSet=DEBUG
      log4j.logger.java.sql.PreparedStatement=DEBUG

7.分页
======

xml配置
-------

.. container:: sourceCode
   :name: cb19

   .. code:: xml

      <!--分页查询学生信息-->
      <select id="getStudentByPage" parameterType="map" resultType="test.pojo.student">
          select * from student limit #{startpoint},#{page};
      </select>

Java代码
--------

.. container:: sourceCode
   :name: cb20

   .. code:: java

      public void selectByPage(){
          SqlSession sqlSession = MybatisUtils.getSqlSession();
          StudentMapper mapper = sqlSession.getMapper(StudentMapper.class);
          HashMap<String, Integer> map = new HashMap<String, Integer>();
          map.put("startpoint",0);
          map.put("page",2);
          List<student> studentByPage = mapper.getStudentByPage(map);
          for (student student : studentByPage) {
              System.out.println(student);
          }
          sqlSession.close();
      }

建议使用此种方法，简单易操作，而且是在SQL层面分页，可以适当修改代码实现想要的分页效果

8.使用注解开发
==============

1.面向接口编程
--------------

.. figure:: assets/image-20220830225818045.png
   :alt: image-20220830225818045

   image-20220830225818045

.. figure:: assets/image-20220830225834177.png
   :alt: image-20220830225834177

   image-20220830225834177

2.使用注解开发增删改查
----------------------

.. container:: sourceCode
   :name: cb21

   .. code:: java

      //使用注解进行简单SQL代码编写 ;
      @Select("select * from student where id = #{uid}")
      student getStudentById2(@Param("uid") int id);

使用@Param()可以为参数重命名，之后直接引用重命名之后的参数，这样可以不用在mapper.xml中配置相应的SQL语句，直接在这里就可以直接写好，但这种只适用于简单的SQL语句，对于稍微复杂点的SQL语句还是建议在mapper中配置

**关于@Param()注解**

-  基本类型的参数或者String类型，需要加上
-  引用类型不需要加
-  如果只有一个基本类型的话，可以忽略，但是建议大家都加上
-  我们在SQL中引用的就是我们这里的@Param()中设定的属性名

9.一对多和多对一
================

案例的SQL语句，可以使用这个SQL语句建立数据库来练习

.. container:: sourceCode
   :name: cb22

   .. code:: sql

      CREATE TABLE `teacher` (
        `id` INT(10) NOT NULL,
        `name` VARCHAR(30) DEFAULT NULL,
        PRIMARY KEY (`id`)
      ) ENGINE=INNODB DEFAULT CHARSET=utf8

      INSERT INTO teacher(`id`, `name`) VALUES (1, '秦老师'); 

      CREATE TABLE `student` (
        `id` INT(10) NOT NULL,
        `name` VARCHAR(30) DEFAULT NULL,
        `tid` INT(10) DEFAULT NULL,
        PRIMARY KEY (`id`),
        KEY `fktid` (`tid`),
        CONSTRAINT `fktid` FOREIGN KEY (`tid`) REFERENCES `teacher` (`id`)
      ) ENGINE=INNODB DEFAULT CHARSET=utf8INSERT INTO `student` (`id`, `name`, `tid`) VALUES ('1', '小明', '1'); 
      INSERT INTO `student` (`id`, `name`, `tid`) VALUES ('2', '小红', '1'); 
      INSERT INTO `student` (`id`, `name`, `tid`) VALUES ('3', '小张', '1'); 
      INSERT INTO `student` (`id`, `name`, `tid`) VALUES ('4', '小李', '1'); 
      INSERT INTO `student` (`id`, `name`, `tid`) VALUES ('5', '小王', '1');

**多对一实例，多个学生对应一个老师**

-  多对一主要解决数据不一致的问题
-  在数据库中，学生信息都有基本数据类型组成，如id,name,tid（学生id，学生姓名，老师id），这些都是基本数据类型，数据库可以直接存储
-  但在java实现中，可以将老师作为一个类来作为学生的一个属性，这样就会导致学生类的属性和数据库的存储类型不一致,association可以解决这个问题，可以将association看成一个类，里面的result标签便是类的具体属性，如果类里面还有类就可以递归的使用association属性来表示

.. container:: sourceCode
   :name: cb23

   .. code:: java

      //学生属性
      public class StudentAndTeacher {
          private int id;
          private String name;
          private Teacher teacher;
      }
      // 老师属性
      public class Teacher {
          private int id;
          private String name;
      }

.. container:: sourceCode
   :name: cb24

   .. code:: xml

      <!--多对一，多个学生对应一个老师-->
      <select id="getStudentList1" resultMap="StudentAndTeacher">
          SELECT s.`id` sid,s.`name` sname,t.`name` tname,t.`id` tid
          FROM testsql.student s ,testsql.teacher t
          WHERE s.`tid` = t.`id`
      </select>
      <!--根据上面select查询出来的结果进行进一步处理，处理结果与类属性不一致的问题，最后得到最终结果-->
      <resultMap id="StudentAndTeacher" type="StudentAndTeacher">
          <!--association外面的都是student的属性，而里面的都是teacher的属性，因为一个student包含一个teacher对象，使用需要使用association来实现-->
          <result property="id" column="sid"></result>
          <result property="name" column="sname"></result>
          <!--这里的property是StudentAndTeacher中的集合属性，teacher,这个属性的类型javaType是Teacher-->
          <association property="teacher" javaType="Teacher">
              <result property="id" column="tid"></result>
              <result property="name" column="tname"></result>
          </association>
      </resultMap>

.. figure:: assets/image-20220831104949131.png
   :alt: image-20220831104949131

   image-20220831104949131

**一对多实例，一个老师对应多个学生**

.. container:: sourceCode
   :name: cb25

   .. code:: java

      //老师属性
      public class Teacher {
          private int id;
          private String name;
          private List<StudentAndTeacher> studentAndTeacher;
      }
      //学生属性
      public class StudentAndTeacher {
          private int id;
          private String name;
          private int tid;
      }

.. container:: sourceCode
   :name: cb26

   .. code:: xml

      <!--一对多，一个老师对应多个学生-->
      <select id="getTeacherList" resultMap="StudentTeacher">
          SELECT s.`id` sid,s.`name` sname,t.`name` tname,t.`id` tid
          FROM testsql.student s ,testsql.teacher t
          WHERE s.`tid` = t.`id` and t.`id` = #{tid}
      </select>
      <!--根据上面select查询出来的结果进行进一步处理，处理结果与类属性不一致的问题，最后得到最终结果-->
      <resultMap id="StudentTeacher" type="Teacher">
          <!--collection外面的都是Teacher的属性，而里面的都是student的属性，因为多个student对应一个teacher对象，查询返回的结果会有多个，集合成一组，使用需要使用collection来实现-->
          <result property="id" column="tid"></result>
          <result property="name" column="tname"></result>
          <!--这里的property是StudentAndTeacher中的集合属性，teacher,这个属性的类型javaType是Teacher-->
          <collection property="studentAndTeacher" ofType="StudentAndTeacher">
              <result property="id" column="sid"></result>
              <result property="name" column="sname"></result>
              <result property="tid" column="tid"></result>
          </collection>
      </resultMap>

**注意**

-  association是对象的意思，也就是说，在数据库中一次查询查询出一条结果，但这条结果里面有对象属性，如果查询的结果属性中有对象，那么就需要使用association，association和javatype一起使用，指定对象的类型
-  collection是集合的意思，如果查询的结果属性有集合就用collection,也就是说在数据库中一次查询会有多条结果，也就是说查询的结果是一个集合，collection和oftype使用，指定集合的类型

.. figure:: assets/image-20220831105931036.png
   :alt: image-20220831105931036

   image-20220831105931036

**小结**

1.管理-association【多对一】

2.集合-collection【一对多】

3.javaType & ofType

​1.javaTypr用来指定实体类中的属性的类型

​2.ofType用来指定映射到List或者集合中的pojo类型，泛型中的约束类型！

注意点

-  保证SQL的可读性，保证通俗易懂
-  注意一对多和多对一中，属性名和字段名的问题
-  通过问题不好排查错误，可以使用日志

面试高频

-  Mysql引擎
-  innoDB底层原理
-  索引
-  索引优化

10.动态SQL
==========

**==动态SQL就是指根据不同的条件生成不同的SQL语句==**

.. container:: sourceCode
   :name: cb27

   .. code:: xml

      如果你之前用过 JSTL 或任何基于类 XML 语言的文本处理器，你对动态 SQL 元素可能会感觉似曾相识。在 MyBatis 之前的版本中，需要花时间了解大量的元素。借助功能强大的基于 OGNL 的表达式，MyBatis 3 替换了之前的大部分元素，大大精简了元素种类，现在要学习的元素种类比原来的一半还要少。

      if
      choose (when, otherwise)
      trim (where, set)
      foreach

搭建环境
--------

.. container:: sourceCode
   :name: cb28

   .. code:: sql

      CREATE TABLE `blog`(
      `id` VARCHAR(50) NOT NULL COMMENT '博客id',
      `title` VARCHAR(100) NOT NULL COMMENT '博客标题',
      `author` VARCHAR(30) NOT NULL COMMENT '博客作者',
      `create_time` DATETIME NOT NULL COMMENT '创建时间',
      `views` INT(30) NOT NULL COMMENT '浏览量'
      )ENGINE=INNODB DEFAULT CHARSET=utf8

**utils就是工具类，所有用来帮助实现功能的代码都放在这里，Dao存放接口和xml配置文件，pojo存放实体类**

IF
--

通过这条语句可以实现搜索的功能，即通过某个条件进行搜索而不必写大量的SQL语句

.. figure:: assets/image-20220831114137629.png
   :alt: image-20220831114137629

   image-20220831114137629

.. _select-1:

Select
------

**select只会选择满足条件的一条执行且是从上至下逐一排查，如果有满足的条件则无视后面的条件**

.. figure:: assets/image-20220831114543472.png
   :alt: image-20220831114543472

   image-20220831114543472

Set
---

**set主要用于更新数据库的操作**

.. figure:: assets/image-20220831115015373.png
   :alt: image-20220831115015373

   image-20220831115015373

SQL片段
-------

SQL片段主要用来实现代码复用

1.使用SQL标签抽取公共部分

2.在需要使用的地方使用include标签引用即可

.. figure:: assets/image-20220831115639638.png
   :alt: image-20220831115639638

   image-20220831115639638

注意事项：

-  最好基于单表来定义SQL片段
-  不要存在where标签

Foreach
-------

.. figure:: assets/image-20220831122123089.png
   :alt: image-20220831122123089

   image-20220831122123089

缓存
====

==缓存主要用来解决高并发的性能问题==

**将在数据库中查询到的结果暂存在内存中以防止下次查询时需要使用，而不需要去数据库中查询**

经常查询且经常更改的数据可以使用缓存

Mybatis有一级缓存和二级缓存

**一级缓存**
------------

-  一级缓存也叫本地会话缓存：SqlSession

   -  与数据库同一次会话期间查询到的数据会放在本地缓存中
   -  以后如果需要获取相同的数据，直接从缓存中拿，没必要再去查询数据库

缓存失效的情况：

1.查询不同的东西

2.增删改操作，可能会改变原来的数据，所以必定会刷新缓存

3.查询不同的Mapper.xml

4.手动清除缓存

小结：一级缓存默认开启的，只在一次sqlsession中有效，也就是拿到连接到关闭连接这个区间段一级缓存相当于一个Map

**二级缓存**
------------

-  二级缓存也叫全局缓存，一级缓存作用域太低了，所以诞生了二级缓存
-  基于namespace级别的缓存，一个名称空间，对应一个二级缓存
-  工作机制

   -  一个会话查询一条数据，这个数据就会被放在当前会话的一级缓存中
   -  如果当前会话关闭了，这个会话对应的一级缓存就没了，但我们想要会话关闭了，一级缓存中的数据被保存到二级缓存中
   -  新的会话查询信息，就可以从二级缓存中获取内容
   -  不同的mapper查出的数据会放在自己对应的缓存中

步骤：

​1.开启全局缓存

.. container:: sourceCode
   :name: cb29

   .. code:: xml

      <!--显示开启全局缓存-->
      <setting name="cacheEnabled" value="true"/>

​2.在要使用二级缓存的mapper中开启

.. container:: sourceCode
   :name: cb30

   .. code:: xml

      <!--在当前mapper.xml中使用二级缓存，二级缓存只对同一个mapper有用-->
      <cache
              eviction="FIFO"
              flushInterval="60000"
              size="512"
              readOnly="true"></cache>

​也可以自定义一些参数

小结：

-  只要开启了二级缓存，在同一个Mapper下就有效
-  所有的数据都会先放在一级缓存中
-  只有当会话提交，或者关闭的时候，才会提交到二级缓存中

自定义缓存Ehcache
-----------------

Ehcache是一种广泛使用的开源java分布式缓存，主要面向通用缓存

1.导包

.. container:: sourceCode
   :name: cb31

   .. code:: xml

      <!-- https://mvnrepository.com/artifact/org.mybatis.caches/mybatis-ehcache -->
      <dependency>
          <groupId>org.mybatis.caches</groupId>
          <artifactId>mybatis-ehcache</artifactId>
          <version>1.2.1</version>
      </dependency>

2.在mapper中指定使用ehcache

.. figure:: assets/image-20220831131143123.png
   :alt: image-20220831131143123

   image-20220831131143123

缓存小结
--------

缓存查找顺序：现在二级缓存中查找数据，如果没有则查一级缓存，然后再去数据库中查询数据
