## Mybaits 插件原理

转自知乎https://zhuanlan.zhihu.com/p/33581658

**版本说明：**

MyBatis 源代码版本为3.4.4

**实现原理分析**

Mybatis支持对Executor、StatementHandler、PameterHandler

和ResultSetHandler接口进行拦截，也就是说会对这4种对象进行代理。

 这里用Exeturor来进行简单的分析下，对应的具体的实现类是：org.apache.ibatis.executor.SimpleExecutor

在执行doUpdate、doQuery、doQueryCursor方法时会执行如下代码：

```java
Configurationconfiguration = ms.getConfiguration();
StatementHandler handler = configuration.newStatementHandler(this, ms, parameter, RowBounds.DEFAULT, null, null);
```

接下来我们看下org.apache.ibatis.session.Configuration：

```java
public StatementHandler newStatementHandler(Executor executor, MappedStatement mappedStatement, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) {

    StatementHandler statementHandler = new RoutingStatementHandler(executor, mappedStatement, parameterObject, rowBounds, resultHandler, boundSql);
    statementHandler = (StatementHandler) interceptorChain.pluginAll(statementHandler);

    return statementHandler;
}
```

这里关键的代码是：

```java
statementHandler = (StatementHandler) interceptorChain.pluginAll(statementHandler); 
```

上面代码功能是：对statementHandler 插入所有的Interceptor以便进行拦截，InterceptorChain里保存了所有的拦截器，它在Configuration 对象被构造出来的时候创建。

我们看下InterceptorChain:

```java
public class InterceptorChain {
    private final List<Interceptor> interceptors = new ArrayList<Interceptor>();

    public Object pluginAll(Object target) {
        for (Interceptor interceptor : interceptors) {
            target = interceptor.plugin(target);
        }
        return target;
    }

    public void addInterceptor(Interceptor interceptor) {
        interceptors.add(interceptor);
    }

    public List<Interceptor> getInterceptors() {
        return Collections.unmodifiableList(interceptors);
    }
}
```

在InterceptorChain中使用List保存了所有的拦截器。这里的interceptor.plugin(target)代码如下：

  ```java
@Override
public Object plugin(Object target) {
    return Plugin.wrap(target, this);
}
  ```

Plugin.wrap方法：

```java
public static Object wrap(Object target, Interceptor interceptor) {
    Map<Class<?>, Set<Method>> signatureMap = getSignatureMap(interceptor);
    Class<?> type = target.getClass();
    Class<?>[] interfaces = getAllInterfaces(type, signatureMap);
    if (interfaces.length > 0) {
        return Proxy.newProxyInstance(
            type.getClassLoader(),
            interfaces,
            new Plugin(target, interceptor, signatureMap));
    }
    return target;
}
```

我们再次结合(Executor)interceptorChain.pluginAll(executor)这个语句来看，这个语句内部对executor执行了多次plugin,第一次plugin后通过Plugin.wrap方法生成了第一个代理类，比如：executorProxy1，这个代理类的target属性是该executor对象。第二次plugin后通过Plugin.wrap方法生成了第二个代理类，比如：executorProxy2，这个代理类的target属性是executorProxy1...这样通过每个代理类的target属性就构成了一个代理链（从最后一个executorProxyN往前查找，通过target属性可以找到最原始的executor类）。

代理链生成后，对原始目标的方法调用都转移到代理者的invoke方法上来了。Plugin作为InvocationHandler的实现类，他的invoke方法是怎么样的呢？

```java
@Override
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    try {
        Set<Method> methods = signatureMap.get(method.getDeclaringClass());
        if (methods != null && methods.contains(method)) {
            return interceptor.intercept(new Invocation(target, method, args));
        }
        return method.invoke(target, args);
    } catch (Exception e) {
        throw ExceptionUtil.unwrapThrowable(e);
    }
}
```

在invoke里，如果方法签名和拦截中的签名一致，就调用拦截器的拦截方法intercept。我们看到传递给拦截器的是一个Invocation对象，这个对象是什么样子的，他的功能又是什么呢？

```java
public class Invocation {

    private Object target;
    private Method method;
    private Object[] args;

    public Invocation(Object target, Method method, Object[] args) {
        this.target = target;
        this.method = method;
        this.args = args;
    }

    public Object getTarget() {
        return target;
    }

    public Method getMethod() {
        return method;
    }

    public Object[] getArgs() {
        return args;
    }

    public Object proceed() throws InvocationTargetException, IllegalAccessException {
        return method.invoke(target, args);
    }
}
```

可以看到，Invocation类保存了代理对象的目标类，执行的目标类方法以及传递给它的参数。在每个拦截器的intercept方法内，最后一个语句一定是returninvocation.proceed()（不这么做的话拦截器链就断了）。invocation.proceed()只是简单的调用了下target的对应方法，如果target还是个代理，就又回到了上面的Plugin.invoke方法了。这样就形成了拦截器的调用链推进。

总结来说就是：

Executor.Method->Plugin.invoke->Interceptor.intercept->Invocation.proceed->method.invoke。