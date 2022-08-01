- [Spring中的设计模式（一）](https://mp.weixin.qq.com/s/3AWW1OX5KwMDX4CM4c39kg)

> 解释器模式：在编程中，我们还需要分析一件事情，并决定它是什么意思。我们可以用解释设计模式来做。`SPEL`
>
> 建造者模式：用于简化复杂对象的构造。`BeanDefinitionBuilder`
>
> 工厂方法模式：允许通过公共静态方法对象进行初始化，称为工厂方法。`<bean>` 标签的 `<constructor-arg>`
>
> 抽象工厂模式：类似于工厂方法。不同之处在于，每个工厂实现类可以生产同一类型的目标类实例，而不是一个。`BeanFactory`

- [Spring中的设计模式（二）](https://mp.weixin.qq.com/s/fc0af9WPlTK_LduChSfLhA)

> 代理模式：使用第三方调用目标类。`AOP`
>
> 复合模式：基于具有共同行为的多个对象的存在，用于构建更大的对象。`org.springframework.beans.BeanMetadataElement`
>
> 策略模式：通过不同方式完成相同事情的几个对象。完成任务的方式取决于采用的策略。`org.springframework.web.servlet.mvc.multiaction.MethodNameResolver`
>
> 模板模式：模式定义了类行为的骨架，并将子步骤的某些步骤的延迟执行。`org.springframework.context.support.AbstractApplicationContext`

- [Spring中的设计模式（三）](https://mp.weixin.qq.com/s/wsTqmXSSM0ixMlk_2af0FA)

> 原型模式：允许通过复制已存在的对象来创建一个对象的实例。`org.springframework.beans.factory.support.AbstractBeanFactory`
>
> 对象池：在一个池中保存特定数量的对象，并根据需要重新使用。`org.springframework.scheduling.concurrent`
>
> 观察者模式：目标对象变化时，观察对象会感知到。`org.springframework.context.ApplicationListener`

- [Spring中的设计模式（四）](https://mp.weixin.qq.com/s/ZJK2U_3EBRmPPKfobjIoug)

> 适配器模式：Spring使用适配器设计模式来处理不同servlet容器中的**加载时编织**(**load-time-weaving**)。在面向切面编程（AOP）中使用**load-time-weaving**，一种方式是在类加载期间将AspectJ的方面注入字节码。另一种方式是对类进行编译时注入或对已编译的类进行静态注入。
>
> 装饰模式：为给定的对象添加补充角色。`org.springframework.cache.transaction.TransactionAwareCacheDecorator`
>
> 单例模式：Spring 的作用域中包含单例类型Singleton。

- [Spring中的设计模式（五）](https://mp.weixin.qq.com/s/RQmhjVRGqBPqd9k3NF_CxA)

> 命令模式：它允许将请求封装在一个对象内并附加一个回调动作(每次遇到所所谓的回调大家就只需要理解为一个函数方法。beanFactory后置处理器的特性中来找到指令设计模式的原理。
>
> 访问者模式：通过另一种类型的对象来使一个对象访问。使用这个设计模式中的对象将被视为访问者或对象可被访问。`org.springframework.beans.factory.config.BeanDefinitionVisitor`



