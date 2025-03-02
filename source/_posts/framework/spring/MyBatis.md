---
title: MyBatis
toc: true
categories: [Framework,Spring]
tags: 
	- MyBatis
	- SSM
date: 2022-06-30 16:48:10
updated: 2022-07-03 21:51
---

MyBatis 是一款优秀的持久层框架，它支持自定义 SQL、存储过程以及高级映射。

- 免除了几乎所有的 [JDBC](https://blog.csdn.net/qq_43640009/article/details/106482059) 代码以及设置参数和获取结果集的工作。
	
- 可以通过简单的 XML 或注解来配置和映射原始类型、接口和 Java POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。

<!-- more -->
	

## XML 配置

更多配置参考：[官方文档 XML配置项](https://mybatis.org/mybatis-3/zh/configuration.html)

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!-- 读取数据库配置文件 -->
    <properties resource="jdbc.properties"/>
    <!-- 设置日志输出 -->
    <settings>
        <setting name="logImpl" value="STDOUT_LOGGING"/>
    </settings>
    <!-- 注册实体类别名 -->
    <typeAliases>
        <!-- 单个注册别名 -->
        <typeAlias type="com.proj.Student" alias="student"/>
        <!-- 批量注册别名：类名的驼峰命名 -->
        <!-- <package name="com.proj"/> -->
    </typeAliases>
    <!-- 数据库配置 -->
    <environments default="development">
        <environment id="development">
            <!-- type：事务管理器
                JDBC：程序员管理
                MANAGED：Spring管理
            -->
            <transactionManager type="JDBC"/>
            <!-- 数据库连接池 -->
            <dataSource type="POOLED">
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>
    <!-- 注册mapper -->
    <mappers>
        <mapper resource="StudentMapper.xml"/>
    </mappers>
</configuration>
```

> 数据库连接池：
>
> 一个中间层，初始化多个连接，建立连接从连接池取出，断开连接放回连接池，开销比直接连接断开小得多。

## XML 映射

### 占位符的区别

- `#{}`: 对非字符串参数的占位符，可以有效防止 SQL 注入
  - 入参是简单数据类型（除 String），`#{}` 内可以任意写
  - 入参是对象类型，`#{}` 内必须是对象成员变量名称
- `${}`: 对字符串的拼接替换，可以替换列名和表名，存在 SQL 注入风险，尽量少用
  - 入参是基本数据类型（有 String），`${}` 内必须是 `value`
  - 入参是对象类型，`${}` 内必须是对象成员变量名称

> **使用 `#{}` 防止 SQL 注入：**
>
> 1. 遇到模糊查询需要拼接字符串时：`where name like '%${name}%'`
> 2. 使用 `concat` 函数改为：`where name like concat('%',#{name},'%')`
>
>  **字符串替换只能使用 `${}`** ，详见下一节代码

### 创建映射配置文件

更多配置参考：[官方文档 XML映射器](https://mybatis.org/mybatis-3/zh/sqlmap-xml.html)

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- namespace：命名空间（包名），用来区分不同文件中相同的id属性 -->
<mapper namespace="com.proj.mapper.UsersMapper">
    <!-- 查询全部数据 -->
    <select id="getAll" resultType="users">
        select id, username, birthday, sex, address
        from users
    </select>
    <!-- 通过主键id查询 -->
    <select id="getById" parameterType="int" resultType="users">
        select id, username, birthday, sex, address
        from users
        where id = #{id}
    </select>
    <!-- 通过名称模糊查询 -->
    <select id="getByName" parameterType="string" resultType="users">
        select id, username, birthday, sex, address
        from users
        where username like concat('%', #{userName}, '%')
    </select>
    <!-- 插入数据
         - 参数对应实体类属性
         - 获取插入后的主键id
    -->
    <insert id="insert" parameterType="users">
        <selectKey keyProperty="id" resultType="int" order="AFTER">
            select last_insert_id()
        </selectKey>
        insert into users (username, birthday, sex, address)
        values (#{userName}, #{birthday}, #{sex}, #{address})
    </insert>
    <!-- 按主键删除数据 -->
    <delete id="delete" parameterType="int">
        delete
        from users
        where id = #{id}
    </delete>
    <!-- 更新数据 -->
    <update id="update" parameterType="users">
        update users
        set username=#{userName},
            birthday=#{birthday},
            sex=#{sex},
            address=#{address}
        where id = #{id}
    </update>
    <!-- 参数超过一个参数就不写 -->
    <select id="getByNameOrAddress" resultType="users">
        select id, username, birthday, sex, address
        from users
        where ${col} like concat('%', #{val}, '%')
    </select>
</mapper>
```

### 测试使用

**增删改需要 commit ！！！**

```java
public class MyTest {
	SqlSession sqlSession;

    @Before
    public void openSqlSession() throws IOException {
        // 文件流读取核心配置文件
        InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
        // 创建SqlSessionFactory工厂
        SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(in);
        // 取出sqlSession对象
        sqlSession = factory.openSession();
    }

    @After
    public void closeSqlSession() {
       // 关闭sqlSession
        sqlSession.close();
    }    

	@Test
    public void test() {
        // 完成查询操作
        List<Student> list = sqlSession.selectList("om.proj.mapper.UsersMapper");
        list.forEach(System.out::println);
        // 增删改需要commit
        // sqlSession.commit();
    }
}
```

## 动态代理

动态代理后就不需要从 xml 中取出执行方法，通过统一的代理接口直接调用。

### 创建代理接口

```java
package com.proj.mapper;

import com.proj.pojo.Users;
import org.apache.ibatis.annotations.Param;

import java.util.List;

public interface UsersMapper {

    List<Users> getAll();

    Users getById(int id);

    List<Users> getByName(String userName);

    int insert(Users users);

    int delete(int id);

    int update(Users users);
	
    // 多个参数使用@Param
    List<Users> getByNameOrAddress(
            @Param("col")
            String col,
            @Param("val")
            String val
    );
}

```

### 修改配置文件

更改为代理接口的 `reference`

```xml
    <mappers>
         <!-- 修改前
        <mapper resource="StudentMapper.xml"/>
		  -->
        <!-- 单个注册 -->
        <mapper class="com.proj.mapper.UsersMapper"/>
        <!-- 批量注册：推荐 -->
        <package name="com.proj.mapper"/>
    </mappers>
```

### 测试使用

```java
public class MyTest {

    SqlSession sqlSession;
    UsersMapper usersMapper;
    SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");


    @Before
    public void openSqlSession() throws IOException {
        // 文件流读取核心配置文件
        InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
        // 创建SqlSessionFactory工厂
        SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(in);
        // 取出sqlSession对象
        sqlSession = factory.openSession();
        // 动态代理对象
        usersMapper = sqlSession.getMapper(UsersMapper.class);
    }

    @After
    public void closeSqlSession() {
        // 关闭sqlSession
        sqlSession.close();
    }

    @Test
    public void testGetAll() {
        List<Users> list = usersMapper.getAll();
        list.forEach(System.out::println);   
    }
  
    @Test
    public void testGetByNameOrAddress(){
		//  List<Users> list1 = usersMapper.getByNameOrAddress("username","小");
		//  list1.forEach(System.out::println);
        List<Users> list2= usersMapper.getByNameOrAddress("address","市");
        list2.forEach(System.out::println);
    }
	
    @Test
    public void testInsert() throws ParseException {
        Users u = new Users("新增", dateFormat.parse("2000-01-01"), "1", "北京市");
        int num = usersMapper.insert(u);
        sqlSession.commit();
        // 获取主键
        System.out.println(u.getId());
    }
```

## 动态 SQL

官方文档：[动态SQL](https://mybatis.org/mybatis-3/zh/dynamic-sql.html)

动态 SQL 使条件判断更为简单：

- 定义代码片段
- 逻辑判断
- 循环（批量）处理

### `<sql>` 和 `<include>`

- `<sql>`：定义代码片段
- `<include>`：使用代码片段。

```xml
<mapper namespace="com.proj.mapper.UsersMapper">
    <!-- 定义代码片段 -->
    <sql id="allCols">
        id, username, birthday, sex, address
    </sql>
    <!-- 查询全部数据 -->
    <select id="getAll" resultType="users">
        select
        <include refid="allCols"/>
        from users
    </select>
</mapper>
```

### `<if>`

- `<if>`：条件判断

#### `<where>`

- `<where>`：多条件拼接，在查删改使用

多条件拼接：

```xml
	<select id="getByCondition" parameterType="users" resultType="users">
        select
        <include refid="allCols"/>
        from users
        <where>
            <if test="userName!=null and userName!=''">
                and username like concat('%',##{userName},'%')
            </if>
            <if test="birthday!=null">
                and birthday=##{userName}
            </if>
            <if test="sex!=null and sex!=''">
                and sex=##{sex}
            </if>
            <if test="address!=null and address!=''">
                and address like concat('%',##{address},'%')
            </if>
        </where>
    </select>
```

测试：

```java
    @Test
    public void testGetByCondition() {
        Users u = new Users();
        u.setSex("1");
        u.setUserName("小");
        List<Users> list = usersMapper.getByCondition(u);
        list.forEach(System.out::println);
    }
```

#### `<set>`

- `set`：有选择地进行更新处理，至少更新一列

有选择地更新：

```xml
	<update id="updateBySet" parameterType="users">
        update users
        <set>
            <if test="userName!=null and userName!=''">
                username=##{userName},
            </if>
            <if test="birthday!=null">
                birthday=##{userName},
            </if>
            <if test="sex!=null and sex!=''">
                sex=##{sex},
            </if>
            <if test="address!=null and address!=''">
                address=##{address},
            </if>
        </set>
        where id = ##{id}
    </update>
```

测试：

```java
    @Test
    public void testUpdateBySet(){
        Users u = new Users();
        u.setId(3);
        u.setUserName("新名字");
        usersMapper.updateBySet(u);
        sqlSession.commit();
    }
```

### `<foreach>`

- `<foreach>`：循环遍历，完成循环条件查询，批量增删改

批量操作：

```xml
    <select id="getByIds" resultType="users">
        select
        <include refid="allCols"/>
        from users
        where id in
        <foreach collection="array" item="item" separator="," open="(" close=")">
            ##{item}
        </foreach>
    </select>
	 <insert id="insertBatch">
        insert into users (username, birthday, sex, address)
        values
        <foreach collection="list" item="u" separator=",">
            (##{u.userName}, #{u.birthday}, #{u.sex}, #{u.address})
        </foreach>
    </insert>
```

测试：

```java
    @Test
    public void testGetByIds() {
        Integer[] array = {2, 4, 6};
        List<Users> list = usersMapper.getByIds(array);
        list.forEach(System.out::println);
    }
```

#### 批量更新

允许多行操作：在 `jdbc.properties` 中的数据库 url 中添加 `&allowMultiQueries=true`

```xml
 	<update id="updateBySet" parameterType="users">
    	<foreach collection="list" item="u" separator=";">
        update users
        <set>
            <if test="u.userName!=null and u.userName!=''">
                username=#{u.userName},
            </if>
            <if test="u.birthday!=null">
                birthday=#{u.userName},
            </if>
            <if test="u.sex!=null and u.sex!=''">
                sex=#{u.sex},
            </if>
            <if test="u.address!=null and u.address!=''">
                address=#{u.address},
            </if>
        </set>
        where id = #{u.id}
        </foreach>
    </update>
```

### Map

#### 入参多个

入参多个，可以指定参数位置传参，是实体类包含不住的条件。

实体类只能封装成员变量的条件，如果某个成员变量有区间范围的判断，或有两个值处理，则实体类无法包含。

根据生日范围查询：

```xml
    <!--
		List<Users> getByBirthday(Date begin, Date end);
	 -->
	<select id="getByBirthday" resultType="users">
        select
        <include refid="allCols"/>
        from users
        where birthday between #{arg0} and #{arg1}
    </select>
```

> 也可以和动态代理里一样，用 `@param` 区分参数

#### 入参 Map

入参超过 1 个以上，使用 map 封装查询条件，查询条件更明确。

**map 的 key 和 xml 中的参数名对应**

```xml
     <!--
		  List<Users> getByMap(Map map);
	 -->
	<select id="getByMap" resultType="users">
        select
        <include refid="allCols"/>
        from users
        where birthday between #{birthdayBegin} and #{birthdayEnd}
    </select>
```

#### 返回 Map

如果返回的数据实体类无法包含，可以使用 map 返回多张表的数据，返回数据没有任何关系，都是 Object 类型。

返回的 map 的 key 是列名或别名。

**返回一行 map**

```xml
    <!--
	   Map getMap(Integer id);
	-->
	<select id="getMap" parameterType="int" resultType="map">
        select
        <include refid="allCols"/>
        from users
        where id=#{id}
    </select>
```

```java
    @Test
    public void testGetResultMap(){
        Map map = usersMapper.getMap(1);
        System.out.println(map);
        System.out.println(map.get("username"));
    }
```

**返回多行 map**

```xml
    <!--
	   List<Map> getMulMap();
	-->
	<select id="getMulMap" resultType="map">
        select
        <include refid="allCols"/>
        from users
    </select>
```

## resultMap

使用 `resultMap` 手动完成映射：

- `property`：实体类成员变量名
- `column`：数据库列名

```xml
    <resultMap id="myMap" type="users">
        <!-- 主键绑定 -->
        <id property="id" column="id"/>
        <!-- 非主键绑定 -->
        <result property="userName" column="username"/>
    </resultMap>
	<!--
	   List<Users> getAllMap();
	-->
	<select id="getAllMap" resultMap="myMap">
        select id, username
        from users
    </select>
```

```java
    @Test
    public void testGetAllMap() {
        List<Users> list = usersMapper.getAllMap();
        list.forEach(System.out::println);
    }
/* 没绑定的为空值
Users{id=1, userName='王五', birthday=null, sex='null', address='null'}
*/
```

### 一对多关系

一个老师有多个学生，假定

- 老师：`id`、`name`、`studentList`
- 学生：`id`、`name`

```xml
    <!-- 绑定老师 -->
	<resultMap id="teacherMap" type="teachers">
        <!-- 主键绑定 -->
        <id property="id" column="id"/>
        <!-- 非主键绑定 -->
        <result property="name" column="name"/>
        <!-- 外键关联 -->
        <collection property="studentList" ofType="students">
        <!-- 绑定学生 -->
        	<id property="id" column="id"/>
        	<result property="name" column="name"/>
        </collection>
    </resultMap>
```

### 多对一关系

学生对应绑定的老师，假定

- 老师：`id`、`name`、`studentList`
- 学生：`id`、`name`、`teacher`

```xml
    <!-- 绑定学生 -->
	<resultMap id="studentMap" type="students">
        <!-- 主键绑定 -->
        <id property="id" column="id"/>
        <!-- 非主键绑定 -->
        <result property="name" column="name"/>
        <!-- 外键关联 -->
        <association property="teacher" javaType="teacher"> <!-- javaType是方法返回值类型 -->
        <!-- 绑定老师 -->
        	<id property="id" column="id"/>
        	<result property="name" column="name"/>
    	</association>
    </resultMap>
```

> `studentList` 不用绑定，否则循环绑死

### 一对一关系

一个班级对应一个班级，一个班级对应一个老师。

**用 `association`**

### 多对多关系

一个文章对应多个标签，一个标签对应多个文章，两个表就是多对多关系。

通过建立中间表处理多对多关联。

**用 `collection`**

> 无论什么关联关系：
>
> - 一方持有另一方的集合，使用 `<collection>`
> - 一方持有另一方的对象，使用 `<association>`

 

## 事务

简而言之，多个 CRUD 操作对应一个事务操作。

在 MyBatis 框架中设置事务：

- `<transactionManager type="JDBC"/>`：程序员自己控制提交和回滚

可设置为自动提交：

- `sqlSession = factory.openSession(true);`：默认是 false，手动提交之后不需要手动 commit

## 缓存

Mybatis 提供两级缓存：一级缓存和二级缓存，默认开启一级缓存。

### 查询流程

缓存查询流程：

1. 查询先查缓存，如果没有则查询数据库，存在缓存中，再返回给客户端
2. 下次访问相同查询时，直接从缓存返回
3. 如果数据库发生 commit 操作，则清空缓存

### 作用域

- 一级缓存：使用 `sqlSession` 的作用域，同一个 `sqlSession` 共享一级缓存的数据
- 二级缓存：使用 `mapper` 的作用域，不同 `sqlSession` 只要访问同一个 `mapper.xml` 文件，则共享二级缓存的数据。

> ORM（Object Relational Mapping）即对象关系映射：
>
> Java 语言中以对象的方式操作数据，数据库中以表的方式存储，对象中的成员变量与表中的列之间的数据互换。
