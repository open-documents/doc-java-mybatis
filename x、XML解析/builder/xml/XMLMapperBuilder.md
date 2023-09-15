# 一、主要方法
## 解析  /mapper  标签

**parse() : void**

```java
public void parse() {  
  if (!configuration.isResourceLoaded(resource)) {  
    configurationElement(parser.evalNode("/mapper"));  
    configuration.addLoadedResource(resource);
    bindMapperForNamespace();  
  }  
  parsePendingResultMaps();  
  parsePendingCacheRefs();  
  parsePendingStatements();  
}
```
-- --
<font color=d4de71>相关方法</font>

### configurationElement(XNode context) : void
```java
private void configurationElement(XNode context) {  
    String namespace = context.getStringAttribute("namespace");  
    builderAssistant.setCurrentNamespace(namespace);  
    cacheRefElement(context.evalNode("cache-ref"));  
    cacheElement(context.evalNode("cache"));  
    parameterMapElement(context.evalNodes("/mapper/parameterMap"));  
    resultMapElements(context.evalNodes("/mapper/resultMap"));  
    sqlElement(context.evalNodes("/mapper/sql"));  
    buildStatementFromContext(context.evalNodes("select|insert|update|delete"));  
}
```

# 二、关键方法
## 解析  /mapper/parameterMap  标签

```xml
<parameterMap id="" type="">  
    <parameter property="" javaType="" jdbcType="" resultMap="" mode=" " typeHandler="" scale=""/>  
</parameterMap>
```

**parameterMapElement(List< XNode > list) : void**

```java
private void parameterMapElement(List<XNode> list) {  
	// 遍历每一个parameterMap
	for (XNode parameterMapNode : list) {
		String id = parameterMapNode.getStringAttribute("id");  
		String type = parameterMapNode.getStringAttribute("type");  
		Class<?> parameterClass = resolveClass(type);
		List<XNode> parameterNodes = parameterMapNode.evalNodes("parameter");  
		List<ParameterMapping> parameterMappings = new ArrayList<>();
		// 遍历每一个parameter
		for (XNode parameterNode : parameterNodes) {  
			String property = parameterNode.getStringAttribute("property");  
			String javaType = parameterNode.getStringAttribute("javaType");  
			String jdbcType = parameterNode.getStringAttribute("jdbcType");  
			String resultMap = parameterNode.getStringAttribute("resultMap");  
			String mode = parameterNode.getStringAttribute("mode");  
			String typeHandler = parameterNode.getStringAttribute("typeHandler");  
			Integer numericScale = parameterNode.getIntAttribute("numericScale");  
			ParameterMode modeEnum = resolveParameterMode(mode);  
			Class<?> javaTypeClass = resolveClass(javaType);  
			JdbcType jdbcTypeEnum = resolveJdbcType(jdbcType);  
			Class<? extends TypeHandler<?>> typeHandlerClass = resolveClass(typeHandler);  
			ParameterMapping parameterMapping = builderAssistant.buildParameterMapping(parameterClass, property, javaTypeClass, jdbcTypeEnum, resultMap, modeEnum, typeHandlerClass, numericScale);  
			parameterMappings.add(parameterMapping);  
		}  
		builderAssistant.addParameterMap(id, parameterClass, parameterMappings);  
	}  
}
```

## 解析  /mapper/resultMap 标签

```java
private void resultMapElements(List<XNode> list) {
	// // 遍历每一个resultMap
    for (XNode resultMapNode : list) {  
		//解析resultMap标签的属性
		Discriminator discriminator = null;  
		List<ResultMapping> resultMappings = new ArrayList<>(additionalResultMappings);  
		List<XNode> resultChildren = resultMapNode.getChildren();  
		for (XNode resultChild : resultChildren) {  
			//解析id、constructor、discriminator标签
		}
	}  
}  
	
	String id = resultMapNode.getStringAttribute("id",  
	        resultMapNode.getValueBasedIdentifier());  
	String extend = resultMapNode.getStringAttribute("extends");  
	Boolean autoMapping = resultMapNode.getBooleanAttribute("autoMapping");  
	ResultMapResolver resultMapResolver = new ResultMapResolver(builderAssistant, id, typeClass, extend, discriminator, resultMappings, autoMapping);  
	try {  
	  return resultMapResolver.resolve();  
	} catch (IncompleteElementException e) {  
	  configuration.addIncompleteResultMap(resultMapResolver);  
	  throw e;  
	}

}

```
-- --
**1.** 解析  `resultMap`  标签的属性
```xml
<resultMap id="" type=""> </resultMap>
<resultMap id="" oftype=""> </resultMap>
<resultMap id="" resultType=""> </resultMap>
<resultMap id="" javaType=""> </resultMap>
```
```java
// type、ofType、resultType、javaType四个属性等价
String type = resultMapNode.getStringAttribute("type",  
	          resultMapNode.getStringAttribute("ofType",  
	          resultMapNode.getStringAttribute("resultType",  
	          resultMapNode.getStringAttribute("javaType"))));
Class<?> typeClass = resolveClass(type); 
if (typeClass == null) {
    typeClass = inheritEnclosingType(resultMapNode, enclosingType);  
}
```
-- --
**2.** 解析  `id、constructor、discriminator`  标签
```xml
<id property="" column="" javaType="" jdbcType="" typeHandler=""/>
```
```java
// id、constructor、discriminator只能同时存在一个
if ("constructor".equals(resultChild.getName())) { // 解析constructor标签
    processConstructorElement(resultChild, typeClass, resultMappings);  
} else if ("discriminator".equals(resultChild.getName())) { // 解析discriminator标签 
    discriminator = processDiscriminatorElement(resultChild, typeClass, resultMappings);  
} else {   // 解析id标签
    List<ResultFlag> flags = new ArrayList<>();
    if ("id".equals(resultChild.getName())) {
      flags.add(ResultFlag.ID);  
    }  
    resultMappings.add(buildResultMappingFromContext(resultChild, typeClass, flags));  
}
```


**1.2  buildStatementFromContext(List< XNode > list) : void**

```java
private void buildStatementFromContext(List<XNode> list) {  
  if (configuration.getDatabaseId() != null) {  
    buildStatementFromContext(list, configuration.getDatabaseId());  
  }  
  buildStatementFromContext(list, null);  
}
```

相关方法

buildStatementFromContext(List< XNode > list, String requiredDatabaseId) : void
```java
  private void buildStatementFromContext(List<XNode> list, String requiredDatabaseId) {  
  for (XNode context : list) {  
    final XMLStatementBuilder statementParser = new XMLStatementBuilder(configuration, builderAssistant, context, requiredDatabaseId);  
    try {  
      statementParser.parseStatementNode();  
    } catch (IncompleteElementException e) {  
      configuration.addIncompleteStatement(statementParser);  
    }  
  }  
}
```

# 、功能方法

## resolveParameterJavaType()

参数：
- resultType : Class< ? > 
- String : property
- Class< ? > : javaType
- JdbcType : jdbcType
返回值 : 
- 

```java
private Class<?> resolveParameterJavaType(Class<?> resultType, String property, Class<?> javaType, JdbcType jdbcType) {
  if (javaType == null) {  
    if (JdbcType.CURSOR.equals(jdbcType)) {  
      javaType = java.sql.ResultSet.class;  
    } else if (Map.class.isAssignableFrom(resultType)) {  
      javaType = Object.class;  
    } else {  
      MetaClass metaResultType = MetaClass.forClass(resultType, configuration.getReflectorFactory());  
      javaType = metaResultType.getGetterType(property);  
    }  
  }  
  if (javaType == null) {  
    javaType = Object.class;  
  }  
  return javaType;  
}
```