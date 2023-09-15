
参考文档：
- 英文参考文档 -- https://mybatis.org/spring/factorybean.html
- 中文参考文档 -- 

在Mybatis中，使用  <font color=44cf57>SqlSessionFactoryBuilder</font> 来创建<font color="FF5500">SqlSessionFactory</font>实例。
在MyBatis-Spring中，使用 <font color="00E0FF">SqlSessionFactoryBean</font> 来创建<font color="FF5500">SqlSessionFactory</font>实例。

在MyBatis-Spring中，往往不会直接使用<font color="00E0FF">SqlSessionFactoryBean</font>和<font color="FF5500">SqlSessionFactory</font>。<font color="FF5500">SqlSessionFactory</font>将会被注入到<font color="FFDD00">MapperFactoryBean</font>或extend了<font color="FFC0CB">SqlSessionDaoSupport</font>的Dao类中。

# 配置SqlSessionFactoryBean

下面有2种方式来配置SqlSessionFactoryBean。

1）xml方式
```xml
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="dataSource" ref="dataSource" />
</bean>
```
2）@Bean方式
```java
@Configuration
public class MyBatisConfig {
    @Bean
    public SqlSessionFactory sqlSessionFactory() throws Exception {
	    SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
	    factoryBean.setDataSource(dataSource());
	    return factoryBean.getObject();
    }
}
```

# 属性（Properties）

）SqlSessionFactoryBean有必须的属性 <font color="00E0FF">DataSource</font>，必须配置。

）SqlSessionFactoryBean有属性 <font color="00E0FF">configLocation</font> ，用来指定Mybatis配置文件位置，有2个场景会出现这样的需求：
- 需要改变Mybatis基础配置（往往是`<settings>`和`<typeAliases>`部分）时使用。
- mapper配置文件和mapper接口不在一个位置，需要在`<mappers>`部分指定。
指定的Mybatis配置文件不必是完整的Mybatis文件，因为environments、dataSource、transactionManager都会被忽略，因为SqlSessionFactoryBean 会创建自己的自定义的Mybatis Environment。

）

）SqlSessionFactoryBean有属性 <font color="00E0FF">mapperLocations</font>，是指定mapper文件的另外一种方式。可以使用Ant风格。
```xml
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="dataSource" ref="dataSource" />
    <property name="mapperLocations" value="classpath*:sample/config/mappers/**/*.xml" />
</bean>
```



# 创建SqlSessionFactory的方法（可选）

下面是创建SqlSessionFactory的方法。
```java
protected SqlSessionFactory buildSqlSessionFactory() throws Exception {  
  
    final Configuration targetConfiguration;  
  
    XMLConfigBuilder xmlConfigBuilder = null;  
    if (this.configuration != null) {  
	    targetConfiguration = this.configuration;  
	    if (targetConfiguration.getVariables() == null) {  
	       targetConfiguration.setVariables(this.configurationProperties);  
	    } else if (this.configurationProperties != null) {  
	        targetConfiguration.getVariables().putAll(this.configurationProperties);  
	    }  
    } else if (this.configLocation != null) {  
	    xmlConfigBuilder = new XMLConfigBuilder(this.configLocation.getInputStream(), null, this.configurationProperties);  
	    targetConfiguration = xmlConfigBuilder.getConfiguration();  
    } else {  
	    LOGGER.debug(  
        () -> "Property 'configuration' or 'configLocation' not specified, using default MyBatis Configuration");  
	    targetConfiguration = new Configuration();  
	    Optional.ofNullable(this.configurationProperties).ifPresent(targetConfiguration::setVariables);  
    }  
  
    Optional.ofNullable(this.objectFactory).ifPresent(targetConfiguration::setObjectFactory);  
    Optional.ofNullable(this.objectWrapperFactory).ifPresent(targetConfiguration::setObjectWrapperFactory);  
    Optional.ofNullable(this.vfs).ifPresent(targetConfiguration::setVfsImpl);  
  
    if (hasLength(this.typeAliasesPackage)) {  
	    scanClasses(this.typeAliasesPackage, this.typeAliasesSuperType).stream()  
        .filter(clazz -> !clazz.isAnonymousClass()).filter(clazz -> !clazz.isInterface())  
        .filter(clazz -> !clazz.isMemberClass()).forEach(targetConfiguration.getTypeAliasRegistry()::registerAlias);  
    }  
  
    if (!isEmpty(this.typeAliases)) {  
	    Stream.of(this.typeAliases).forEach(typeAlias -> {  
	        targetConfiguration.getTypeAliasRegistry().registerAlias(typeAlias);  
	        LOGGER.debug(() -> "Registered type alias: '" + typeAlias + "'");  
        });  
    }  
  
    if (!isEmpty(this.plugins)) {  
	    Stream.of(this.plugins).forEach(plugin -> {  
	        targetConfiguration.addInterceptor(plugin);  
	        LOGGER.debug(() -> "Registered plugin: '" + plugin + "'");  
        });  
    }  
  
  if (hasLength(this.typeHandlersPackage)) {  
    scanClasses(this.typeHandlersPackage, TypeHandler.class).stream().filter(clazz -> !clazz.isAnonymousClass())  
        .filter(clazz -> !clazz.isInterface()).filter(clazz -> !Modifier.isAbstract(clazz.getModifiers()))  
        .forEach(targetConfiguration.getTypeHandlerRegistry()::register);  
  }  
  
  if (!isEmpty(this.typeHandlers)) {  
    Stream.of(this.typeHandlers).forEach(typeHandler -> {  
      targetConfiguration.getTypeHandlerRegistry().register(typeHandler);  
      LOGGER.debug(() -> "Registered type handler: '" + typeHandler + "'");  
    });  
  }  
  
  targetConfiguration.setDefaultEnumTypeHandler(defaultEnumTypeHandler);  
  
  if (!isEmpty(this.scriptingLanguageDrivers)) {  
    Stream.of(this.scriptingLanguageDrivers).forEach(languageDriver -> {  
      targetConfiguration.getLanguageRegistry().register(languageDriver);  
      LOGGER.debug(() -> "Registered scripting language driver: '" + languageDriver + "'");  
    });  
  }  
    Optional.ofNullable(this.defaultScriptingLanguageDriver)  
            .ifPresent(targetConfiguration::setDefaultScriptingLanguage);  
  
    if (this.databaseIdProvider != null) {// fix #64 set databaseId before parse mapper xmls  
	    try {  
	        targetConfiguration.setDatabaseId(this.databaseIdProvider.getDatabaseId(this.dataSource));  
	    } catch (SQLException e) {  
	        throw new IOException("Failed getting a databaseId", e);  
	    }  
    }  
  
    Optional.ofNullable(this.cache).ifPresent(targetConfiguration::addCache);  
  
    if (xmlConfigBuilder != null) {  
	    try {  
	        xmlConfigBuilder.parse();  
	        LOGGER.debug(() -> "Parsed configuration file: '" + this.configLocation + "'");  
	    } catch (Exception ex) {  
	        throw new IOException("Failed to parse config resource: " + this.configLocation, ex);  
	    } finally {  
	        ErrorContext.instance().reset();  
	    }  
	}  
  
    targetConfiguration.setEnvironment(
	    new Environment(
		    this.environment,  
		    // this.transactionFactory:
		    // 1. 类型 TransactionFactory
		    // 2. 默认初值null, 提供set()方法 
	        this.transactionFactory == null ? new SpringManagedTransactionFactory() : this.transactionFactory,  
            this.dataSource
        )
    );  
  
  if (this.mapperLocations != null) {  
    if (this.mapperLocations.length == 0) {  
      LOGGER.warn(() -> "Property 'mapperLocations' was specified but matching resources are not found.");  
    } else {  
      for (Resource mapperLocation : this.mapperLocations) {  
        if (mapperLocation == null) {  
          continue;  
        }  
        try {  
          XMLMapperBuilder xmlMapperBuilder = new XMLMapperBuilder(mapperLocation.getInputStream(),  
              targetConfiguration, mapperLocation.toString(), targetConfiguration.getSqlFragments());  
          xmlMapperBuilder.parse();  
        } catch (Exception e) {  
          throw new IOException("Failed to parse mapping resource: '" + mapperLocation + "'", e);  
        } finally {  
          ErrorContext.instance().reset();  
        }  
        LOGGER.debug(() -> "Parsed mapper file: '" + mapperLocation + "'");  
      }  
    }  
  } else {  
    LOGGER.debug(() -> "Property 'mapperLocations' was not specified.");  
  }  

	// this.sqlSessionFactoryBuilder:
	// 1. 类型 SqlSessionFactoryBuilder
	// 2. 初始值 new SqlSessionFactoryBuilder(), 提供set()方法
    return this.sqlSessionFactoryBuilder.build(targetConfiguration);  
}
```

