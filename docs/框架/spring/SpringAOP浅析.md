### Spring AOP 浅析

Spring 2.0 开始支持基于 XML 和注解的 AOP 实现。

##### XML

<Aop />

##### 注解

@AspectJ

其最终实现可以参考 `org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator` ，其会自动实现所有切面增强。

![](C:\work\note\java-note\docs\spring\images\DefaultAdvisorAutoProxyCreator.png)

`DefaultAdvisorAutoProxyCreator` ` 实现了 `BeanPostProcessor` ，在 bean 初始化过程中为被代理类创建代理。

```java
// org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#initializeBean(java.lang.String, java.lang.Object, org.springframework.beans.factory.support.RootBeanDefinition)
Object wrappedBean = bean;
if (mbd == null || !mbd.isSynthetic()) {
    
    // 调用 BeanPostProcessor 的 postProcessBeforeInitialization 方法
    wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
}
...
    // 调用 init-method 指定的方法
    invokeInitMethods(beanName, wrappedBean, mbd);
...
if (mbd == null || !mbd.isSynthetic()) {
    // 调用 BeanPostProcessor 的 postProcessAfterInitialization 方法
    // AOP 增强在这里实现
    wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
}
```

AOP 增强在这里实现在 `applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName)` ， 具体是在`org.springframework.aop.framework.autoproxy.AbstractAutoProxyCreator#wrapIfNecessary` 中通过动态代理为 bean 添加了 AOP 的逻辑

```java
// 找到所有匹配的 advisor
Object[] specificInterceptors = getAdvicesAndAdvisorsForBean(bean.getClass(), beanName, null);
// 通过代理实现 AOP
if (specificInterceptors != DO_NOT_PROXY) {
    this.advisedBeans.put(cacheKey, Boolean.TRUE);
    // !重点
    Object proxy = createProxy(
        bean.getClass(), beanName, specificInterceptors, new SingletonTargetSource(bean));
    this.proxyTypes.put(cacheKey, proxy.getClass());
    return proxy;
}
```

具体为 bean 实现代理的代码在 `ProxyFactory`

