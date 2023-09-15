# 前言

在阅读这部分时，需要对  `org.w3c.dom.Node`  接口熟悉。

# 属性

```java
public class XNode {
	private final Node node;  
	private final String name;  // = node.getNodeName()
	private final String body;  // = parseBody(node)

	private final Properties attributes;  
	private final Properties variables;  

	private final XPathParser xpathParser;
}
```

# 构造方法

```java
public class XNode {
	public XNode(XPathParser xpathParser, Node node, Properties variables) {  
		this.xpathParser = xpathParser;  
		this.node = node;  
		this.name = node.getNodeName();  
		this.variables = variables;  
		this.attributes = parseAttributes(node);  // parseAttributes(Node)
		this.body = parseBody(node);  
	}
}
```

parseAttributes


# 主要方法

## getBooleanAttribute : Boolean

**1.  getBooleanAttribute(String name) : Boolean**
```java
public Boolean getBooleanAttribute(String name) {  
  return getBooleanAttribute(name, null);  
}
```
**2.  getBooleanAttribute(String name, Boolean def) : Boolean**
```java
public Boolean getBooleanAttribute(String name, Boolean def) {  
  String value = attributes.getProperty(name);  
  return value == null ? def : Boolean.valueOf(value);  
}
```
-- --
## getIntAttribute : Integer

**1.  getIntAttribute(String name) : Integer**
```java
public Integer getIntAttribute(String name) {  
  return getIntAttribute(name, null);  
}
```
**2.  getIntAttribute(String name, Integer def) : Integer**
```java
public Integer getIntAttribute(String name, Integer def) {  
  String value = attributes.getProperty(name);  
  return value == null ? def : Integer.valueOf(value);  
}
```
-- --
## getLongAttribute : Long

**1.  getLongAttribute(String name) : Long**
```java
public Long getLongAttribute(String name) {  
  return getLongAttribute(name, null);  
}
```
**2.  getLongAttribute(String name, Long def) : Long**
```java
public Long getLongAttribute(String name, Long def) {  
  String value = attributes.getProperty(name);  
  return value == null ? def : Long.valueOf(value);  
}
```
-- --
## getFloatAttribute : Float

**1.  getFloatAttribute(String name) : Float**
```java
public Float getFloatAttribute(String name) {  
  return getFloatAttribute(name, null);  
}
```
**2.  getFloatAttribute(String name, Boolean def) : Float**
```java
public Float getFloatAttribute(String name, Float def) {  
  String value = attributes.getProperty(name);  
  return value == null ? def : Float.valueOf(value);  
}
```
-- --
## getDoubleAttribute : Double

**1.  getDoubleAttribute(String name) : Boolean**
```java
public Double getDoubleAttribute(String name) {  
  return getDoubleAttribute(name, null);  
}
```
**2.  getDoubleAttribute(String name, Boolean def) : Boolean**
```java
public Double getDoubleAttribute(String name, Double def) {  
  String value = attributes.getProperty(name);  
  return value == null ? def : Double.valueOf(value);  
}
```
-- --
## getStringAttribute : String

**1.  getStringAttribute(String name) : String**
```java
public String getStringAttribute(String name) {  
  return getStringAttribute(name, (String) null);  
}
```
**2.  getStringAttribute(String name, String def) : String**
```java
public String getStringAttribute(String name, String def) {  
  String value = attributes.getProperty(name);  
  return value == null ? def : value;  
}
```
**3.  getStringAttribute(String name, Supplier< String> defSupplier) : String
```java
public String getStringAttribute(String name, Supplier<String> defSupplier) {  
  String value = attributes.getProperty(name);  
  return value == null ? defSupplier.get() : value;  
}
```




## getChildren() : List< XNode >

描述：仅获得`Node.ELEMENT_NODE`类型的节点，并对其进行封装得到XNode


```java
public List<XNode> getChildren() {  
	List<XNode> children = new ArrayList<>();  
	NodeList nodeList = node.getChildNodes();  
	if (nodeList != null) {  
	  for (int i = 0, n = nodeList.getLength(); i < n; i++) {  
	    Node node = nodeList.item(i);  
	    if (node.getNodeType() == Node.ELEMENT_NODE) {  
	      children.add(new XNode(xpathParser, node, variables));  
	    }  
	  }  
	}  
	return children;  
}

```

## getChildrenAsProperties() : Properties





## getStringAttribute() : String

```java
public String getStringAttribute(String name){
	return getStringAttribute(name, (String) null);
}

public String getStringAttribute(String name, String def){
	String value = attributes.getProperty(name);  
	return value == null ? def : value;
}
```


