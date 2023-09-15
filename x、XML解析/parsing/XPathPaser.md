# class

## 1. XPathParser

### 属性
```java
public class XPathParser {
	// 对XPath对象的简单封装，XPathParser的方法主要使用XPath的方法来完成
	private XPath xpath;  
	
}
```

### public方法

```java
public class XPathParser {
	// 调用XNode evalNode(Object root, String expression)，即后面这个方法
	public XNode evalNode(String expression);
	// 调用private Object evaluate(String expression, Object root, QName returnType)
	// 返回XNode对象
	public XNode evalNode(Object root, String expression);
}

```

### 非public方法

```java
public class XPathParser {
	// 调用XPath对象的evaluate(String expression, Object root, QName returnType)方法
	private Object evaluate(String expression, Object root, QName returnType){
		return xpath.evaluate(expression, root, returnType);
	}

}
```

## 2. 