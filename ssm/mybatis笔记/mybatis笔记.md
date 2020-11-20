<img src="mybatis%E7%AC%94%E8%AE%B0/01%E4%B8%89%E5%B1%82%E6%9E%B6%E6%9E%84.png" alt="01三层架构" style="zoom: 200%;" />



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

~~~java
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





## 8.测试类中的设计模式

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
<!-- 配置保存时获取插入的 id -->
<selectKey keyColumn="id" keyProperty="id" resultType="int">
select last_insert_id();
</selectKey>
insert into user(username,birthday,sex,address)
values(#{username},#{birthday},#{sex},#{address})
</insert>
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
public void testFindByName(){//5.执行查询一个方法
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
QueryVo vo = new QueryVo();
User user = new User();
user.setUserName("%王%");
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























p30