

MyBatis-Spring-Boot-Starter提供 `SqlSessionFactoryBeanCustomizer` 接口来改变 使用Java Config通过自动配置产生的 `SqlSessionFactoryBean`。
MyBatis-Spring-Boot-Starter将会自动扫描实现了`SqlSessionFactoryBeanCustomizer`接口的类，然后调用其custom()方法来改变Java Config，以此改变创建的`SqlSessionFactoryBean`实例。
```java
@Configuration
public class MyBatisConfig {
    @Bean
    SqlSessionFactoryBeanCustomizer sqlSessionFactoryBeanCustomizer() {
	    return new SqlSessionFactoryBeanCustomizer() {
	        @Override
	        public void customize(SqlSessionFactoryBean factoryBean) {
	        // customize ...
	        }
	    };
    }
}
```
