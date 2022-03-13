# 推断构造方法

​		spring在创建对象时，使用反射调用构造方法时创建bean对象的一种方式。如果存在多个构造方法如确定使用哪个构造方法呢？

​		如果只有一个无参的构造方法，那么实例化就只能使用这个构造方法了。 如果只有一个有参的构造方法，那么实例化时能使用这个构造方法吗？

>   1.   如果开发者指定了想要使用的构造方法，那么就用这个构造方法 
>   2.   如果开发者没有指定想要使用的构造方法，则看开发者有没有让Spring自动去选择构造方法 
>   3.   如果开发者也没有让Spring自动去选择构造方法，则Spring利用无参构造方法，如果没有无参构 造方法，则报错。



## 1，指定构造方法

### 1.1 xml中指定

>   xml中的标签，<constructor-arg>这个标签表示构造方法参数，即指定了构造方法。
>
>   这种方式开发者手动指定了构造方法的参数值。

### 1.2 javaconfig中指定

>    通过@Autowired注解，@Autowired注解可以写在构造方法就是指定该构造方法。这种方式需要spring通过byType或byName的方式去找符合条件的参数值。
>
>   **注意：**多个构造方法上写了@Autowired注解，那么此时Spring会报错。



## 2，过程

>   1. AbstractAutowireCapableBeanFactory类中的createBeanInstance()方法会去创建一个Bean实例
>   2. 根据BeanDefinition加载类得到Class对象
>   3. 如果BeanDefinition绑定了一个Supplier，那就调用Supplier的get方法得到一个对象并直接返回
>   4. 如果BeanDefinition中存在factoryMethodName，那么就调用该工厂方法得到一个bean对象并返回
>   5. 如果BeanDefinition已经自动构造过了，那就调用autowireConstructor()自动构造一个对象
>   6. 调用SmartInstantiationAwareBeanPostProcessor的determineCandidateConstructors()方法得到哪些构造方法是可以用的
>   7. 如果存在可用得构造方法，或者当前BeanDefinition的autowired是AUTOWIRE_CONSTRUCTOR，或者BeanDefinition中指定了构造方法参数值，或者创建Bean的时候指定了构造方法参数值，那么就调用**autowireConstructor()**方法自动构造一个对象
>   8. 最后，如果不是上述情况，就根据无参的构造方法实例化一个对象。

