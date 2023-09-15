
通过Spring使用Mybatis，需要：
- 至少一个SqlSessionFactory 
- 至少一个Mapper接口

MyBatis-Spring-Boot-Starter 将会：
- 自动检测已经存在的DataSource。
- 使用检测到的DataSource来创建SqlSessionFactoryBean实例。
- 使用SqlSessionFactoryBean实例来创建SqlSessionFactory实例，并将该实例注册到容器中。
- 使用SqlSessionFactory实例来创建SqlSessionTemplate，并将其注册到容器中。
- 自动扫描Mapper，将其链接到SqlSessionTemplate，同时将Mapper实例注册到容器中以便注入到其他组件中。

假定我们有下面的Mapper。
```java
@Mapper
public interface CityMapper {
    @Select("SELECT * FROM CITY WHERE state = #{state}")
    City findByState(@Param("state") String state);
}
```
创建正常的SpringBoot应用，并向其中注册CityMapper实例。
```java
@SpringBootApplication
public class SampleMybatisApplication implements CommandLineRunner {
    private final CityMapper cityMapper;
    public SampleMybatisApplication(CityMapper cityMapper) {
	    this.cityMapper = cityMapper;
    }

    public static void main(String[] args) {
	    SpringApplication.run(SampleMybatisApplication.class, args);
    }

    @Override
    public void run(String... args) throws Exception {
	    System.out.println(this.cityMapper.findByState("CA"));
    }
}
```

# 扫描

MyBatis-Spring-Boot-Starter默认会搜索标注@Mapper注解的Mapper。

如果需要自定义被扫描的注解，使用@MapperScan注解。

如果容器中存在 `MapperFactoryBean` ，那么MyBatis-Spring-Boot-Starter不会开启扫描过程，因此需要使用@Bean来注册Mapper。

