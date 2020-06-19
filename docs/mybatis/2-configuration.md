# Configuration

Mybatis 的配置文件有以下属性

### properties 

`<properties>` 可以从 properties 文件中读取配置，并赋值给变量

```xml
<properties resource="org/mybatis/example/config.properties">
  <property name="username" value="dev_user"/>
  <property name="password" value="F2Fa3!33TYyg"/>
</properties>
```

上面的配置可以在 mybatis-conf.xml 文件中的其他地方引用，语法是 `${property_name}`

```xml
<dataSource type="POOLED">
  <property name="driver" value="${driver}"/>
  <property name="url" value="${url}"/>
  <property name="username" value="${username}"/>
  <property name="password" value="${password}"/>
</dataSource>
```

`SqlSessionFactoryBuilder.build()` 方法同样可以引用 `<properties>` 的配置

```java
SqlSessionFactory factory = sqlSessionFactoryBuilder.build(reader, props);
// ... or ...
SqlSessionFactory factory = new SqlSessionFactoryBuilder.build(reader, environment, props);
```

如果属性存在于多个地方，加载顺序如下

- `<properties>` 直接定义的属性
- 从类路径资源或 properties 元素的 url 属性加载的属性将被读取，并覆盖已经指定的所有重复属性
- 作为方法参数传递的属性将最后读取，并覆盖可能已从属性主体和资源的  URL 属性加载的所有重复属性。

