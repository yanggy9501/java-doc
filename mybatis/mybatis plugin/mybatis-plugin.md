# mybatis-插件开发

一个接口，两个注解，四大对象

## 1. 插件开发流程

> mybatis 的插件使用的是拦截器实现，在SQL的执行前后进行拦截

### 1.1 拦截器接口

> mybatis拦截器: org.apache.ibatis.plugin.Interceptor.
>
> 实现插件功能必须实现该接口，在intercept方法中实现自己的逻辑。

```java
public interface Interceptor {

  Object intercept(Invocation invocation) throws Throwable;

  default Object plugin(Object target) {
    return Plugin.wrap(target, this);
  }

  default void setProperties(Properties properties) {
    // NOP
  }
}
```

> 方法：intercept就是我们的拦截逻辑，拦截逻辑都写在这里面。
>
> > 参数：Invocation 
> >
> > 方法：proceed() 在invoke前面执行就是前拦截，在invoke后面面执行就是后拦截
> >
> > ```java
> > public Object proceed() throws InvocationTargetException, IllegalAccessException {
> >     return method.invoke(target, args);
> >   }
> > ```
>
> 方法：plugin-就是插件本身，把插件包装成代理对象，有默认实现，一般不动。
>
> 方法：serPropertis-使用xml配置拦截器属性，spring使用属性注入，一般用注解注入



## 2. mybatis四大对象

> mybatis 的拦截器并不是什么都拦截，只拦截4大对象（自己实现接口的实现）

### 2.1 Executor

> org.apache.ibatis.executor.Executor - sql的执行器
>
> 如果实现的插件需要拦截sql的执行或者事务的话，那么就需要拦截这个对象。

### 2.2 ParameterHandler

> org.apache.ibatis.executor.parameter.ParameterHandler - 参数处理
>
> 这个对象主要做参数处理 - 可以针对sql入参进行处理如：传入的是枚举，枚举如何转成真正的值



### 2.3 ResultSetHandler

> org.apache.ibatis.executor.resultset.ResultSetHandler - 结果集处理
>
> 如果想干预结果集的处理，那么拦截这个对象
>
> 例如：数据脱敏，杨*银



### 2.4 StatementHandler

> org.apache.ibatis.executor.statement.StatementHandler - sql语法构建



## 3. 两个注解

### 3.1 Intercepts

> @Intercepts该注解告诉mybatis该类是mybatis的拦截器



### 3.2 @Signature

> mybatis虽然知道了哪个类是拦截器，但是还需要知道该拦截器拦截什么对象（四大对象）。
>
> @Signature(type = ResultSetHandler.class, method = "handleResultSets", args = Statement.class)
>
> 属性：
>
> type：拦截的对象有（Executor，ParameterHandler，StatementHandler，StatementHandler）
>
> method：拦截该拦截对象的什么方法
>
> args：拦截方法的参数类型

```java
@Intercepts(@Signature(type = ResultSetHandler.class, method = "handleResultSets", args = Statement.class))
@Component
public class TuominPlugin implements Interceptor {
    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        List<Object> result = (List<Object>) invocation.proceed();
        return result;
    }
}
```

**注意：**@Intercepts虽然标记了为插件，@Component还是要加的不然不起作用

## 4. 案例-数据脱敏

1. 脱敏注解

   > 使用在实体类上，表名哪个字段需要脱敏

   ```java
   @Target(ElementType.FIELD)
   @Retention(RetentionPolicy.RUNTIME)
   public @interface Tuomin {
   
       TuominStrategy strategy();
   }
   ```

2. 实体类

   ```java
   @Data
   public class User {
   
       private Long userId;
   
       @Tuomin(strategy = TuominStrategy.USERNAME)
       private String nickName;
   
       private String sex;
   
       private String address;
   }
   
   ```

3. 脱敏策略（不同的字段有不同的脱敏策略如：姓名 杨*银；手机：139\*\*\*\*1234）

   ```java
   public enum TuominStrategy {
       USERNAME(s -> s.replaceAll("(\\S)\\S(\\S*)", "$1*$2")),
       PHONE(s -> s.replaceAll("(\\d{4})\\d{10}(\\w{4})", "$1****$2"));
   
       private final Desensitizer desensitizer;
   
       private TuominStrategy(Desensitizer desensitizer) {
           this.desensitizer = desensitizer;
       }
   
       public Desensitizer getDesensitizer() {
           return desensitizer;
       }
   }
   
   ```

   ```java
   public interface Desensitizer extends Function<String, String> {
   }
   
   ```

4.  插件

   ```java
   @Intercepts(@Signature(type = ResultSetHandler.class, method = "handleResultSets", args = Statement.class))
   @Component
   public class TuominPlugin implements Interceptor {
       @Override
       public Object intercept(Invocation invocation) throws Throwable {
           List<Object> result = (List<Object>) invocation.proceed();
           for (Object o : result) {
               tuomin(o);
           }
           return result;
       }
   
       private void tuomin(Object source) {
           Class<?> clazz = source.getClass();
           MetaObject metaObject = SystemMetaObject.forObject(source);
           Field[] declaredFields = clazz.getDeclaredFields();
           Stream.of(declaredFields).filter(field -> field.isAnnotationPresent(Tuomin.class))
               .forEach(field -> doTuoim(metaObject, field));
       }
   
       private void doTuoim(MetaObject metaObject, Field field) {
           String name = field.getName();
           Object value = metaObject.getValue(name);
           if (String.class == metaObject.getGetterType(name) && value != null) {
               Tuomin annotation = field.getAnnotation(Tuomin.class);
               TuominStrategy strategy = annotation.strategy();
               Object o = strategy.getDesensitizer().apply((String) value);
               metaObject.setValue(name, o);
           }
       }
   }
   ```

   

## 5. 案例-SQL打印插件 

