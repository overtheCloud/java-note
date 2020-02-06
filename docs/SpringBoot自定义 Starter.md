### SpringBoot 自定义 starter

有两种实现方式

- @Import 注解
- ImportSelector，首先在 `META-INF`中定义的 `spring.factories` 中配置

**案例**：定义一个过滤所有请求的过滤器

1. 定义注解 @EnableLogFilter

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Import(LogFilterAutoConfiguration.class)
public @interface EnableLogFilter {
}
```

2. 注册过滤器的 bean

```java
@Configuration
@ConditionalOnClass({LogFilterRegistrationBean.class, LogFilter.class})
public class LogFilterAutoConfiguration {
    @Bean
    @ConditionalOnMissingBean(LogFilterRegistrationBean.class)
    public LogFilterRegistrationBean logFilterRegistrationBean() {
        return new LogFilterRegistrationBean();
    }
}
```

3. 添加过滤器

```java
import org.springframework.boot.web.servlet.FilterRegistrationBean;

public class LogFilterRegistrationBean extends FilterRegistrationBean<LogFilter> {
    public LogFilterRegistrationBean() {
        super();
        this.setFilter(new LogFilter());
        this.addUrlPatterns("/*");
        this.setName("LogFilter");
        this.setOrder(1);
    }
}
```

4. 定义过滤器

```java
@Slf4j
public class LogFilter implements Filter {
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletRequest httpServletRequest = (HttpServletRequest) servletRequest;
        log.info("request url : {}", httpServletRequest.getRequestURI());
        filterChain.doFilter(servletRequest, servletResponse);
    }
}
```

