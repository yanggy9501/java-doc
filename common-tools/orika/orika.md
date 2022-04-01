# orika&对象复制



**属性拷贝工具**

```java
import ma.glasnost.orika.MapperFacade;
import ma.glasnost.orika.MapperFactory;
import ma.glasnost.orika.impl.DefaultMapperFactory;
import ma.glasnost.orika.metadata.ClassMapBuilder;

import java.util.*;
import java.util.concurrent.ConcurrentHashMap;

/**
 * bean 属性拷贝工具
 *
 * @author yanggy
 */
public class PropertyMapper {

    /**
     * 默认映射属性字段工厂
     */
    private static final MapperFactory MAPPER_FACTORY = new DefaultMapperFactory.Builder().build();

    /**
     * 默认映射字段实例
     */
    private static final MapperFacade MAPPER_FACADE = MAPPER_FACTORY.getMapperFacade();

    /**
     * 默认映射字段实例集合(缓存)
     */
    private static final Map<String, MapperFacade> CACHE_MAPPER_FACADE_MAP = new ConcurrentHashMap<>();

    /**
     *  属性映射
     *  映射规则：属性名相同，属性类型相同
     *
     * @param data 源数据对象
     * @param toClass 映射目标对象的字节码对象
     * @param <E> 目标对象泛型
     * @param <T> 源数据对象泛型
     * @return 目标类型对象
     */
    public <E, T> E map(T data, Class<E> toClass) {
        return MAPPER_FACADE.map(data, toClass);
    }

    /**
     *  自定义属性映射
     *  映射规则：属性名可以不同，对应属性类型相同
     *
     * @param data 源数据对象
     * @param toClass 映射目标对象的字节码对象
     * @param <E> 目标对象泛型
     * @param <T> 源数据对象泛型
     * @return 目标类型对象
     */
    public <E, T> E map(T data, Class<E> toClass, Map<String, String> fieldMapConfig) {
        MapperFacade mapperFacade = getMapperFacade(data.getClass(), toClass, fieldMapConfig);
        return mapperFacade.map(data, toClass);
    }

    /**
     * 映射集合(默认字段）
     * 映射规则：属性名相同，属性类型相同
     *
     * @param data 源数据对象集合
     * @param toClass 映射目标对象的字节码对象
     * @param <E> 目标对象泛型
     * @param <T> 源数据对象泛型
     * @return List<target>
     */
    public <E, T> List<E> mapAsList(Collection<T> data, Class<E> toClass) {
        return MAPPER_FACADE.mapAsList(data, toClass);
    }

    /**
     * 映射集合(默认字段）
     * 映射规则：属性名相同，属性类型相同
     *
     * @param data 源数据对象集合
     * @param toClass 映射目标对象的字节码对象
     * @param <E> 目标对象泛型
     * @param <T> 源数据对象泛型
     * @return List<target>
     */
    public <E, T> List<E> mapAsList(Collection<T> data, Class<E> toClass, Map<String, String> fieldMapConfig) {
        Optional<T> od = data.stream().findFirst();
        if (od.isPresent()) {
            MapperFacade mapperFacade = this.getMapperFacade(od.get().getClass(), toClass, fieldMapConfig);
            return mapperFacade.mapAsList(data, toClass);
        }
        return new ArrayList<>(0);
    }

    /**
     * 获取自定义的属性映射
     * 优先从缓存中获取，若缓存中不存在先创建对象在保存到缓存中
     * 映射规则：属性类型相同，属性名可以不同
     *
     * @param dataClass 源数据class对象
     * @param toClass   目标对象class对象
     * @param fieldMapConfig  source对象和target对象的属性映射规则
     * @param <E> 泛型
     * @param <T> 泛型
     * @return MapperFacade
     */
    private <E, T> MapperFacade getMapperFacade(Class<T> dataClass, Class<E> toClass, Map<String, String> fieldMapConfig) {
        String mapKey = dataClass.getCanonicalName() + "_" + toClass.getCanonicalName();
        MapperFacade mapperFacade = CACHE_MAPPER_FACADE_MAP.get(mapKey);
        if (Objects.isNull(mapperFacade)) {
            DefaultMapperFactory factory = new DefaultMapperFactory.Builder().build();
            // 设置映射的源和目标对象字节码
            ClassMapBuilder<T, E> classMapBuilder = factory.classMap(dataClass, toClass);
            // MapBuilder映射构造器中设置source 和 target的属性映射规则
            fieldMapConfig.forEach(classMapBuilder::field);
            classMapBuilder.byDefault().register();
            mapperFacade = factory.getMapperFacade();
            // 保存到缓存中
            CACHE_MAPPER_FACADE_MAP.put(mapKey, mapperFacade);
        }
        return mapperFacade;
    }
}

```

