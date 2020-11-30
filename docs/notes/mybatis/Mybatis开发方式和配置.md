

> 本系列文章Github [后端进阶指南 ](https://github.com/CodeGeekLee/data-structures-and-algorithms)已收录，此项目正在完善中，欢迎star。

## 1. Mybatis的开发方式

此处使用的是JDK的动态代理方式，延迟加载使用的cglib动态代理方式

### 1.1 代理理解

代理分为静态代理和动态代理。此处先不说静态代理，因为Mybatis中使用的代理方式是动态代理。

动态代理分为两种方式：

- 基于JDK的动态代理--针对有**接口的类**进行动态代理
- 基于CGLIB的动态代理--通过**子类**继承**父类**的方式去进行代理。

### 1.2 XML方式

- 开发方式

  只需要开发Mapper接口（dao接口）和Mapper映射文件，不需要编写实现类。

- 开发规范

  Mapper接口开发方式需要遵循以下规范：

  1、 Mapper接口的类路径与Mapper.xml文件中的namespace相同。

  2、 Mapper接口方法名称和Mapper.xml中定义的每个statement的id相同。

  3、 Mapper接口方法的输入参数类型和mapper.xml中定义的每个sql 的parameterType的类型相同。

  4、 Mapper接口方法的返回值类型和mapper.xml中定义的每个sql的resultType的类型相同。

- mapper映射文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.kkb.mybatis.mapper.UserMapper">
<!-- 根据id获取用户信息 -->
	<select id="findUserById" parameterType="int" resultType="com.kkb.mybatis.po.User">
		select * from user where id = #{id}
	</select>
</mapper>
```

- mapper接口

```java
/**
 * 用户管理mapper
 */
public interface UserMapper {
	//根据用户id查询用户信息
	public User findUserById(int id) throws Exception;
}
```

- 全局配置文件中加载映射文件

```xml
<!-- 加载映射文件 -->
  <mappers>
    <mapper resource="mapper/UserMapper.xml"/>
  </mappers>

```

- 测试代码

```java
public class UserMapperTest{

	private SqlSessionFactory sqlSessionFactory;

@Before
	public void setUp() throws Exception {
		//mybatis配置文件
		String resource = "SqlMapConfig.xml";
		InputStream inputStream = Resources.getResourceAsStream(resource);
		//使用SqlSessionFactoryBuilder创建sessionFactory
		sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
	}
	@Test
	public void testFindUserById() throws Exception {
		//获取session
		SqlSession session = sqlSessionFactory.openSession();
		//获取mapper接口的代理对象
		UserMapper userMapper = session.getMapper(UserMapper.class);
		//调用代理对象方法
		User user = userMapper.findUserById(1);
		System.out.println(user);
		//关闭session
		session.close();
		
	}
}

```

### 1.3 注解方式

* 开发方式

  只需要编写mapper接口文件接口。

- mapper接口

```java
public interface AnnotationUserMapper {
	// 查询
	@Select("SELECT * FROM user WHERE id = #{id}")
	public User findUserById(int id);

	// 模糊查询用户列表
	@Select("SELECT * FROM user WHERE username LIKE '%${value}%'")
	public List<User> findUserList(String username);

	// 添加并实现主键返回
	@Insert("INSERT INTO user (username,birthday,sex,address) VALUES (#{username},#{birthday},#{sex},#{address})")
	@SelectKey(statement = "SELECT LAST_INSERT_ID()", keyProperty = "id", resultType = int.class, before = false)
	public void insertUser(User user);

}
```



- 测试代码

```java
public class AnnotationUserMapperTest {

	private SqlSessionFactory sqlSessionFactory;

	/**
	 * @Before注解的方法会在@Test注解的方法之前执行
	 * 
	 * @throws Exception
	 */
	@Before
	public void init() throws Exception {
		// 指定全局配置文件路径
		String resource = "SqlMapConfig.xml";
		// 加载资源文件（全局配置文件和映射文件）
		InputStream inputStream = Resources.getResourceAsStream(resource);
		// 还有构建者模式，去创建SqlSessionFactory对象
		sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
	}

	@Test
	public void testFindUserById() {
		SqlSession sqlSession = sqlSessionFactory.openSession();
		AnnotationUserMapper userMapper = sqlSession.getMapper(AnnotationUserMapper.class);

		User user = userMapper.findUserById(1);
		System.out.println(user);
	}

	@Test
	public void testFindUserList() {
		SqlSession sqlSession = sqlSessionFactory.openSession();
		AnnotationUserMapper userMapper = sqlSession.getMapper(AnnotationUserMapper.class);

		List<User> list = userMapper.findUserList("老郭");
		System.out.println(list);
	}

	@Test
	public void testInsertUser() {
		SqlSession sqlSession = sqlSessionFactory.openSession();
		AnnotationUserMapper userMapper = sqlSession.getMapper(AnnotationUserMapper.class);

		User user = new User();
		user.setUsername("开课吧-2");
		user.setSex("1");
		user.setAddress("致真大厦");
		userMapper.insertUser(user);
		System.out.println(user.getId());
	}

}
```



## 2. 全局配置文件

### 2.1 配置内容

SqlMapConfig.xml中配置的内容和顺序如下：

```
properties（属性）

settings（全局配置参数）

typeAliases（类型别名）

typeHandlers（类型处理器）--Java类型--JDBC类型--->数据库类型转换

objectFactory（对象工厂）

plugins（插件）--可以在Mybatis执行SQL语句的流程中，横叉一脚去实现一些功能增强，比如PageHelper分页插件，就是第三方实现的一个插件

environments（环境集合属性对象）

environment（环境子属性对象）
       transactionManager（事务管理）
       dataSource（数据源）
mappers（映射器）
```

### 2.2 properties标签

SqlMapConfig.xml可以引用java属性文件中的配置信息。

1、在classpath下定义db.properties文件，

```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/ssm?characterEncoding=utf-8
jdbc.username=root
jdbc.password=root
```

2、在SqlMapConfig.xml文件中，引用db.properties中的属性，具体如下：

```xml
   <properties resource="db.properties"/>
	<environments default="development">
		<environment id="development">
			<transactionManager type="JDBC"/>
			<dataSource type="POOLED">
				<property name="driver" value="${jdbc.driver}"/>
				<property name="url" value="${jdbc.url}"/>
				<property name="username" value="${jdbc.username}"/>
				<property name="password" value="${jdbc.password}"/>
			</dataSource>
		</environment>
	</environments>
```

properties标签除了可以使用resource属性，引用properties文件中的属性。还可以在properties标签内定义property子标签来定义属性和属性值，具体如下：

```xml
<properties>
	<property name="driver" value="com.mysql.jdbc.Driver"/>
</properties>
```

**注意： MyBatis 将按照下面的顺序来加载属性**：

- 读取properties 元素体内定义的属性。 
- 读取properties 元素中resource或 url 加载的属性，它会覆盖已读取的同名属性。 

### 2.3 typeAlias标签

**别名的作用**：就是为了简化映射文件中parameterType和ResultType中的POJO类型名称编写。

#### 2.3.1 默认支持别名

| 别名       | 映射的类型 |
| ---------- | ---------- |
| _byte      | byte       |
| _long      | long       |
| _short     | short      |
| _int       | int        |
| _integer   | int        |
| _double    | double     |
| _float     | float      |
| _boolean   | boolean    |
| string     | String     |
| byte       | Byte       |
| long       | Long       |
| short      | Short      |
| int        | Integer    |
| integer    | Integer    |
| double     | Double     |
| float      | Float      |
| boolean    | Boolean    |
| date       | Date       |
| decimal    | BigDecimal |
| bigdecimal | BigDecimal |
| map        | Map        |

#### 2.3.2 自定义别名

在SqlMapConfig.xml中进行如下配置：

```xml
<typeAliases>
	<!-- 单个别名定义 -->
	<typeAlias alias="user" type="com.kkb.mybatis.po.User"/>
	<!-- 批量别名定义，扫描整个包下的类，别名为类名（首字母大写或小写都可以） -->
	<package name="com.kkb.mybatis.po"/>
</typeAliases>
```

### 2.4 mappers标签

#### \<mapper resource=""/>

使用相对于类路径的资源

如：

```
<mapper resource="sqlmap/User.xml" />
```

#### \<mapper url="">

使用绝对路径加载资源

如：

```
<mapper url="file://d:/sqlmap/User.xml" />
```

#### \<mapper class=""/>

使用mapper接口类路径，加载映射文件。

如：

```
<mapper class="com.kkb.mybatis.mapper.UserMapper"/>
```

**注意：此种方法要求mapper接口名称和mapper映射文件名称相同，且放在同一个目录中。**

#### \<package name=""/>

注册指定包下的所有mapper接口，来加载映射文件。

如：

```
<package name="com.kkb.mybatis.mapper"/>
```

**注意：此种方法要求mapper接口名称和mapper映射文件名称相同，且放在同一个目录中。**



## 3. 输入映射和输出映射

### 3.1 parameterType(输入类型)

parameterType属性可以映射的输入参数Java类型有：**简单类型、POJO类型、Map类型、List类型（数组）**。

* Map类型和POJO类型的用法类似，这里只讲POJO类型的相关配置。

* List类型在动态SQL部分进行讲解。

#### 传递简单类型

参考《Mybatis基础》（在我的主页内查找）中用户查询的案例。

#### 传递pojo对象

参考《Mybatis基础》（在我的主页内查找）中的添加用户的案例。

#### 传递pojo包装对象

包装对象：pojo类中嵌套pojo。



##### 需求

通过包装POJO传递参数，完成用户查询。

##### QueryVO

定义包装对象QueryVO

```java
public class QueryVO {
     private User user;
}
```

##### SQL语句

```XML
SELECT * FROM user where username like '%小明%'
```

##### Mapper文件

```xml
	<!-- 使用包装类型查询用户 
		使用ognl从对象中取属性值，如果是包装对象可以使用.操作符来取内容部的属性
	-->
	<select id="findUserList" parameterType="queryVo" resultType="user">
		SELECT * FROM user where username like '%${user.username}%'
	</select>

```

##### Mapper接口

```java
/**
 * 用户管理mapper
 */
public interface UserMapper {
 	//综合查询用户列表
	public List<User> findUserList(QueryVo queryVo)throws Exception; 
}
```

 

##### 测试方法

在UserMapperTest测试类中，添加以下测试代码：

```java
@Test
	public void testFindUserList() throws Exception {
		SqlSession sqlSession = sqlSessionFactory.openSession();
		//获得mapper的代理对象
		UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
		//创建QueryVo对象
		QueryVo queryVo = new QueryVo();
		//创建user对象
		User user = new User();
		user.setUsername("小明");

		queryVo.setUser(user);
		//根据queryvo查询用户
		List<User> list = userMapper.findUserList(queryVo);
		System.out.println(list);
		sqlSession.close();
	}

```

 

### 3.2 resultType(输出类型)

resultType属性可以映射的java类型有：**简单类型、POJO类型、Map类型**。

不过Map类型和POJO类型的使用情况类似，所以只需讲解POJO类型即可。

#### 3.2.1 使用要求

使用resultType进行输出映射时，要求sql语句中**查询的列名**和要映射的**pojo的属性名**一致。

#### 3.2.2 映射简单类型

##### 案例需求

查询用户记录总数。

##### Mapper映射文件

```xml
<!-- 获取用户列表总数 -->
	<select id="findUserCount" resultType="int">
	   select count(1) from user
	</select>
```

##### Mapper接口

```java
	//查询用户总数
	public int  findUserCount() throws Exception; 
```

##### 测试代码

在UserMapperTest测试类中，添加以下测试代码：

```java
     @Test
	public void testFindUserCount() throws Exception {
		SqlSession sqlSession = sessionFactory.openSession();
		//获得mapper的代理对象
		UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
		
         int count = userMapper.findUserCount(queryVo);
         System.out.println(count);

		sqlSession.close();
	}


```

**注意：输出简单类型必须查询出来的结果集只有一列。**



#### 3.2.3 映射pojo对象

**注意：不管是单个POJO还是POJO集合，在使用resultType完成映射时，用法一样。**

参考《Mybatis基础》（在我的主页内查找）中根据用户ID查询用户信息和根据名称模糊查询用户列表的案例



### 4. resultMap

#### 4.1 使用要求

如果sql查询列名和pojo的属性名可以不一致，通过resultMap将列名和属性名作一个对应关系，最终将查询结果映射到指定的pojo对象中。

**注意：resultType底层也是通过resultMap完成映射的。**

#### 4.2 需求

将以下sql的查询结果进行映射：

```
SELECT id id_,username username_,birthday birthday_ FROM user
```



#### 4.3 Mapper接口

```java
// resultMap入门
public List<User> findUserListResultMap() throws Exception; 
```

#### 4.4 Mapper映射文件

由于sql查询列名和User类属性名不一致，所以不能使用resultType进行结构映射。

需要定义一个resultMap将sql查询列名和User类的属性名对应起来，完成结果映射。

```xml
	<!-- 定义resultMap：将查询的列名和映射的pojo的属性名做一个对应关系 -->
	<!-- 
		type：指定查询结果要映射的pojo的类型
		id：指定resultMap的唯一标示
	 -->
	<resultMap type="user" id="userListResultMap">
		<!-- 
		id标签：映射查询结果的唯一列（主键列）
			column：查询sql的列名
			property：映射结果的属性名
		-->
		<id column="id_" property="id"/>
		<!-- result标签：映射查询结果的普通列 -->
		<result column="username_" property="username"/>
		<result column="birthday_" property="birthday"/>
	</resultMap>

	<!-- resultMap入门 -->
	<select id="findUserListResultMap" resultMap="userListResultMap">
		SELECT id id_,username username_,birthday birthday_ FROM user
	</select>


```

 

- \<id/>：表示查询结果集的唯一标识，非常重要。如果是多个字段为复合唯一约束则定义多个<id />
  - Property：表示User类的属性。
  - Column：表示sql查询出来的字段名。
  - Column和property放在一块儿表示将sql查询出来的字段映射到指定的pojo类属性上。

- \<result/>：普通结果，即pojo的属性。



> 本系列文章Github [后端进阶指南 ](https://github.com/CodeGeekLee/data-structures-and-algorithms)已收录，此项目正在完善中，欢迎star。
>
> 公众号内文章都是博主原创，并且会一直更新。如果你想见证或和博主一起成长，欢迎关注！
>
> ![欢迎扫码关注哦！！！](https://i.loli.net/2019/11/24/fXyOTLCBcGMNKoj.png)