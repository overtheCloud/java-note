#### Lombok 是什么
Lombok 可以通过注解在编译阶段为 `bean` 生成构造器、getter/setter、`toString()`、`hashCode()`、`equals()` 等方法。
代码中没有这些方法的代码，但是生成的字节码文件中有。

#### Lombok 的原理
JSR 269自JDK6加入，javac在执行的时候会调用实现了该API的程序。
Lombok本质上就是一个实现了“JSR 269 API”的程序。在使用javac的过程中，它产生作用的具体流程如下：
- javac对源代码进行分析，生成了一棵抽象语法树（AST）
- 运行过程中调用实现了“JSR 269 API”的Lombok程序
- 此时Lombok就对第一步骤得到的AST进行处理，找到@Data注解所在类对应的语法树（AST），然后修改该语法树（AST），增加getter和setter方法定义的相应树节点
- javac使用修改后的抽象语法树（AST）生成字节码文件，即给class增加新的节点（代码块）

#### Lombok 如何使用
1. IDEA 添加插件 `Lombok`
2. 应用程序的 pom.xml 中添加 `Lombok` 依赖
```xml
<dependency>
	<groupId>org.projectlombok</groupId>
	<artifactId>lombok</artifactId>
</dependency>
```
3. 在需要的类上添加相应注解，如 @Data

### 注解详解
- **@Data** 生成构造器、getter/setter、`toString()`、`hashCode()`、`equals()` 等方法
- **@Getter** 生成 getter 方法
- **@Setter** 生成 setter 方法
- **@EqualsAndHashCode** 生成 `hashCode()`、`equals()` 方法
- **@ToString** 生成 `toString()` 方法
- **@NoArgsConstructor** 生成无参构造器
- **@RequiredArgsConstructor** 生成部分参数构造器
- **@AllArgsConstructor** 生成全参构造器
- **@Builder** 生成构造器
- **@SL4J** 生成日志实例对象
- **@Accessors** 指定 setter/getter 方法生成方式
- @Cleanup 生达自动释放资源的代码
- @NonNull 为方法或构造函数的参数提供非空检查
- @SneakyThrows 生成 catch 异常的代码
- @Synchronized 生成 synchronized 代码块
- val 将变量申明是final类型

#### @Data 
生成构造器、getter/setter、`toString()`、`hashCode()`、`equals()` 等方法  
**源代码**
```java
import lombok.Data;

@Data
public class LombokBean {
    private int id;
    private String name;
}
```
**编译后代码**
```java
public class LombokBean {
    private int id;
    private String name;

    public LombokBean() {
    }

    public int getId() {
        return this.id;
    }

    public String getName() {
        return this.name;
    }

    public void setId(final int id) {
        this.id = id;
    }

    public void setName(final String name) {
        this.name = name;
    }

    public boolean equals(final Object o) {
        if (o == this) {
            return true;
        } else if (!(o instanceof LombokBean)) {
            return false;
        } else {
            LombokBean other = (LombokBean)o;
            if (!other.canEqual(this)) {
                return false;
            } else if (this.getId() != other.getId()) {
                return false;
            } else {
                Object this$name = this.getName();
                Object other$name = other.getName();
                if (this$name == null) {
                    if (other$name != null) {
                        return false;
                    }
                } else if (!this$name.equals(other$name)) {
                    return false;
                }

                return true;
            }
        }
    }

    protected boolean canEqual(final Object other) {
        return other instanceof LombokBean;
    }

    public int hashCode() {
        int PRIME = true;
        int result = 1;
        int result = result * 59 + this.getId();
        Object $name = this.getName();
        result = result * 59 + ($name == null ? 43 : $name.hashCode());
        return result;
    }

    public String toString() {
        return "LombokBean(id=" + this.getId() + ", name=" + this.getName() + ")";
    }
}
```

#### @Getter/@Setter/@EqualsAndHashCode/@ToString 
生成的代码都包含在 @Data 中，此处就省略了  

#### @NoArgsConstructor 
生成无参构造器，有 4 个参数，这些参数除了 force 是 @NoArgsConstructor独有，其他 3 个参数和 @RequiredArgsConstructor 、 @AllArgsConstructor 一样  
**参数**：
- staticName 生成一个静态方法，构造器修饰符变成 private
- onConstructor 指定注解
- access 指定修饰符，值范围 AccessLevel
- force 为所有属性赋予初始化值 `0 / null / false`，否则报错，当属性被 `final` 修饰时使用可以避免编译报错  

**源代码**
```java
@NoArgsConstructor(staticName = "instance", access = AccessLevel.PUBLIC, force = true)
public class LombokBean {}
```
**编译后代码**
```java
public class LombokBean {
    private LombokBean() { // 此处修饰符是 private 而不是 public 是由于指定了 staticName，说明 staticName 优先于 access
    }

    public static LombokBean instance() {
        return new LombokBean();
    }
}
```

#### @RequiredArgsConstructor
生成无参或有参的构造函数。默认无参，如果有属性被 `@NonNull` 修饰或者被 `final` 修饰且未初始化则生成入参为这几个属性的有参构造器  
**源代码**
```java
@RequiredArgsConstructor()
public class LombokBean {
    @NonNull
    private int i;
    private final String firstName;
    private final String lastName = "hhh";
    private int age;
}
```
**编译后代码**
```java
public class LombokBean {
    @NonNull
    private int i; // 被 @NonNull 修饰的属性会加入到有参构造器的参数列表中
    private final String firstName; // 被 final 修饰且未初始化的属性会加入到有参构造器的参数列表中
    private final String lastName = "hhh"; // 属性被初始化后不会加入到有参构造器的参数列表中
    private int age; 

    public LombokBean(@NonNull final int i, final String firstName) {
        this.i = i;
        this.firstName = firstName;
    }
}
```
#### @AllArgsConstructor 
生成全参数的构造函数，不包括被 final 修饰且已经初始化的属性，因为被 final 修饰的属性不能被修改
**源代码**

```Java
@AllArgsConstructor
public class LombokBean {
        private int i;
        private final String firstName = "hhh";
        private final String lastName;
}
```
**编译后代码**
```Java
public class LombokBean {
    private int i;
    private final String firstName = "hhh"; // 被初始化的 final 修饰的属性不在构造器参数列表中
    private final String lastName;

    public LombokBean(final int i, final String lastName) {
        this.i = i;
        this.lastName = lastName;
    }
}
```
> 注意的是，同时使用@Data 和 @AllArgsConstructor 后 ，默认的无参构造函数失效，如果需要它，要重新设置 @NoArgsConstructor

#### @Builder

生成方便创建被修饰的类（以目标类代称）的实例的创建类（以创建类代称），这是建造者模式的一种实现，创建类中有目标类的除 `final` 修饰的全部属性。可以用于类、接口、方法、构造函数。

**参数**：

- builderMethodName  目标类中获取创建类实例的方法名称
- buildMethodName 创建类中创建目标类实例的方法名称
- builderClassName 创建类名称
- toBuilder  目标类中生成 `toBUilder()` 方法，可以创建对象的副本或者相近副本。
- access  指定创建类和目标类的 `toBuilder` 、 `builder()` 等方法修饰符，值参考 `lombok.AccessLevel`
- @Builder.Default 使用创建类生成实例时如果不设置属性的值，则实例中对应属性值是默认值而不是定义的值，此注解指定了未设置属性时创建的实例的属性值是定义的值。
- @Builder.ObtainVia 指定属性值从其他哪个属性或者方法。分别通过 `field`绑定字段 、`method`绑定方法、`isStaic` 指定方法是否静态方法，配合`method`使用。

**源代码**

```java
import lombok.*;

@Builder(buildMethodName = "create", builderMethodName = "creater", builderClassName = "child",
        toBuilder = true, access = AccessLevel.PROTECTED)
@ToString
public class LombokDemo {
    private final int id = 1;
    // 使默认值生效
    @Builder.Default 
    private String name = "li";
    // age = id
    @Builder.ObtainVia(field = "id") 
    private int age;
    // sex = getSex()
    // isStatic = true 编译时会报错
    @Builder.ObtainVia(method = "getSex", isStatic = true)
    private String sex;

    public static String getSex() {
        return "男";
    }

    public static void main(String[] args) {
        LombokDemo zhang = LombokDemo.creater().name("zhang").create();
        System.out.println(zhang);
        LombokDemo wang = zhang.toBuilder().name("wang").create();
        System.out.println(wang);
        LombokDemo li = LombokDemo.creater().create();
        System.out.println(li);
    }
}
```

**执行结果**

```java
LombokDemo(id=1, name=zhang, age=0, sex=null)
LombokDemo(id=1, name=wang, age=1, sex=男) // 只有 toBuilder 会为未设置的属性赋值
LombokDemo(id=1, name=li, age=0, sex=null)
```

**编译后代码**

```java
public class LombokDemo {
    private final int id = 1;
    private String name;
    private int age;
    private String sex;

    public static String getSex() {
        return "男";
    }

    public static void main(String[] args) {
        LombokDemo zhang = creater().name("zhang").create();
        System.out.println(zhang);
        LombokDemo wang = zhang.toBuilder().name("wang").create();
        System.out.println(wang);
        LombokDemo li = creater().create();
        System.out.println(li);
    }

    private static String $default$name() {
        return "li";
    }

    LombokDemo(final String name, final int age, final String sex) {
        this.name = name;
        this.age = age;
        this.sex = sex;
    }

    protected static LombokDemo.child creater() {
        return new LombokDemo.child();
    }

    // @Builder.ObtainVia 实际上只在 toBuilder 中生效了
    protected LombokDemo.child toBuilder() {
        LombokDemo.child var10000 = (new LombokDemo.child()).name(this.name);
        this.getClass();
        return var10000.age(1).sex(getSex());
    }

    public String toString() {
        StringBuilder var10000 = (new StringBuilder()).append("LombokDemo(id=");
        this.getClass();
        return var10000.append(1).append(", name=").append(this.name).append(", age=").append(this.age).append(", sex=").append(this.sex).append(")").toString();
    }

    protected static class child {
        // true 表示设置了 name 的值，false 表示使用默认值
        private boolean name$set;
        private String name;
        private int age;
        private String sex;

        child() {
        }

        public LombokDemo.child name(final String name) {
            this.name = name;
            this.name$set = true;
            return this;
        }

        public LombokDemo.child age(final int age) {
            this.age = age;
            return this;
        }

        public LombokDemo.child sex(final String sex) {
            this.sex = sex;
            return this;
        }

        public LombokDemo create() {
            String name = this.name;
            if (!this.name$set) {
                name = LombokDemo.$default$name();
            }

            return new LombokDemo(name, this.age, this.sex);
        }

        public String toString() {
            return "LombokDemo.child(name=" + this.name + ", age=" + this.age + ", sex=" + this.sex + ")";
        }
    }
}
```

#### @Slf4j

生成 Slf4j 对象

```java
private static final Logger log = LoggerFactory.getLogger(LombokDemo.class);
```

> 类似的还有 `@CommonsLog`、 `@LOG`、 `@JBossLog`、 `@Log4j`、`@Log4j2`、 `@Slf4j2`

#### @Value

使类不可变

**源代码**

```java
@Value
public class LombokDemo {}
```

**编译后代码**

```java
public final class LombokDemo {
    public LombokDemo() {
    }

    public boolean equals(final Object o) {...}

    public int hashCode() {...}

    public String toString() {...}
}
```

#### @Accessors

指定 setter/getter 生成方式，配合 @Data、@Getter、@Setter使用

***参数***

- fluent `true`表示使用属性名称作为 setter/getter 方法名
- chain `true`表示使用链式调用，方法返回的是对象实例
- prefix 指定忽略的属性名称的前缀

**源代码**

```java
@Data
@Accessors(chain = true, fluent = true, prefix = "p")
public class LombokDemo {
    private int id;
    private String pName;
}
```

**编译后代码**

```java
public class LombokDemo {
    private int id;
    private String pName;
    
     public LombokDemo() {
    }
	// 方法名中不包含 'p'
    public LombokDemo name(final String pName) {
        this.pName = pName;
        return this;
    }
    
    public String name() {
        return this.pName;
    }
    // 省略其他方法
}
```

#### @Synchronized 

生成 synchronized 代码块

源代码
```java
import lombok.Synchronized;

public class LombokBean {
    @Synchronized
    public synchronized void init() {
        System.out.println(1);
        System.out.println(2);
    }
}
```
编译后代码
```java
public class LombokBean {
    private final Object $lock = new Object[0];// new Object() 不会序列化，空数组会序列化

    public LombokBean() {
    }

    public void init() {
        synchronized(this.$lock) {
            System.out.println(1);
            System.out.println(2);
        }
    }
}
```
#### val 将变量申明是final类型
源代码
```java
import lombok.val;

public class LombokBean {
    public void method() {
        val a = new HashSet<>();
        System.out.println(a); // 如果不使用变量，编译器编译时会删除变量引用 a
    }
}
```
编译后代码
```java
public class LombokBean {
public class LombokBean {
    public LombokBean() {
    }

    public void method() {
        HashSet<Object> a = new HashSet();
        System.out.println(a);
    }
}
}

```