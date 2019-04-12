---
title: Spring Boot 学习笔记：对接数据库 ORM 框架 Mybatis 
tags:
---


官方链接：[mybatis 官方 spring boot starter 仓库连接]https://github.com/mybatis/spring-boot-starter/wiki/Quick-Start



这两天启动了一个新项目因为项目组成员一直都使用的是 Mybatis，虽然个人比较喜欢 Jpa 这种极简的模式，但是为了项目保持统一性技术选型还是定了 Mybatis 。到网上找了一下关于 Spring Boot 和 Mybatis 组合的相关资料，各种各样的形式都有，看的人心累，结合了 Mybatis 的官方 Demo 和文档终于找到了最简的两种模式，花了一天时间总结后分享出来。

Orm 框架的本质是简化编程中操作数据库的编码，发展到现在基本上就剩两家了，一个是宣称可以不用写一句 Sql 的 Hibernate，一个是可以灵活调试动态 Sql 的 Mybatis ,两者各有特点，在企业级系统开发中可以根据需求灵活使用。发现一个有趣的现象：传统企业大都喜欢使用 Hibernate ,互联网行业通常使用 Mybatis 。

Hibernate 特点就是所有的 Sql 都用 Java 代码来生成，不用跳出程序去写（看） Sql ，有着编程的完整性，发展到最顶端就是 Spring Data Jpa 这种模式了，基本上根据方法名就可以生成对应的 Sql 了，有不太了解的可以看我的上篇文章Spring Boot(五)： Spring Data Jpa 的使用。

Mybatis 初期使用比较麻烦，需要各种配置文件、实体类、Dao 层映射关联、还有一大推其它配置。当然 Mybatis 也发现了这种弊端，初期开发了generator可以根据表结果自动生产实体类、配置文件和 Dao 层代码，可以减轻一部分开发量；后期也进行了大量的优化可以使用注解了，自动管理 Dao 层和配置文件等，发展到最顶端就是今天要讲的这种模式了，mybatis-spring-boot-starter 就是 Spring Boot+ Mybatis 可以完全注解不用配置文件，也可以简单配置轻松上手。


关于 mysql 最新的驱动
```yaml
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```

mybatis-spring-boot-starter
官方说明：MyBatis Spring-Boot-Starter will help you use MyBatis with Spring Boot
其实就是 Mybatis 看 Spring Boot 这么火热也开发出一套解决方案来凑凑热闹，但这一凑确实解决了很多问题，使用起来确实顺畅了许多。mybatis-spring-boot-starter主要有两种解决方案，一种是使用注解解决一切问题，一种是简化后的老传统。

当然任何模式都需要首先引入mybatis-spring-boot-starter的 Pom 文件，现在最新版本是 2.0.0

<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.0.0</version>
</dependency>
好了下来分别介绍两种开发模式

无配置文件注解版
就是一切使用注解搞定。

1 添加相关 Maven 文件
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>2.0.0</version>
    </dependency>
     <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>
</dependencies>
完整的 Pom 包这里就不贴了，大家直接看源码

2、application.properties 添加相关配置
mybatis.type-aliases-package=com.neo.model

spring.datasource.url=jdbc:mysql://localhost:3306/test?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8&useSSL=true
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
Spring Boot 会自动加载 spring.datasource.* 相关配置，数据源就会自动注入到 sqlSessionFactory 中，sqlSessionFactory 会自动注入到 Mapper 中，对了，你一切都不用管了，直接拿起来使用就行了。

在启动类中添加对 mapper 包扫描@MapperScan

@SpringBootApplication
@MapperScan("com.neo.mapper")
public class MybatisAnnotationApplication {

    public static void main(String[] args) {
        SpringApplication.run(MybatisAnnotationApplication.class, args);
    }
}
或者直接在 Mapper 类上面添加注解@Mapper，建议使用上面那种，不然每个 mapper 加个注解也挺麻烦的

3、开发 Mapper
第三步是最关键的一块， Sql 生产都在这里

public interface UserMapper {

    @Select("SELECT * FROM users")
    @Results({
        @Result(property = "userSex",  column = "user_sex", javaType = UserSexEnum.class),
        @Result(property = "nickName", column = "nick_name")
    })
    List<UserEntity> getAll();

    @Select("SELECT * FROM users WHERE id = #{id}")
    @Results({
        @Result(property = "userSex",  column = "user_sex", javaType = UserSexEnum.class),
        @Result(property = "nickName", column = "nick_name")
    })
    UserEntity getOne(Long id);

    @Insert("INSERT INTO users(userName,passWord,user_sex) VALUES(#{userName}, #{passWord}, #{userSex})")
    void insert(UserEntity user);

    @Update("UPDATE users SET userName=#{userName},nick_name=#{nickName} WHERE id =#{id}")
    void update(UserEntity user);

    @Delete("DELETE FROM users WHERE id =#{id}")
    void delete(Long id);

}
为了更接近生产我特地将 user_sex、nick_name 两个属性在数据库加了下划线和实体类属性名不一致，另外 user_sex 使用了枚举

@Select 是查询类的注解，所有的查询均使用这个
@Result 修饰返回的结果集，关联实体类属性和数据库字段一一对应，如果实体类属性和数据库属性名保持一致，就不需要这个属性来修饰。
@Insert 插入数据库使用，直接传入实体类会自动解析属性到对应的值
@Update 负责修改，也可以直接传入对象
@delete 负责删除
了解更多属性参考这里

注意，使用#符号和$符号的不同：

// This example creates a prepared statement, something like select * from teacher where name = ?;
@Select("Select * from teacher where name = #{name}")
Teacher selectTeachForGivenName(@Param("name") String name);

// This example creates n inlined statement, something like select * from teacher where name = 'someName';
@Select("Select * from teacher where name = '${name}'")
Teacher selectTeachForGivenName(@Param("name") String name);
4、使用
上面三步就基本完成了相关 Mapper 层开发，使用的时候当作普通的类注入进入就可以了

@RunWith(SpringRunner.class)
@SpringBootTest
public class UserMapperTest {

    @Autowired
    private UserMapper userMapper;

    @Test
    public void testInsert() throws Exception {
        userMapper.insert(new User("aa1", "a123456", UserSexEnum.MAN));
        userMapper.insert(new User("bb1", "b123456", UserSexEnum.WOMAN));
        userMapper.insert(new User("cc1", "b123456", UserSexEnum.WOMAN));

        Assert.assertEquals(3, userMapper.getAll().size());
    }

    @Test
    public void testQuery() throws Exception {
        List<User> users = userMapper.getAll();
        System.out.println(users.toString());
    }


    @Test
    public void testUpdate() throws Exception {
        User user = userMapper.getOne(30l);
        System.out.println(user.toString());
        user.setNickName("neo");
        userMapper.update(user);
        Assert.assertTrue(("neo".equals(userMapper.getOne(30l).getNickName())));
    }
}
源码中 Controller 层有完整的增删改查，这里就不贴了

如何选择
两种模式各有特点，注解版适合简单快速的模式，其实像现在流行的这种微服务模式，一个微服务就会对应一个自已的数据库，多表连接查询的需求会大大的降低，会越来越适合这种模式。

老传统模式比适合大型项目，可以灵活的动态生成 Sql ，方便调整 Sql ，也有痛痛快快，洋洋洒洒的写 Sql 的感觉。


# 问题勘误

Spring Boot 启动时时区报错的问题

```
The server time zone value 'ÖÐ¹ú±ê×¼Ê±¼ä' is unrecognized or represents more than one time zone
```

解决办法如下：
```yml
spring.datasource.url=jdbc:mysql://localhost:3306/test_mybatis?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=GMT%2B8

```
这里的 GMT%2B8 代表是东八区。

# 参考链接 



- [文章配套代码连接](https://github.com/llyer/learn-spring/tree/master/spring-boot-mybatis)

- [基于 SpringBoot2.0+优雅整合 SpringBoot+Mybatis](https://segmentfault.com/a/1190000017211657#articleHeader1)