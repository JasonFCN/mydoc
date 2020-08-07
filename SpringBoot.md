### SpringBoot学习

#### spring自动配置原理

配置文件（yaml,properties文件）中配置的值通过`@ConfigurationProperties`注解绑定到对应的XXXProperties类中。

![image-20200623154903795](SpringBoot.assets/image-20200623154903795.png)

而后通过`@EnableConfigurationProperties`注解指定XXXProperties来加载配置信息到`XXXAutoConfiguration`类中。

![image-20200623155027202](SpringBoot.assets/image-20200623155027202.png)

spring-boot-autoconfigure-2.3.1.RELEASE.jar中META-INF/spring.factories定义了大量的`XXXAutoConfiguration`：

![image-20200623154706533](SpringBoot.assets/image-20200623154706533.png)

SpringBoot应用的启动类都有一个注解：`@SpringBootApplication`

该注解的定义：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
```

其中的`@EnableAutoConfiguration`通过`@Import({AutoConfigurationImportSelector.class})`完成对AutoConfiguration类的加载：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}
```

总结：

1. 当应用启动时，@SpringBootApplication中的@EnableAutoConfiguration通过@Import({AutoConfigurationImportSelector.class})加载spring-boot-autoconfigure-2.3.1.RELEASE.jar下的META-INF/spring.factories中定义自动配置组件。
2. 每一个组件又通过`@EnableConfigurationProperties`注解来加载ServerProperties类信息。
3. ServerProperties类通过@ConfigurationProperties注解去加载配置文件（yaml,properties）中定义的值。

#### 配置java8新的时间API对序列化，反序列化，参数绑定

```java
package com.cwj.config;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializationFeature;
import com.fasterxml.jackson.databind.deser.std.DateDeserializers;
import com.fasterxml.jackson.databind.ser.std.DateSerializer;
import com.fasterxml.jackson.datatype.jsr310.JavaTimeModule;
import com.fasterxml.jackson.datatype.jsr310.deser.LocalDateTimeDeserializer;
import com.fasterxml.jackson.datatype.jsr310.ser.LocalDateTimeSerializer;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.autoconfigure.jackson.Jackson2ObjectMapperBuilderCustomizer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.converter.json.Jackson2ObjectMapperBuilder;

import java.text.SimpleDateFormat;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.Date;

@Configuration
public class LocalDateTimeSerializerConfig {

    @Value("${spring.jackson.date-format:yyyy-MM-dd HH:mm:ss}")
    private String pattern;

    @Bean
    public LocalDateTimeSerializer localDateTimeSerializer() {
        return new LocalDateTimeSerializer(DateTimeFormatter.ofPattern(pattern));
    }

    @Bean
    public DateSerializer dateSerializer() {
        return new DateSerializer(true, new SimpleDateFormat(pattern));
    }

    @Bean
    public Jackson2ObjectMapperBuilderCustomizer jackson2ObjectMapperBuilderCustomizer() {
        return builder -> builder
                .serializerByType(LocalDateTime.class, localDateTimeSerializer())
//                .serializerByType(Date.class, dateSerializer())
                ;
    }

    @Bean
    public ObjectMapper serializingObjectMapper() {
        JavaTimeModule module = new JavaTimeModule();
        LocalDateTimeDeserializer dateTimeDeserializer = new LocalDateTimeDeserializer(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
//        MyLocalDateTimeDeserializer myLocalDateTimeDeserializer = new MyLocalDateTimeDeserializer(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
        DateDeserializers.DateDeserializer deserializer = new DateDeserializers.DateDeserializer(DateDeserializers.DateDeserializer.instance, new SimpleDateFormat(pattern), pattern);
        module.addDeserializer(LocalDateTime.class, dateTimeDeserializer);
        module.addSerializer(LocalDateTime.class, localDateTimeSerializer());
        module.addSerializer(Date.class, DateSerializer.instance.withFormat(false, new SimpleDateFormat(pattern)));
        module.addDeserializer(Date.class, deserializer);
        return Jackson2ObjectMapperBuilder.json().modules(module)
                .featuresToDisable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS).build();
    }
}
```

```java
package com.cwj.config;

import lombok.SneakyThrows;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.convert.converter.Converter;

import java.text.SimpleDateFormat;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.Date;

@Configuration
public class DateConverterConfig {
    @Bean
    public Converter<String, LocalDate> localDateConverter() {
        return new Converter<String, LocalDate>() {
            @Override
            public LocalDate convert(String source) {
                return LocalDate.parse(source, DateTimeFormatter.ofPattern("yyyy-MM-dd"));
            }
        };
    }

    @Bean
    public Converter<String, LocalDateTime> localDateTimeConverter() {
        return new Converter<String, LocalDateTime>() {
            @Override
            public LocalDateTime convert(String source) {
                return LocalDateTime.parse(source, DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
            }
        };
    }

    @Bean
    public Converter<String, Date> dateConverter() {
        return new Converter<String, Date>() {
            @SneakyThrows
            @Override
            public Date convert(String source) {
                return new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").parse(source);
            }
        };
    }
}
```

