# Spring AOP 与注解

> 合并自：`Spring AOP.md`、`Spring中注解.md`。原文未删改（仅修正已知错误、调整格式）。

## 一、AOP 术语与原理

> 来源：`Spring/Spring AOP.md`

面向切面编程：在运行时，动态地将代码切入到类的指定方法、指定位置上的编程思想就是面向切面的编程。原理是使用代理模式，面向切面主要可以帮助我们减少重复代码、专注业务

AOP面向切面编程术语：

Aspect（切面）

aspect 由 pointcount 和 advice 组成, 它既包含了横切逻辑的定义, 也包括了连接点的定义. Spring AOP就是负责实施切面的框架, 它将切面所定义的横切逻辑织入到切面所指定的连接点中.AOP的工作重心在于如何将增强织入目标对象的连接点上, 这里包含两个工作:

- 如何通过 pointcut 和 advice 定位到特定的 joinpoint 上
- 如何在 advice 中编写切面代码.

可以简单地认为, 使用 @Aspect 注解的类就是切面.

advice（增强）

由 aspect 添加到特定的 join point(即满足 point cut 规则的 join point) 的一段代码.

许多 AOP框架, 包括 Spring AOP, 会将 advice 模拟为一个拦截器(interceptor), 并且在 join point 上维护多个 advice, 进行层层拦截.

例如 HTTP 鉴权的实现, 我们可以为每个使用 RequestMapping 标注的方法织入 advice, 当 HTTP 请求到来时, 首先进入到 advice 代码中, 在这里我们可以分析这个 HTTP 请求是否有相应的权限, 如果有, 则执行 Controller, 如果没有, 则抛出异常. 这里的 advice 就扮演着鉴权拦截器的角色了.

join point（连接点）

程序运行中的一些时间点, 例如一个方法的执行, 或者是一个异常的处理.

Spring AOP中，join point总是方法的执行点，即只有方法连接点

所有有可能被织入的候选点

point cut（切点）

Advice 是和特定的 point cut 关联的, 并且在 point cut 相匹配的 join point 中执行.

在 Spring 中, 所有的方法都可以认为是 joinpoint, 但是我们并不希望在所有的方法上都添加 Advice, 而 pointcut 的作用就是提供一组规则(使用 AspectJ pointcut expression language 来描述) 来匹配join point, 给满足规则的 joinpoint 添加 Advice.

满足条件的一组连接点

advice 是在 join point 上执行的, 而 point cut 规定了哪些 join point 可以执行哪些 advice

target（目标对象）

织入 advice 的目标对象. 目标对象也被称为 advised object.

因为 Spring AOP 使用运行时代理的方式来实现 aspect, 因此 adviced object 总是一个代理对象(proxied object)

注意, adviced object 指的不是原来的类, 而是织入 advice 后所产生的代理类.

AOP proxy

一个类被 AOP 织入 advice, 就会产生一个结果类, 它是融合了原类和增强逻辑的代理类.

在 Spring AOP 中, 一个 AOP 代理是一个 JDK 动态代理对象或 CGLIB 代理对象.

织入(Weaving)

将 aspect 和其他对象连接起来, 并创建 adviced object 的过程.

根据不同的实现技术, AOP织入有三种方式:

- 编译器织入, 这要求有特殊的Java编译器.
- 类装载期织入, 这需要有特殊的类装载器.
- 动态代理织入, 在运行期为目标类添加增强(Advice)生成子类的方式.

Spring 采用动态代理织入, 而AspectJ采用编译器织入和类装载期织入.

advice 的类型

- before advice, 在 join point 前被执行的 advice. 虽然 before advice 是在 join point 前被执行, 但是它并不能够阻止 join point 的执行, 除非发生了异常(即我们在before advice 代码中, 不能人为地决定是否继续执行 join point 中的代码)

- after return advice, 在一个 join point 正常返回后执行的 advice
- after throwing advice, 当一个 join point 抛出异常后执行的 advice

- after(final) advice, 无论一个 join point 是正常退出还是发生了异常, 都会被执行的 advice.
- around advice, 在 join point 前和 joint point 退出后都执行的 advice. 这个是最常用的 advice.

关于 AOP Proxy

Spring AOP 默认使用标准的 JDK 动态代理(dynamic proxy)技术来实现 AOP 代理, 通过它, 我们可以为任意的接口实现代理.

如果需要为一个类实现代理, 那么可以使用 CGLIB 代理. 当一个业务逻辑对象没有实现接口时, 那么Spring AOP 就默认使用 CGLIB 来作为 AOP 代理了. 即如果我们需要为一个方法织入 advice, 但是这个方法不是一个接口所提供的方法, 则此时 Spring AOP 会使用 CGLIB 来实现动态代理. 鉴于此, Spring AOP 建议基于接口编程, 对接口进行 AOP 而不是类.

## 二、常用注解

> 来源：`Spring/Spring中注解.md`

**配置类上添加@Configuration注解**

在Spring中如果一个类添加了@Configuration表明这个类的功能是类似于之前版本的xml bean配置文件的作用，但是如果我们不加这个注解，直接使用@Bean注解也可以将bean注入到Spring容器中。

通过下面的代码来观察

```java
public class A {
    private Object obj;

    public A(){
        System.out.println("A construct");
    }
}
```

```java
public class B {
    private A a;

    public B (A a) {
        System.out.println("B Construction");
    }
}
```

```java
@Configuration
public class Configuration1 {

    @Bean
    public A a(){
        return new A();
    }

    @Bean
    public B b(){
        A a = a();
        return new B(a);
    }
}
```

```java
public class Configuration2 {

    @Bean
    public A a(){
        return new A();
    }

    @Bean
    public B b(){
        A a = a();
        return new B(a);
    }
}
```

测试类

```java
public class Main {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(Configuration2.class);
    }
}
```

控制台打印结果

```java
A construct
B Construction

A construct
A construct
B Construction
```

在使用@Configuration注解之后是使用cglib代理增强Configuration类，在访问@Bean注解的方法时，会优先从beanFactory中获取Bean的实例，保证单例，如果不加的话如果出现依赖就会重新new对象无法保证单例

@ComponentScan

@ComponentScan用于批量注册bean，这个注解会让Spring去扫描某些包及其子包中所有的类，然后将满足一定条件的类作为bean注册到Spring容器中。

这个注解定义如下

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE})
@Documented
@Repeatable(ComponentScans.class)
public @interface ComponentScan {
    @AliasFor("basePackages")
    String[] value() default {};

    @AliasFor("value")
    String[] basePackages() default {};

    Class<?>[] basePackageClasses() default {};

    Class<? extends BeanNameGenerator> nameGenerator() default BeanNameGenerator.class;

    Class<? extends ScopeMetadataResolver> scopeResolver() default AnnotationScopeMetadataResolver.class;

    ScopedProxyMode scopedProxy() default ScopedProxyMode.DEFAULT;

    String resourcePattern() default "**/*.class";

    boolean useDefaultFilters() default true;

    ComponentScan.Filter[] includeFilters() default {};

    ComponentScan.Filter[] excludeFilters() default {};

    boolean lazyInit() default false;

    @Retention(RetentionPolicy.RUNTIME)
    @Target({})
    public @interface Filter {
        FilterType type() default FilterType.ANNOTATION;

        @AliasFor("classes")
        Class<?>[] value() default {};

        @AliasFor("value")
        Class<?>[] classes() default {};

        String[] pattern() default {};
    }
}
```

具体参数含义：

定义上可以看出此注解可以用在任何类型上面，不过我们通常将其用在类上面。

常用参数：

value：指定需要扫描的包，如：com.javacode2018

basePackages：作用同value；value和basePackages不能同时存在设置，可二选一

basePackageClasses：指定一些类，spring容器会扫描这些类所在的包及其子包中的类

nameGenerator：自定义bean名称生成器

resourcePattern：需要扫描包中的那些资源，默认是：`**/*.class`，即会扫描指定包中所有的class文件

useDefaultFilters：对扫描的类是否启用默认过滤器，默认为true

includeFilters：过滤器：用来配置被扫描出来的那些类会被作为组件注册到容器中

excludeFilters：过滤器，和includeFilters作用刚好相反，用来对扫描的类进行排除的，被排除的类不会被注册到容器中

lazyInit：是否延迟初始化被注册的bean

@Repeatable(ComponentScans.class)，这个注解可以同时使用多个。

@ComponentScan工作流程

1. Spring会扫描指定的包，且会递归下面子包，得到一批类的数组
2. 然后这些类会经过上面的过滤器，最后剩下的类会被注册到容器中

默认情况下，任何参数都不设置的情况下，此时会将@ComponentScan修饰的类所在的包作为扫描器；默认情况下useDefaultFilter为true，这个为true的时候，Spring容器内部会使用默认过滤器，规则是：凡是类上有@Component、@Controller、@Service、@Repository这几个注解中的任何一个，那么这个类就会被作为bean注册到Spring容器中，所以默认情况下，只需在类上加上这几个注解中的任何一个，这些类就会自动交给Spring容器来管理

@Component、@Controller、@Service、@Repository这几个注解对于Spring容器没有区分，被扫描的时候会被作为bean注册到容器中

@Autowired 字段注入流程

![图片](../../assets/Spring%20AOP与注解/img-01.webp)

@Autowired在方法上注入流程

![图片](../../assets/Spring%20AOP与注解/img-02.webp)

@Autowored 先按照类型找，再按名称找

当@Autowired在方法上的时候，会通过容器将方法所有的参数全部注入，并且执行该方法

@Resource标注在字段上查找流程如下

![图片](../../assets/Spring%20AOP与注解/img-03.png)

@Resource标注在方法或者方法参数上流程

![图片](../../assets/Spring%20AOP与注解/img-04.webp)

注意：当@Resource标注在方法上时，这个方法只能有一个参数，否则容器启动会报错

```sql
nested exception is java.lang.IllegalStateException: @Resource annotation requires a single-arg method:
```

@Qulifier主要是起到限定的作用，可以在依赖注入查找候选者的过程中对候选者进行过滤，当这个注解在类上时，相当于给类打上tag，一般和@Autowired一同使用来进行名称的限制

@Primary的功能是设置主要候选者，可以用在类上后者方法上，通常定义bean常见的两种方式中：1.在类上标注@Component注解，此时可以配合@Primary，标注这个为主要候选者。2.在配置文件中使用@Bean注解标注方法来注册备案，可以在@Bean标注的方法上加上@Primary，标注这个bean为主要候选bean
