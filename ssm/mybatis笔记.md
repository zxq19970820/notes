<img src="mybatis%E7%AC%94%E8%AE%B0/01%E4%B8%89%E5%B1%82%E6%9E%B6%E6%9E%84.png" alt="01三层架构" style="zoom: 140%;" />



# 一.搭建 Mybatis 开发环境

![image-20200606205804786](mybatis%E7%AC%94%E8%AE%B0/image-20200606205804786.png)

## 1.添加依赖

~~~java
<dependencies>

        <!--单元测试-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.4</version>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.45</version>
        </dependency>

        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.17</version>
        </dependency>

    </dependencies>
~~~



## 2.添加配置文件

~~~xml
<?xml version="1.0" encoding="UTF-8" ?>
<!--    mybatis 从 XML 中构建 SqlSessionFactory  中引入-->
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<!--mybatis的主配置文件-->
<configuration>

    <!--    类型别名-->
    <typeAliases>
        <typeAlias alias="Users" type="com.zxq.domain.User"/>
    </typeAliases>

    <!--    配置环境-->
    <environments default="development">
        <!--        配置mysql的环境-->
        <environment id="development">
            <!--            配置事务类型-->
            <transactionManager type="JDBC"/>
            <!--            配置数据源(连接池)-->
            <dataSource type="POOLED">

                <!--                获取properties文件参数值，根据key获取value-->
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url"
                          value="jdbc:mysql://127.0.0.1:3306/hmybaytis?useUnicode=true&amp;characterEncoding=UTF-8"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
            </dataSource>
        </environment>
    </environments>


    <!--        xml映射文件名有要求，xxxMapper，xxx表示实体类（表一一对应）
    users表  users类  usersMapper.xml
    UsersMapper.xml  中包含各种sql语句
    如果要执行sql语句，通过映射接口UsersMapper；跟UsersMapper.xml关联，接口名要求必须跟xml文件一致 -->

<!--    指定映射配置文件的位置，映射配置文件指的是每个dao独立的配置文件-->
    <mappers>

        <mapper resource="dao/IUserDao.xml"/>

<!--        如果是用注解来配置的话，此处应该试用版class属性指定被注解的dao全限定类名-->
<!--               <mapper class="com.zxq.dao.IUserDao"/>-->
    </mappers>


</configuration>


~~~



## 3.创建实体类

~~~java
public class User implements Serializable {
    private Integer id;
    private String username;
    private Date birthday;
    private String sex;
    private String address;

    public User(Integer id, String username, Date birthday, String sex, String address) {
        this.id = id;
        this.username = username;
        this.birthday = birthday;
        this.sex = sex;
        this.address = address;
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public Date getBirthday() {
        return birthday;
    }

    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", username='" + username + '\'' +
                ", birthday=" + birthday +
                ", sex='" + sex + '\'' +
                ", address='" + address + '\'' +
                '}';
    }
}
~~~



## 4.编写持久层mapper接口

~~~java
public interface IUserDao {

    //这是注解配置，还要改sqlMapConfig.xml
//    @Select("select * from user")
    List<User> findAll();


}
~~~



### 4.1mapper接口实现类（一般不写）

~~~java
public class UserDaoImpl implements IUserDao {

    private SqlSessionFactory factory;

    public UserDaoImpl(SqlSessionFactory factory) {
        this.factory = factory;
    }
    
    public List<User> findAll() {
//        1.使用工厂创建Session对象
        SqlSession session = factory.openSession();

//         2.使用session执行查询所有方法     使用namespace="com.zxq.dao.IUserDao"的值  和id值
        List<User> users = session.selectList("com.zxq.dao.IUserDao.findAll");
        session.close();
        return users;
    }
}
~~~





## 5.编写持久层映射接口的xml文件

~~~java
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">


<mapper namespace="com.zxq.dao.IUserDao">

    <!--    配置查询所有-->
    <select id="findAll" resultType="com.zxq.domain.User">
         select * from user
    </select>
</mapper>

~~~



## 6.编写测试类（一般使用）

~~~java
public class test {
    public static void main(String[] args) {

        String resource = "sqlMapConfig.xml";
        InputStream inputStream = null;
        
        try {
//        1读取配置文件
            in = Resources.getResourceAsStream(resource);
        } catch (IOException e) {
            e.printStackTrace();
        }
        
//       2创建sqlSessionFactory对象
 SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
        SqlSessionFactory sqlSessionFactory=sqlSessionFactoryBuilder.build(in);

//       3使用工厂对象创建Sqlsession
        SqlSession session = sqlSessionFactory.openSession(true);

//       4 使用sqlsession对象创建dao/mapper的代理对象
        IUserDao userDao = session.getMapper(IUserDao.class);

//        5使用代理对象执行方法
        List<User> users = userDao.findAll();
        for (User user : users) {
            System.out.println(user);
        }


//        6释放资源
        session.close();
        try {
            inputStream.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
        ;
    }
}
~~~



### 6.1编写测试类（自定义mapper接口实现类）

~~~java
public class DiyTest {
    public static void main(String[] args) {

        String resource = "sqlMapConfig.xml";
        InputStream inputStream = null;
        try {
            //        1读取配置文件
            inputStream = Resources.getResourceAsStream(resource);

        } catch (IOException e) {
            e.printStackTrace();
        }

//2创建sqlSessionFactory对象
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

//        3使用工厂创建dao对象
        IUserDao userDao=new UserDaoImpl(sqlSessionFactory);

//        使用自定义对象执行方法
        List<User> users = userDao.findAll();
        for (User user : users) {
            System.out.println(user);
        }

//        释放资源
        try {
            inputStream.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
~~~





## 7.测试类中的设计模式

~~~java
//创建工厂使用  构建者模式：  把对象的创建细节隐藏，使用者直接调用方法即可拿到对象
SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
        SqlSessionFactory sqlSessionFactory=sqlSessionFactoryBuilder.build(in);

//使用工厂创建Sqlsession，使用了   工厂模式：  优势，解耦（降低类之间的依赖关系）
   SqlSession session = sqlSessionFactory.openSession(true);

//使用Session对象创建Dao接口实现类使用了   代理模式   优势，不修改源码的基础上对已有方法增强
        IUserDao userDao = session.getMapper(IUserDao.class);

~~~







#  二、基于代理 Dao 实现 CRUD 操作

## 1.根据 ID 查询

### 1.1在持久层接口中添加 findById 方法

~~~java
User findById(Integer userId);
~~~





### 1.2在用户的映射配置文件中配置

~~~java
<!-- 根据 id 查询 -->
<select id="findById" resultType="com.itheima.domain.User" parameterType="int">
		select * from user where id = #{uid}
</select>
~~~

**resultType** 属性：用于指定**返回结果集**的类型。
**parameterType** 属性：用于指定**传入参数**的类型。
sql 语句中使用**#{**}字符：它代表占位符，相当于原来 jdbc 部分所学的?，都是用于执行语句时替换实际的数据。
											具体的数据是由#{}里面的内容决定的。
#{}中内容的写法：由于数据类型是基本类型，所以此处可以随意写。







### 1.3在测试类添加测试

~~~java
public class MybatisTest {

    private InputStream in;
    private SqlSession sqlSession;
    private IUserDao userDao;

    @Before//用于在测试方法执行之前执行
    public void init()throws Exception{
        //1.读取配置文件，生成字节输入流
        in = Resources.getResourceAsStream("SqlMapConfig.xml");
        //2.获取SqlSessionFactory
        SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(in);
        //3.获取SqlSession对象  //自动更新操作的保存
        sqlSession = factory.openSession(true);
        //4.获取dao的代理对象
        userDao = sqlSession.getMapper(IUserDao.class);
    }

    @After//用于在测试方法执行之后执行
    public void destroy()throws Exception{
        //提交事务
//        sqlSession.commit();
        //6.释放资源
        sqlSession.close();
        in.close();
    }

    /**
     * 测试按id查询
     */
   
  	 @Test
	public void testFindOne() {
	/	/6.执行操作
		User user = userDao.findById(41);
		System.out.println(user);
	}
}
~~~



## 2.保存操作



### 2.1 在持久层接口中添加新增方法



~~~java
/**
* 保存用户
* @param user
* @return 影响数据库记录的行数
*/
int saveUser(User user);
~~~



### 2.2在用户的映射配置文件中配置

~~~java
<!-- 保存用户-->
<insert id="saveUser" parameterType="com.itheima.domain.User">
	insert into user(username,birthday,sex,address)
	values(#{username},#{birthday},#{sex},#{address})
</insert>
~~~





### 2.3 添加测试类中的测试方法

~~~java
@Test
public void testSave(){
User user = new User();
user.setUsername("modify User property");
user.setAddress("北京市顺义区");
user.setSex("男");
user.setBirthday(new Date());
System.out.println("保存操作之前："+user);
//5.执行保存方法
userDao.saveUser(user);
System.out.println("保存操作之后："+user);
}
~~~



#### 2.3.1 问题扩展：新增用户 id 的返回值



新增用户后，同时还要返回当前新增用户的 id 值.测试类中的id就有值了

~~~java
<insert id="saveUser" parameterType="USER">

    <!-- 配置保存时获取插入的 id      keyColumn数据库的列   keyProperty类名属性 -->
	<selectKey keyColumn="id" keyProperty="id" resultType="int">
		select last_insert_id();
	</selectKey>
	insert into user(username,birthday,sex,address)
	values(#{username},#{birthday},#{sex},#{address})
</insert>
~~~



#### 测试类

~~~java
    /**
     * 测试保存操作
     */
    @Test
    public void testSave() {

        User user = new User();
        user.setUsername("艾伦");
        user.setBirthday(new Date());
        System.out.println("保存操作之前：" + user);
        //5.执行保存方法
        userDao.saveUser(user);

        System.out.println(user);
        System.out.println("保存操作之后：" + user);
        System.out.println(user);
    }
~~~





## 3.更新操作

### 3.1在持久层接口中添加更新方法

```java
/**
* 更新用户
* @param user
* @return 影响数据库记录的行数
*/
int updateUser(User user);
```



### 3.2 在用户的映射配置文件中配置

~~~java
<!-- 更新用户 -->
<update id="updateUser" parameterType="com.itheima.domain.User">
	update user set username=#{username},birthday=#{birthday},sex=#{sex},
	address=#{address} where id=#{id}
</update>
~~~



### 3.3 加入更新的测试方法

~~~java
@Test
public void testUpdateUser()throws Exception{
	//1.根据 id 查询
	User user = userDao.findById(52);
	//2.更新操作
	user.setAddress("北京市顺义区");
	int res = userDao.updateUser(user);
	System.out.println(res);
}
~~~







## 4.删除操作

### 4.1 在持久层接口中添加删除方法



~~~java
/**
* 根据 id 删除用户
* @param userId
* @return
*/
int deleteUser(Integer userId);
~~~



### 4.2 在用户的映射配置文件中配置



~~~java
<!-- 删除用户 -->
<delete id="deleteUser" parameterType="java.lang.Integer">
	delete from user where id = #{uid}
</delete>
~~~



### 4.3 加入删除的测试方法

~~~java
@Test
public void testDeleteUser() throws Exception {
//6.执行操作
int res = userDao.deleteUser(52);
System.out.println(res);
}
~~~





## 5模糊查询

### 5.1 在持久层接口中添加模糊查询方法



~~~java
/**
* 根据名称模糊查询
* @param username
* @return
*/
List<User> findByName(String username)
~~~



### 5.2 在用户的映射配置文件中配置

~~~java
<!-- 根据名称模糊查询 -->
<select id="findByName" resultType="com.itheima.domain.User" parameterType="String">
	select * from user where username like #{username}
</select>
~~~





### 5.3 加入模糊查询的测试方法

~~~java
@Test
public void testFindByName(){	//5.执行查询一个方法
    
	List<User> users = userDao.findByName("%王%");
    
	for(User user : users){
		System.out.println(user);
	}
}
~~~

在控制台输出的执行 SQL 语句如下：

![image-20200618094816474](mybatis%E7%AC%94%E8%AE%B0/image-20200618094816474.png)

我们在配置文件中没有加入%来作为模糊查询的条件，所以在传入字符串实参时，就需要给定模糊查询的标
识%。配置文件中的#{username}也只是一个占位符，所以 SQL 语句显示为“？”。







## 5.4 模糊查询的另一种配置方式



~~~java
第一步：修改 SQL 语句的配置，配置如下：
<!-- 根据名称模糊查询 -->
<select id="findByName" parameterType="string" resultType="com.itheima.domain.User">
select * from user where username like '%${value}%'
</select>
我们在上面将原来的#{}占位符，改成了${value}。注意如果用模糊查询的这种写法，那么${value}的写
法就是固定的，不能写成其它名字。
    
第二步：测试，如下：
/**
* 测试模糊查询操作
*/
@Test
public void testFindByName(){
//5.执行查询一个方法
List<User> users = userDao.findByName("王");
for(User user : users){
System.out.println(user);
}
}
~~~



在控制台输出的执行 SQL 语句如下

![image-20200618095042268](mybatis%E7%AC%94%E8%AE%B0/image-20200618095042268.png)

可以发现，我们在程序代码中就不需要加入模糊查询的匹配符%了，这两种方式的实现效果是一样的，但执行
的语句是不一样的。



## 5.5 #{}与${}的区别

~~~java
#{}表示一个占位符号
通过#{}可以实现 preparedStatement 向占位符中设置值，自动进行 java 类型和 jdbc 类型转换，
#{}可以有效防止 sql 注入。 #{}可以接收简单类型值或 pojo 属性值。 如果 parameterType 传输单个简单类
型值，#{}括号中可以是 value 或其它名称。
~~~

~~~java
${}表示拼接 sql 串
通过${}可以将 parameterType 传入的内容拼接在 sql 中且不进行 jdbc 类型转换， ${}可以接收简
单类型值或 pojo 属性值，如果 parameterType 传输单个简单类型值，${}括号中只能是 value。
~~~





## 6.查询使用聚合函数

### 6.1 在持久层接口中添加模糊查询方法

~~~java
/**
* 查询总记录条数
* @return
*/
int findTotal();
~~~





### 6.2 在用户的映射配置文件中配置

~~~java
<!-- 查询总记录条数 -->
<select id="findTotal" resultType="int">
	select count(*) from user;
</select>
~~~



### 6.3 加入聚合查询的测试方法

~~~java
@Test
public void testFindTotal() throws Exception {
	//6.执行操作
	int res = userDao.findTotal();
	System.out.println(res);
}
~~~





# 三、2.7 Mybatis 与 JDBC 编程的比较

1.数据库链接创建、释放频繁造成系统资源浪费从而影响系统性能，如果使用数据库链接池可解决此问题。
解决：
		在 SqlMapConfig.xml 中配置数据链接池，使用连接池管理数据库链接。

2.Sql 语句写在代码中造成代码不易维护，实际应用 sql 变化的可能较大，sql 变动需要改变 java 代码。
解决：
		将 Sql 语句配置在 XXXXmapper.xml 文件中与 java 代码分离。

3.向 sql 语句传参数麻烦，因为 sql 语句的 where 条件不一定，可能多也可能少，占位符需要和参数对应。
解决：
		Mybatis 自动将 java 对象映射至 sql 语句，通过 statement 中的 parameterType 定义输入参数的
类型。

4.对结果集解析麻烦，sql 变化导致解析代码变化，且解析前需要遍历，如果能将数据库记录封装成 pojo 对
象解析比较方便。
解决：
		Mybatis 自动将 sql 执行结果映射至 java 对象，通过 statement 中的 resultType 定义输出结果的
类型



# 四.传递pojo包装类进行查询

开发中通过 pojo 传递查询条件 ，查询条件是综合的查询条件，不仅包括用户查询条件还包括其它的查
询条件（比如将用户购买商品信息也作为查询条件），这时可以使用包装对象传递输入参数。
Pojo 类中包含 pojo。
需求：根据用户名查询用户信息，查询条件放到 QueryVo 的 user 属性中。



### 4.1编写 QueryVo

~~~jav
public class QueryVo implements Serializable {
	private User user;
	public User getUser() {
		return user;
	}
	public void setUser(User user) {
		this.user = user;
	}
}
~~~



### 4.2编写持久层接口

~~~java
public interface IUserDao {
/**
* 根据 QueryVo 中的条件查询用户
* @param vo
* @return
*/
List<User> findByVo(QueryVo vo);
~~~



### 4.3持久层接口的映射文件

#{user.username}默认是parameterType的属性

~~~java
<select id="findByVo" resultType="com.itheima.domain.User"
				parameterType="com.itheima.domain.QueryVo">
	select * from user where username like #{user.username};
</select>
~~~



### 4.4测试包装类作为参数



~~~~java
@Test
public void testFindByQueryVo() {
    User user = new User();
	user.setUserName("%王%");
    
	QueryVo vo = new QueryVo();
	vo.setUser(user);
	List<User> users = userDao.findByVo(vo);
    
	for(User u : users) {
	System.out.println(u);
	}
}
~~~~



# 五.resultType 配置结果类型



## 5.1resultType 属性可以指定结果集的类型，它支持**基本类型**和**实体类类型**。



它和 parameterType 一样，如果注册过类型别名的，可以直接使用别名。没有注册过的必须使用全限定类名。例如：我们的实体类此时必须是全限定类名（今天最后一个章节会讲解如何配置实体类的别名）

同时，当是实体类名称是，还有一个要求，实体类中的**属性名称**必须和**查询语句**中的列名保持一致，否则无法
实现封装。



![image-20200619220535198](mybatis%E7%AC%94%E8%AE%B0/image-20200619220535198.png)





当实体类中的**属性名称**和**查询语句**中的列名不一致时，有两种方法将返回值封装成对象



### 1.取别名

![image-20200619224159334](mybatis%E7%AC%94%E8%AE%B0/image-20200619224159334.png)





### 2.定义 resultMap



 建立 User 实体和数据库表的对应关系type 属性：指定实体类的全限定类名
id 属性：给定一个唯一标识，是给查询 select 标签引用用的。

id 标签：用于指定主键字段
result 标签：用于指定非主键字段
column 属性：用于指定数据库列名
property 属性：用于指定实体类属性名称

~~~jav
<resultMap id="userMap" type="com.itheima.domain.User" >
	<id column="id" property="userId"/>
	
	<result column="username" property="userName"/>
	<result column="sex" property="userSex"/>
	<result column="address" property="userAddress"/>
	<result column="birthday" property="userBirthday"/>
</resultMap>
~~~



<配置查询所有操作 

~~~java
<select id="findAll" resultMap="userMap">
	select * from user
</select>
~~~





resultMap 标签可以建立查询的列名和实体类的属性名称不一致时建立对应关系。从而实现封装。
在 select 标签中使用 resultMap 属性指定引用即可。同时 resultMap 可以实现将查询结果映射为复杂类
型的 pojo，比如在查询结果映射对象中包括 pojo 和 list 实现一对一查询和一对多查询。











# properties 标签配置

![image-20200714215134426](mybatis%E7%AC%94%E8%AE%B0/image-20200714215134426.png)





# 六.typeAliases取别名

可以直接使用包，在mybatisconfig.xml中

~~~java
    <typeAliases>

<!--        直接打包，包下所有类自动取别名，别名就是类名-->
        <package name="com.zxq.domain"/>

    </typeAliases>

~~~









# 第三天

![image-20200714222226504](mybatis%E7%AC%94%E8%AE%B0/image-20200714222226504.png)





# 七.动态 SQL 

## 1.\<if>标签



### 1.1	dao接口

~~~java
/**
* 根据用户信息，查询用户列表
* @param user
* @return
*/
List<User> findByUser(User user);
~~~

### 1.2 持久层 Dao 映射配置

~~~java
 <select id="findUserByCondition" resultType="User" parameterType="User">
        select  * from user where 1=1
        <if test="username!=null">
            and  username=#{username}
        </if>
        <if test="sex!=null">
            and sex=#{sex}
        </if>
  </select>
~~~

### 1.3 测试

~~~java
  @Test
    public void testFindByCondition() {
        User u = new User();
        u.setUsername("老王");
        u.setSex("女");

        List<User> userByCondition = userDao.findUserByCondition(u);
        for (User user : userByCondition) {
            System.out.println(user);
        }
    }
~~~



## 2.、\<where>标签



为了**简化**上面 where 1=1 的条件拼装，我们可以采用\<where>标签来简化开发。





### 持久层 Dao 映射配置

~~~java
<select id="findUserByCondition" resultType="User" parameterType="User">
        select  * from user
        <where>
            <if test="username!=null">
                and  username=#{username}
            </if>
            <if test="sex!=null">
                and sex=#{sex}
            </if>
        </where>

</select>
~~~





## 3.\<foreach>标签

**需求**

传入多个 id 查询用户信息，用下边两个 sql 实现：
SELECT * FROM USERS WHERE username LIKE '%张%' AND (id =10 OR id =89 OR id=16)
SELECT * FROM USERS WHERE username LIKE '%张%' AND id IN (10,89,16)
这样我们在进行范围查询时，就要将一个集合中的值，作为参数动态添加进来。
这样我们将如何进行参数的传递？



### 3.1 QueryVo 中加入一个 List 集合用于封装参数

~~~java
public class QueryVo implements Serializable {
    private User user;
    private List<Integer> ids;
}
~~~



### 3.2	持久层 Dao 接口

~~~java
/**
* 根据 id 集合查询用户
* @param vo
* @return
*/
List<User> findInIds(QueryVo vo);
~~~



### 3.3 持久层 Dao 映射配置

~~~java
<select id="findUserInIds" resultType="com.zxq.domain.User" parameterType="QueryVo">
   select * from user
   <where>
    id in
     <if test="ids!=null and ids.size()>0">
       <foreach collection="ids" open="(" close=")" item="id" separator=",">
                    #{id}
       </foreach>

       </if>
    </where>

</select>

~~~

```
<foreach>标签用于遍历集合，它的属性：
collection:代表要遍历的集合元素，注意编写时不要写#{}
open:代表语句的开始部分
close:代表结束部分
item:代表遍历集合的每个元素，生成的变量名
sperator:代表分隔符
```





**SQL 语句：**
select 字段 from user where id in (?)







### 3.4 编写测试方法



~~~java
@Test
public void testFindInIds() {
    QueryVo vo=new QueryVo();
    List<Integer> list=new ArrayList<>();
    list.add(41);
    list.add(42);
    list.add(43);
    vo.setIds(list);

    List<User> userByCondition = userDao.findUserInIds(vo);
    for (User user : userByCondition) {
        System.out.println(user);
    }
}
~~~





# 八.Mybatis 中简化编写的 SQL 片段

## 8.1 抽取重复的语句代码片段 

~~~java
<sql id="defaultSql">
		select * from user
</sql>
~~~



## 8.2引用代码片段

~~~java
<!-- 配置查询所有操作 -->
<select id="findAll" resultType="user">
	<include refid="defaultSql"></include>
</select>
    
<!-- 根据 id 查询 -->
	<select id="findById" resultType="UsEr" parameterType="int">
    <include refid="defaultSql"></include>
		where id = #{uid}
</select>
~~~





# 九.mybatis中的多表查询

![image-20200715190325620](mybatis%E7%AC%94%E8%AE%B0/image-20200715190325620.png)





![image-20200715190643803](mybatis%E7%AC%94%E8%AE%B0/image-20200715190643803.png)

![image-20200715191343892](mybatis%E7%AC%94%E8%AE%B0/image-20200715191343892.png)





## 9.1一对一查询(多对一)



### 方法一

需求
查询所有账户信息，关联查询下单用户信息。
注意：
因为一个账户信息只能供某个用户使用，所以从查询账户信息出发关联查询用户信息为一对一查询。如
果从用户信息出发查询用户下的账户信息则为一对多查询，因为一个用户可以有多个账户。



#### 9.1.1定义账户信息的实体类



~~~java
public class Account implements Serializable {
    
	private Integer id;
	private Integer uid;
	private Double money;
~~~



#### 9.1.2定义 AccountUser 类

为了能够封装上面 SQL 语句的查询结果，定义 AccountCustomer 类中要包含账户信息同时还要包含用户信
息，所以我们要在定义 AccountUser 类时可以继承 User 类。

~~~java
public class AccountUser extends Account implements Serializable {
	private String username;
	private String address;
    
 	public String toString() {
		return super.toString() + " AccountUser [username=" + username + ",
			address=" + address + "]";
}
~~~



#### 9.1.3 定义账户的持久层 Dao 接口

~~~java
public interface IAccountDao {
/**
* 查询所有账户，同时获取账户的所属用户名称以及它的地址信息
* @return
*/
List<AccountUser> findAll();
}
~~~



#### 9.1.4定义 AccountDao.xml 文件的查询配置信息

~~~java
  	<!-- 配置查询所有操作-->
<select id="findAll" resultType="accountuser">
	select a.*,u.username,u.address from account a,user u where a.uid =u.id;
</select>
~~~



注意：因为上面查询的结果中包含了账户信息同时还包含了用户信息，所以我们的返回值类型 returnType
的值设置为 **AccountUser** 类型，这样就可以接收账户信息和用户信息了。



#### 9.1.5创建 AccountTest 测试类

~~~java'
/**
*
* <p>Title: MybastisCRUDTest</p>
* <p>Description: 一对多账户的操作</p>
* <p>Company: http://www.itheima.com/ </p>
*/

public class AccountTest {
	private InputStream in ;
	private SqlSessionFactory factory;
	private SqlSession session;
	private IAccountDao accountDao;

@Test
public void testFindAll() {
//6.执行操作
	List<AccountUser> accountusers = accountDao.findAll();
	for(AccountUser au : accountusers) {
		System.out.println(au);
}
}
~~~



#### 小结

定义专门的 po 类作为输出类型，其中定义了 sql 查询结果集所有的字段。此方法较为简单，企业中使用普
遍。





### 方法二

使用 resultMap，定义专门的 resultMap 用于映射一对一查询结果。
通过面向对象的(has a)关系可以得知，我们可以在 Account 类中加入一个 User 类的对象来代表这个账户
是哪个用户的。



#### 9.1.1修改 Account 类

~~~java
public class Account implements Serializable {
    private Integer id;
	private Integer uid;
	private Double money;
    
	private User user;
}
~~~





#### 9.1.2修改 AccountDao 接口中的方法

~~~java
public interface IAccountDao {
/**
* 查询所有账户，同时获取账户的所属用户名称以及它的地址信息
* @return
*/
List<Account> findAll();
}
~~~





#### 9.1.3重新定义 AccountDao.xml 文件

~~~java
<!-- 建立对应关系 -->
<resultMap type="accountMap" id="accountMap">
	<id column="aid" property="id"/>
    
	<result column="uid" property="uid"/>
	<result column="money" property="money"/>

    <!-- 它是用于指定从表方的引用实体属性的 -->
	<association property="user" javaType="user">
		<id column="id" property="id"/>
    
		<result column="username" property="username"/>
		<result column="sex" property="sex"/>
		<result column="birthday" property="birthday"/>
		<result column="address" property="address"/>
	</association>
</resultMap>
    
<select id="findAll" resultMap="accountMap">
		select u.*,a.id as aid,a.uid,a.money from account a,user u where a.uid =u.id;
</select>
~~~





#### 9.1.4在 AccountTest 类中加入测试方法

~~~java
@Test
public void testFindAll() {
	List<Account> accounts = accountDao.findAll();
	for(Account au : accounts) {
		System.out.println(au);
		System.out.println(au.getUser());
	}
}
~~~







## 9.2一对多查询

需求：
查询所有用户信息及用户关联的账户信息。

分析：
用户信息和他的账户信息为一对多关系，并且查询过程中如果用户没有账户信息，此时也要将用户信息
查询出来，我们想到了左外连接查询比较合适。



### 9.2.1编写 SQL 语句

~~~java
SELECT
	u.*, acc.id id,acc.uid,acc.money
	FROM user u
	LEFT JOIN account acc
    ON u.id = acc.uid
~~~





### 9.2.2User 类加入 List\<Account>

~~~java
public class User implements Serializable {
	private Integer id;
	private String username;
	private Date birthday;
	private String sex;
	private String address;
    
	private List<Account> accounts;
}
~~~





### 9.2.3用户持久层 Dao 接口中加入查询方法

~~~java
List<User> findAll();
~~~



### 9.2.4 用户持久层 Dao 映射文件配置

~~~java
<resultMap type="user" id="userMap">
    
	<id column="id" property="id"></id>
    
	<result column="username" property="username"/>
	<result column="address" property="address"/>
	<result column="sex" property="sex"/>
	<result column="birthday" property="birthday"/>
    
	<!-- collection 是用于建立一对多中集合属性的对应关系，
    ofType 用于指定集合元素的数据类型
	-->
  <collection property="accounts" ofType="account">
		<id column="aid" property="id"/>
		<result column="uid" property="uid"/>
		<result column="money" property="money"/>
 </collection>
</resultMap>
    
<!-- 配置查询所有操作 -->
<select id="findAll" resultMap="userMap">
	select u.*,a.id as aid ,a.uid,a.money
    from user u left outer
    join account	a on u.id =a.uid
</select>
~~~



collection
部分定义了用户关联的账户信息。表示关联查询结果集
property="accounts"：
关联查询的结果集存储在 User 对象的上哪个属性。
ofType="account"：
指定关联查询的结果集中的对象类型即 List 中的对象类型。此处可以使用别名，也可以使用全限定名







### 9.2.5测试方法

~~~JAVA
@Test
public void testFindAll() {
//6.执行操作
List<User> users = userDao.findAll();
 	for(User user : users) {
	System.out.println("-------每个用户的内容---------");
	System.out.println(user);
	System.out.println(user.getAccounts());
  }

~~~







## 9.3多对多

通过前面的学习，我们使用 Mybatis 实现一对多关系的维护。多对多关系其实我们看成是双向的一对多关
系。







### 9.3.1业务要求及实现 SQL





需求：
实现查询所有对象并且加载它所分配的用户信息。
分析：
查询角色我们需要用到Role 表，但角色分配的用户的信息我们并不能直接找到用户信息，而是要通过中
间表(USER_ROLE 表)才能关联到用户信息。

下面是实现的 SQL 语句：

~~~java
SELECT
r.*,u.id uid,
u.username username,
u.birthday birthday,
u.sex sex,
u.address address
FROM
ROLE r
INNER JOIN
USER_ROLE ur
ON ( r.id = ur.rid)
INNER JOIN
USER u
ON (ur.uid = u.id);
~~~





### 9.3.2编写角色实体类





~~~java
public class Role implements Serializable {
private Integer roleId;
private String roleName;
private String roleDesc;
~~~









### 9.3.3编写 Role 持久层接口

~~~java
public interface IRoleDao {
/**
* 查询所有角色
* @return
*/
    
	List<Role> findAll();
}
~~~



### 9.3.4编写映射文件

~~~java
<!--定义 role 表的 ResultMap-->
<resultMap id="roleMap" type="role">
    
	<id property="roleId" column="rid"></id>
	<result property="roleName" column="role_name"></result>
	<result property="roleDesc" column="role_desc"></result>
    
	<collection property="users" ofType="user">
		<id column="id" property="id"></id>
		<result column="username" property="username"></result>
		<result column="address" property="address"></result>
		<result column="sex" property="sex"></result>
		<result column="birthday" property="birthday"></result>
	</collection>
</resultMap>
    
<!--查询所有-->
<select id="findAll" resultMap="roleMap">
	select u.*,r.id as rid,r.role_name,r.role_desc from role r
    left outer join user_role ur on r.id = ur.rid
	left outer join user u on u.id = ur.uid
</select>
</mapper>
~~~





### 9.3.5编写测试类

~~~java
@Test
public void testFindAll(){
List<Role> roles = roleDao.findAll();
    for(Role role : roles){
	System.out.println("---每个角色的信息----");
	System.out.println(role);
	System.out.println(role.getUsers());
	}
}
~~~







# 十.Mybatis中的延迟加载

​	问题：**在一对多中，当我们有一个用户，它有100个账户。**

​	      在查询用户的时候，要不要把关联的账户查出来？
​	      在查询账户的时候，要不要把关联的用户查出来？

    在查询用户时，用户下的账户信息应该是，什么时候使用，什么时候查询的。
    在查询账户时，账户的所属用户信息应该是随着账户查询时一起查询出来。		



什么是延迟加载

	在真正使用数据时才发起查询，不用的时候不查询。按需加载（懒加载）


什么是立即加载

~~~
不管用不用，只要一调用方法，马上发起查询。
~~~



在对应的四种表关系中：一对多，多对一，一对一，多对多

~~~
	一对多，多对多：通常情况下我们都是采用延迟加载。
	多对一，一对一：通常情况下我们都是采用立即加载。
~~~









# 十一. Mybatis 缓存





像大多数的持久化框架一样，Mybatis 也提供了缓存策略，通过缓存策略来减少数据库的查询次数，从而提
高性能。
Mybatis 中缓存分为一级缓存，二级缓存。

![image-20200717142230021](mybatis%E7%AC%94%E8%AE%B0/image-20200717142230021.png)



什么是缓存
		存在于内存中的临时数据。



为什么使用缓存
	减少和数据库的交互次数，提高执行效率。
	什么样的数据能使用缓存，什么样的数据不能使用



适用于缓存：
			经常查询并且不经常改变的。
			数据的正确与否对最终结果影响不大的。



不适用于缓存：
			经常改变的数据
			数据的正确与否对最终结果影响很大的。
			例如：商品的库存，银行的汇率，股市的牌价



Mybatis中的一级缓存和二级缓存
		一级缓存：
			它指的是Mybatis中==SqlSession==对象的缓存。
			当我们执行查询之后，查询的结果会同时存入到SqlSession为我们提供一块区域中。
			该区域的结构是一个Map。当我们再次查询同样的数据，mybatis会先去sqlsession中
			查询是否有，有的话直接拿出来用。
			当SqlSession对象消失时，mybatis的一级缓存也就消失了。
		

二级缓存:
	它指的是Mybatis中==SqlSessionFactory==对象的缓存。由同一个SqlSessionFactory对象创建的==SqlSession==共享其缓存。
	二级缓存的使用步骤：

​		第一步：让Mybatis框架支持二级缓存（在SqlMapConfig.xml中配置）

~~~xml
<settings>
	<!-- 开启二级缓存的支持 -->
	<setting name="cacheEnabled" value="true"/>
</settings>
~~~

​		第二步：让当前的映射文件支持二级缓存（在IUserDao.xml中配置）

~~~xml
<mapper namespace="com.itheima.dao.IUserDao">
	<!-- 开启二级缓存的支持 -->
	<cache></cache>
</mapper>
~~~



​		第三步：让当前的操作支持二级缓存（在select标签中配置）

~~~xml
<!-- 根据 id 查询 -->
<select id="findById" resultType="user" parameterType="int" useCache="true">
	select * from user where id = #{uid}
</select>
~~~

将 UserDao.xml 映射文件中的<select>标签中设置 useCache=”true”代表当前这个 statement 要使用
二级缓存，如果不使用二级缓存可以设置为 false。
注意：针对每次查询都需要最新的数据 sql，要设置成 useCache=false，禁用二级缓存。









# 十二.分页查询







# 十三.大于小于等于

第一种写法（1）：

```
原符号       <        <=      >       >=       &        '        "
替换符号    &lt;    &lt;=   &gt;    &gt;=   &amp;   &apos;  &quot;
例如：sql如下：
create_date_time &gt;= #{startTime} and  create_date_time &lt;= #{endTime}1234
```

第二种写法（2）：

```
大于等于
<![CDATA[ >= ]]>
小于等于
<![CDATA[ <= ]]>
例如：sql如下：
create_date_time <![CDATA[ >= ]]> #{startTime} and  create_date_time <![CDATA[ <= ]]> #{endTime}
```







































































p50