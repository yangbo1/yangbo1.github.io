---
layout: post
title:  "mybatis"
categories: Mybatis
tags:  mybatis ssm
author: yangbo
---

* content
{:toc}
mybatis学习笔记










# 什么是mybatis
MyBatis 本是apache的一个开源项目iBatis, 2010年这个项目由apache software foundation 迁移到了google code，并且改名为MyBatis，实质上Mybatis对ibatis进行一些改进。 后来又迁移到了github。
MyBatis是一个优秀的持久层框架，它对jdbc的操作数据库的过程进行封装，使开发者只需要关注 SQL 本身，而不需要花费精力去处理例如注册驱动、创建connection、创建statement、手动设置参数、结果集检索等jdbc繁杂的过程代码。
Mybatis通过xml或注解的方式将要执行的各种statement（statement、preparedStatemnt、CallableStatement）配置起来，并通过java对象和statement中的sql进行映射生成最终执行的sql语句，最后由mybatis框架执行sql并将结果映射成java对象并返回。
# Mybatis的入门
### 目录结构
![image](http://a4.qpic.cn/psb?/V11TGf4q1ABjBk/aSG8uvqEQVb7K4pq*UaX.nPbIleF4a.176FvKeulxSw!/m/dN8AAAAAAAAAnull&bo=GwF9ARsBfQEDCSw!&rf=photolist&t=5)
### 表结构
![image](http://a3.qpic.cn/psb?/V11TGf4q1ABjBk/mZJ3A5Jmx2UziRk6lrbKxd6H8mVvH5k9iPqmGeKS6dg!/m/dG4BAAAAAAAAnull&bo=hAGbAIQBmwADCSw!&rf=photolist&t=5)
### SqlMapConfig.xml
```
 <?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!-- 加载配置文件或者在 properties内部定义-->
    <properties resource="db.properties">
        <!-- 加载顺序，先加载节点内容部定义的变量的值，然后再加载外部配置文件，
            如果有同名变量会将其覆盖。
         -->
        <property name="jdbc.driver" value="com.mysql.jdbc.Driver"/>
    </properties>

    <!-- 全局配置 -->
    <settings>
        <setting name="cacheEnabled" value="true"/>
    </settings>
    <!-- 别名配置 -->
    <typeAliases>
        <!-- 配置po类的别名 -->
        <!-- <typeAlias type="cn.itcast.mybatis.po.User" alias="user"/> -->
        <!-- 配置po类的包，别名就是类名，不区分大小写 -->
        <package name="com.itheima.mybatis.po" />
    </typeAliases>

    <!-- 和spring整合后 environments配置将废除-->
    <environments default="development">
        <environment id="development">
            <!-- 使用jdbc事务管理-->
            <transactionManager type="JDBC" />
            <!-- 数据库连接池-->
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driver}" />
                <property name="url" value="${jdbc.url}" />
                <property name="username" value="${jdbc.username}" />
                <property name="password" value="${jdbc.password}" />
            </dataSource>
        </environment>
    </environments>
    <!-- 加载mapper文件 -->
    <mappers>
        <!--resource是基于claspath加载配置文件  -->
        <mapper resource="sqlmap/User.xml"/>
    </mappers>
</configuration>
```
### Mapper文件
### User.xml
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="test">
    <!-- id:起名叫statementid -->
    <!--
        parameterType:输入参数的类型
        resultType：返回结果的类型。如果是pojo类型的话需要给一个pojo的全路径。
     -->
    <!--  #{id}:占位符，相当于jdbc中的"?"
           输入参数是简单数据类型 时，括号中的名字可以随便起。
     -->
    <select id="findUserById" parameterType="int" resultType="com.itheima.mybatis.po.User">
        select * from user where id = #{id}
    </select>

    <!-- 根据姓名查询用户 -->
    <!-- resultType：返回结果中一条记录的类型 -->
    <!-- ${value}:sql字符串拼接指令，如果参数是简单数据类型时必须使用value取值
        这种方式不能防止sql注入。
    -->
    <select id="findUserByUsername" parameterType="string" resultType="com.itheima.mybatis.po.User">
        select * from user where username like '%${value}%'
    </select>

    <!-- 添加用户 -->
    <!--
        #{username}:User类中属性，取值时要和user类中的属性向对应。
        mybatis取对象中的属性值使用OGNL完成的。
     -->
    <insert id="insertUser" parameterType="com.itheima.mybatis.po.User">
        <!--
                keyProperty:返回主键放到user对象中的那个属性
                order:指定返回主键的操作是在sql语句执行之前还是执行之后
                resultType：返回主键的数据类型
                返回主键的sql：select LAST_INSERT_ID();mysql的语句，返回当前事务中最后一次生成的id的值。
                 -->
        <selectKey keyProperty="id" order="AFTER" resultType="int">
            select LAST_INSERT_ID();
        </selectKey>
        insert into user (sex, username, age, address) values (#{sex}, #{username}, #{age}, #{address})
    </insert>

    <!--根据id更新user-->
    <update id="updateUser" parameterType="com.itheima.mybatis.po.User">
        update user set username=#{username}, address=#{address} where id = #{id}
    </update>

    <delete id="deleteUser" parameterType="com.itheima.mybatis.po.User">
        delete from user where id = #{id}
    </delete>
</mapper>

```
### PO类
```java
package com.itheima.mybatis.po;

/**
 * Created by 杨波 on 2017/3/1.
 */
public class User {
    private int id;
    private String sex;
    private String username;
    private int age;
    private String address;

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", sex='" + sex + '\'' +
                ", username='" + username + '\'' +
                ", age=" + age +
                ", address='" + address + '\'' +
                '}';
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }
}
```
### 测试类
```java
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Test;

import java.io.InputStream;
import java.util.List;

/**
 * Created by 杨波 on 2017/3/1.
 */
public class MybatisTest {
    private SqlSessionFactory sessionFactory = null;
    private SqlSession sqlSession =null;
    public void init() throws Exception{
        InputStream inputStream = Resources.getResourceAsStream("SqlMapConfig.xml");
        //获得sqlsessionFactory对象
         sessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        //获得sqlsession
         sqlSession = sessionFactory.openSession();
    }
    @Test
    public void findUserById() throws Exception{
        init();

        User user = sqlSession.selectOne("test.findUserById",1);
        System.out.println(user);

        sqlSession.close();
    }
    @Test
    public void findUserByUsername() throws Exception{
        init();
        List<User> list = sqlSession.selectList("test.findUserByUsername","杨");
        System.out.println(list);

        sqlSession.close();
    }
    @Test
    public void insertUser() throws Exception{
        init();
        User user = new User();
        user.setSex("男");
        user.setUsername("黄晓明");
        user.setAge(17);
        user.setAddress("日本");
        sqlSession.insert("test.insertUser",user);
        System.out.println(user.getId());
        sqlSession.commit();
        sqlSession.close();

    }

    @Test
    public void updateUser() throws Exception{
        init();
        User user = new User();
        user.setId(6);
        user.setUsername("东尼");
        user.setAddress("中国");
        sqlSession.update("test.updateUser",user);
        sqlSession.commit();
        sqlSession.close();
    }
    @Test
    public void deleteUser() throws Exception{
        init();
        sqlSession.delete("test.deleteUser",4);
        sqlSession.commit();
        sqlSession.close();
    }
}
```
****
# Mapper代理方式
* 命名规范，习惯上接口的命名UserMapper.java(表名+mapper)，每个接口必须对应一个mapper文件，文件的文件名必须和接口的文件名一致（UserMapper.xml）。



* Mapper文件的namespace必须是接口的全路径。
  `<mapper namespace="com.itheima.mybatis.mapper.UserMapper">`


* 接口中的方法名称必须和mapper文件中的statementID一致。

* 方法的参数必须和statement的参数类型parameterType一致。
  方法的返回值，必须和statement的返回类型一致resultType。
  `User findUserById(int id);`
```
 <select id="findUserById" parameterType="int" resultType="com.itheima.mybatis.po.User">
        select * from user where id = #{id}
 </select>
```
## 测试
```java
public class UserMapperTest {
    private SqlSessionFactory sessionFactory = null;
    private SqlSession sqlSession =null;
    private UserMapper userMapper =null;
    public void init() throws Exception{
        InputStream inputStream = Resources.getResourceAsStream("SqlMapConfig.xml");
        //获得sqlSessionFactory对象
        sessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        //获得sqlSession
        sqlSession = sessionFactory.openSession();
        //获得mapper代理对象
        userMapper = sqlSession.getMapper(UserMapper.class);
    }

    @Test
    public void testFindUserById()throws Exception{
        init();
        User user = userMapper.findUserById(1);
        System.out.println(user);
        sqlSession.close();
    }

}
```

## 使用包装pojo作为查询条件取对象中的属性值是使用OGNL来获得的。
### POJO类
```
package com.itheima.mybatis.po;

public class QueryVo {
	private User user;
	
	private Integer[] ids;

	public User getUser() {
		return user;
	}

	public void setUser(User user) {
		this.user = user;
	}

	public Integer[] getIds() {
		return ids;
	}

	public void setIds(Integer[] ids) {
		this.ids = ids;
	}
	
	
	
}

```
### Mapper接口
```
List<User> findUserByQueryVo(QueryVo queryVo);
```
### 测试类
```
 //根据QueryVo查询用户列表
    @Test
    public void testFindUserByQueryVo() throws Exception {
        init();
        QueryVo queryVo = new QueryVo();
        User user = new User();
        user.setUsername("杨");
        queryVo.setUser(user);
        //执行查询
        List<User> userList = userMapper.findUserByQueryVo(queryVo);
        System.out.println(userList);
        sqlSession.close();
    }
```
## 返回Map类型
```
<!--
		type:返回结果的数据类型
		id：resultMap的id
	 -->
    <resultMap type="user" id="userResultMap">
        <!-- 配置映射关系 -->
        <!--
            column:结果集的列名
            property：返回结果类型中的属性名
         -->
        <id column="id" property="id"/>
        <result column="name" property="username"/>
        <result column="sex" property="sex"/>
        <result column="addr" property="address"/>

    </resultMap>
    <!-- 使用map数量类型作为参数 -->
    <sql id="fieldList">
        id,username ,sex,address ,age
    </sql>
    <select id="findUserResultMap" parameterType="string" resultMap="userResultMap">
        select
        <include refid="fieldList"/>
        from user
        where username like '%${value}%'
    </select>
```
### Mapper接口
```
List<User> findUserResultMap(String username);
```
### 测试类
```
 @Test
    public void testFindUserResultMap () throws Exception {
        init();
        List<User> list = userMapper.findUserResultMap("杨");
        System.out.println(list);
        sqlSession.close();
    }
```
# 动态sql
## if标签
**判断是否添加一些sql语句**
```
<update id="updateUser" parameterType="com.itheima.mybatis.po.User">
        update user set
        <if test="username != null and username!=''">
            username = #{username},
        </if>
        <if test="address != null and address != ''">
            address = #{address}
        </if>
        where id = #{id}
</update>
```
## where标签
**Where标签会自动添加where条件，如果如果标签内容部没有条件的话sql语句中就没有where关键字。如果条件中有多余的and 会自动去掉。**
```
<where>
	<if test="user.sex != null">
          and sex = #{user.sex}
        </if>
        <if test="user.username != null">
            and username = #{user.username}
        </if>
</where>
```
## foreach标签
```
<if test="ids !=null">
            <!--
                collection：集合属性
                item：集合中每个元素的值
                open：字符串拼接的前缀
                close：字符串后缀
                separator：分隔符
             -->
            <foreach collection="ids" item="item" open="and id in(" close=")" separator=",">
                #{item}
            </foreach>
</if>
```
## sql标签
**Sql标签就是声明一个sql片段，可以使用include标签引用。**
* 多个查询都会用到的查询条件会提取出来。
* 字段的列表提取出来。
```
<sql id="fieldList">
        id,username ,sex,address ,age
</sql>
 <select id="findUserResultMap" parameterType="string" resultMap="userResultMap">
        select
        <include refid="fieldList"/>
        from user
        where username like '%${value}%'
 </select>
```
# 关系模型

![](http://a3.qpic.cn/psb?/V11TGf4q1ABjBk/gYiYNu6RMIF7kuD1aF9r2y39FHou0G2Wes37oNvXVMM!/m/dAoBAAAAAAAAnull&bo=3wJFAt8CRQIFCSo!&rf=photolist&t=5)

表结构

* user( id , username , birthday , sex , address)
* order( id , user_id , number , createtime , note)
* orderdetail( id , orders_id , items_id , items_num)
* item( id , name , price , detail , pic , createtime)

# 一对一查询

## 使用resultType来实现
sql语句
```sql
SELECT
            o.id,
            o.user_id userId,
            o.number,
            o.createtime,
            o.note,
            u.username,
            u.birthday,
            u.sex,
            u.address
        FROM
            orders o
        JOIN `user` u
        ON
            o.user_id = u.id
```



Mapper文件

```xml
<select id="selectOrderWithUser" resultType="com.itheima.mybatis.po1.OrdersUser">
        SELECT
            o.id,
            o.user_id userId,
            o.number,
            o.createtime,
            o.note,
            u.username,
            u.birthday,
            u.sex,
            u.address
        FROM
            orders o
        JOIN `user` u
        ON
            o.user_id = u.id
    </select>
```

接口

```java
List<OrdersUser> selectOrderWithUser() throws Exception;
```

创建po类存放参数

```java
public class OrdersUser extends Orders{
    private String username;// 用户姓名
    private String sex;// 性别
    private Date birthday;// 生日
    private String address;// 地址
    //省略getset方法
}
```

测试类

```java
@Test
    public void testSelectOrderWithUser() throws Exception {
        SqlSession sqlSession = sessionFactory.openSession();
        OrdersMapper ordersMapper = sqlSession.getMapper(OrdersMapper.class);
        List<OrdersUser> ordersUserList = ordersMapper.selectOrderWithUser();
        System.out.println(ordersUserList);
    }
```

## 使用resultMap接收一对一映射

在po类orders中添加

```java
 //用户信息
    private User user;
//省略getset
```

Mapper文件

```xml
<resultMap id="OrderWithUserResultMap" type="orders">
        <id column="id" property="id"/>
        <result column="userId" property="userId"/>
        <result column="number" property="number"/>
        <result column="createtime" property="createtime"/>
        <result column="note" property="note"/>
        <!-- property:orders对象中user属性
			javaType:user对象的类型
		-->
        <association property="user" javaType="com.itheima.mybatis.po.User">
            <!-- 对应user表的id column：结果集中user的id-->
            <id column="userId" property="id"/>
            <!--普通字段-->
            <result column="username" property="username"/>
            <result column="sex" property="sex"/>
            <result column="address" property="address"/>
        </association>

    </resultMap>
    <!--一对一简单查询，使用ResultMap-->
    <select id="selectOrderWithUserResultMap" resultMap="OrderWithUserResultMap">
      SELECT
            o.id,
            o.user_id userId,
            o.number,
            o.createtime,
            o.note,
            u.username,
            u.birthday,
            u.sex,
            u.address
        FROM
            orders o
        JOIN `user` u
        ON
            o.user_id = u.id
    </select>
```

测试

```java
@Test
    public void testSelectOrderWithUserResultMap() throws Exception {
        SqlSession sqlSession = sessionFactory.openSession();
        OrdersMapper ordersMapper = sqlSession.getMapper(OrdersMapper.class);
        List<Orders> ordersList = ordersMapper.selectOrderWithUserResultMap();
        System.out.println(ordersList);
    }
```

# 一对多查询

sql语句

```sql
SELECT
            o.id,
            o.user_id userId,
            o.number,
            o.createtime,
            o.note,
            u.username,
            u.birthday,
            u.sex,
            u.address,
            od.id detail_id,
            od.items_id,
            od.items_num
        FROM
            orders o
        JOIN `user` u ON o.user_id = u.id
        JOIN orderdetail od ON o.id = od.orders_id
```

在po类orders中添加

```java
//订单明细
    private List<Orderdetail> orderdetails;
//省略getset
```

Mapper文件

```xml
<resultMap id="orderWithDetailResultMap" type="orders" extends="OrderWithUserResultMap">
        <!-- 订单的关联映射 -->
        <!-- 用户的关联映射 -->
        <!-- 订单明细的关联映射 -->
        <!--property orders对象中的订单列表属性detailList
            ofType:列表中一个元素的类型
         -->
        <collection property="orderdetails" ofType="com.itheima.mybatis.po.Orderdetail">
            <!-- column:结果集中订单明细的主键列
                id：Orderdetail类中主键的属性
             -->
            <id column="detail_id" property="id"/>
            <!-- 普通列 -->
            <result column="items_id" property="itemsId"/>
            <result column="items_num" property="itemsNum"/>
        </collection>
    </resultMap>
    <!--一对多简单查询，使用ResultMap-->
    <select id="selectOrderWithDetail" resultMap="orderWithDetailResultMap">
        SELECT
            o.id,
            o.user_id userId,
            o.number,
            o.createtime,
            o.note,
            u.username,
            u.birthday,
            u.sex,
            u.address,
            od.id detail_id,
            od.items_id,
            od.items_num
        FROM
            orders o
        JOIN `user` u ON o.user_id = u.id
        JOIN orderdetail od ON o.id = od.orders_id
    </select>
```

测试

```java
@Test
    public void testSelectOrderWithDetail() throws Exception {
        SqlSession sqlSession = sessionFactory.openSession();
        OrdersMapper ordersMapper = sqlSession.getMapper(OrdersMapper.class);
        List<Orders> ordersList = ordersMapper.selectOrderWithDetail();
        System.out.println(ordersList);
    }
```

# 一对多复杂查询

sql语句

```sql
SELECT
            o.id,
            o.user_id userId,
            o.number,
            o.createtime,
            o.note,
            u.username,
            u.birthday,
            u.sex,
            u.address,
            od.id detail_id,
            od.items_id,
            od.items_num,
            i.createtime items_createtime,
            i.detail,
            i.`name`,
            i.pic,
            i.price
        FROM
            orders o
        JOIN `user` u ON o.user_id = u.id
        JOIN orderdetail od ON o.id = od.orders_id
        JOIN items i ON i.id = od.items_id
```

在po类orders中添加

```java
 //用户信息
    private User user;
    
    //订单明细
    private List<Orderdetail> orderdetails;
```

在po类orderdetail中添加

```java
 private Items items;
```

Mapper文件

```xml
<resultMap type="orders" id="userOrderDetailItemsResultMap">
        <!-- 订单表的映射关系 -->
        <id column="orders_id" property="id"/>
        <result column="id" property="userId"/>
        <result column="number" property="number"/>
        <result column="createtime" property="createtime"/>
        <result column="note" property="note"/>
        <!-- property:orders对象中user属性
            javaType:user对象的类型
        -->
        <association property="user" javaType="com.itheima.mybatis.po.User">
            <!-- 对应user表的id column：结果集中user的id-->
            <id column="id" property="id"/>
            <!-- 普通字段 -->
            <result column="username" property="username"/>
            <result column="birthday" property="birthday"/>
            <result column="sex" property="sex"/>
            <result column="address" property="address"/>
        </association>
        <!-- 订单明细的关联映射 -->
        <!--property orders对象中的订单列表属性detailList
            ofType:列表中一个元素的类型
         -->
        <collection property="orderdetails" ofType="com.itheima.mybatis.po.Orderdetail">
            <!-- column:结果集中订单明细的主键列
                id：Orderdetail类中主键的属性
             -->
            <id column="detail_id" property="id"/>
            <!-- 普通列 -->
            <result column="items_id" property="itemsId"/>
            <result column="items_num" property="itemsNum"/>
            <!-- 商品表映射关系 -->
            <association property="items" javaType="com.itheima.mybatis.po.Items">
                <id column="items_id" property="id"/>
                <result column="name" property="name"/>
                <result column="price" property="price"/>
                <result column="pic" property="pic"/>
                <result column="detail" property="detail"/>
                <result column="items_createtime" property="createtime"/>
            </association>
        </collection>
    </resultMap>
    <!--一对多复杂查询，使用ResultMap-->
    <select id="selectUserOrderDetailItems" resultMap="userOrderDetailItemsResultMap">
        SELECT
            o.id,
            o.user_id userId,
            o.number,
            o.createtime,
            o.note,
            u.username,
            u.birthday,
            u.sex,
            u.address,
            od.id detail_id,
            od.items_id,
            od.items_num,
            i.createtime items_createtime,
            i.detail,
            i.`name`,
            i.pic,
            i.price
        FROM
            orders o
        JOIN `user` u ON o.user_id = u.id
        JOIN orderdetail od ON o.id = od.orders_id
        JOIN items i ON i.id = od.items_id
    </select>
```

测试

```java
@Test
    public void testSelectUserOrderDetailItems() throws Exception {
        SqlSession sqlSession = sessionFactory.openSession();
        OrdersMapper ordersMapper = sqlSession.getMapper(OrdersMapper.class);
        List<Orders> ordersList = ordersMapper.selectUserOrderDetailItems();
        System.out.println(ordersList);
    }
```

# 延迟加载

![](http://a4.qpic.cn/psb?/V11TGf4q1ABjBk/BCLytoE8CSn..GSs0KsEcqDv8hMZ0qbpA0G7uW3GNK8!/m/dIMBAAAAAAAAnull&bo=ygIjAcoCIwEFCSo!&rf=photolist&t=5)

```xml
<!-- 全局配置参数 -->
	<settings>
		<!-- 延迟加载总开关 -->
		<setting name="lazyLoadingEnabled" value="true" />	
		<!-- 设置按需加载 -->
		<setting name="aggressiveLazyLoading" value="false" />
	</settings>
```

## 配置sqlMapConfig

```xml
<!-- 全局配置 -->
	<!-- 全局配置参数 -->
	<settings>
		<!-- 延迟加载总开关 -->
		<setting name="lazyLoadingEnabled" value="true" />	
		<!-- 设置按需加载 -->
		<setting name="aggressiveLazyLoading" value="false" />
		<!-- 开启二级缓存 -->
		<setting name="cacheEnabled" value="true"/>
	</settings>
```

## Mapper文件

```xml
<!-- 延迟加载查询订单信息，订单明细延迟加载 -->
	<!-- 配置延迟加载的resultMap -->
	<resultMap type="orders" id="orderWithDetailLazyResultMap">
		<!-- 订单表映射 -->
		<id column="id" property="id"/>
		<result column="userId" property="userId"/>
		<result column="number" property="number"/>
		<result column="createtime" property="createtime"/>
		<result column="note" property="note"/>
		<!-- 订单明细的映射 -->
		<!-- 
			select:子查询的statementid，如果使用到其他mapper文件中的statementid时需要加上namespace。
			
		 -->
		<collection property="detailList" ofType="cn.itcast.mybatis.po.Orderdetail" select="findOrderDetail" column="id">
			<!-- column:结果集中订单明细的主键列
				id：Orderdetail类中主键的属性
			 -->
			<id column="id" property="id"/>
			<!-- 普通列 -->
			<result column="items_id" property="itemsId"/>
			<result column="items_num" property="itemsNum"/>
			<result column="orders_id" property="ordersId"/>
		</collection>
	</resultMap>
	<select id="selectOrderWithDetail" resultMap="orderWithDetailLazyResultMap">
		SELECT
			o.id,
			o.user_id userId,
			o.number,
			o.createtime,
			o.note
		FROM
			orders o
	</select>
	<!-- 查询订单明细 -->
	<select id="findOrderDetail" parameterType="int" resultType="cn.itcast.mybatis.po.Orderdetail">
		select * from orderdetail where orders_id=#{id}
	</select>
```

# 查询缓存

![](http://a4.qpic.cn/psb?/V11TGf4q1ABjBk/AXmDyhgah9O2qbNLyiDU0XdROI5O5rkc.WqXMc8OA.A!/m/dB8BAAAAAAAAnull&bo=2QItAdkCLQEFCSo!&rf=photolist&t=5)

Mybatis一级缓存的作用域是同一个SqlSession，在同一个sqlSession中两次执行相同的sql语句，第一次执行完毕会将数据库中查询的数据写到缓存（内存），第二次会从缓存中获取数据将不再从数据库查询，从而提高查询效率。当一个sqlSession结束后该sqlSession中的一级缓存也就不存在了。Mybatis默认开启一级缓存。

 

Mybatis二级缓存是多个SqlSession共享的，其作用域是mapper的同一个namespace，不同的sqlSession两次执行相同namespace下的sql语句且向sql中传递参数也相同即最终执行相同的sql语句，第一次执行完毕会将数据库中查询的数据写到缓存（内存），第二次会从缓存中获取数据将不再从数据库查询，从而提高查询效率。Mybatis默认没有开启二级缓存需要在setting全局参数中配置开启二级缓存。

## 一级缓存

一级缓存默认开启。

```java
//一级缓存测试
	@Test
	public void testSelectOrderWithUserLevel1Cache() throws Exception {
		//开启一个sqlsession
		SqlSession sqlSession = sessionFactory.openSession();
		OrdersMapper ordersMapper = sqlSession.getMapper(OrdersMapper.class);
		//第一查询数据库，从数据库中取数据
		List<OrdersUser> list = ordersMapper.selectOrderWithUser();
		System.out.println(list);
		//第二次查询，从一级缓存中取数据
		List<OrdersUser> list2 = ordersMapper.selectOrderWithUser();
		System.out.println(list2);
		//关闭sqlsession
		sqlSession.close();
	}
```

如果在sqlsession中执行了更新数据库的操作缓存会清空。

```java
/一级缓存测试
	@Test
	public void testSelectOrderWithUserLevel1Cache() throws Exception {
		//开启一个sqlsession
		SqlSession sqlSession = sessionFactory.openSession();
		OrdersMapper ordersMapper = sqlSession.getMapper(OrdersMapper.class);
		UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
		//第一查询数据库，从数据库中取数据
		List<OrdersUser> list = ordersMapper.selectOrderWithUser();
		System.out.println(list);
		//执行数据库的更新操作
		User user = new User();
		user.setId(1);
		user.setUsername("杨幂");
		userMapper.updateUser(user);
		sqlSession.commit();
		//第二次查询，从一级缓存中取数据
		List<OrdersUser> list2 = ordersMapper.selectOrderWithUser();
		System.out.println(list2);
		//关闭sqlsession
		sqlSession.close();
	}
```

## 二级缓存

### 原理

多个sqlSession请求UserMapper的二级缓存图解。

![](http://a3.qpic.cn/psb?/V11TGf4q1ABjBk/bWMFGw.ozn*wdZ5P1DAQlWC92ENXhzzMK1yNMeZbKn8!/m/dPYAAAAAAAAAnull&bo=3AKXAdwClwEFCSo!&rf=photolist&t=5)

二级缓存区域是根据mapper的namespace划分的，相同namespace的mapper查询数据放在同一个区域，如果使用mapper代理方法每个mapper的namespace都不同，此时可以理解为二级缓存区域是根据mapper划分。

每次查询会先从缓存区域找，如果找不到从数据库查询，查询到数据将数据写入缓存。

Mybatis内部存储缓存使用一个HashMap，key为hashCode+sqlId+Sql语句。value为从查询出来映射生成的java对象

sqlSession执行insert、update、delete等操作commit提交后会清空缓存区

### 开启二级缓存

在核心配置文件SqlMapConfig.xml中加入

`<setting name="cacheEnabled" value="true"/>`

要在Mapper映射文件中添加一行：  <cache /> ，表示此mapper开启二级缓存。

#### sqlMapConfig.xml

```xml
<!-- 全局配置参数 -->
	<settings>
		<!-- 延迟加载总开关 -->
		<setting name="lazyLoadingEnabled" value="true" />	
		<!-- 设置按需加载 -->
		<setting name="aggressiveLazyLoading" value="false" />
		<!-- 开启二级缓存 -->
		<setting name="cacheEnabled" value="true"/>
	</settings>
```

#### Mapper文件

```xml
<!-- 此mapper使用二级缓存 -->
	<cache type="org.mybatis.caches.ehcache.EhcacheCache"/>
```

#### POJO实现Serializable接口

```java
public class OrdersUser extends Orders implements Serializable {

	private String username;// 用户姓名
	private String sex;// 性别
	private Date birthday;// 生日
	private String address;// 地址
```

#### 测试

```java
/测试二级缓存
	@Test
	public void testLevel2cache() throws Exception {
		//开启一个sqlsession
		SqlSession sqlSession = sessionFactory.openSession();
		//取orderMapper代理对象
		OrdersMapper ordersMapper = sqlSession.getMapper(OrdersMapper.class);
		List<OrdersUser> list = ordersMapper.selectOrderWithUser();
		System.out.println(list);
		sqlSession.close();
		
		//开启第二个sqlsession
		SqlSession sqlSession2 = sessionFactory.openSession();
		//取orderMapper代理对象
		OrdersMapper ordersMapper2 = sqlSession2.getMapper(OrdersMapper.class);
		List<OrdersUser> list2 = ordersMapper2.selectOrderWithUser();
		System.out.println(list2);
		sqlSession2.close();
	}
```

#### 测试二级缓存包含更新

```java
@Test
	public void testLevel2cache() throws Exception {
		//开启一个sqlsession
		SqlSession sqlSession = sessionFactory.openSession();
		//取orderMapper代理对象
		OrdersMapper ordersMapper = sqlSession.getMapper(OrdersMapper.class);
		List<OrdersUser> list = ordersMapper.selectOrderWithUser();
		System.out.println(list);
		sqlSession.close();
		//执行更新数据库操作，会清空二级缓存
		//开启一个sqlsession
		SqlSession sqlSession1 = sessionFactory.openSession();
		OrdersMapper ordersMapper1 = sqlSession1.getMapper(OrdersMapper.class);
		ordersMapper1.updateOrders();
		sqlSession1.commit();
		sqlSession1.close();
		//开启第二个sqlsession
		SqlSession sqlSession2 = sessionFactory.openSession();
		//取orderMapper代理对象
		OrdersMapper ordersMapper2 = sqlSession2.getMapper(OrdersMapper.class);
		List<OrdersUser> list2 = ordersMapper2.selectOrderWithUser();
		System.out.println(list2);
		sqlSession2.close();
	}
```

### 不使用二级缓存

`useCache = "false"`

### 缓存刷新

默认执行数据库的更新操作会清空二级缓存，但是可以手工指定更新后不清空缓存，会造成数据库中的数据和缓存中的数据不同步，出现脏读的情况。

`flushCache = "false"`

# 和spring整合

* 使用spring框架关联sqlsessionfactory，作为一个单例存在容器中。
* 如果是传统的dao的开发方式，sqlsession最好是从spring容器中获得。
* 如果mapper代理的方式，最好是直接从spring容器中获得mapper对象。
* 数据库连接、连接池、事务关联都使用spring来管理。
# 需要的jar包

maven坐标

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.itheima.mybatis_03</groupId>
  <artifactId>mybatis_03</artifactId>
  <packaging>war</packaging>
  <version>1.0-SNAPSHOT</version>
  <name>mybatis_03 Maven Webapp</name>
  <url>http://maven.apache.org</url>
  <dependencies>
    <!-- mybatis整合spring jar包 -->
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis-spring</artifactId>
      <version>1.3.0</version>
    </dependency>
    <!-- spring相关jar包 -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>4.3.3.RELEASE</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/org.springframework/spring-core -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-core</artifactId>
      <version>4.3.3.RELEASE</version>
    </dependency>

    <!-- https://mvnrepository.com/artifact/org.springframework/spring-beans -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-beans</artifactId>
      <version>4.3.3.RELEASE</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/org.springframework/spring-web -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-web</artifactId>
      <version>4.3.3.RELEASE</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/org.springframework/spring-webmvc -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>4.3.3.RELEASE</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/org.springframework/spring-jdbc -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-jdbc</artifactId>
      <version>4.3.3.RELEASE</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/org.springframework/spring-aop -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-aop</artifactId>
      <version>4.3.3.RELEASE</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/org.springframework/spring-orm -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-orm</artifactId>
      <version>4.3.3.RELEASE</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/org.springframework/spring-context-support -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context-support</artifactId>
      <version>4.3.3.RELEASE</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/org.springframework/spring-aspects -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-aspects</artifactId>
      <version>4.3.3.RELEASE</version>
    </dependency>
    <!-- mysql的jar包 -->
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.38</version>
    </dependency>
    <!-- junit jar包 -->
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.12</version>
    </dependency>
    <!-- mybatis jar包 -->
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>3.4.2</version>
    </dependency>
    <!-- commons相关 -->
    <!-- https://mvnrepository.com/artifact/commons-pool/commons-pool -->
    <dependency>
      <groupId>commons-pool</groupId>
      <artifactId>commons-pool</artifactId>
      <version>1.6</version>
    </dependency>
    <dependency>
      <groupId>commons-dbcp</groupId>
      <artifactId>commons-dbcp</artifactId>
      <version>1.4</version>
    </dependency>
    <dependency>
      <groupId>commons-logging</groupId>
      <artifactId>commons-logging</artifactId>
      <version>1.1.1</version>
    </dependency>
    <!-- log4j -->
    <dependency>
      <groupId>log4j</groupId>
      <artifactId>log4j</artifactId>
      <version>1.2.17</version>
    </dependency>
    <dependency>
      <groupId>org.apache.logging.log4j</groupId>
      <artifactId>log4j-core</artifactId>
      <version>2.6.2</version>
    </dependency>
    <dependency>
      <groupId>org.apache.logging.log4j</groupId>
      <artifactId>log4j-api</artifactId>
      <version>2.7</version>
    </dependency>

  </dependencies>
  <build>
    <finalName>mybatis_03</finalName>
  </build>
</project>

```

# spring配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:mvc="http://www.springframework.org/schema/mvc"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
		http://www.springframework.org/schema/beans/spring-beans-3.2.xsd 
		http://www.springframework.org/schema/mvc 
		http://www.springframework.org/schema/mvc/spring-mvc-3.2.xsd 
		http://www.springframework.org/schema/context 
		http://www.springframework.org/schema/context/spring-context-3.2.xsd 
		http://www.springframework.org/schema/aop 
		http://www.springframework.org/schema/aop/spring-aop-3.2.xsd 
		http://www.springframework.org/schema/tx 
		http://www.springframework.org/schema/tx/spring-tx-3.2.xsd ">
	<!-- 加载配置文件 -->
	<context:property-placeholder location="classpath:db.properties"/>
	<!-- 数据库连接池 -->
	<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
	       <property name="driverClassName" value="${jdbc.driver}"/>
			<property name="url" value="${jdbc.url}"/>
			<property name="username" value="${jdbc.username}"/>
			<property name="password" value="${jdbc.password}"/>
			<property name="maxActive" value="10"/>
			<property name="maxIdle" value="5"/>
	</bean>	
	<!-- sqlsessionFactory的配置 -->
	<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
		<property name="dataSource" ref="dataSource"/>
		<!-- 加载mybatis的配置文件 -->
		<property name="configLocation" value="classpath:SqlMapConfig.xml"/>
	</bean>
	<!-- 使用代理对象开发dao 使用MapperFactoryBean创建代理对象-->
	<!-- <bean id="ordersMapper" class="org.mybatis.spring.mapper.MapperFactoryBean">
		要代理的接口
		<property name="mapperInterface" value="cn.itcast.mybatis.mapper.OrdersMapper"/>
		<property name="sqlSessionFactory" ref="sqlSessionFactory"/>
	</bean> -->
	<!-- 配置包扫描的方式扫描mapper，扫描之后bean的id就是类名首字母小写-->
	<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
		<!-- 指定扫描的包，如果要扫描多个包可以把多个包都配置到vale中，使用半角逗号分隔 -->
		<property name="basePackage" value="com.itheima.mybatis.mapper"/>
		<property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
	</bean>
</beans>
```

# sqlMapConfig.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <typeAliases>
        <!-- 配置po类的别名 -->
        <!-- <typeAlias type="cn.itcast.mybatis.po.User" alias="user"/> -->
        <!-- 配置po类的包，别名就是类名，不区分大小写 -->
        <package name="com.itheima.mybatis.po" />
    </typeAliases>


    <!-- 加载mapper文件 -->
    <mappers>
        <mapper resource="sqlmap/OrdersMapper.xml"/>
    </mappers>
</configuration>
```

# dao开发（略）

# Mapper代理

```java
package com.itheima.mybatis.mapper;

import com.itheima.mybatis.po.Orders;
import com.itheima.mybatis.po1.OrdersUser;

import java.util.List;

/**
 * Created by 杨波 on 2017/3/3.
 */
public interface OrdersMapper {
    List<OrdersUser> selectOrderWithUser() throws Exception;
    List<Orders> selectOrderWithUserResultMap() throws Exception;
    List<Orders> selectOrderWithDetail() throws Exception;
    List<Orders> selectUserOrderDetailItems() throws Exception;
}

```

## 测试

```java
package com.itheima.mybatis.mapper;

import com.itheima.mybatis.po1.OrdersUser;
import org.junit.Before;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import java.util.List;

/**
 * Created by 杨波 on 2017/3/6.
 */
public class OrdersMapperTest {
    //spring容器
    private ApplicationContext applicationContext = null;
    @Before
    public void setUp() throws Exception {
        applicationContext = new ClassPathXmlApplicationContext("ApplicationContext.xml");
    }

    @Test
    public void testSelectOrderWithUser() throws Exception {
        OrdersMapper ordersMapper = (OrdersMapper) applicationContext.getBean("ordersMapper");
        List<OrdersUser> list = ordersMapper.selectOrderWithUser();
        System.out.println(list);
    }

}

```

# 逆向工程

## 配置文件generatorConfig.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
  PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
  "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
	<context id="testTables" targetRuntime="MyBatis3">
		<commentGenerator>
			<!-- 是否去除自动生成的注释 true：是 ： false:否 -->
			<property name="suppressAllComments" value="true" />
		</commentGenerator>
		<!--数据库连接的信息：驱动类、连接地址、用户名、密码 -->
		<jdbcConnection driverClass="com.mysql.jdbc.Driver"
			connectionURL="jdbc:mysql://localhost:3306/mybatis01" userId="root"
			password="134679">
		</jdbcConnection>
		<!-- <jdbcConnection driverClass="oracle.jdbc.OracleDriver"
			connectionURL="jdbc:oracle:thin:@127.0.0.1:1521:yycg" 
			userId="yycg"
			password="yycg">
		</jdbcConnection> -->

		<!-- 默认false，把JDBC DECIMAL 和 NUMERIC 类型解析为 Integer，为 true时把JDBC DECIMAL 和 
			NUMERIC 类型解析为java.math.BigDecimal -->
		<javaTypeResolver>
			<property name="forceBigDecimals" value="false" />
		</javaTypeResolver>

		<!-- targetProject:生成PO类的位置 -->
		<javaModelGenerator targetPackage="com.itheima.mybatis.po"
			targetProject=".\src">
			<!-- enableSubPackages:是否让schema作为包的后缀 -->
			<property name="enableSubPackages" value="false" />
			<!-- 从数据库返回的值被清理前后的空格 -->
			<property name="trimStrings" value="true" />
		</javaModelGenerator>
        <!-- targetProject:mapper映射文件生成的位置 -->
		<sqlMapGenerator targetPackage="sql\mapper"
			targetProject=".\src">
			<!-- enableSubPackages:是否让schema作为包的后缀 -->
			<property name="enableSubPackages" value="false" />
		</sqlMapGenerator>
		<!-- targetPackage：mapper接口生成的位置 -->
		<javaClientGenerator type="XMLMAPPER"
			targetPackage="com.itheima.mybatis.mapper"
			targetProject=".\src">
			<!-- enableSubPackages:是否让schema作为包的后缀 -->
			<property name="enableSubPackages" value="false" />
		</javaClientGenerator>
		<!-- 指定数据库表 -->
		<table schema="" tableName="user"></table>
		<table schema="" tableName="orders"></table>
		<table schema="" tableName="items"></table>
		<table schema="" tableName="orderdetail"></table>
		
		<!-- 有些表的字段需要指定java类型
		 <table schema="" tableName="">
			<columnOverride column="" javaType="" />
		</table> -->
	</context>
</generatorConfiguration>

```

## GeneratorAqlmap.java

```java


import java.io.File;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

import org.mybatis.generator.api.MyBatisGenerator;
import org.mybatis.generator.config.Configuration;
import org.mybatis.generator.config.xml.ConfigurationParser;
import org.mybatis.generator.exception.XMLParserException;
import org.mybatis.generator.internal.DefaultShellCallback;

public class GeneratorSqlmap {

	public void generator() throws Exception{

		List<String> warnings = new ArrayList<String>();
		boolean overwrite = true;
		//指定 逆向工程配置文件
		File configFile = new File("generatorConfig.xml"); 
		ConfigurationParser cp = new ConfigurationParser(warnings);
		Configuration config = cp.parseConfiguration(configFile);
		DefaultShellCallback callback = new DefaultShellCallback(overwrite);
		MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config,
				callback, warnings);
		myBatisGenerator.generate(null);

	} 
	public static void main(String[] args) throws Exception {
		try {
			GeneratorSqlmap generatorSqlmap = new GeneratorSqlmap();
			generatorSqlmap.generator();
		} catch (Exception e) {
			e.printStackTrace();
		}
		
	}

}

```