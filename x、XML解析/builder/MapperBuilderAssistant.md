```java
public class MapperBuilderAssistant extends BaseBuilder
```
# 属性
```java
private String currentNamespace;
```
# 属性方法
```java
// currentNamespace
public String getCurrentNamespace()
// 设置传入参数不为空且必须和当前已有的currentNamespace相同
public void setCurrentNamespace(String currentNamespace)
```
# 一、主要方法

buildParameterMapping() : ParameterMapping

```java
public ParameterMapping buildParameterMapping(  
    Class<?> parameterType,  
    String property,  
    Class<?> javaType,  
    JdbcType jdbcType,  
    String resultMap,  
    ParameterMode parameterMode,  
    Class<? extends TypeHandler<?>> typeHandler,  
    Integer numericScale) {  

  resultMap = applyCurrentNamespace(resultMap, true);  
  
  // Class parameterType = parameterMapBuilder.type();  
  Class<?> javaTypeClass = resolveParameterJavaType(parameterType, property, javaType, jdbcType);  
  TypeHandler<?> typeHandlerInstance = resolveTypeHandler(javaTypeClass, typeHandler);  
  
  return new ParameterMapping.Builder(configuration, property, javaTypeClass)  
      .jdbcType(jdbcType)  
      .resultMapId(resultMap)  
      .mode(parameterMode)  
      .numericScale(numericScale)  
      .typeHandler(typeHandlerInstance)  
      .build();  
}
```