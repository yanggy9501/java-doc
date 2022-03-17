# Spring启动过程

spring的启动过程，通常就是指创建ApplicationContext对象以及调用refresh（）方法的过程。

## 1，前言

### 1.1 spring的启动过程

>   1.   构造一个BeanFactory对象（在父类进行创建DefualtListableBeanFactory）
>   2.   生成默认的BeanPostProcessor对象，并添加到BeanFactory中（在属性处理之后，执行Aware）
>        1.   AutowiredAnnotationBeanPostProcessor：处理@Autowired、@Value
>        2.   CommonAnnotationBeanPostProcessor：处理@Resource、@PostConstruct，@PreDestroy
>        3.    ApplicationContextAwareProcessor：处理ApplicationContextAware等回调
>   3.   `解析配置类`（在refresh（）方法中，invokexxx方法中完成扫描），得到BeanDefinition，并注册到BeanFactory中
>        1.   解析@ComponentScan，得到扫描路径
>        2.   <font color="red">解析@Import注解（@Import可以注册一个bean对象）</font>
>        3.   <font color="red">解析@Bean</font>
>        4.   ......
>   4.   初始化MessageSource对象（`国际化`）
>   5.   初始化ApplicationEventMulticaster对象，ApplicationContext的`事件发布`
>   6.   `事件监听`，把用户定义的ApplicationListener对象添加到ApplicationContext中
>   7.   `创建非懒加载的单例Bean对象`，并存在BeanFactory的单例池中
>   8.    调用Lifecycle Bean的start()方法
>   9.    发布ContextRefreshedEvent事件



### 1.2 BeanFactoryPostProcessor

>   BeanPostProcessor（初始化前执行）表示Bean的后置处理器，是用来对Bean进行加工的，类似的，BeanFactoryPostProcessor理解为BeanFactory的后置处理器，用来用对**BeanFactory**进行加工的。
>
>   Spring支持用户定义BeanFactoryPostProcessor的实现类Bean，来对BeanFactory进行加工。
>
>   **在spring启动时，创建单例bean之前执行**

```java
@Component
public class MyBeanFactoryPostProcessor implements BeanFactoryPostProcessor {
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory)
        throws BeansException {
        BeanDefinition beanDefinition = beanFactory.getBeanDefinition("userService");
        beanDefinition.setAutowireCandidate(false);
    }
}
```

**注意：**DefaultListableBeanFactory 实现ConfigurableListableBeanFactory和BeanDefinitionRegistry接口，所以以ApplicationContext和 DefaultListableBeanFactory是可以注册BeanDefinition的，但是ConfigurableListableBeanFactory不可以注册，只能获取。

### 1.3 BeanDefinitionRegistryPostProcessor

>   由于BeanFactoryPostProcessor不能注册BeanDefinition，spring提供了另外一个接口BeanDefinitionRegistryPostProcessor，其extends BeanFactoryPostProcessor接口，但是可以注册bean定义的功能。

```java
@Component
public class MyBeanDefinitionRegistryPostProcessor implements
    BeanDefinitionRegistryPostProcessor {
    @Override
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws
        BeansException {
        AbstractBeanDefinition beanDefinition =
            BeanDefinitionBuilder.genericBeanDefinition().getBeanDefinition();
        beanDefinition.setBeanClass(User.class);
        registry.registerBeanDefinition("user", beanDefinition);
    }
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory)
        throws BeansException {
        BeanDefinition beanDefinition = beanFactory.getBeanDefinition("userService");
        beanDefinition.setAutowireCandidate(false);
    }
}
```



## 2, refresh()底层流程

**refresh（）理解**

>   加载或刷新持久化的配置，可能是XML文件、属性文件或关系数据库中存储的。
>
>   由于这是一个启动方法，如果失败，它应该销毁已经创建的单例，以避免暂用资源（应该是程序可能没停止）。
>
>   换句话说，在调用该方法之后，应该实例化所有的单例， 或者根本不实例化单例 。
>
>   **有个理念需要注意：ApplicationContext关闭之后不代表JVM也关闭了，ApplicationContext是 属于JVM的，说白了ApplicationContext也是JVM中的一个对象。说白了就是ApplicationContext关闭不代表bean被回收，如果还被外界引用呢。**
>
>   以刷新的ApplicationContext和不可以刷新的ApplicationContext。
>
>   *   AbstractRefreshableApplicationContext 可刷新的
>   *   GenericApplicationContext 不可刷新的
>   *   AnnotationConfigApplicationContext继承的是GenericApplicationContext
>   *   AnnotationConfigWebApplicationContext继承的是AbstractRefreshableWebApplicationContext
>
>   不能刷新是指不能重复执行refresh（）方法。

Spring启动流程详解 https://www.processon.com/view/link/5f60a7d71e08531edf26a919#pc#pc



### 2.1 底层流程

以AnnotationConfigApplicationContext为例：

>   1.   调用`AnnotationConfigApplicationContext`的构造方法之前，会调用父类 `GenericApplicationContext`的无参构造方法，会构造一个BeanFactory，为`DefaultListableBeanFactory`。
>   2.   构造`AnnotatedBeanDefinitionReader`（主要作用添加一些基础的PostProcessor（包括BeanFactory的），同时创建reader，其可以对BeanDefinition的注册）
>        1.   设置dependencyComparator：AnnotationAwareOrderComparator它是一个 Comparator, 会获取某个对象上的Order注解，用来排序。
>        2.    设置`autowireCandidateResolver`: ContextAnnotationAutowireCandidateResolver， 用来解析某个Bean能不能进行自动注入。个Bean的autowireCandidate属性是否等于true。
>        3.   向BeanFactory中添加`ConfigurationClassPostProcessor`对应的BeanDefinition，它是一个`BeanDefinitionRegistryPostProcessor`，并且实现了PriorityOrdered接口。
>        4.   向BeanFactory中添加`AutowiredAnnotationBeanPostProcessor`(处理@Autowared，@Value等)对应的BeanDefinition，它是一个InstantiationAwareBeanPostProcessorAdapter， MergedBeanDefinitionPostProcessor。
>        5.   向BeanFactory中添加`CommonAnnotationBeanPostProcessor`（处理@Resource, @PostConstruct）对应的BeanDefinition，它是一个InstantiationAwareBeanPostProcessor， InitDestroyAnnotationBeanPostProcessor。
>        6.    向BeanFactory中添加EventListenerMethodProcessor对应的BeanDefinition，它是一个 BeanFactoryPostProcessor，SmartInitializingSingleton.
>        7.   向BeanFactory中添加DefaultEventListenerFactory对应的BeanDefinition，它是一个 EventListenerFactory
>   3.   构造ClassPathBeanDefinitionScanner（主要作用可以用来扫描得到并注册 BeanDefinition）
>        1.   设置this.includeFilters = AnnotationTypeFilter(Component.class
>        2.   设置environment
>        3.   设置resourceLoade
>   4.   利用reader注册配置类AppConfig为BeanDefinition，类型为AnnotatedGenericBeanDefinitio。
>   5.   `接下来就是调用refresh方法`
>   6.   prepareRefresh()
>   7.   obtainFreshBeanFactory()：**进行BeanFactory的refresh**，在这里会去调用子类的 refreshBeanFactory方法，具体子类是怎么刷新的得看子类，然后再调用子类的 getBeanFactory方法，重新得到一个BeanFactory。
>   8.   prepareBeanFactory(beanFactory)
>        1.   设置beanFactory的类加载器
>        2.   设置表达式解析器：StandardBeanExpressionResolver（el表达式）
>        3.   添加PropertyEditorRegistrar：ResourceEditorRegistrar，PropertyEditor类型转化器注册器，注册一些默认的PropertyEditor。
>        4.   添加一个Bean的后置处理器：ApplicationContextAwareProcessor
>        5.   添加ignoredDependencyInterface
>        6.   添加resolvableDependencies：在byType进行依赖注入时，会先从这个属性中根据类型 找bean
>        7.   添加一个Bean的后置处理器：ApplicationListenerDetector用来判断某个Bean是不是ApplicationListener
>        8.   ...
>   9.   postProcessBeanFactory(beanFactory) 
>   10.   invokeBeanFactoryPostProcessors(beanFactory)：执行BeanFactoryPostProcessor
>         1.   此时在BeanFactory中会存在一个BeanFactoryPostProcessor：ConfigurationClassPostProcessor，它也是一个BeanDefinitionRegistryPostProcesso
>         2.   从BeanFactory中找到类型为BeanDefinitionRegistryPostProcessor的beanName，也就 是ConfigurationClassPostProcessor， 然后调用BeanFactory的getBean方法得到实例对象
>         3.   执行**ConfigurationClassPostProcessor的postProcessBeanDefinitionRegistry()**方法：
>              1.   解析AppConfig类
>              2.   扫描得到BeanDefinition并注册
>              3.   解析@Import，@Bean等注解得到BeanDefinition并注册
>              4.   执行完ConfigurationClassPostProcessor的postProcessBeanDefinitionRegistry()方 法后，还需要继续执行其他BeanDefinitionRegistryPostProcessor的 postProcessBeanDefinitionRegistry()方法
>              5.   执行其他BeanDefinitionRegistryPostProcessor的 **postProcessBeanDefinitionRegistry()**方法
>              6.   从BeanFactory中找到类型为BeanFactoryPostProcessor的beanName，而这些 BeanFactoryPostProcessor包括了上面的BeanDefinitionRegistryPostProcessor
>              7.   . 执行还没有执行过的BeanFactoryPostProcessor的**postProcessBeanFactory()**方法
>   11.   到此，所有的BeanFactoryPostProcessor的逻辑都执行完了，主要做的事情就是得到 BeanDefinition并注册到BeanFactory中
>   12.   registerBeanPostProcessors(beanFactory)：因为上面的步骤完成了扫描，这个过程中程序员 可能自己定义了一些BeanPostProcessor，在这一步就会把BeanFactory中所有的 BeanPostProcessor找出来并实例化得到一个对象，并添加到BeanFactory中去（属性 beanPostProcessors）
>   13.    initMessageSource()
>   14.   initApplicationEventMulticaster()
>   15.   onRefresh()：没用
>   16.   egisterListeners()：从BeanFactory中获取ApplicationListener类型的beanName，然后添加 到ApplicationContext中的事件广播器applicationEventMulticaster中去。
>   17.   finishBeanFactoryInitialization(beanFactory)：完成BeanFactory的初始化，主要就是实例化 非懒加载的单例Bean。
>   18.   finishRefresh()：BeanFactory的初始化完
>   19.   ....

### 2.2 执行BeanFactoryPostProcessor



1.   执行通过ApplicationContext添加进来的BeanDefinitionRegistryPostProcessor的 postProcessBeanDefinitionRegistry()方法 
2.   执行BeanFactory中实现了PriorityOrdered接口的BeanDefinitionRegistryPostProcessor的 postProcessBeanDefinitionRegistry()方法 
3.   执行BeanFactory中实现了Ordered接口的BeanDefinitionRegistryPostProcessor的 postProcessBeanDefinitionRegistry()方法 
4.   执行BeanFactory中其他的BeanDefinitionRegistryPostProcessor的 postProcessBeanDefinitionRegistry()方法
5.   执行上面所有的BeanDefinitionRegistryPostProcessor的postProcessBeanFactory()方法
6.   执行通过ApplicationContext添加进来的BeanFactoryPostProcessor的 postProcessBeanFactory()方法
7.   执行BeanFactory中实现了PriorityOrdered接口的BeanFactoryPostProcessor的 postProcessBeanFactory()方法
8.   执行BeanFactory中实现了Ordered接口的BeanFactoryPostProcessor的 postProcessBeanFactory()方法
9.   执行BeanFactory中其他的BeanFactoryPostProcessor的postProcessBeanFactory()方法

