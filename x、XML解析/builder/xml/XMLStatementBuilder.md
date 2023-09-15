# 主要方法

## parseStatementNode() : void

解析如下xml中的 `select` 标签。

```xml
<mapper namespace="org.example.dao.UserDao" >  
    <select id="selectUser" resultType="org.example.entity.User">  
        select * from user_tbl where id = #{id}  
    </select>  
</mapper>
```

select标签不常用属性：flushCache  useCache  resultOrdered


```java
public void parseStatementNode() {  
  String id = context.getStringAttribute("id");  
  String databaseId = context.getStringAttribute("databaseId"); // 先略过 
  
  if (!databaseIdMatchesCurrent(id, databaseId, this.requiredDatabaseId)) {  
    return;  
  }  
  
  String nodeName = context.getNode().getNodeName();  // se
  SqlCommandType sqlCommandType = SqlCommandType.valueOf(nodeName.toUpperCase(Locale.ENGLISH));  
  boolean isSelect = sqlCommandType == SqlCommandType.SELECT;  
  boolean flushCache = context.getBooleanAttribute("flushCache", !isSelect);  
  boolean useCache = context.getBooleanAttribute("useCache", isSelect);  
  boolean resultOrdered = context.getBooleanAttribute("resultOrdered", false);  
  
  // Include Fragments before parsing  
  XMLIncludeTransformer includeParser = new XMLIncludeTransformer(configuration, builderAssistant);  
  includeParser.applyIncludes(context.getNode());  
  
  String parameterType = context.getStringAttribute("parameterType");  
  Class<?> parameterTypeClass = resolveClass(parameterType);  
  
  String lang = context.getStringAttribute("lang");  
  LanguageDriver langDriver = getLanguageDriver(lang);  
  
  // Parse selectKey after includes and remove them.  
  processSelectKeyNodes(id, parameterTypeClass, langDriver);  
  
  // Parse the SQL (pre: <selectKey> and <include> were parsed and removed)  
  KeyGenerator keyGenerator;  
  String keyStatementId = id + SelectKeyGenerator.SELECT_KEY_SUFFIX;  
  keyStatementId = builderAssistant.applyCurrentNamespace(keyStatementId, true);  
  if (configuration.hasKeyGenerator(keyStatementId)) {  
    keyGenerator = configuration.getKeyGenerator(keyStatementId);  
  } else {  
    keyGenerator = context.getBooleanAttribute("useGeneratedKeys",  
        configuration.isUseGeneratedKeys() && SqlCommandType.INSERT.equals(sqlCommandType))  
        ? Jdbc3KeyGenerator.INSTANCE : NoKeyGenerator.INSTANCE;  
  }  
  
  SqlSource sqlSource = langDriver.createSqlSource(configuration, context, parameterTypeClass);  
  StatementType statementType = StatementType.valueOf(context.getStringAttribute("statementType", StatementType.PREPARED.toString()));  
  Integer fetchSize = context.getIntAttribute("fetchSize");  
  Integer timeout = context.getIntAttribute("timeout");  
  String parameterMap = context.getStringAttribute("parameterMap");  
  String resultType = context.getStringAttribute("resultType");  
  Class<?> resultTypeClass = resolveClass(resultType);  
  String resultMap = context.getStringAttribute("resultMap");  
  String resultSetType = context.getStringAttribute("resultSetType");  
  ResultSetType resultSetTypeEnum = resolveResultSetType(resultSetType);  
  if (resultSetTypeEnum == null) {  
    resultSetTypeEnum = configuration.getDefaultResultSetType();  
  }  
  String keyProperty = context.getStringAttribute("keyProperty");  
  String keyColumn = context.getStringAttribute("keyColumn");  
  String resultSets = context.getStringAttribute("resultSets");  
  
  builderAssistant.addMappedStatement(id, sqlSource, statementType, sqlCommandType,  
      fetchSize, timeout, parameterMap, parameterTypeClass, resultMap, resultTypeClass,  
      resultSetTypeEnum, flushCache, useCache, resultOrdered,  
      keyGenerator, keyProperty, keyColumn, databaseId, langDriver, resultSets);  
}
```