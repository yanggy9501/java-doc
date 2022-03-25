# AOP面向切面编程

## 1，动态代理

>   代理模式的解释：为其他对象提供一种代理以控制对这个对象的访问，增强一个类中的某个方法，对程序进行扩展。
>
>   可以实现在不改变原有代码的基础上对代码增强，补充一些，装饰器模式同样有增强类功能的作用。
>
>   代理对象不再是原来的实例对象了，代理技术在jvm中生成代理类的字节码。

### 1.1 cglib动态代理

>   通过cglib来实现的代理对象的创建，其是基于extends的，被代理类（target）是父类，代理类extends父类扩展父类的功能，所以代理类是子类（cglib是怎么创建代理类的程序员不需要关心）。

#### 1.1.1 创建拦截器

>   创建一个拦截器，实现接口net.sf.cglib.proxy.MethodInterceptor，用于方法的拦截回调。

intercept方法参数说明

*   target：表示要进行增强的对象
*   method：表示要被拦截的方法
*   args：表示要被拦截方法的参数
*   methodProxy：表示要触发父类的方法对象

```java
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;

import java.lang.reflect.Method;

/**
 * @author yanggy
 */
public class LogInterceptor implements MethodInterceptor {
    /**
     * @param target 表示要进行增强的对象(被代理对象)
     * @param method 表示拦截的方法（被代理对象的方法，但是执行target方法不用该method）
     * @param args 被代理对象的参数列表
     * @param methodProxy 表示对方法的代理，invokeSuper方法表示对被代理对象方法的调用（表示要触发父类的方法对象）
     * @return 一般被代理对象方法执行的返回结果
     * @throws Throwable t
     */
    @Override
    public Object intercept(Object target, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        // 增强逻辑
        before(method.getName());
        // 注意这里是调用invokeSuper而不是invoke，否则死循环;
        // methodProxy.invokeSuper执行的是原始类的方法;
        // method.invoke执行的是子类的方法;
        Object result = methodProxy.invokeSuper(target, args);
        // 增强逻辑
        after(method.getName());
        return result;
    }

    /**
     * 调用invoke方法之前执行
     */
    private void before(String methodName) {
        System.out.println("调用方法" + methodName +"之【前】的日志处理");
    }

    /**
     * 调用invoke方法之后执行
     */
    private void after(String methodName) {
        System.out.println("调用方法" + methodName +"之【后】的日志处理");
    }
}
```

>   *在方法的内部主要调用的methodProxy.invokeSuper，执行的原始类的方法。虽然method是target的方法，如果调用invoke方法否会出现死循环，所以 ~~method.invoke(target, args)~~。
>
>   



#### 1.1.2 客户端使用

```java
public static void main(String[] args) {
    // 通过CGLIB动态代理获取代理对象的过程
    // 创建Enhancer对象，类似于JDK动态代理的Proxy类
    Enhancer enhancer = new Enhancer();
    // 设置目标类的字节码文件
    enhancer.setSuperclass(UserService.class);
    // 设置回调函数
    enhancer.setCallback(new LogInterceptor());
    // create方法正式创建代理类
    UserService userService = (UserService) enhancer.create();
    // 调用代理类的具体业务方法
    userService.test();
}
```

**ps:** 发现debug和run不一样，debug时控制台会多打印，即好像拦截多次。

#### 1.1.3 过程&流程

从反编译的源码可以看出，代理对象继承Target类，拦截器调用intercept()方法，intercept()方法由自定义的LogInterceptor实现，所以最后调用LogInterceptor中的intercept()方法，从而完成了由代理对象访问到目标对象的动态代理实现。

**流程**

1.   创建拦截器-方法回调，是增强的逻辑以及控制target的执行
2.   创建cglib帮助类Enhancer
3.   Enhancer设值父类（cglib是基于父子类实现的）
4.   设值回调
5.   Enhancer.ceate创建代理对象

#### 1.1.4 JDK动态代理与CGLIB对比

*   JDK动态代理：基于Java反射机制实现，必须要实现了接口的业务类才生成代理对象
*   CGLIB动态代理：基于ASM机制实现，通过生成业务类的子类作为代理类

****



