### 动态代理

运行期创建接口的代理类，通过类加载器加载到内存中，实际返回的是被代理类的子类，主要实现类：`java.lang.reflect.Proxy` 和 `java.lang.reflect.InvocationHandler`

```java
Hello hello = new HelloImpl();
Hello proxyHello = (Hello)Proxy.newProxyInstance(hello.getClass().getClassLoader(), new Class[]{Hello.class}, new InvocationHandler() {
	@Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    	System.out.println("before");
        method.invoke(hello, null);
        System.out.println("after");
        return null;
    }
});

proxyHello.hello();
```

生产的代理对象代码：

```java
public final class $Proxy0 extends Proxy implements Hello {
    private static Method m1;
    private static Method m3;
    private static Method m2;
    private static Method m0;
    static {
        try {
            m1 = Class.forName("java.lang.Object").getMethod("equals", Class.forName("java.lang.Object"));
            m3 = Class.forName("com.oc.proxy.Hello").getMethod("hello");
            m2 = Class.forName("java.lang.Object").getMethod("toString");
            m0 = Class.forName("java.lang.Object").getMethod("hashCode");
        } catch (NoSuchMethodException var2) {
            throw new NoSuchMethodError(var2.getMessage());
        } catch (ClassNotFoundException var3) {
            throw new NoClassDefFoundError(var3.getMessage());
        }
    }
    public $Proxy0(InvocationHandler var1) throws  {
        super(var1);
    }
    public final void hello() throws  {
        try {
            super.h.invoke(this, m3, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }
    // 以下省略了 equals、toString、hashCode三个方法
}
```

#### `Proxy.newProxyInstance` 源码分析

```java
// 为代理类生成子类的字节码文件并加载到 JVM
Class<?> cl = getProxyClass0(loader, intfs) {
    // 生成字节码文件
    byte[] proxyClassFile = ProxyGenerator.generateProxyClass(proxyName, interfaces, accessFlags);
    // 通过类加载器将字节码文件加载到 JVM 中
    defineClass0(loader, proxyName, proxyClassFile, 0, proxyClassFile.length); 
}
// 通过反射得到生成子类的实例
Constructor<?> cons = cl.getConstructor(constructorParams);
return cons.newInstance(new Object[]{h});
```

### CGLIB

CGLIB底层使用了ASM（一个短小精悍的字节码操作框架）来操作字节码生成新的类。

```java
import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;
import java.lang.reflect.Method;

public class CGlibDemo implements MethodInterceptor {
    // Enhancer创建一个被代理对象的子类并且拦截所有的方法调用，不能够拦截final方法
    private Enhancer enhancer = new Enhancer();

    public Object getProxy(Class clazz) {
        enhancer.setSuperclass(clazz);
        enhancer.setCallback(this);
        return enhancer.create();
    }

    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        System.out.println("before");
        // 此处调用 invokeSuper() 而不是 invoke()，否则会 StackOverflowError
        Object invoke = methodProxy.invokeSuper(o, objects);
        System.out.println("after");
        System.out.println("proxy method : " + method.getName());
        return invoke;
    }

    public static void main(String[] args) {
        CGlibDemo cGlibDemo = new CGlibDemo();
        Hello hello = (Hello)cGlibDemo.getProxy(HelloImpl.class);
        hello.hello();
    }
}
```

#### 动态代理和 CGLIB 的区别

1. Java动态代理只能够对接口进行代理，不能对普通的类进行代理（因为所有生成的代理类的父类为Proxy，Java类继承机制不允许多重继承）；CGLIB能够代理普通类；

2. Java动态代理使用Java原生的反射API进行操作，在生成类上比较高效；CGLIB使用ASM框架直接对字节码进行操作，在类的执行过程中比较高效

