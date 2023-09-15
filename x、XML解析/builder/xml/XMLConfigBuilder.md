<font color=44cf57>功能</font>：通过 `parse()` 解析XML，将xml转换成Configure对象

<font color=44cf57>代码</font>：

```java
public class XMLConfigBuilder extends BaseBuilder
```

# 属性

```java
private final XPathParser parser;  
protected final Set<String> loadedResources = new HashSet<>();

```

# 主要方法

##  parse() : Configuration

```java
public Configuration parse() {
	...
	parseConfiguration(parser.evalNode("/configuration"));  
	return configuration;
}
```

## 1. parseConfiguration(XNode root) : void

```java
private void parseConfiguration(XNode root){
	propertiesElement(root.evalNode("properties"));  
	Properties settings = settingsAsProperties(root.evalNode("settings"));  
	loadCustomVfs(settings);  
	loadCustomLogImpl(settings);  
	typeAliasesElement(root.evalNode("typeAliases"));  
	pluginElement(root.evalNode("plugins"));  
	objectFactoryElement(root.evalNode("objectFactory"));  
	objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));  
	reflectorFactoryElement(root.evalNode("reflectorFactory"));  
	settingsElement(settings);  
	// read it after objectFactory and objectWrapperFactory issue #631  
	environmentsElement(root.evalNode("environments"));  
	databaseIdProviderElement(root.evalNode("databaseIdProvider"));  
	typeHandlerElement(root.evalNode("typeHandlers"));  
	mapperElement(root.evalNode("mappers"));
}
```

## 2. propertiesElement(XNode context) : void

```java
private void propertiesElement(XNode context) throws Exception {  
	if (context != null) {  
		Properties defaults = context.getChildrenAsProperties();  
		String resource = context.getStringAttribute("resource");  
		String url = context.getStringAttribute("url");  
		if (resource != null && url != null) {  
			throw new BuilderException("The properties element cannot specify both a URL and a resource based property file reference.  Please specify one or the other.");  
    }  
    if (resource != null) {  
	    defaults.putAll(Resources.getResourceAsProperties(resource));  
    } else if (url != null) {  
	    defaults.putAll(Resources.getUrlAsProperties(url));  
    }  
    Properties vars = configuration.getVariables();  
    if (vars != null) {  
	    defaults.putAll(vars);  
    }  
    parser.setVariables(defaults);  
    configuration.setVariables(defaults);  
  }  
}

```


## 5. typeAliasesElement(XNode parent) : void

```xml
<typeAliases>
   <package name="org.example.entity"/>  
   <typeAlias type="" alias="" />
   <!-- 这样注册就必须在类名上使用Alias注解-->
   <typeAlias type=""/>
</typeAliases>  
```

下面的代码完成


```java
// parent = root.evalNode("typeAliases")
private void typeAliasesElement(XNode parent) {  
	for (XNode child : parent.getChildren()) {  
		if ("package".equals(child.getName())) {  
			String typeAliasPackage = child.getStringAttribute("name");  
			configuration.getTypeAliasRegistry().registerAliases(typeAliasPackage);  
		}
		else {  
			String alias = child.getStringAttribute("alias");  
			String type = child.getStringAttribute("type");  
			try {  
				Class<?> clazz = Resources.classForName(type);  
				if (alias == null) {  
					typeAliasRegistry.registerAlias(clazz);  
				} else {  
					typeAliasRegistry.registerAlias(alias, clazz);  
				}  
			} catch (ClassNotFoundException e) {  
				throw new BuilderException(...);  
			}  
		}
	}  
}
```


## 6. pluginElement(XNode parent) : void 

```java
<plugins>  
    <plugin interceptor="">  
        <property name="" value=""/>  
        <property name="" value=""/>  
    </plugin>
</plugins>
```

```java
// parent = root.evalNode("plugins")
private void pluginElement(XNode parent) throws Exception {  
	for (XNode child : parent.getChildren()) {  
		String interceptor = child.getStringAttribute("interceptor");  
		Properties properties = child.getChildrenAsProperties();  
		Interceptor interceptorInstance = (Interceptor) resolveClass(interceptor).getDeclaredConstructor().newInstance();  
		interceptorInstance.setProperties(properties);  
		configuration.addInterceptor(interceptorInstance);  
	}  
}
```

## 7. objectFactoryElement(XNode context) : void

```xml
<objectFactory type="">  
    <property name="" value=""/>  
    <property name="" value=""/>  
</objectFactory>
```

```java
// context = root.evalNode("objectFactory")
private void objectFactoryElement(XNode context) throws Exception {  
	String type = context.getStringAttribute("type");  
	Properties properties = context.getChildrenAsProperties();  
	ObjectFactory factory = (ObjectFactory) resolveClass(type).getDeclaredConstructor().newInstance();  
	factory.setProperties(properties);  
	configuration.setObjectFactory(factory);  
}
```

## 8. objectWrapperFactoryElement(XNode context) : void

```xml
<objectWrapperFactory type=""/>
```

```java
// context = root.evalNode("objectWrapperFactory")
private void objectWrapperFactoryElement(XNode context) throws Exception {  
    String type = context.getStringAttribute("type");  
    ObjectWrapperFactory factory = (ObjectWrapperFactory) resolveClass(type).getDeclaredConstructor().newInstance();  
    configuration.setObjectWrapperFactory(factory);  
}
```

## 9. reflectorFactoryElement(XNode context) : void

```xml
<reflectorFactory type=""/>
```

```java
private void reflectorFactoryElement(XNode context) throws Exception {  
    String type = context.getStringAttribute("type");  
    ReflectorFactory factory = (ReflectorFactory) resolveClass(type).getDeclaredConstructor().newInstance();  
    configuration.setReflectorFactory(factory);  
}
```

## 10. settingsElement(Properties props) : void


```java
private void settingsElement(Properties props) {  
  configuration.setAutoMappingBehavior(AutoMappingBehavior.valueOf(props.getProperty("autoMappingBehavior", "PARTIAL")));  
  configuration.setAutoMappingUnknownColumnBehavior(AutoMappingUnknownColumnBehavior.valueOf(props.getProperty("autoMappingUnknownColumnBehavior", "NONE")));  
  configuration.setCacheEnabled(booleanValueOf(props.getProperty("cacheEnabled"), true));  
  configuration.setProxyFactory((ProxyFactory) createInstance(props.getProperty("proxyFactory")));  
  configuration.setLazyLoadingEnabled(booleanValueOf(props.getProperty("lazyLoadingEnabled"), false));  
  configuration.setAggressiveLazyLoading(booleanValueOf(props.getProperty("aggressiveLazyLoading"), false));  
  configuration.setMultipleResultSetsEnabled(booleanValueOf(props.getProperty("multipleResultSetsEnabled"), true));  
  configuration.setUseColumnLabel(booleanValueOf(props.getProperty("useColumnLabel"), true));  
  configuration.setUseGeneratedKeys(booleanValueOf(props.getProperty("useGeneratedKeys"), false));  
  configuration.setDefaultExecutorType(ExecutorType.valueOf(props.getProperty("defaultExecutorType", "SIMPLE")));  
  configuration.setDefaultStatementTimeout(integerValueOf(props.getProperty("defaultStatementTimeout"), null));  
  configuration.setDefaultFetchSize(integerValueOf(props.getProperty("defaultFetchSize"), null));  
  configuration.setDefaultResultSetType(resolveResultSetType(props.getProperty("defaultResultSetType")));  
  configuration.setMapUnderscoreToCamelCase(booleanValueOf(props.getProperty("mapUnderscoreToCamelCase"), false));  
  configuration.setSafeRowBoundsEnabled(booleanValueOf(props.getProperty("safeRowBoundsEnabled"), false));  
  configuration.setLocalCacheScope(LocalCacheScope.valueOf(props.getProperty("localCacheScope", "SESSION")));  
  configuration.setJdbcTypeForNull(JdbcType.valueOf(props.getProperty("jdbcTypeForNull", "OTHER")));  
  configuration.setLazyLoadTriggerMethods(stringSetValueOf(props.getProperty("lazyLoadTriggerMethods"), "equals,clone,hashCode,toString"));  
  configuration.setSafeResultHandlerEnabled(booleanValueOf(props.getProperty("safeResultHandlerEnabled"), true));  
  configuration.setDefaultScriptingLanguage(resolveClass(props.getProperty("defaultScriptingLanguage")));  
  configuration.setDefaultEnumTypeHandler(resolveClass(props.getProperty("defaultEnumTypeHandler")));  
  configuration.setCallSettersOnNulls(booleanValueOf(props.getProperty("callSettersOnNulls"), false));  
  configuration.setUseActualParamName(booleanValueOf(props.getProperty("useActualParamName"), true));  
  configuration.setReturnInstanceForEmptyRow(booleanValueOf(props.getProperty("returnInstanceForEmptyRow"), false));  
  configuration.setLogPrefix(props.getProperty("logPrefix"));  
  configuration.setConfigurationFactory(resolveClass(props.getProperty("configurationFactory")));  
  configuration.setShrinkWhitespacesInSql(booleanValueOf(props.getProperty("shrinkWhitespacesInSql"), false));  
  configuration.setDefaultSqlProviderType(resolveClass(props.getProperty("defaultSqlProviderType")));  
  configuration.setNullableOnForEach(booleanValueOf(props.getProperty("nullableOnForEach"), false));  
}
```

## 11. environmentsElement(XNode context)

```xml
<environments default="development">  
    <!-- 环境变量：支持多套环境变量，例如开发环境、生产环境 -->  
    <environment id="development">  
        <!-- 事务管理器：默认JDBC -->  
        <transactionManager type="JDBC">  
            <property name="" value=""/>  
            <property name="" value=""/>  
        </transactionManager>        <!-- 数据源：使用连接池，并加载mysql驱动连接数据库 -->  
        <dataSource type="POOLED">  
            <property name="driver" value="com.mysql.cj.jdbc.Driver"/>  
            <property name="url" value="jdbc:mysql://localhost:3306/mybatis_learning"/>  
            <property name="username" value="root"/>  
            <property name="password" value="xcxwrka1314"/>  
        </dataSource>
    </environment>
</environments>
```

```java
// context = root.evalNode("environments")
private void environmentsElement(XNode context) throws Exception {  
	// String enviroment是这个类中的一个属性
	if (environment == null) {  
		environment = context.getStringAttribute("default");  
	}  
	for (XNode child : context.getChildren()) {  
		String id = child.getStringAttribute("id");  
		if (isSpecifiedEnvironment(id)) {  
			TransactionFactory txFactory = transactionManagerElement(child.evalNode("transactionManager"));  
			DataSourceFactory dsFactory = dataSourceElement(child.evalNode("dataSource"));  
			DataSource dataSource = dsFactory.getDataSource();  
			Environment.Builder environmentBuilder = new Environment.Builder(id)  
			    .transactionFactory(txFactory)
			    .dataSource(dataSource);  
			configuration.setEnvironment(environmentBuilder.build());  
			break;
		}  
	}  
}

```

## 12. databaseIdProviderElement(XNode context) : void

```xml
<databaseIdProvider type="">  
    <property name="" value=""/>  
    <property name="" value=""/>  
</databaseIdProvider>
```

```java
private void databaseIdProviderElement(XNode context) throws Exception {  
	DatabaseIdProvider databaseIdProvider = null;  
	if (context != null) {  
		String type = context.getStringAttribute("type");  
		// awful patch to keep backward compatibility  
		if ("VENDOR".equals(type)) {  
		  type = "DB_VENDOR";  
		}  
		Properties properties = context.getChildrenAsProperties();  
		databaseIdProvider = (DatabaseIdProvider) resolveClass(type).getDeclaredConstructor().newInstance();  
		databaseIdProvider.setProperties(properties);  
	}  
	Environment environment = configuration.getEnvironment();  
	if (environment != null && databaseIdProvider != null) {  
		String databaseId = databaseIdProvider.getDatabaseId(environment.getDataSource());  
		configuration.setDatabaseId(databaseId);  
	}  
}
```

## 13. typeHandlerElement(XNode parent) : void

```xml
<typeHandlers>  
    <package name=""/>  
    <package name=""/>  
    <typeHandler handler="" jdbcType="" javaType=""/>  
    <typeHandler handler="" jdbcType="" javaType=""/>  
</typeHandlers>
```

```java
private void typeHandlerElement(XNode parent) {  
	for (XNode child : parent.getChildren()) {  
		if ("package".equals(child.getName())) {  
			String typeHandlerPackage = child.getStringAttribute("name");  
			typeHandlerRegistry.register(typeHandlerPackage);  
		} else {  
			String javaTypeName = child.getStringAttribute("javaType");  
			String jdbcTypeName = child.getStringAttribute("jdbcType");  
			String handlerTypeName = child.getStringAttribute("handler");  
			Class<?> javaTypeClass = resolveClass(javaTypeName);  
			JdbcType jdbcType = resolveJdbcType(jdbcTypeName);  
			Class<?> typeHandlerClass = resolveClass(handlerTypeName);  
			if (javaTypeClass != null) {  
				if (jdbcType == null) {  
					typeHandlerRegistry.register(javaTypeClass, typeHandlerClass);  
				} else {  
					typeHandlerRegistry.register(javaTypeClass, jdbcType, typeHandlerClass);  
				}  
			} 
			else {  
				typeHandlerRegistry.register(typeHandlerClass);  
			}  
		}  
	}  
}
```

## 14. 解析mapper标签

```xml
<mappers>  
    <mapper resource="UserDao.xml"/>  
    <mapper url=""/>  
    <mapper class=""/>  
</mappers>
```
-- --
**主要方法：mapperElement(XNode parent) : void**

```java
private void mapperElement(XNode parent) throws Exception {  
	for (XNode child : parent.getChildren()) {  
		if ("package".equals(child.getName())) {  
		  String mapperPackage = child.getStringAttribute("name");  
		  configuration.addMappers(mapperPackage);  
		} else {  
		  String resource = child.getStringAttribute("resource");  
		  String url = child.getStringAttribute("url");  
		  String mapperClass = child.getStringAttribute("class");  
		  if (resource != null && url == null && mapperClass == null) {  
		    ErrorContext.instance().resource(resource);  
		    try(InputStream inputStream = Resources.getResourceAsStream(resource)) {  
		      XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, resource, configuration.getSqlFragments());  
		      mapperParser.parse();  
		    }  
		  } else if (resource == null && url != null && mapperClass == null) {  
		    ErrorContext.instance().resource(url);  
		    try(InputStream inputStream = Resources.getUrlAsStream(url)){  
		      XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, url, configuration.getSqlFragments());  
		      mapperParser.parse();  
		    }  
		  } else if (resource == null && url == null && mapperClass != null) {  
		    Class<?> mapperInterface = Resources.classForName(mapperClass);  
		    configuration.addMapper(mapperInterface);  
		  } else {  
		    throw new BuilderException("A mapper element may only specify a url, resource or class, but not more than one.");  
		  }  
		}  
	}  
}
```


# 补充方法

## 与environmentsElement有关的方法

### transactionManagerElement(XNode context) : TransactionFactory

```java
// context = child.evalNode("transactionManager")
private TransactionFactory transactionManagerElement(XNode context) throws Exception {  
	if (context != null) {  
		String type = context.getStringAttribute("type");  
		Properties props = context.getChildrenAsProperties();  
		TransactionFactory factory = (TransactionFactory) resolveClass(type).getDeclaredConstructor().newInstance();  
		factory.setProperties(props);  
		return factory;
	}  
	throw new BuilderException("Environment declaration requires a TransactionFactory.");  
}
```

### dataSourceElement(XNode context) : DataSourceFactory

```java
private DataSourceFactory dataSourceElement(XNode context) throws Exception {  
	if (context != null) {  
		String type = context.getStringAttribute("type");  
		Properties props = context.getChildrenAsProperties();  
		DataSourceFactory factory = (DataSourceFactory) resolveClass(type).getDeclaredConstructor().newInstance();  
		factory.setProperties(props);  
		return factory;
	}  
	throw new BuilderException("Environment declaration requires a DataSourceFactory.");  
}
```