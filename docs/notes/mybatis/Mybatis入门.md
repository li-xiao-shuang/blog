

> 本系列文章Github [后端进阶指南 ](https://github.com/CodeGeekLee/data-structures-and-algorithms)已收录，此项目正在完善中，欢迎star。

## 认识MyBatis

mybatis参考网址：http://www.mybatis.org/mybatis-3/zh/index.html

Github源码地址：https://github.com/mybatis/mybatis-3

### Mybatis是什么

MyBatis 是一款优秀的**持久层框架**，它支持定制化SQL、存储过程以及高级映射。MyBatis 避免了几乎所有的 JDBC代码和手动设置参数以及获取结果集，它可以使用简单的**XML**或**注解**来配置和映射SQL信息，将接口和 Java 的 POJOs(Plain Old Java Objects,普通的 Java对象)映射成数据库中的记录。

### Mybatis的由来

- MyBatis 本是apache的一个开源项目iBatis。
- 2010年这个项目由apache software foundation 迁移到了google code，并且改名为MyBatis 。
- 2013年11月迁移到Github。 

### ORM是什么

对象-关系映射（OBJECT/RELATIONALMAPPING，简称ORM），是随着面向对象的[软件开发方法](https://baike.baidu.com/item/%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91%E6%96%B9%E6%B3%95)发展而产生的。用来把对象模型表示的对象映射到基于SQL 的关系模型数据库结构中去。这样，我们在具体的操作实体对象的时候，就不需要再去和复杂的 SQL 语句打交道，只需简单的操作实体对象的属性和方法 。ORM 技术是在对象和关系之间提供了一条桥梁，前台的对象型数据和数据库中的关系型的数据通过这个桥梁来相互转化。

### ORM框架和MyBatis的区别 

| 对比项       | Mybatis                | Hibernate              |
| ------------ | ---------------------- | ---------------------- |
| 市场占有率   | 高                     | 高                     |
| 适合的行业   | 互联网 电商 项目       | 传统的(ERP CRM OA)     |
| 性能         | 高                     | 低                     |
| Sql灵活性    | 高                     | 低                     |
| 学习门槛     | 低                     | 高                     |
| Sql配置文件  | 全局配置文件、映射文件 | 全局配置文件、映射文件 |
| ORM          | 半自动化               | 完全的自动化           |
| 数据库无关性 | 低                     | 高                     |

## 编码流程

1. 编写全局配置文件：xxxConfig.xml
2. POJO类
3. 映射文件：xxxMapper.xml
4. 编写dao代码：xxxDao接口、xxxDaoImpl实现类
5. 单元测试类

## 需求

1、根据用户id查询一个用户信息

2、根据用户名称模糊查询用户信息列表

3、添加用户

## 项目搭建

- 创建maven工程：mybatis-demo
- POM文件

```xml
<dependencies>
		<!-- mybatis依赖 -->
		<dependency>
			<groupId>org.mybatis</groupId>
			<artifactId>mybatis</artifactId>
			<version>3.4.6</version>
		</dependency>
		<!-- mysql依赖 -->
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>5.1.35</version>
		</dependency>

		<!-- 单元测试 -->
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.12</version>
		</dependency>
	</dependencies>

```

- SqlMapConfig.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	<properties resource="db.properties"></properties>
	<environments default="development">
		<environment id="development">
			<transactionManager type="JDBC" />
			<dataSource type="POOLED">
				<property name="driver" value="${db.driver}" />
				<property name="url" value="${db.url}" />
				<property name="username" value="${db.username}" />
				<property name="password" value="${db.password}" />
			</dataSource>
		</environment>
	</environments>
	<mappers>
		<mapper resource="UserMapper.xml" />
	</mappers>
</configuration>

```

- UserMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="test">
</mapper>
```

- PO类

```java
public class User {

	private int id;
	private String username;
	private Date birthday;
	private String sex;
	private String address;
	// getter\setter方法
}
```

 

## 需求实现

### 查询用户

#### 映射文件

```xml
<!-- 根据id获取用户信息 -->
<select id="findUserById" parameterType="int" resultType="com.kkb.mybatis.po.User">
	select * from user where id = #{id} 
</select>

<!-- 根据名称模糊查询用户列表 -->
<select id="findUserByUsername" parameterType="java.lang.String" 
			resultType="com.kkb.mybatis.po.User">
	 select * from user where username like '%${value}%' 
</select>
```

**配置说明：**

```
 - parameterType：定义输入参数的Java类型，
 - resultType：定义结果映射类型。
 - #{}：相当于JDBC中的？占位符
 - #{id}表示使用preparedstatement设置占位符号并将输入变量id传到sql。
 - ${value}：取出参数名为value的值。将${value}占位符替换。	
	
注意：如果是取简单数量类型的参数，括号中的参数名称必须为value
```

#### dao接口和实现类

```java
public interface UserDao {
	public User findUserById(int id) throws Exception;
  public List<User> findUsersByName(String name) throws Exception;
}
```

- 生命周期（作用范围）

1. sqlsession：方法级别
2. sqlsessionFactory：全局范围（应用级别）
3. sqlsessionFactoryBuilder：方法级别

```java
public class UserDaoImpl implements UserDao {
	//注入SqlSessionFactory
	public UserDaoImpl(SqlSessionFactory sqlSessionFactory){
		this. sqlSessionFactory = sqlSessionFactory;
	}
	
	private SqlSessionFactory sqlSessionFactory;
    
	@Override
	public User findUserById(int id) throws Exception {
		SqlSession session = sqlSessionFactory.openSession();
		User user = null;
		try {
			//通过sqlsession调用selectOne方法获取一条结果集
			//参数1：指定定义的statement的id,参数2：指定向statement中传递的参数
			user = session.selectOne("test.findUserById", id);
			System.out.println(user);	
		} finally{
			session.close();
		}
		return user;
	}
    
    
	@Override
	public List<User> findUsersByName(String name) throws Exception {
		SqlSession session = sqlSessionFactory.openSession();
		List<User> users = null;
		try {
			users = session.selectList("test.findUsersByName", name);
			System.out.println(users);		
		} finally{
			session.close();
		}
		return users;
	}
}
```



#### 测试代码

```java
public class MybatisTest {
	
	private SqlSessionFactory sqlSessionFactory;
	
	@Before
	public void init() throws Exception {
		SqlSessionFactoryBuilder sessionFactoryBuilder = new SqlSessionFactoryBuilder();
		InputStream inputStream = Resources.getResourceAsStream("SqlMapConfig.xml");
		sqlSessionFactory = sessionFactoryBuilder.build(inputStream);
	}

	@Test
	public void testFindUserById() {
		UserDao userDao = new UserDaoImpl(sqlSessionFactory);
		User user = userDao.findUserById(22);
		System.out.println(user);
	}
    @Test
	public void testFindUsersByName() {
		UserDao userDao = new UserDaoImpl(sqlSessionFactory);
		List<User> users = userDao.findUsersByName("老郭");
		System.out.println(users);
	}
}
```



#### #{}和${}区别

- 区别1

```
#{} ：相当于JDBC SQL语句中的占位符? (PreparedStatement)

${}  : 相当于JDBC SQL语句中的连接符合 + (Statement)
```

- 区别2

```
#{} ： 进行输入映射的时候，会对参数进行类型解析（如果是String类型，那么SQL语句会自动加上’’）

${}  :进行输入映射的时候，将参数原样输出到SQL语句中
```

- 区别3

```
#{} ： 如果进行简单类型（String、Date、8种基本类型的包装类）的输入映射时，#{}中参数名称可以任意

${}  : 如果进行简单类型（String、Date、8种基本类型的包装类）的输入映射时，${}中参数名称必须是value
```

- 区别4 

```
${} :存在SQL注入问题 ，使用OR 1=1 关键字将查询条件忽略
```



### 添加用户

**#{}：是通过反射获取数据的---StaticSqlSource** 

**${}：是通过OGNL表达式会随着对象的嵌套而相应的发生层级变化 --DynamicSqlSource**

#### 映射文件

```xml
<!-- 添加用户 -->
	<insert id="insertUser" parameterType="com.kkb.mybatis.po.User">
	  insert into user(username,birthday,sex,address) 
	  values(#{username},#{birthday},#{sex},#{address})
	</insert>
```

#### dao接口和实现类

```java
public interface UserDao {
	public void insertUser(User user) throws Exception;
}
```



```java
public class UserDaoImpl implements UserDao {
	//注入SqlSessionFactory
	public UserDaoImpl(SqlSessionFactory sqlSessionFactory){
		this. sqlSessionFactory = sqlSessionFactory;
	}
	
	private SqlSessionFactory sqlSessionFactory;
    
	@Override
	Public void insertUser(User user) throws Exception {
		SqlSession sqlSession = sqlSessionFactory.openSession();
		try {
			sqlSession.insert("test.insertUser", user);
			sqlSession.commit();
		} finally{
			session.close();
		}	
	}
}

```



#### 测试代码

```java
	@Override
	Public void insertUser(User user) throws Exception {
		SqlSession sqlSession = sqlSessionFactory.openSession();
		try {
			sqlSession.insert("insertUser", user);
			sqlSession.commit();
		} finally{
			session.close();
		}
		
	}
```

#### 主键返回

```xml
<insert id="insertUser" parameterType="com.kkb.mybatis.po.User">
		<!-- selectKey将主键返回，需要再返回 -->
		<selectKey keyProperty="id" order="AFTER" resultType="java.lang.Integer">
			select LAST_INSERT_ID()
		</selectKey>
	   insert into user(username,birthday,sex,address)
	    values(#{username},#{birthday},#{sex},#{address});
	</insert>

```

添加selectKey标签实现主键返回。

- **keyProperty**:指定返回的主键，存储在pojo中的哪个属性
- **order**：selectKey标签中的sql的执行顺序，是相对与insert语句来说。由于mysql的自增原理，执行完insert语句之后才将主键生成，所以这里selectKey的执行顺序为after。
- **resultType**:返回的主键对应的JAVA类型
- **LAST_INSERT_ID()**:是mysql的函数，返回auto_increment自增列新记录id值。



> 本系列文章Github [后端进阶指南 ](https://github.com/CodeGeekLee/data-structures-and-algorithms)已收录，此项目正在完善中，欢迎star。
>
> 公众号内文章都是博主原创，并且会一直更新。如果你想见证或和博主一起成长，欢迎关注！
>
> ![欢迎扫码关注哦！！！](https://i.loli.net/2019/11/24/fXyOTLCBcGMNKoj.png)