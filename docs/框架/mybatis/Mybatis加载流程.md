# MyBatis 加载流程

### MyBatis 初始化

首先读取所有 XML 或者 DAO 上的注解，此过程省略。 读取到的所有 SQL 信息存储在 Configuration 中。

 `org.apache.ibatis.session.Configuration`

```java
// key 是全限定类名 + . + 方法名
Map<String, MappedStatement> mappedStatements = new StrictMap<MappedStatement>("Mapped Statements collection");
```

每一个 `MappedStatement` 对应存储 XML 中一个 <select> <update> <delete> <insert> 等标签。

```java
org.apache.ibatis.mapping.MappedStatement {
private String resource;
      private Configuration configuration;
      // 对应标签的 id
      private String id;
      private Integer fetchSize;
      private Integer timeout;
      private StatementType statementType;
      private ResultSetType resultSetType;
      private SqlSource sqlSource;
      private Cache cache;
      private ParameterMap parameterMap;
      private List<ResultMap> resultMaps;
      private boolean flushCacheRequired;
      private boolean useCache;
      private boolean resultOrdered;
      private SqlCommandType sqlCommandType;
      private KeyGenerator keyGenerator;
      private String[] keyProperties;
      private String[] keyColumns;
      private boolean hasNestedResultMaps;
      private String databaseId;
      private Log statementLog;
      private LanguageDriver lang;
      private String[] resultSets;
}
```

### MyBatis.getMapper()

以下代码按照后缀注释的数字升序查看。

```java
// 通过 SqlSession 生成 DAO 接口的代理类，SqlSession 默认实现是 SqlSessionTemplate
DepartmentMapper mapper = sqlSession.getMapper(DepartmentMapper.class); // 1 
List<Department> departments = mapper.listDepartment();
```

#### 获取 Mapper

`org.mybatis.spring.SqlSessionTemplate`

```java
// sqlSessionProxy 是通过代理生成的，主要逻辑在 SqlSessionInterceptor.invoke() 里
sqlSessionProxy = (SqlSession)Proxy.newProxyInstance(SqlSessionFactory.class.getClassLoader(), new Class[]{SqlSession.class}, new SqlSessionTemplate.SqlSessionInterceptor());
public <T> T getMapper(Class<T> type) { // 2
	return this.getConfiguration().getMapper(type, this);
}
```

`org.apache.ibatis.session.Configuration`

```JAVA
public <T> T getMapper(Class<T> type, SqlSession sqlSession) { // 3
    return mapperRegistry.getMapper(type, sqlSession);
}
```

`org.apache.ibatis.binding.MapperRegistry`

```JAVA
public <T> T getMapper(Class<T> type, SqlSession sqlSession) { // 4
    final MapperProxyFactory<T> mapperProxyFactory = (MapperProxyFactory<T>) knownMappers.get(type); // 5
    if (mapperProxyFactory == null) {
      throw new BindingException("Type " + type + " is not known to the MapperRegistry.");
    }
    try {
      // 主要是这句代码生成代理类
      return mapperProxyFactory.newInstance(sqlSession); // 7
    } catch (Exception e) {
      throw new BindingException("Error getting mapper instance. Cause: " + e, e);
    }
} 
// 读取配置文件时会添加索引 MapperProxyFactory 到 knownMappers
public <T> void addMapper(Class<T> type) { // 6
	... ...
     knownMappers.put(type, new MapperProxyFactory<T>(type));
     ... ...
}
```

`org.apache.ibatis.binding.MapperProxyFactory`

```java
private final Map<Method, MapperMethod> methodCache = new ConcurrentHashMap<Method, MapperMethod>(); // 14
public T newInstance(SqlSession sqlSession) { // 8
    final MapperProxy<T> mapperProxy = new MapperProxy<T>(sqlSession, mapperInterface, methodCache);
    return newInstance(mapperProxy);
}
protected T newInstance(MapperProxy<T> mapperProxy) { // 9
    // mapperInterface 是被代理的接口，在上面的 MapperRegistry.addMapper() 中设置的
    return (T) Proxy.newProxyInstance(mapperInterface.getClassLoader(), new Class[] { mapperInterface }, mapperProxy);
}
```

`org.apache.ibatis.binding.MapperProxy`

```java
public class MapperProxy<T> implements InvocationHandler, Serializable { // 10
    private final Map<Method, MapperMethod> methodCache;
    public MapperProxy(SqlSession sqlSession, Class<T> mapperInterface, Map<Method, MapperMethod> methodCache) {
        this.sqlSession = sqlSession;
        this.mapperInterface = mapperInterface;
        this.methodCache = methodCache;
    }
    // 重点
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable { // 11
    	try {
        	if (Object.class.equals(method.getDeclaringClass())) {
        		return method.invoke(this, args);
      		} else if (isDefaultMethod(method)) {
       	 		return invokeDefaultMethod(proxy, method, args);
      		}
    	} catch (Throwable t) {
      		throw ExceptionUtil.unwrapThrowable(t);
    	}
    	final MapperMethod mapperMethod = cachedMapperMethod(method); // 12
        // 当调用 DAO 的方法时实际是调用的这个地方
    	return mapperMethod.execute(sqlSession, args); // 15
  	}
    // 从缓存中获取 MapperMethod
    private MapperMethod cachedMapperMethod(Method method) { // 13
        MapperMethod mapperMethod = methodCache.get(method);
        if (mapperMethod == null) {
            mapperMethod = new MapperMethod(mapperInterface, method, sqlSession.getConfiguration());
            methodCache.put(method, mapperMethod);
        }
        return mapperMethod;
    }
}

```

#### 执行 sql

`org.apache.ibatis.binding.MapperMethod`

```java
public MapperMethod(Class<?> mapperInterface, Method method, Configuration config) {
    this.command = new SqlCommand(config, mapperInterface, method); // 18
    this.method = new MethodSignature(config, mapperInterface, method);
}
public Object execute(SqlSession sqlSession, Object[] args) { // 16
    Object result;
    switch (command.getType()) { // 17 ，此处 command 在上方的构造函数中创建的
        case INSERT: {
            Object param = method.convertArgsToSqlCommandParam(args);
            result = rowCountResult(sqlSession.insert(command.getName(), param)); // 19 
            break;
        }
        case UPDATE: {
            Object param = method.convertArgsToSqlCommandParam(args);
            result = rowCountResult(sqlSession.update(command.getName(), param));
            break;
        }
        case DELETE: {
            Object param = method.convertArgsToSqlCommandParam(args);
            result = rowCountResult(sqlSession.delete(command.getName(), param));
            break;
        }
        case SELECT:
            if (method.returnsVoid() && method.hasResultHandler()) {
                executeWithResultHandler(sqlSession, args);
                result = null;
            } else if (method.returnsMany()) {
                result = executeForMany(sqlSession, args);
            } else if (method.returnsMap()) {
                result = executeForMap(sqlSession, args);
            } else if (method.returnsCursor()) {
                result = executeForCursor(sqlSession, args);
            } else {
                Object param = method.convertArgsToSqlCommandParam(args);
                result = sqlSession.selectOne(command.getName(), param); 
            }
            break;
        case FLUSH:
            result = sqlSession.flushStatements();
            break;
        default:
            throw new BindingException("Unknown execution method for: " + command.getName());
    }
    if (result == null && method.getReturnType().isPrimitive() && !method.returnsVoid()) {
        throw new BindingException("Mapper method '" + command.getName() 
                                   + " attempted to return null from a method with a primitive return type (" + method.getReturnType() + ").");
    }
    return result;
}
// 插入、修改和删除操作的返回结果处理
private Object rowCountResult(int rowCount) { // 20
    final Object result;
    if (method.returnsVoid()) {
        result = null;
    } else if (Integer.class.equals(method.getReturnType()) || Integer.TYPE.equals(method.getReturnType())) {
        result = rowCount;
    } else if (Long.class.equals(method.getReturnType()) || Long.TYPE.equals(method.getReturnType())) {
        result = (long)rowCount;
    } else if (Boolean.class.equals(method.getReturnType()) || Boolean.TYPE.equals(method.getReturnType())) {
        result = rowCount > 0;
    } else {
        throw new BindingException("Mapper method '" + command.getName() + "' has an unsupported return type: " + method.getReturnType());
    }
    return result;
}
```

上面所有 sqlSession 的方法都会进入 `org.mybatis.spring.SqlSessionTemplate.SqlSessionInterceptor.invoke()` 。

`org.mybatis.spring.SqlSessionTemplate.SqlSessionInterceptor` 位于 `SqlSessionTemplate` 类中

```java
private class SqlSessionInterceptor implements InvocationHandler { 
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable { // 21
        // 注意此处的 sqlSession 和之前的 sqlSession 不是同一个
        SqlSession sqlSession = SqlSessionUtils.getSqlSession(SqlSessionTemplate.this.sqlSessionFactory, SqlSessionTemplate.this.executorType, SqlSessionTemplate.this.exceptionTranslator); // 22
        Object unwrapped;
        try {
            // 实际调用的 sqlSession 的方法
            Object result = method.invoke(sqlSession, args);
            if (!SqlSessionUtils.isSqlSessionTransactional(sqlSession, SqlSessionTemplate.this.sqlSessionFactory)) {
                sqlSession.commit(true);
            }
            unwrapped = result;
        } catch (Throwable var11) {
            unwrapped = ExceptionUtil.unwrapThrowable(var11);
            if (SqlSessionTemplate.this.exceptionTranslator != null && unwrapped instanceof PersistenceException) {
                SqlSessionUtils.closeSqlSession(sqlSession, SqlSessionTemplate.this.sqlSessionFactory);
                sqlSession = null;
                Throwable translated = SqlSessionTemplate.this.exceptionTranslator.translateExceptionIfPossible((PersistenceException)unwrapped);
                if (translated != null) {
                    unwrapped = translated;
                }
            }
            throw (Throwable)unwrapped;
        } finally {
            // 关闭 session
            if (sqlSession != null) {
                SqlSessionUtils.closeSqlSession(sqlSession, SqlSessionTemplate.this.sqlSessionFactory);
            }
        }
        return unwrapped;
    }
}
```

`org.mybatis.spring.SqlSessionUtils`

```java
public static SqlSession getSqlSession(SqlSessionFactory sessionFactory, ExecutorType executorType, PersistenceExceptionTranslator exceptionTranslator) { // 23
    Assert.notNull(sessionFactory, "No SqlSessionFactory specified");
    Assert.notNull(executorType, "No ExecutorType specified");
    SqlSessionHolder holder = (SqlSessionHolder)TransactionSynchronizationManager.getResource(sessionFactory);
    SqlSession session = sessionHolder(executorType, holder);
    if (session != null) {
        return session;
    } else {
        if (LOGGER.isDebugEnabled()) {
            LOGGER.debug("Creating a new SqlSession");
        }
		// 重新获取一个 sqlSession，此处的 sessionFactory 是 DefaultSqlSessionFactory
        session = sessionFactory.openSession(executorType); // 24
        registerSessionHolder(sessionFactory, executorType, exceptionTranslator, session);
        return session;
    }
}
```

`org.apache.ibatis.session.defaults.DefaultSqlSessionFactory`

```java
public SqlSession openSession(ExecutorType execType) { // 25
    return openSessionFromDataSource(execType, null, false); 
}
private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit) { // 26
    Transaction tx = null;
    try {
        final Environment environment = configuration.getEnvironment();
        final TransactionFactory transactionFactory = getTransactionFactoryFromEnvironment(environment);
        tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);
        final Eecutor executor = configuration.newExecutor(tx, execType);
        // 也就是 19 处实际调用的是 DefaultSqlSession.insert(String statement, Object parameter)
        return new DefaultSqlSession(configuration, executor, autoCommit); // 27
    } catch (Exception e) {
        closeTransaction(tx); // may have fetched a connection so lets call close()
        throw ExceptionFactory.wrapException("Error opening session.  Cause: " + e, e);
    } finally {
        ErrorContext.instance().reset();
    }
}
```

`org.apache.ibatis.session.defaults.DefaultSqlSession`

```java
private final Executor executor;
public int insert(String statement, Object parameter) { // 28
    return update(statement, parameter);
}
public int update(String statement, Object parameter) { // 29
    try {
        dirty = true;
        // 此处的 MappedStatement 在文章开头部分简单描述了
        MappedStatement ms = configuration.getMappedStatement(statement); // 30
        // executor 才是实际执行 sql 的执行器
        // wrapCollection 是将 parameter 转成集合
        return executor.update(ms, wrapCollection(parameter)); // 31
    } catch (Exception e) {
        throw ExceptionFactory.wrapException("Error updating database.  Cause: " + e, e);
    } finally {
        ErrorContext.instance().reset();
    }
}
// 参数转集合
private Object wrapCollection(final Object object) { // 32
    if (object instanceof Collection) {
        StrictMap<Object> map = new StrictMap<Object>();
        map.put("collection", object);
        if (object instanceof List) {
            map.put("list", object);
        }
        return map;
    } else if (object != null && object.getClass().isArray()) {
        StrictMap<Object> map = new StrictMap<Object>();
        map.put("array", object);
        return map;
    }
    return object;
}
public int update(MappedStatement ms, Object parameter) throws SQLException { // 33
    ErrorContext.instance().resource(ms.getResource()).activity("executing an update").object(ms.getId());
    if (closed) {
        throw new ExecutorException("Executor was closed.");
    }
    clearLocalCache();
    // Executor 根据 ExecutorType 有三个实现 SimpleExecutor、ReuseExecutor、BatchExecutor，此处选择
    return doUpdate(ms, parameter); //34
}
```

`org.apache.ibatis.executor.SimpleExecutor`

```java
public int doUpdate(MappedStatement ms, Object parameter) throws SQLException { // 35
    Statement stmt = null;
    try {
        Configuration configuration = ms.getConfiguration();
        StatementHandler handler = configuration.newStatementHandler(this, ms, parameter, RowBounds.DEFAULT, null, null);
        // 此处得到的是 Statement 而不是 PreparedStatement，在query 中获取的是 PreparedStatement
        stmt = prepareStatement(handler, ms.getStatementLog()); // 36
        return handler.update(stmt); // 38
    } finally {
        closeStatement(stmt);
    }
}
private Statement prepareStatement(StatementHandler handler, Log statementLog) throws SQLException { 
    Statement stmt;
    // 数据库链接
    Connection connection = getConnection(statementLog);
    stmt = handler.prepare(connection, transaction.getTimeout()); // 37
    handler.parameterize(stmt);
    return stmt;
}
// org.apache.ibatis.executor.BaseExecutor 中的
protected Connection getConnection(Log statementLog) throws SQLException {
    // 从数据库连接池获取 connection
    Connection connection = transaction.getConnection();
    if (statementLog.isDebugEnabled()) {
        return ConnectionLogger.newInstance(connection, statementLog, queryStack);
    } else {
        return connection;
    }
}
```

`org.apache.ibatis.executor.statement.SimpleStatementHandler`

```java
public int update(Statement statement) throws SQLException { // 39
    String sql = boundSql.getSql();
    Object parameterObject = boundSql.getParameterObject();
    KeyGenerator keyGenerator = mappedStatement.getKeyGenerator();
    int rows;
    if (keyGenerator instanceof Jdbc3KeyGenerator) {
        statement.execute(sql, Statement.RETURN_GENERATED_KEYS);
        rows = statement.getUpdateCount();`
        keyGenerator.processAfter(executor, mappedStatement, statement, parameterObject);
    } else if (keyGenerator instanceof SelectKeyGenerator) {
        statement.execute(sql);
        rows = statement.getUpdateCount();
        keyGenerator.processAfter(executor, mappedStatement, statement, parameterObject);
    } else {
        statement.execute(sql);
        rows = statement.getUpdateCount();
    }
    return rows;
}
```

### 总结

简化以上流程：

```java
// 1 创建 Mapper 代理
DepartmentMapper mapper = sqlSession.getMapper(DepartmentMapper.class); 
// 2 通过动态代理生成 DAO 接口的实现类，所以第一步得到的是通过动态代理生成的 DepartmentMapper 的子类
Proxy.newProxyInstance(DepartmentMapper.class.getClassLoader(), new Class[] { DepartmentMapper.class }, mapperProxy);
// 3 执行 DAO 的插入方法
mapper.insert();
// 4 第四步会进入动态代理的 MapperProxy 的 invoke 方法
mapperMethod.execute(sqlSession, args);
// 5 根据 SQL 类型选择具体的 sqlSession 的方法
switch (command.getType()) {
    case INSERT: {
        Object param = method.convertArgsToSqlCommandParam(args);
        // 
        result = rowCountResult(sqlSession.insert(command.getName(), param)); // 19 
        break;
    }    
}
// 6 sqlSession 是通过动态代理生成的，所以接下来进入 SqlSessionInterceptor.invoke()
SqlSessionInterceptor.invoke();
// 7 通过 SqlSessionUtils 获取 sqlSession，实际得到的是 DefaultSqlSession
SqlSession sqlSession = SqlSessionUtils.getSqlSession();
// 8 调用 DefaultSqlSession 的方法执行 SQL
Object result = method.invoke(sqlSession, args);
// 9 DefaultSqlSession 中执行 SQL 首先获取 MappedStatement，然后执行器 Executor 执行 SQL
MappedStatement ms = configuration.getMappedStatement(statement);
executor.update(ms, wrapCollection(parameter));
// 10 Executor 会使用 JDBC 操作数据库
statement.execute(sql);
```

### INSERT  useGeneratedKeys 是如何注入主键的

主要是使用 JDBC 的 getGeneratedKeys 方法来取出由数据库内部生成的主键， `Eeecutor`  执行 `INSERT` 时

```java
public Object execute(SqlSession sqlSession, Object[] args) {
    Object result;
    switch (command.getType()) {
      case INSERT: {
        Object param = method.convertArgsToSqlCommandParam(args);
          // 主键处理是在 sqlSession.insert
        result = rowCountResult(sqlSession.insert(command.getName(), param));
        break;
          ...
      }
...
}
```

进入  `org.apache.ibatis.session.defaults.DefaultSqlSession#insert(java.lang.String, java.lang.Object)`

```java
// statement 是方法id , 形式是 package.class.method 
public int insert(String statement, Object parameter) {
    return update(statement, parameter);
}

public int update(String statement, Object parameter) {
...
      return executor.update(ms, wrapCollection(parameter));
...
}
```

进入  `org.apache.ibatis.executor.CachingExecutor#update`

```java
public int update(MappedStatement ms, Object parameterObject) throws SQLException {
    flushCacheIfRequired(ms);
    // **
    return delegate.update(ms, parameterObject);
}
```

进入 `org.apache.ibatis.executor.BaseExecutor#update`

```java
public int update(MappedStatement ms, Object parameter) throws SQLException {
    ...
    // **
    return doUpdate(ms, parameter);
  }
```

进入 `org.apache.ibatis.executor.SimpleExecutor#doUpdate`

```java
public int doUpdate(MappedStatement ms, Object parameter) throws SQLException {
    Statement stmt = null;
    try {
      Configuration configuration = ms.getConfiguration();
      StatementHandler handler = configuration.newStatementHandler(this, ms, parameter, RowBounds.DEFAULT, null, null);
      stmt = prepareStatement(handler, ms.getStatementLog());
      // **
      return handler.update(stmt);
    } finally {
      closeStatement(stmt);
    }
  }
```

进入 `org.apache.ibatis.executor.statement.RoutingStatementHandler#update`

```java
public int update(Statement statement) throws SQLException {
    return delegate.update(statement);
  }
```

进入 `org.apache.ibatis.executor.statement.PreparedStatementHandler#update`

```java
public int update(Statement statement) throws SQLException {
    PreparedStatement ps = (PreparedStatement) statement;
    ps.execute();
    int rows = ps.getUpdateCount();
    Object parameterObject = boundSql.getParameterObject();
    // keyGenerator 是 Jdbc3KeyGenerator
    KeyGenerator keyGenerator = mappedStatement.getKeyGenerator();
    // 此处注入主键
    keyGenerator.processAfter(executor, mappedStatement, ps, parameterObject);
    return rows;
  }
```

`mappedStatement.getKeyGenerator()` 是在解析 mapper 时决定的

```java
// 需要数据库生成的主键时，useGeneratedKeys = true
keyGenerator = context.getBooleanAttribute("useGeneratedKeys",
          configuration.isUseGeneratedKeys() && SqlCommandType.INSERT.equals(sqlCommandType))
          ? Jdbc3KeyGenerator.INSTANCE : NoKeyGenerator.INSTANCE;
```

返回看 `org.apache.ibatis.executor.keygen.Jdbc3KeyGenerator#processAfter`

```java
public void processAfter(Executor executor, MappedStatement ms, Statement stmt, Object parameter) {
    processBatch(ms, stmt, parameter);
  }
  
public void processBatch(MappedStatement ms, Statement stmt, Object parameter) {
    // useGeneratedKeys 要和 keyproperties 配置使用，指定主键要赋值给那个字段
    final String[] keyProperties = ms.getKeyProperties();
    if (keyProperties == null || keyProperties.length == 0) {
      return;
    }
    try (ResultSet rs = stmt.getGeneratedKeys()) {
      final ResultSetMetaData rsmd = rs.getMetaData();
      final Configuration configuration = ms.getConfiguration();
      if (rsmd.getColumnCount() < keyProperties.length) {
        // Error?
      } else {
          // 注入主键在此通过反射实现就不再深入 
          // method.invoke(object, params); method=set方法,object=参数model,params=主键
        assignKeys(configuration, rs, rsmd, keyProperties, parameter);
      }
    } catch (Exception e) {
      throw new ExecutorException("Error getting generated key or setting result to parameter object. Cause: " + e, e);
    }
  }  
```





### 问题

#### #{} 和 ${} 的区别

在组装 SQL 的过程中，${} 中的参数会被替换为实际参数，而 #{} 中的参数会被替换成 `?`，然后 JDBC 的PrepareStatement 会对 `?` 参数进行转义，避免 SQL 注入风险。

尽量使用 #{} 而不是 ${}。