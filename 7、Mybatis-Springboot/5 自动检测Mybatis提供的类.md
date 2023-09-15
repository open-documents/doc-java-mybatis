

# 自定义

参考文档：http://mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/#detecting-mybatis-components

MyBatis-Spring-Boot-Starter会自动检测Mybatis提供的接口的实现类：
- Interceptor
- TypeHandler
- LanguageDriver
- DatabaseIdProvider

```java
@Configuration
public class MyBatisConfig {
  @Bean
  MyInterceptor myInterceptor() {
    return MyInterceptor();
  }
  @Bean
  MyTypeHandler myTypeHandler() {
    return MyTypeHandler();
  }
  @Bean
  MyLanguageDriver myLanguageDriver() {
    return MyLanguageDriver();
  }
  @Bean
  VendorDatabaseIdProvider databaseIdProvider() {
    VendorDatabaseIdProvider databaseIdProvider = new VendorDatabaseIdProvider();
    Properties properties = new Properties();
    properties.put("SQL Server", "sqlserver");
    properties.put("DB2", "db2");
    properties.put("H2", "h2");
    databaseIdProvider.setProperties(properties);
    return databaseIdProvider;
  }  
}
```

如果需要改变通过自动配置产生的 `LanguageDriver`，直接向容器中注册自定义的 `LanguageDriver`即可。
```java
@Configuration
public class MyBatisConfig {
    @Bean
    ThymeleafLanguageDriverConfig thymeleafLanguageDriverConfig() {
        return ThymeleafLanguageDriverConfig.newInstance(c -> {
	        // ... customization code
        });
    }
}

@Configuration
public class MyBatisConfig {
    @Bean
    FreeMarkerLanguageDriverConfig freeMarkerLanguageDriverConfig() {
        return FreeMarkerLanguageDriverConfig.newInstance(c -> {
	        // ... customization code
        });
    }
}

@Configuration
public class MyBatisConfig {
    @Bean
    VelocityLanguageDriverConfig velocityLanguageDriverConfig() {
        return VelocityLanguageDriverConfig.newInstance(c -> {
	        // ... customization code
        });
    }
}
```