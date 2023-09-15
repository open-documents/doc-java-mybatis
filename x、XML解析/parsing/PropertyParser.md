## 属性

```java
private static final String KEY_PREFIX = "org.apache.ibatis.parsing.PropertyParser.";

public static final String KEY_ENABLE_DEFAULT_VALUE = KEY_PREFIX + "enable-default-value";
public static final String KEY_DEFAULT_VALUE_SEPARATOR = KEY_PREFIX + "default-value-separator";

private static final String ENABLE_DEFAULT_VALUE = "false";  
private static final String DEFAULT_VALUE_SEPARATOR = ":";
```

## 关键方法

## parse(String string, Properties variables) : String



```java
public static String parse(String string, Properties variables) {  
  VariableTokenHandler handler = new VariableTokenHandler(variables);  
  GenericTokenParser parser = new GenericTokenParser("${", "}", handler);  
  return parser.parse(string);  
}
```

## 内部类