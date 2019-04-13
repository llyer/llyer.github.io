---
title: Spring Boot Redis + Spring Boot Cache 使用教程
date: 2019-04-12 14:44:45
tags:
    - 缓存
    - Redis
    - Mysql
    - Spring Boot
---

**本文配套代码地址：**

[Spring Boot Redis + Spring Boot Cache 使用教程](https://github.com/llyer/learn-spring/tree/master/spring-boot-mybatis)，**强烈建议下载代码配套阅读！**

`Redis` 是开发者们使用最多的内存数据库，内存数据库的很大的一个特性就是用来做缓存，本片文章我们通过 `Spring Boot Cache` 来整合 `Redis`，将 `Redis` 作为我们关系型数据库的缓存。

# 项目配置

在 `pom.xml` 文件中导入依赖：

```xml
        <!-- redis -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>

        <!-- cache -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-cache</artifactId>
        </dependency>

        <!-- jpa -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
```

> 其他的更多依赖以及可选项请查看源代码

**配置文件如下图所示:**

```Properties
# port
server.port=8080

# actuator 健康监控
info.name=service-redis
info.server.port=${server.port}

# redis
spring.redis.database=0
spring.redis.host=127.0.0.1
spring.redis.port=6379
spring.redis.password=your password

# spring cache type 指明使用的缓存类型是Redis
#spring.cache.type=redis

# database settings
spring.jpa.database=mysql
spring.jpa.show-sql=false
spring.jpa.hibernate.ddl-auto=update
spring.jpa.hibernate.naming-strategy=org.hibernate.cfg.ImprovedNamingStrategy
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect

# data source
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/learn_redis?useUnicode=true&characterEncoding=utf8&useSSL=false&serverTimezone=GMT%2B8
spring.datasource.username=root
spring.datasource.password=password
spring.datasource.driverClassName=com.mysql.jdbc.Driver
```

当我们在 `Spring boot` 中配置好 `Redis` 并且在 `Service` 层代码中设置对应 `Cache`，`Spring` 会自动帮我们检测缓存数据源，如果发现当前项目中有可用的 `Redis`，那么就会设置缓存数据源为 `Redis`

**`Service` 层的示例代码如下：**

```java
@Service
public class UserService {

    @Autowired
    UserRepository userRepository;

    public List<User> list() {
        System.out.println("进入 list users service 方法");
        return userRepository.findAll();
    }

    // 查询用户，并且值缓存用户ID小于10的用户
    @Cacheable(value = "user", key = "#id", condition = "#id < 10")
    public User get(Integer id) {
        System.out.println("进入 service get 方法");
        return userRepository.findById(id).orElse(null);
    }


    @CachePut(value = "user", key = "#user.id", condition = "#user.id < 10")
    public User save(User user) {
        System.out.println("进入 service save 方法");
        return userRepository.save(user);
    }

    @CachePut(value = "user", key = "#user.id", condition = "#user.id < 10")
    public User update(User user) {
        System.out.println("进入 service update 方法");
        return userRepository.save(user);
    }

    // Evict 清除缓存，一般用在删除方法之上，在删除时，也将缓存数据清空
    @CacheEvict(value = "user", key = "#id")
    public void delete(Integer id) {
        System.out.println("进入 service delete 方法");
        userRepository.deleteById(id);
    }
}
```

上图中有一些是属于 `org.springframework.cache.annotation.*` 包下面的注解。这些注解定义我们要缓存的数据，规则等信息。

- `Cacheable` 对应 `get` 方法。查询数据中进行缓存，`value` 对应的是仓库，缓存的数据为方法 `return` 的数据

- `CachePut` 对应 `post`，`put` 方法。用在插入或者更新时，将变动的信息更新到缓存。

- `CacheEvict` 对应 `delete` 方法。用在需要将缓存中的数据移除的时候。

更多详细的文档请访问：https://docs.spring.io/spring/docs/4.3.15.RELEASE/spring-framework-reference/html/cache.html


# 如何验证缓存已经正常运行？

在项目中执行以下测试用例
```java
    @Test
    public void getTest3() throws Exception {
        User u1 = new User();
        u1.setId(1);
        u1.setName("曾阿牛");
        u1.setAge(18);
        userService.save(u1);
        User u3 = userService.get(1);
        System.out.println("第一次查询：" + u3.getAge());
    }
```

发现 `Mysql` 和 `Redis` 缓存中同时多了以下数据：

`redis: `
```sh
127.0.0.1:6379> keys *
1) "user::1"
2) "runtime"
3) "caches"
127.0.0.1:6379> 
```

`mysql: `
![](https://blog-1251468774.cos.ap-shanghai.myqcloud.com/20190412_redis_01.png)

> 以上两张图片证明了，我们不仅将数据保存到了 `Mysql` 数据库中，还将数据同时添加到了缓存当中。此时，如果我们再单独的执行 `userService.get(1)` 方法，那么查询会直接将 `Redis` 缓存中的数据返回，这样相对于从数据库查询数据速度要快很多

# 如何保证 Mysql 和 Redis 数据的一致性？

**Question:** 这个时候可能有人会问？那我数据的增删改查怎么办？我的数据要更新，更新后数据库里面的数据和缓存不一致怎么办？那岂不是缓存里面的数据就变成脏数据了？缓存的设置怎么设置条件，比如：我们只想缓存 `id` 小于 `10` 的用户的数据，应该怎么做？

**Answer:** 其实解决办法也不是很复杂，我们只需要在所有可能操作要这个对应数据的地方加上对应的缓存注解就可以，例如：更新数据库时也更新缓存就可以了。具体的代码在上面的代码上已经有了。大家可以下载代码自行检验。添加以下条件进行过滤就可以了：

```java
@CachePut(value = "user", key = "#user.id", condition = "#user.id < 10")
```

接下来我们来验证一下过滤条件是否生效，分别运行一下两个测试用例：

```java
    @Test
    public void getTest3() throws Exception {
        User u1 = new User();
        u1.setId(9);
        u1.setName("灭绝师太");
        u1.setAge(50);
        userService.save(u1);
        User u3 = userService.get(9);
        System.out.println("第一次查询：" + u3.getAge());
    }

    @Test
    public void getTest3() throws Exception {
        User u1 = new User();
        u1.setId(11);
        u1.setName("张三丰");
        u1.setAge(108);
        userService.save(u1);
        User u3 = userService.get(11);
        System.out.println("第一次查询：" + u3.getAge());
    }
```

此时在 `Mysql` 和 `Redis` 中的数据状态如下：

`redis: `
```sh
127.0.0.1:6379> keys *
1) "user::1"
2) "runtime"
3) "caches"
4) "user::9"
127.0.0.1:6379> 
```

`mysql: `
![](https://blog-1251468774.cos.ap-shanghai.myqcloud.com/20190412_redis_03.png)


> 可以发现系统并没有将 `id` 为 `11` 的数据保存在缓存当中。


# 参考链接

- [配套代码](https://github.com/llyer/learn-spring/tree/master/spring-boot-mybatis)

- [Spring Cache 官方文档](https://docs.spring.io/spring/docs/4.3.15.RELEASE/spring-framework-reference/html/cache.html)

- [Redis 常用命令整理](https://www.cnblogs.com/kevinws/p/6281395.html)

- [设置 Redis 服务可以被远程访问](https://www.jianshu.com/p/0ed7e88325dd)





