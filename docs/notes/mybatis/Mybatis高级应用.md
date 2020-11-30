> 本系列文章Github [后端进阶指南 ](https://github.com/CodeGeekLee/data-structures-and-algorithms)已收录，此项目正在完善中，欢迎star。

## 1. 关联查询

举例：因为一个订单信息只会是一个人下的订单，所以从查询订单信息出发，关联查询用户信息为一对一查询。如果从用户信息出发，查询用户下的订单信息则为一对多查询，因为一个用户可以下多个订单。

### 1.1 一对一查询

#### 需求

查询所有订单信息，关联查询下单用户信息。

#### SQL语句

**主信息：订单表**

**从信息：用户表**

```mysql
SELECT 
  orders.*,
  user.username,
  user.address
FROM
  orders LEFT JOIN user 
  ON orders.user_id = user.id

```



#### 方法一：resultType

返回resultType方式比较简单，也比较常用，就不做介绍了。

#### 方法二：resultMap

使用resultMap进行结果映射，定义专门的resultMap用于映射一对一查询结果。

##### 创建扩展po类

创建OrdersExt类（**该类用于结果集封装**），加入User属性，user属性中用于存储关联查询的用户信息，因为订单关联查询用户是一对一关系，所以这里使用单个User对象存储关联查询的用户信息。

```java
public class OrdersExt extends Orders {

    private User user;// 用户对象
	// get/set。。。。
}
```

##### Mapper映射文件

在UserMapper.xml中，添加以下代码：

```xml
<!-- 查询订单关联用户信息使用resultmap -->
	<resultMap type="OrdersExt" id="ordersAndUserRstMap">
		<id column="id" property="id"/>
		<result column="user_id" property="userId"/>
		<result column="number" property="number"/>
		<result column="createtime" property="createtime"/>
		<result column="note" property="note"/>
		<!-- 一对一关联映射 -->
		<!-- 
		property:Orders对象的user属性
		javaType：user属性对应 的类型
		 -->
		<association property="user" javaType="com.kkb.mybatis.po.User">
			<!-- column:user表的主键对应的列  property：user对象中id属性-->
			<id column="user_id" property="id"/>
			<result column="username" property="username"/>
			<result column="address" property="address"/>
		</association>
	</resultMap>
	<select id="findOrdersAndUserRstMap" resultMap="ordersAndUserRstMap">
		SELECT
			o.id,
			o.user_id,
			o.number,
			o.createtime,
			o.note,
			u.username,
			u.address
		FROM
			orders o
		JOIN `user` u ON u.id = o.user_id
	</select>

```

- **association**：表示进行一对一关联查询映射
- **property**：表示关联查询的结果存储在com.kkb.mybatis.po.Orders的user属性中
- **javaType**：表示关联查询的映射结果类型

 

##### Mapper接口

在UserMapper接口中，添加以下接口方法：

```java
public List<OrdersExt> findOrdersAndUserRstMap() throws Exception;
```

 

##### 测试代码

在UserMapperTest测试类中，添加测试代码：

```java
public void testfindOrdersAndUserRstMap()throws Exception{
		//获取session
		SqlSession session = sqlSessionFactory.openSession();
		//获限mapper接口实例
		UserMapper userMapper = session.getMapper(UserMapper.class);
		//查询订单信息
		List<OrdersExt> list = userMapper.findOrdersAndUserRstMap();
		System.out.println(list);
		//关闭session
		session.close();
	}
```



##### 小结

使用resultMap进行结果映射时，具体是使用association完成关联查询的映射，将关联查询信息映射到pojo对象中。

### 1.2 一对多查询

#### 需求

查询所有用户信息及用户关联的订单信息。

#### SQL语句

**主信息：用户信息**

**从信息：订单信息**

```mysql
SELECT
	u.*, 
	o.id oid,
	o.number,
	o.createtime,
	o.note
FROM
	`user` u
LEFT JOIN orders o ON u.id = o.user_id
```



#### 分析

在一对多关联查询时，只能使用resultMap进行结果映射：

1、一对多关联查询时，sql查询结果有多条，而映射对象是一个。

2、resultType完成结果映射的方式的一条记录映射一个对象。

3、resultMap完成结果映射的方式是以[主信息]为主对象，[从信息]映射为集合或者对象，然后封装到主对象中。

#### 修改po类

在User类中加入List<Orders> orders属性。 

#### Mapper映射文件

在UserMapper.xml文件中，添加以下代码：

```xml
<resultMap type="user" id="userAndOrderRstMap">
		<!-- 用户信息映射 -->
		<id property="id" column="id"/>
		<result property="username" column="username"/>
		<result property="birthday" column="birthday"/>
		<result property="sex" column="sex"/>
		<result property="address" column="address"/>
		<!-- 一对多关联映射 -->
		<collection property="orders" ofType="orders">
			<id property="id" column="oid"/>	
			<result property="userId" column="id"/>
			<result property="number" column="number"/>
			<result property="createtime" column="createtime"/>
			<result property="note" column="note"/>
		</collection>
	</resultMap>
	<select id="findUserAndOrderRstMap" resultMap="userAndOrderRstMap">
		SELECT
		u.*,
    o.id oid,
		o.number,
		o.createtime,
		o.note
		FROM
		`user` u
		LEFT JOIN orders o ON u.id = o.user_id
	</select>
```

 

Collection标签：定义了一对多关联的结果映射。

- **property="orders"**：关联查询的结果集存储在User对象的上哪个属性。
- **ofType="orders"**：指定关联查询的结果集中的对象类型即List中的对象类型。此处可以使用别名，也可以使用全限定名。

#### Mapper接口

```java
// resultMap入门
public List<User> findUserAndOrdersRstMap() throws Exception; 

```

#### 测试代码

```java
@Test
	public void testFindUserAndOrdersRstMap() {
		SqlSession session = sqlSessionFactory.openSession();
		UserMapper userMapper = session.getMapper(UserMapper.class);
		List<User> result = userMapper.findUserAndOrdersRstMap();
		for (User user : result) {
			System.out.println(user);
		}
		session.close();
	}

```

## 2. 延迟加载

### 2.1 什么是延迟加载

* MyBatis中的延迟加载，也称为**懒加载**，是指在进行关联查询时，按照设置延迟规则推迟对关联对象的select查询。延迟加载可以有效的减少数据库压力。

* Mybatis的延迟加载，需要通过**resultMap标签中的association和collection**子标签才能演示成功。

* Mybatis的延迟加载，也被称为是嵌套查询，对应的还有**嵌套结果**的概念，可以参考一对多关联的案例。

* 注意：**MyBatis的延迟加载只是对关联对象的查询有延迟设置，对于主加载对象都是直接执行查询语句的sql**。

### 2.2 延迟加载的分类

MyBatis根据对关联对象查询的select语句的**执行时机**，分为三种类型：**直接加载、侵入式加载与深度延迟加载**

- **直接加载：** 执行完对主加载对象的select语句，马上执行对关联对象的select查询。
- **侵入式延迟**：执行对主加载对象的查询时，不会执行对关联对象的查询。但当要访问主加载对象的某个属性（该属性不是关联对象的属性）时，就会马上执行关联对象的select查询。
- **深度延迟：**执行对主加载对象的查询时，不会执行对关联对象的查询。访问主加载对象的详情时也不会执行关联对象的select查询。只有当真正访问关联对象的详情时，才会执行对关联对象的select查询。

> 延迟加载策略需要在Mybatis的全局配置文件中，通过<settings>标签进行设置。

### 2.3 案例准备

查询订单信息及它的下单用户信息。

### 2.4 直接加载

通过对全局参数：lazyLoadingEnabled进行设置，默认就是false。

```xml
<settings>
    <!-- 延迟加载总开关 -->
    <setting name="lazyLoadingEnabled" value="false"/>
</settings>
```

### 2.5 侵入式延迟加载

```xml
<settings>
    <!-- 延迟加载总开关 -->
    <setting name="lazyLoadingEnabled" value="true"/>
    <!-- 侵入式延迟加载开关 -->
    <setting name="aggressiveLazyLoading" value="true"/>
</settings>
```

### 2.6 深度延迟加载

```xml
<settings>
    <!-- 延迟加载总开关 -->
    <setting name="lazyLoadingEnabled" value="true"/>
    <!-- 侵入式延迟加载开关 -->
    <setting name="aggressiveLazyLoading" value="false"/>
</settings>
```



### 2.7 N+1问题

- 深度延迟加载的使用会提升性能。
- 如果延迟加载的表数据太多，此时会产生N+1问题，主信息加载一次算1次，而从信息是会根据主信息传递过来的条件，去查询从表多次。

## 3. 动态SQL

动态SQL的思想：就是使用不同的动态SQL标签去完成字符串的拼接处理、循环判断。

解决的问题是：

1. 在映射文件中，会编写很多有重叠部分的SQL语句，比如SELECT语句和WHERE语句等这些重叠语句，该如何处理

2. SQL语句中的where条件有多个，但是页面只传递过来一个条件参数，此时会发生问题。

### 3.1 if标签

综合查询的案例中，查询条件是由页面传入，页面中的查询条件可能输入用户名称，也可能不输入用户名称。

```xml
	<select id="findUserList" parameterType="queryVo" resultType="user">
		SELECT * FROM user where 1=1
		<if test="user != null">
			<if test="user.username != null and user.username != ''">
				AND username like '%${user.username}%'
			</if>
		</if>
	</select>
```

 

**注意：要做『不等于空』字符串校验。**

### 3.2 where标签

上边的sql中的1=1，虽然可以保证sql语句的完整性：但是存在性能问题。Mybatis提供where标签解决该问题。

代码修改如下：



```xml
	<select id="findUserList" parameterType="queryVo" resultType="user">
		SELECT * FROM user
		<!-- where标签会处理它后面的第一个and -->
		<where>
			<if test="user != null">
				<if test="user.username != null and user.username != ''">
					AND username like '%${user.username}%'
				</if>
			</if>
		</where>
	</select>
```

 

### 3.3 sql片段

在映射文件中可使用sql标签将重复的sql提取出来，然后使用include标签引用即可，最终达到sql重用的目的，具体实现如下：

 

- 原映射文件中的代码：

  ```xml
  <select id="findUserList" parameterType="queryVo" resultType="user">
  		SELECT * FROM user
  		<!-- where标签会处理它后面的第一个and -->
  		<where>
  			<if test="user != null">
  				<if test="user.username != null and user.username != ''">
  					AND username like '%${user.username}%'
  				</if>
  			</if>			
  		</where>
  	</select>
  
  ```

- 将where条件抽取出来：

```xml
<sql id="query_user_where">
		<if test="user != null">
			<if test="user.username != null and user.username != ''">
				AND username like '%${user.username}%'
			</if>
		</if>
</sql>

```

- 使用include引用：

```xml
	<!-- 使用包装类型查询用户 使用ognl从对象中取属性值，如果是包装对象可以使用.操作符来取内容部的属性 -->
	<select id="findUserList" parameterType="queryVo" resultType="user">
		SELECT * FROM user
		<!-- where标签会处理它后面的第一个and -->
		<where>
			<include refid="query_user_where"></include>
		</where>

	</select>

```



**注意：**

**1、如果引用其它mapper.xml的sql片段，则在引用时需要加上namespace，如下：**

```
<include refid="namespace.sql片段”/>
```



### 3.4 foreach

#### 需求

综合查询时，传入多个id查询用户信息，用下边两个sql实现：

```
SELECT * FROM USER WHERE username LIKE '%老郭%' AND (id =1 OR id =10 OR id=16)

SELECT * FROM USER WHERE username LIKE '%老郭%'  AND  id  IN (1,10,16)
```

 

#### POJO

在pojo中定义list属性ids存储多个用户id，并添加getter/setter方法

#### Mapper映射文件

```xml
<sql id="query_user_where">
		<if test="user != null">
			<if test="user.username != null and user.username != ''">
				AND username like '%${user.username}%'
			</if>
		</if>
		<if test="ids != null and ids.size() > 0">
			<!-- collection：指定输入的集合参数的参数名称 -->
			<!-- item：声明集合参数中的元素变量名 -->
			<!-- open：集合遍历时，需要拼接到遍历sql语句的前面 -->
			<!-- close：集合遍历时，需要拼接到遍历sql语句的后面 -->
			<!-- separator：集合遍历时，需要拼接到遍历sql语句之间的分隔符号 -->
			<foreach collection="ids" item="id" open=" AND id IN ( "
				close=" ) " separator=",">
				#{id}
			</foreach>
		</if>
	</sql>

```



#### 测试代码

在UserMapperTest测试代码中，修改testFindUserList方法，如下：

```java
@Test
	public void testFindUserList() throws Exception {
		SqlSession sqlSession = sqlSessionFactory.openSession();
		// 获得mapper的代理对象
		UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
		// 创建QueryVo对象
		QueryVo queryVo = new QueryVo();
		// 创建user对象
		User user = new User();
		user.setUsername("老郭");

		queryVo.setUser(user);

		List<Integer> ids = new ArrayList<Integer>();
		ids.add(1);// 查询id为1的用户
		ids.add(10); // 查询id为10的用户
		queryVo.setIds(ids);

		// 根据queryvo查询用户
		List<User> list = userMapper.findUserList(queryVo);
		System.out.println(list);
		sqlSession.close();
	}

```

 

#### 注意事项

**如果parameterType不是POJO类型，而是List或者Array的话，那么foreach语句中，collection属性值需要固定写死为list或者array。**



> 本系列文章Github [后端进阶指南 ](https://github.com/CodeGeekLee/data-structures-and-algorithms)已收录，此项目正在完善中，欢迎star。
>
> 公众号内文章都是博主原创，并且会一直更新。如果你想见证或和博主一起成长，欢迎关注！
>
> ![](https://user-gold-cdn.xitu.io/2020/2/19/1705c47c1b2eb7b8?w=300&h=300&f=png&s=11666)
