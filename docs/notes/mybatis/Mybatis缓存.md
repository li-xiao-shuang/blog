### 1. 缓存介绍

Mybatis提供**查询缓存**，如果缓存中有数据就不用从数据库中获取，用于减轻数据压力，提高系统性能。

Mybatis的查询**缓存总共有两级**，我们称之为一级缓存和二级缓存：            

- 一级缓存是**SqlSession级别**的缓存。在操作数据库时需要构造 sqlSession对象，在对象中有一个数据结构（HashMap）用于存储缓存数据。不同的sqlSession之间的缓存数据区域（HashMap）是互相不影响的。 

- 二级缓存是Mapper（namespace）级别的缓存。多个SqlSession去操作同一个Mapper的sql语句，多个SqlSession可以共用二级缓存，二级缓存是跨SqlSession的。

### 2. 一级缓存

**Mybatis默认开启了一级缓存**

![img](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2018a/6e38df6a.jpg)

**说明：**

- 第一次发起查询用户id为1的用户信息，先去找缓存中是否有id为1的用户信息，如果没有，从数据库查询用户信息，将查询到的用户信息存储到一级缓存中。
- 如果中间sqlSession去执行commit操作（执行插入、更新、删除），清空SqlSession中的一级缓存，这样做的目的为了让缓存中存储的是最新的信息，避免脏读。
- 第二次发起查询用户id为1的用户信息，先去找缓存中是否有id为1的用户信息，缓存中有，直接从缓存中获取用户信息。 

#### 2.1 测试1

```java
@Test
	public void testOneLevelCache() {
		SqlSession sqlSession = sqlSessionFactory.openSession();
		UserMapper mapper = sqlSession.getMapper(UserMapper.class);
		// 第一次查询ID为1的用户，去缓存找，找不到就去查找数据库
		User user1 = mapper.findUserById(1);
		System.out.println(user1);
		
		// 第二次查询ID为1的用户
		User user2 = mapper.findUserById(1);
		System.out.println(user2);

		sqlSession.close();
	}

```



#### 2.2 测试2

```java
@Test
	public void testOneLevelCache() {
		SqlSession sqlSession = sqlSessionFactory.openSession();
		UserMapper mapper = sqlSession.getMapper(UserMapper.class);
		// 第一次查询ID为1的用户，去缓存找，找不到就去查找数据库
		User user1 = mapper.findUserById(1);
		System.out.println(user1);
		
		User user = new User();
		user.setUsername("隔壁老詹1");
		user.setAddress("洛杉矶湖人");
		//执行增删改操作，清空缓存
		mapper.insertUser(user);
		
		// 第二次查询ID为1的用户
		User user2 = mapper.findUserById(1);
		System.out.println(user2);

		sqlSession.close();
	}

```



#### 2.3 具体应用

正式开发，是将mybatis和spring进行整合开发，事务控制在service中。

一个service方法中包括 很多mapper方法调用：

```
service{
    //开始执行时，开启事务，创建SqlSession对象
    //第一次调用mapper的方法findUserById(1)
    
    //第二次调用mapper的方法findUserById(1)，从一级缓存中取数据
    //方法结束，sqlSession关闭
}
```

如果是执行两次service调用查询相同 的用户信息，是不走一级缓存的，因为mapper方法结束，sqlSession就关闭，一级缓存就清空。

### 3. 二级缓存

#### 3.1 原理

二级缓存是mapper（namespace）级别的。

![img](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2018a/28399eba.png)        

**说明：**

1. 第一次调用mapper下的SQL去查询用户信息。查询到的信息会存到该mapper对应的**二级缓存区域**内。
2. 第二次调用相同namespace下的mapper映射文件中相同的SQL去查询用户信息。会去对应的二级缓存内取结果。
3. 如果调用相同namespace下的mapper映射文件中的增删改SQL，并执行了commit操作。此时会清空该namespace下的二级缓存。

#### 3.2 开启二级缓存

**Mybatis默认是没有开启二级缓存，开启步骤如下：**

1. 在核心配置文件SqlMapConfig.xml中加入以下内容（开启二级缓存总开关）：

```xml
<!-- 开启二级缓存总开关 -->
<settings>
    <setting name="cacheEnabled" value="true"/>
</settings>
```



1. 在UserMapper映射文件中，加入以下内容，开启二级缓存： 

```xml
<!-- 开启本mapper下的namespace的二级缓存，默认使用的是mybatis提供的PerpetualCache -->
<cache></cache>
```



#### 3.3 实现序列化

由于二级缓存的数据不一定都是存储到内存中，它的存储介质多种多样，比如说存储到文件系统中，所以需要给缓存的对象执行序列化。如果该类存在父类，那么父类也要实现序列化。       

#### 3.4 测试1

```java
@Test
	public void testTwoLevelCache() {
		SqlSession sqlSession1 = sqlSessionFactory.openSession();
		SqlSession sqlSession2 = sqlSessionFactory.openSession();
		SqlSession sqlSession3 = sqlSessionFactory.openSession();

		UserMapper mapper1 = sqlSession1.getMapper(UserMapper.class);
		UserMapper mapper2 = sqlSession2.getMapper(UserMapper.class);
		UserMapper mapper3 = sqlSession3.getMapper(UserMapper.class);
		// 第一次查询ID为1的用户，去缓存找，找不到就去查找数据库
		User user1 = mapper1.findUserById(1);
		System.out.println(user1);
		// 关闭SqlSession1
		sqlSession1.close();

		// 第二次查询ID为1的用户
		User user2 = mapper2.findUserById(1);
		System.out.println(user2);
		// 关闭SqlSession2
		sqlSession2.close();
	}

```



#### 3.5 测试2

```mysql
@Test
	public void testTwoLevelCache() {
		SqlSession sqlSession1 = sqlSessionFactory.openSession();
		SqlSession sqlSession2 = sqlSessionFactory.openSession();
		SqlSession sqlSession3 = sqlSessionFactory.openSession();

		UserMapper mapper1 = sqlSession1.getMapper(UserMapper.class);
		UserMapper mapper2 = sqlSession2.getMapper(UserMapper.class);
		UserMapper mapper3 = sqlSession3.getMapper(UserMapper.class);
		// 第一次查询ID为1的用户，去缓存找，找不到就去查找数据库
		User user1 = mapper1.findUserById(1);
		System.out.println(user1);
		// 关闭SqlSession1
		sqlSession1.close();

		//修改查询出来的user1对象，作为插入语句的参数
		user1.setUsername("隔壁老詹1");
		user1.setAddress("洛杉矶湖人");

		mapper3.insertUser(user1);

		// 提交事务
		sqlSession3.commit();
		// 关闭SqlSession3
		sqlSession3.close();

		// 第二次查询ID为1的用户
		User user2 = mapper2.findUserById(1);
		System.out.println(user2);
		// 关闭SqlSession2
		sqlSession2.close();
	}

```

 

#### 3.6 禁用二级缓存

默认二级缓存的粒度是Mapper级别的，但是如果在同一个Mapper文件中某个查询不想使用二级缓存的话，就需要对缓存的控制粒度更细。

在select标签中设置**useCache=false**，可以禁用当前select语句的二级缓存，即每次查询都是去数据库中查询，**默认情况下是true**，即该statement使用二级缓存。

```xml
<select id="findUserById" parameterType="int" resultType="com.kkb.mybatis.po.User" useCache="true">
    SELECT * FROM user WHERE id = #{id}

</select>
```

#### 3.7 刷新二级缓存

**通过flushCache属性，可以控制select、insert、update、delete标签的是否属性二级缓存**

**默认设置**

- 默认情况下如果是select语句，那么flushCache是false。

- 如果是insert、update、delete语句，那么flushCache是true。

**默认配置解读**

- 如果查询语句设置成true，那么每次查询都是去数据库查询，即意味着该查询的二级缓存失效。

- 如果增删改语句设置成false，即使用二级缓存，那么如果在数据库中修改了数据，而缓存数据还是原来的，这个时候就会出现脏读。

flushCache设置如下：

```xml
<select id="findUserById" parameterType="int"
        resultType="com.kkb.mybatis.po.User" useCache="true" flushCache="true">
        SELECT * FROM user WHERE id = #{id}
</select>
```

#### 3.8 应用场景

- 使用场景：

  对于访问响应速度要求高，但是实时性不高的查询，可以采用二级缓存技术。

- 注意事项：

  在使用二级缓存的时候，要设置一下**刷新间隔**（cache标签中有一个**flashInterval**属性）来定时刷新二级缓存，这个刷新间隔根据具体需求来设置，比如设置30分钟、60分钟等，**单位为毫秒**。

#### 3.9 局限性

**Mybatis二级缓存对细粒度的数据级别的缓存实现不好。**

- 场景：

  对商品信息进行缓存，由于商品信息查询访问量大，但是要求用户每次查询都是最新的商品信息，此时如果使用二级缓存，就无法实现当一个商品发生变化只刷新该商品的缓存信息而不刷新其他商品缓存信息，因为二级缓存是mapper级别的，当一个商品的信息发送更新，所有的商品信息缓存数据都会清空。

- 解决方法

  此类问题，需要在业务层根据需要对数据有针对性的缓存。

  比如可以对经常变化的 数据操作单独放到另一个namespace的mapper中。