---
title: SpringBoot中时间类型 序列化、反序列化、格式处理
tags:
  - SpringBoot
  - 序列化
  - 时间格式
categories:
  - SpringBoot
---
# SpringBoot中时间类型 序列化、反序列化、格式处理

## Date

**yml全局配置**

```yaml
spring:  
  jackson:
    time-zone: GMT+8
    date-format: yyyy-MM-dd HH:mm:ss #配置POST请求Body中Date时间类型序列化格式处理，并返回
```

**请求参数类型转换**

```java
/**
 * 时间Date转换
 * 配置GET请求，Query查询Date时间类型参数转换
 */
@Component
public class DateConverter implements Converter<String, Date> {
  @Override
  public Date convert(String source) {
    if (StringUtils.isBlank(source)) {
      return null;
    }
    if (source.matches("^\\d{4}-\\d{1,2}-\\d{1,2}$")) {
      return parseDate(source.trim(), "yyyy-MM-dd");
    }
    if (source.matches("^\\d{4}-\\d{1,2}-\\d{1,2} {1}\\d{1,2}:\\d{1,2}:\\d{1,2}$")) {
      return parseDate(source.trim(), "yyyy-MM-dd HH:mm:ss");
    }
    throw new IllegalArgumentException("Invalid value '" + source + "'");
  }

  public Date parseDate(String dateStr, String format) {
    Date date = null;
    try {
      date = new SimpleDateFormat(format).parse(dateStr);
    } catch (ParseException e) {
      log.warn("转换{}为日期(pattern={})错误！", dateStr, format);
    }
    return date;
  }
}
```

## JDK8-时间类型-LocalDateTime、LocalDate、LocalTime

```java
/**
 * 序列化,反序列化,格式处理
 *
 * @author zc
 * @date 2020/7/9 01:42
 */
@Slf4j
@Configuration
public class JacksonCustomizerConfig {

    @Value("${spring.jackson.date-format:yyyy-MM-dd HH:mm:ss}")
    private String localDateTimePattern;

    @Value("${spring.jackson.local-date-format:yyyy-MM-dd}")
    private String localDatePattern;

    @Value("${spring.jackson.local-time-format:HH:mm:ss}")
    private String localTimePattern;

    @Bean
    public Jackson2ObjectMapperBuilderCustomizer jackson2ObjectMapperBuilderCustomizer() {
        return builder -> {
            builder.serializerByType(LocalDateTime.class, new LocalDateTimeSerializer(DateTimeFormatter.ofPattern(localDateTimePattern)));
            builder.serializerByType(LocalDate.class, new LocalDateSerializer(DateTimeFormatter.ofPattern(localDatePattern)));
            builder.serializerByType(LocalTime.class, new LocalTimeSerializer(DateTimeFormatter.ofPattern(localTimePattern)));
            builder.deserializerByType(LocalDateTime.class, new LocalDateTimeDeserializer(DateTimeFormatter.ofPattern(localDateTimePattern)));
            builder.deserializerByType(LocalDate.class, new LocalDateDeserializer(DateTimeFormatter.ofPattern(localDatePattern)));
            builder.deserializerByType(LocalTime.class, new LocalTimeDeserializer(DateTimeFormatter.ofPattern(localTimePattern)));
        };
    }
  
  	/**
     * 时间LocalDateTime转换
     */
    @Component
    public static class LocalDateTimeConverter implements Converter<String, LocalDateTime> {
        @Override
        public LocalDateTime convert(String source) {
            if (StringUtils.isBlank(source)) {
                return null;
            }
            if (source.matches("^\\d{4}-\\d{1,2}-\\d{1,2} {1}\\d{1,2}:\\d{1,2}:\\d{1,2}$")) {
                return LocalDateTime.parse(source, DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
            }
            throw new IllegalArgumentException("Invalid value '" + source + "'");
        }
    }

    /**
     * 时间LocalDate转换
     */
    @Component
    public static class LocalDateConverter implements Converter<String, LocalDate> {
        @Override
        public LocalDate convert(String source) {
            if (StringUtils.isBlank(source)) {
                return null;
            }
            if (source.matches("^\\d{4}-\\d{1,2}-\\d{1,2}$")) {
                return LocalDate.parse(source, DateTimeFormatter.ofPattern("yyyy-MM-dd"));
            }
            throw new IllegalArgumentException("Invalid value '" + source + "'");
        }
    }
  
}
```

