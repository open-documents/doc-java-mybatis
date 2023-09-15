
参考文档：
- 英文参考文档 -- https://mybatis.org/spring/mappers.html

除了直接使用 `SqlSessionDaoSupport`实现类 或者 SqlSessionTemplate， Mybatis-Spring能够创建线程安全的Mapper实例，然后可以将其注入到容器中需要的组件中。

下面是一个简单的例子。
```java
public class FooServiceImpl implements FooService {
    private final UserMapper userMapper;
    public FooServiceImpl(UserMapper userMapper) {
	    this.userMapper = userMapper;
    }
    public User doSomeBusinessStuff(String userId) {
	    return this.userMapper.getUser(userId);
    }
}
```
```xml
<bean id="fooService" class="org.mybatis.spring.sample.service.FooServiceImpl">
    <constructor-arg ref="userMapper" />
</bean>
```

# 注册Mapper

Mybatis-Spring提供 MapperFactoryBean类 来完成向容器中注册Mapper的功能。

下面是MapperFactoryBean类的重要的部分（不是全部内容）。
```java
public class MapperFactoryBean<T> extends SqlSessionDaoSupport implements FactoryBean<T> {

	private Class<T> mapperInterface;
	public void setMapperInterface(Class<T> mapperInterface) {  
	    this.mapperInterface = mapperInterface;  
	}

	public MapperFactoryBean() {}   
	public MapperFactoryBean(Class<T> mapperInterface) {  
	    this.mapperInterface = mapperInterface;  
	}

	public T getObject() throws Exception {  
	    return getSqlSession().getMapper(this.mapperInterface);  
	}
}
```
使用xml方式向容器中注册MapperFactoryBean组件：
```xml
<bean id="userMapper" class="org.mybatis.spring.mapper.MapperFactoryBean">
    <property name="mapperInterface" value="org.mybatis.spring.sample.mapper.UserMapper" />
    <property name="sqlSessionFactory" ref="sqlSessionFactory" />
</bean>
```
使用@Bean方式向容器中注册MapperFactoryBean组件：
```java
@Configuration
public class MyBatisConfig {
    @Bean
    public MapperFactoryBean<UserMapper> userMapper() throws Exception {
	    MapperFactoryBean<UserMapper> factoryBean = new MapperFactoryBean<>(UserMapper.class);
	    factoryBean.setSqlSessionFactory(sqlSessionFactory());
	    return factoryBean;
    }
}
```

需要注意的是：如果Java Mapper接口和Mapper配置文件位于同一位置，那么不需要指定MapperFactoryBean的mapperInterface属性。注意，还是需要MapperFactoryBean实例的，只是不需要指定mapperInterface属性。

# 扫描Mapper

除了上述显示地在MapperFactoryBean实例中指定Mapper接口，还可以通过扫描的方式来自动注册Mapper组件。

MyBatis-Spring提供3种不同的方式来实现：
- 使用 `<mybatis:scan>`         （MyBatis-Spring 1.2.0引入）
- 使用 @MapperScan注解   （MyBatis-Spring 1.2.0引入）
- 向容器种注册 MapperScannerConfigurer组件  （需要 Spring 3.1+）

## <mybatis:scan>和@MapperScan

下面是<mybatis:scan>的全部属性：
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:mybatis="http://mybatis.org/schema/mybatis-spring"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://mybatis.org/schema/mybatis-spring http://mybatis.org/schema/mybatis-spring.xsd">

	<mybatis:scan 
		base-package=""  
        annotation=""  
        marker-interface=""  
        factory-ref=""  
        template-ref=""  
        mapper-factory-bean-class=""  
        name-generator=""  
        lazy-initialization=""  
        default-scope=""/>

</beans>
```
下面是@MapperScan的全部属性（相比之下多了扫描类所在包的属性）：
```java
@Retention(RetentionPolicy.RUNTIME)  
@Target(ElementType.TYPE)  
@Documented  
@Import(MapperScannerRegistrar.class)  
@Repeatable(MapperScans.class)
public @interface MapperScan {

	@AliasFor("basePackages")  
	String[] value() default {};
	@AliasFor("value")  
	String[] basePackages() default {};
	Class<?>[] basePackageClasses() default {};

	Class<? extends Annotation> annotationClass() default Annotation.class;
	Class<?> markerInterface() default Class.class;

	String sqlSessionTemplateRef() default "";
	String sqlSessionFactoryRef() default "";
	Class<? extends MapperFactoryBean> factoryBean() default MapperFactoryBean.class;

	Class<? extends BeanNameGenerator> nameGenerator() default BeanNameGenerator.class;

	String lazyInitialization() default "";
	String defaultScope() default AbstractBeanDefinition.SCOPE_DEFAULT;
}
```

下面以@MapperScan的全部属性展开介绍。

- basePackages()（对应base-package属性）：需要扫描的包，在包下所有满足条件的Mapper都会被注册为组件。
- annotationClass()（对应annotation属性）：指定注解，过滤扫描到的Mapper，只有Mapper上标注了指定的注解才会被注册到容器中。
- markerInterface()（对应marker-interface属性）：指定接口，过滤扫描到的Mapper，只有Mapper上继承了指定的接口才会被注册到容器中。
- sqlSessionTemplateRef()（对应template-ref属性）：指定MapperFactoryBean需要使用到的SqlSessionTemplate Bean的名称。
- sqlSessionFactoryRef()（对应factory-ref属性）：指定MapperFactoryBean需要使用到的SqlSessionTemplate Bean所需要的SqlSessionFactoryBean Bean的名称，已经指定了sqlSessionTemplateRef()后，该属性会被忽略。
- nameGenerator()（对应name-generator属性）：指定注册到容器中Mapper Bean名称的生成策略。

除了上述属性，将lazyInitialization() 和 defaultScope()单独介绍。

1）lazyInitialization()和lazy-initialization

从2.0.2开始支持mapper扫描这一特性，是为了springboot2.2懒加载的特性。默认为false，即不是懒加载。

出现下面的情况时，不应该使用懒加载的特征：
- When refers to the statement of **other mapper** using `<association>`(`@One`) and `<collection>`(`@Many`)
- When includes to the fragment of **other mapper** using `<include>`
- When refers to the cache of **other mapper** using `<cache-ref>`(`@CacheNamespaceRef`)
- When refers to the result mapping of **other mapper** using `<select resultMap="...">`(`@ResultMap`)

## MapperScannerConfigurer

MapperScannerConfigurer其中包含的就是上述提到的属性（下面没有列出与之无关的属性，所有的属性都提供setXxx()方法）。
```java
public class MapperScannerConfigurer implements BeanDefinitionRegistryPostProcessor, InitializingBean, ApplicationContextAware, BeanNameAware {

	private String basePackage;  

	private SqlSessionFactory sqlSessionFactory;  
	private SqlSessionTemplate sqlSessionTemplate;  
	private String sqlSessionFactoryBeanName;  
	private String sqlSessionTemplateBeanName;  
	
	private Class<? extends MapperFactoryBean> mapperFactoryBeanClass;  
	
	private Class<? extends Annotation> annotationClass;  
	private Class<?> markerInterface;  

	private BeanNameGenerator nameGenerator; 

	private ApplicationContext applicationContext;  

	private String lazyInitialization;  
	private String defaultScope;

}
```
通过向容器中注册自定义的MapperScannerConfigurer组件完成扫描Mapper的自定义。
```xml
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <property name="basePackage" value="org.mybatis.spring.sample.mapper" />
</bean>
```