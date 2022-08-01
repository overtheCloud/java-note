###  Spring-session + redis 实现 Spring-boot 多实例 session 共享

>spring-boot 版本 2.0.4.RELEASE
>
>redis 版本 5.05

#### 依赖

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.4.RELEASE</version>
    <relativePath />
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.session</groupId>
        <artifactId>spring-session-data-redis</artifactId>
    </dependency>
</dependencies>
```



#### 启动类上添加注解 @EnableRedisHttpSession

### 问题

##### 1. ERR unknown command `CONFIG`, with args beginning with: `GET`, `notify-keyspace-events`, 

> [Spring session redis ERR unknown command 'CONFIG'](https://www.cnblogs.com/coderzl/p/7512644.html) -- 来自 cnblogs

开启 redis 的 `notify-keyspace-events Ex ` 后重启redis，然后项目中添加如下代码：

```java
@Configuration
public class RedisConfig {
    @Bean
    public static ConfigureRedisAction configureRedisAction() {
        return ConfigureRedisAction.NO_OP;
    }
}
```



