# Spring-02Bean的生成过程|生命周期

## 1，生成BeanDefinition

>   spring 启动的时候会进行扫描配置类@ComponentScan(value = "a.b")获取类的文件路径，并生成到BeanDefinition的Set集合。
>
>   **扫描方法：**可以在这个方法断点调试，查看调用堆栈从而得知入口
>
>   ```text
>   org.springframework.context.annotation.ClassPathBeanDefinitionScanner#doScan
>   ```

### 1.1 spring启动到扫描开始

>   AnnotationConfigApplicationContext调用this()先创建BeanDefinitionReader和BeanDefinitionScanner工具类，super()构造方法则创建DefaultListableBeanFactory容器。↓
>
>   register(class)方法利用this()创建的读取器进行注册配置类，生成配置类的BeanDefinition，以便扫描配置类的@ComponentScan做准备。↓
>
>   refresh() -> invokeBeanFactoryPostProcessors(beanFactory)在这里完成扫描。↓
>
>   通过执行<font color="red">beanFactoryPostProcessor</font>方法完成扫描。 -> PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors ↓
>
>    invokeBeanDefinitionRegistryPostProcessors() -> ..... -> ↓
>
>   org.springframework.context.annotation.ConfigurationClassPostProcessor#processConfigBeanDefinitions这里parse（）解析配置类。
>
>   .... doScan()



### 1.2 扫描底层流程

1.   已经在上一步通过配置类获取包路径即@ComponentScan的属性值，扫描时spring通过工具类获取指定包路径下的所有 .class 文件路径packageSearchPath，通过资源加载器ResourcePatternResolver获取资源并封装成了Resource对象。

     ```java
     Resource[] resources = getResourcePatternResolver().getResources(packageSearchPath);
     ```

2.   利用MetadataReaderFactory解析Resource对象得到MetadataReader元数据读取器。利用MetadataReader进行excludeFilters和includeFilters，以及条件注解@Conditional的筛选。

     ```java
     MetadataReader metadataReader = getMetadataReaderFactory().getMetadataReader(resource);
     ```

3.   通过metadataReader生成BeanDefinition(metadataReader能够读取一个类，获取类元数据)

     ```java
     ScannedGenericBeanDefinition sbd = new ScannedGenericBeanDefinition(metadataReader);
     ```

4.   再基于metadataReader判断是不是对应的类，是不是接口或抽象类(是否是一个bean)，是，则加入结果集。

#### 1.2.1 合并BeanDefinition

>   实际用的比较少

#### 1.2.2 加载类

>   BeanDefinition合并之后，就可以去创建Bean对象了，创建对象需要先加载当前BeanDefinition所对应的class文件，得到Class字节码对象。
>
>   ClassUtils.getDefaultClassLoader()获取类加载器

```java
Class<?> resolvedClass = resolveBeanClass(mbd, beanName);
```



>   1.   实例化前-InstantiationAwareBeanPostProcessor
>   2.   创建bean对象：利用该类的构造方法来实例化得到一个对象(推断构造方法)
>   3.   依赖注入：Spring会判断该对象中是否存在被@Autowired注解了的属性，有进行1步骤然后注入
>   4.   Bean增强：Spring会判断该对象是否实现了Aware接口（类.class.isAssignableFrom(clazz) 或者 instanceof判断），那Spring就会调用这些方法并传入相应的参数，我们就可以得到某些参数了。
>        1.   BeanNameAware接口（可以获取beanName）
>        2.   BeanClassLoaderAware接口（可以获取类加载器）
>        3.   BeanFactoryAware接口（可以获取BeanFactory）
>        4.   ApplicationContextAware接口（获取ApplicationContext）
>   5.   初始化前-BeanPostProcessor接口或@PostConstruct注解： Aware回调后，Spring就将调用接口的postProcessBeforeInitialization()方法；Spring会判断该对象中是否存在某个方法被@PostConstruct注解 了，如果存在，Spring会调用当前对象的此方法。
>   6.   初始化-实现InitializingBean接口或者@InitMethod注解：Spring会判断该对象是否实现了InitializingBean接口，springg就会调用当前对象中的afterPropertiesSet()完成初始化；spring也会判断是否有该注解。
>   7.   初始化后-aop：Spring会判断当前对象需不需要进行AOP，否Bean就创建完了，是则会进行动态代理并生成一个代理对象做为Bean

## 2，实例化前

>   当前BeanDefinition对应的类成功加载后，就可以实例化对象。实例化前的操作可以干涉bean的生成过程。
>
>   *   扩展点-InstantiationAwareBeanPostProcessor：InstantiationAwareBeanPostProcessor.postProcessBeforeInstantiation()实例化前操作。该接口继承接口BeanPostProcessor。

```java
@Component
public class MyBeanPostProcessor implements InstantiationAwareBeanPostProcessor {
    @Override
    public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
        if ("userService".equals(beanName)) {
        	System.out.println("实例化前");
        	return new UserService();
        }
        // 该bean不会被创建，spring不实例化对象
        return null;
    }
}
```



## 3，实例化

>   在这个步骤中就会根据BeanDefinition去创建一个对象了。根据字节码反射创建一个对象，或获取创建对象的方法创建对象。

### 3.1 Supplier创建对象

>   BeanDefinition中是否设置了Supplier，如果设置了则调用Supplier的get()得到对象。

```java
AbstractBeanDefinition beanDefinition = BeanDefinitionBuilder.genericBeanDefinition().getBeanDefinition();

beanDefinition.setInstanceSupplier(new Supplier<Object>() {
    @Override
    public Object get() {
    return new UserService();
    }
});

context.registerBeanDefinition("userService", beanDefinition);
```

### 3.2 工厂方法创建对象

>   @Bean
>
>   检查BeanDefinition中是否设置了factoryMethod，也就是工厂方法。
>
>   `xml方式1：`
>
>   ```xml
>   <bean id="userService" class="com.xxx.service.UserService" factory-method="createUserService" />
>   ```
>
>   `xml方式2：`
>
>   ```xml
>   <bean id="commonService" class="com.xxx.service.CommonService"/>
>   <bean id="userService1" factory‐bean="commonService" factory-method="createUserService"/>
>   ```
>
>   
>
>   `@Bean`所定义的BeanDefinition，是存在factoryMethod和factoryBean的，也就是和上面的方式二非常类似，@Bean所注解的方法就是factoryMethod，AppConfig对象就是factoryBean。如果@Bean所所注解的方法是static的，那么对应的就是方式一。



### 3.3 推断构造方法-反射创建对象

>   `反射创建对象`推断构造方法逻辑会去选择构造方法以及查找入参对象意外，会还判断类中是否存在使用**@Lookup注解**的方法。
>
>   ​	是：存在则把该方法封装为LookupOverride对象添加到BeanDefinition中，生成代理对象。
>
>   ​	否：直接用构造方法反射得到一个实例对象。



## 4， BeanDefinition的后置处理

>   （BeanDefinitionPostProcesso）经过实例化过程创建对象之后，接下来就应该给对象的属性赋值了。在属性赋值之前，Spring又提供了一个扩展点-`BeanDefinition的后置处理`
>
>   主要是对BeanDefinition的干涉-bean对象已经创建出来了，其只能对BeanDefinition进行修改，影响属性注入，或者对bean的属性，注解等进行干涉。
>
>   AutowiredAnnotationBeanPostProcessor就是一个MergedBeanDefinitionPostProcessor，它在postProcessMergedBeanDefinition()中会去查找注入点并保存到Map中。

```java
@Component
    public class MyMergedBeanDefinitionPostProcessor implements MergedBeanDefinitionPostProcessor {
    @Override
    public void postProcessMergedBeanDefinition(RootBeanDefinition beanDefinition, Class<?> beanType, String beanName) {
        if ("userService".equals(beanName)) {
       		beanDefinition.getPropertyValues().add("orderService", new OrderService());
        }
    }
}
```



## 5，实例化后

>   在处理完BeanDefinition后置处理后，这个到了实例化后阶段，这个时候bean对象已经创建，但是属性还未注入。Spring又设计了一个实例化后扩展点，可以用来决定该bean是否要交给spring管理。
>
>   **InstantiationAwareBeanPostProcessor** 同实例化前接口。

```java
@Component
public class ZhouyuInstantiationAwareBeanPostProcessor implements InstantiationAwareBeanPostProcessor {
    @Override
    public boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
        if ("userService".equals(beanName)) {
            UserService userService = (UserService) bean;
            userService.test();
        }
        return true;
	}
}
```



## 6，属性注入

>   <font color="red">依赖注入</font>



## 7，属性处理

>   这个步骤中，可以自己处理@Autowired、@Resource、@Value等注解，或者属性注入。也是通过**InstantiationAwareBeanPostProcessor.postProcessProperties()**扩展点来实现的。比如：

```java
@Component
public class ZhouyuInstantiationAwareBeanPostProcessor implements InstantiationAwareBeanPostProcessor {
    @Override
    public PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName) throws BeansException {
        if ("userService".equals(beanName)) {
            for (Field field : bean.getClass().getFields()) {
                if (field.isAnnotationPresent(ZhouyuInject.class)) {
                    field.setAccessible(true);
                    try {
                    	field.set(bean, "123");
                    } catch (IllegalAccessException e) {
                    	e.printStackTrace();
                    }
                }
            }
        }
        return pvs;
    }
}

```



## 8, 执行Aware

>   1.   BeanNameAware接口（可以获取beanName）：回传beanName给bean对象
>   2.   BeanClassLoaderAware接口（可以获取类加载器）：：回传classLoader给bean对象。
>   3.   BeanFactoryAware接口（可以获取BeanFactory）：回传beanFactory给对象。
>   4.   ApplicationContextAware接口（获取ApplicationContext）：回传ApplicationContext给对象。
>
>   **注意：**需要交给spring

```java
@Component
public class SpringApplicationContext implements ApplicationContextAware {
    /**
     * Spring应用上下文环境
     */
    public static ApplicationContext beanFactory;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        beanFactory = applicationContext;
    }
}
```



## 9，初始化前

>   到这里，基本上bean对象已经完成了。利用初始化前，可以对进行了依赖注入的Bean进行处理。
>
>   1.   @PostConstruct：执行某个方法，做一些初始化操作，一般操作与属性赋值无关。
>   2.   BeanPostProcessor：



## 10，初始化

>   1.   查看当前Bean对象是否实现了InitializingBean接口，如果实现了就调用其afterPropertiesSet() 方法
>   2.   @InitMethod：
>   3.   执行BeanDefinition中指定的初始化方法。



## 11，初始化后

>   这个步骤中，对Bean最终进行处理，Spring中的AOP就是基于初始化后实现的，初始化后返回的对象才是最终的Bean对象。

```java
@Component
public class ZhouyuBeanPostProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        if ("userService".equals(beanName)) {
        	System.out.println("初始化后");
        }
    return bean;
    }
}
```



# Bean的销毁过程

>   略
