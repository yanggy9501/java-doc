# Jackson 使用

​		SpringBoot默认的json解析格式就是Jackson，所以不需要额外的引入依赖。

​		Jackson是一个简洁的方式去解析JSON开源包。Jackson可以解析JSON从String，Stream，或者file的方式去创建Java对象。Jackson不仅仅可以解析JSON到Java对象，也可以将Java对象解析为JSON字符串。

## 1，Jackson配置

springboot的这些配置都是全局配置，对所有的使用到的jackson自动序列化和反序列化都有效，对自己new的对象无效。

### 1.1 properties 全局配置

```properties
spring.jackson.date-format指定日期格式，比如yyyy-MM-dd HH:mm:ss，或者具体的格式化类的全限定名

spring.jackson.time-zone指定日期格式化时区，比如America/Los_Angeles或者GMT+10.

spring.jackson.deserialization是否开启Jackson的反序列化

spring.jackson.generator是否开启json的generators.

spring.jackson.joda-date-time-format指定Joda date/time的格式，比如yyyy-MM-ddHH:mm:ss). 如果没有配置的话，dateformat会作为backup

spring.jackson.locale指定json使用的Locale.

spring.jackson.mapper是否开启Jackson通用的特性.

spring.jackson.parser是否开启jackson的parser特性.

spring.jackson.property-naming-strategy指定PropertyNamingStrategy(CAMEL_CASE_TO_LOWER_CASE_WITH_UNDERSCORES)或者指定PropertyNamingStrategy子类的全限定类名.

spring.jackson.serialization是否开启jackson的序列化.

spring.jackson.serialization-inclusion指定序列化时属性的inclusion方式，具体查看JsonInclude.Include枚举.
```

### 1.2 yaml 全局配置

```yaml
spring:
  jackson:
    # 日期格式化
    date-format: yyyy-MM-dd HH:mm:ss
    time-zone: GMT+8
    # 设置空如何序列化
    default-property-inclusion: non_null    
    serialization:
       # 格式化输出 
      indent_output: true
      # 忽略无法转换的对象
      fail_on_empty_beans: false
    deserialization:
      # 允许对象忽略json中不存在的属性
      fail_on_unknown_properties: false
    parser:
      # 允许出现特殊字符和转义符
      allow_unquoted_control_chars: true
      # 允许出现单引号
      allow_single_quotes: true

```

### 1.3 重新注入ObjectMapper

```java
@Bean
@Primary
@ConditionalOnMissingBean(ObjectMapper.class)
public ObjectMapper jacksonObjectMapper(Jackson2ObjectMapperBuilder builder{
   ObjectMapper objectMapper = builder.createXmlMapper(false).build();

   // 通过该方法对mapper对象进行设置，所有序列化的对象都将按改规则进行系列化
   // Include.Include.ALWAYS 默认
   // Include.NON_DEFAULT 属性为默认值不序列化
   // Include.NON_EMPTY 属性为 空（""） 或者为 NULL 都不序列化，则返回的json是没有这个字段的。这样对移动端会更省流量
   // Include.NON_NULL 属性为NULL 不序列化
   objectMapper.setSerializationInclusion(JsonInclude.Include.NON_EMPTY);
   objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
   // 允许出现特殊字符和转义符
   objectMapper.configure(JsonParser.Feature.ALLOW_UNQUOTED_CONTROL_CHARS, true);
   // 允许出现单引号
   objectMapper.configure(JsonParser.Feature.ALLOW_SINGLE_QUOTES, true);
   // 字段保留，将null值转为""
   objectMapper.getSerializerProvider().setNullValueSerializer(new JsonSerializer<Object> () {
       @Override
       public void serialize(Object o, JsonGenerator jsonGenerator,SerializerProvider serializerProvider) throws IOExcepti {
           jsonGenerator.writeString("");
       }
   });
   return objectMapper;
}
```

## 2，常用API

**注意使用json序列化和反序列化必须有setter，getter方法**

### 2.1 封装工具类

```java
import com.fasterxml.jackson.annotation.JsonInclude;
import com.fasterxml.jackson.core.JsonParser;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.DeserializationFeature;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializationFeature;
import lombok.NonNull;
import lombok.extern.slf4j.Slf4j;

@Slf4j
public class JsonUtils {
    private static ObjectMapper mapper = new ObjectMapper();
    
    static {
        // 对于空的对象转json的时候不抛出错误
        mapper.disable(SerializationFeature.FAIL_ON_EMPTY_BEANS);
        // 允许属性名称没有引号
        mapper.configure(JsonParser.Feature.ALLOW_UNQUOTED_FIELD_NAMES, true);
        // 允许单引号
        mapper.configure(JsonParser.Feature.ALLOW_SINGLE_QUOTES, true);
        // 设置输入时忽略在json字符串中存在但在java对象实际没有的属性
        mapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);
        // 设置输出时包含属性的风格
        mapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);
    }

    /**
     * 序列化，将对象转化为json字符串
     *
     * @param data
     * @return
     */
    public static String toJsonString(Object data) {
        if (data == null) {
            return null;
        }

        String json = null;
        try {
            json = mapper.writeValueAsString(data);
        } catch (JsonProcessingException e) {
            log.error("[{}] toJsonString error：{{}}", data.getClass().getSimpleName(), e);
        }
        return json;
    }


    /**
     * 反序列化，将json字符串转化为对象
     *
     * @param json
     * @param clazz
     * @param <T>
     * @return
     */
    public static <T> T parse(@NonNull String json, Class<T> clazz) {
        T t = null;
        try {
            t = mapper.readValue(json, clazz);
        } catch (Exception e) {
            log.error(" parse json [{}] to class [{}] error：{{}}", json, clazz.getSimpleName(), e);
        }
        return t;
    }

}

```

### 2.2 使用注解

| 注解                  | 用法                                                         |
| :-------------------- | :----------------------------------------------------------- |
| @JsonProperty         | 用于属性，把属性的名称序列化时转换为另外一个名称。示例：@JsonProperty("birth_date") private Date birthDate |
| @JsonIgnore           | 可用于字段、getter/setter、构造函数参数上，作用相同，都会对相应的字段产生影响。使相应字段不参与序列化和反序列化。 |
| @JsonIgnoreProperties | 该注解是类注解。该注解在Java类和JSON不完全匹配的时候使用。   |
| @JsonFormat           | 用于属性或者方法，把属性的格式序列化时转换成指定的格式。示例：@JsonFormat(timezone = "GMT+8", pattern = "yyyy-MM-dd HH:mm") public Date getBirthDate() |
| @JsonPropertyOrder    | 用于类， 和 @JsonProperty 的index属性类似，指定属性在序列化时 json 中的顺序 ， 示例：@JsonPropertyOrder({ "birth_Date", "name" }) public class Person |
| @JsonCreator          | 用于构造方法，和 @JsonProperty 配合使用，适用有参数的构造方法。示例：@JsonCreator public Person(@JsonProperty("name")String name) {…} |
| @JsonAnySetter        | 用于属性或者方法，设置未反序列化的属性名和值作为键值存储到 map 中 @JsonAnySetter public void set(String key, Object value) { map.put(key, value); } |
| @JsonAnyGetter        | 用于方法 ，获取所有未序列化的属性 public Map<String, Object> any() { return map; } |
| @JsonNaming           | 类注解。序列化的时候该注解可将驼峰命名的字段名转换为下划线分隔的小写字母命名方式。反序列化的时候可以将下划线分隔的小写字母转换为驼峰命名的字段名。示例：@JsonNaming(PropertyNamingStrategy.SnakeCaseStrategy.class) |
| @JsonRootName         | 类注解。需开启mapper.enable(SerializationFeature.WRAP_ROOT_VALUE)，用于序列化时输出带有根属性名称的 JSON 串，形式如 {"root_name":{"id":1,"name":"zhangsan"}}。但不支持该 JSON 串反序列化。 |

#### 2.1.1 案例

````java
// 用于类,指定属性在序列化时 json 中的顺序
@JsonPropertyOrder({"date", "user_name"})
// 批量忽略属性，不进行序列化
@JsonIgnoreProperties(value = {"other"})
// 用于序列化与反序列化时的驼峰命名与小写字母命名转换
@JsonNaming(PropertyNamingStrategy.SnakeCaseStrategy.class)
public static class User {
    @JsonIgnore
    private Map<String, Object> other = new HashMap<>();
 
    // 正常case
    @JsonProperty("user_name")
    private String userName;
    // 空对象case
    private Integer age;
    @JsonFormat(timezone = "GMT+8", pattern = "yyyy-MM-dd HH:mm:ss")
    // 日期转换case
    private Date date;
    // 默认值case
    private int height;
 
    public User() {
    }
 
    // 反序列化执行构造方法
    @JsonCreator
    public User(@JsonProperty("user_name") String userName) {
        System.out.println("@JsonCreator 注解使得反序列化自动执行该构造方法 " + userName);
        // 反序列化需要手动赋值
        this.userName = userName;
    }
 
    @JsonAnySetter
    public void set(String key, Object value) {
        other.put(key, value);
    }
 
    @JsonAnyGetter
    public Map<String, Object> any() {
        return other;
    }
}
````

### 2.3 日期处理

不同类型的日期类型，JackSon 的处理方式也不同。

#### 2.3.1 普通日期

>   对于日期类型为 `Calendar`, `GregorianCalendar`, `Date`,  若不指定格式，将序列化为 `long` 类型的数据。

**JackSon 普通日期转换格式**

*   注解方式：使用 `@JsonFormat` 注解指定日期格式。
*   全局配置：在springboot项目中，可以全局配置。
*   ObjectMapper 方式：ObjectMapper 的方法 `setDateFormat`，将序列化为指定格式的 string 类型的数据。

#### 2.3.2 Local日期

>   对于日期类型为 `LocalDate`, `LocalDateTime`，还需要添加代码添加代码 `mapper.registerModule(new JavaTimeModule())`

## 3，序列化

### 3.1 JSON字符串转换为Java对象

```java
ObjectMapper objectMapper = new ObjectMapper();
String carJson ="{ \"brand\" : \"Mercedes\", \"doors\" : 5 }";
Car car = objectMapper.readValue(carJson, Car.class);
```

### 3.2 JSON Reader转换为Java对象

```java
ObjectMapper objectMapper = new ObjectMapper();
String carJson = "{ \"brand\" : \"Mercedes\", \"doors\" : 4 }";
Reader reader = new StringReader(carJson);
Car car = objectMapper.readValue(reader, Car.class);
```

### 3.3 从File 文件中转换为Java对象

```java
ObjectMapper objectMapper = new ObjectMapper();
File file = new File("data/car.json");
Car car = objectMapper.readValue(file, Car.class);
```

### 3.4 从URL 资源转换为Java对象

```java
ObjectMapper objectMapper = new ObjectMapper();
URL url = new URL("file:data/car.json");
Car car = objectMapper.readValue(url, Car.class);
```

### 3.5 从InputStream 输入流中转换为Java对象

```java
ObjectMapper objectMapper = new ObjectMapper();
InputStream input = new FileInputStream("data/car.json");
Car car = objectMapper.readValue(input, Car.class);
```

### 3.6 从Byte Array 中转换为Java对象

```java
ObjectMapper objectMapper = new ObjectMapper();
String carJson = "{ \"brand\" : \"Mercedes\", \"doors\" : 5 }";
byte[] bytes = carJson.getBytes("UTF-8");
Car car = objectMapper.readValue(bytes, Car.class);
```

### 3.7 从JSON字符数组中转换为Java对象数组

```java
String jsonArray = "[{\"brand\":\"ford\"}, {\"brand\":\"Fiat\"}]";
ObjectMapper objectMapper = new ObjectMapper();
Car[] cars2 = objectMapper.readValue(jsonArray, Car[].class);
```

### 3.8 从JSON字符数组中转换为Java对象列表

```java
String jsonArray = "[{\"brand\":\"ford\"}, {\"brand\":\"Fiat\"}]";
ObjectMapper objectMapper = new ObjectMapper();
List<Car> cars1 = objectMapper.readValue(jsonArray, new TypeReference<List<Car>>(){});
```

### 3.9 从JSON字符串中转换为Java的Map存储

```java
String jsonObject = "{\"brand\":\"ford\", \"doors\":5}";
ObjectMapper objectMapper = new ObjectMapper();
Map<String, Object> jsonMap = objectMapper.readValue(jsonObject, new TypeReference<Map<String,Object>>(){});
```

