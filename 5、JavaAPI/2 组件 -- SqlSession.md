
接口对应的mapper通过 SqlSession#getMapper() 获取。下面来看具体实现。
```java
/* ------------------------------ DefaultSqlSession ------------------------------*/
public <T> T getMapper(Class<T> type) {  
    return configuration.getMapper(type, this);  
}
/* -------------------------------- Configuration --------------------------------*/
public <T> T getMapper(Class<T> type, SqlSession sqlSession) {  
    return mapperRegistry.getMapper(type, sqlSession);  
}
/* ------------------------------- MapperRegistry ---------------------------------*/
public <T> T getMapper(Class<T> type, SqlSession sqlSession) {  
	// knownMappers: 类型Map<Class<?>, MapperProxyFactory<?>>
    final MapperProxyFactory<T> mapperProxyFactory = (MapperProxyFactory<T>) knownMappers.get(type);  
    if (mapperProxyFactory == null) {  
	    throw new BindingException("Type " + type + " is not known to the MapperRegistry.");  
    }  
	try {  
	    return mapperProxyFactory.newInstance(sqlSession);  
    } catch (Exception e) {  
	    throw new BindingException("Error getting mapper instance. Cause: " + e, e);  
    }  
}
```

下面单独看如何从MapperProxyFactory中获取接口对应的Mapper。
```java
/* ------------------------------- MapperProxyFactory --------------------------------- */
public T newInstance(SqlSession sqlSession) {  
	// 1. mapperInterface
	// 2. methodCache: 
	// - 类型 Map<Method, MapperMethodInvoker>
    final MapperProxy<T> mapperProxy = new MapperProxy<>(sqlSession, mapperInterface, methodCache);  
    return newInstance(mapperProxy);  
}
/* ------------------------------- MapperProxyFactory --------------------------------- */
protected T newInstance(MapperProxy<T> mapperProxy) {  
	// 使用jdk动态代理生成代理对象
    return (T) Proxy.newProxyInstance(mapperInterface.getClassLoader(), new Class[] { mapperInterface }, mapperProxy);  
}
```

## 代理逻辑

下面来看MapperProxy提供的代理逻辑。
```java
/* ----------------------------------- MapperProxy ----------------------------------- */
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {  
    try {  
	    if (Object.class.equals(method.getDeclaringClass())) {  
	        return method.invoke(this, args);  
        }  
	    return cachedInvoker(method).invoke(proxy, method, args, sqlSession);  
    } catch (Throwable t) {  
	    throw ExceptionUtil.unwrapThrowable(t);  
    }  
}
```
下面来看
```java
/* ----------------------------------- MapperProxy ----------------------------------- */
private MapperMethodInvoker cachedInvoker(Method method) throws Throwable {  
    try {  
	    return MapUtil.computeIfAbsent(
	    
		    methodCache, method, m -> {  
		    
            if (!m.isDefault()) {  
	            return new PlainMethodInvoker(new MapperMethod(mapperInterface, method, sqlSession.getConfiguration())); 
	        }  
            try {  
	            if (privateLookupInMethod == null) {  
		            return new DefaultMethodInvoker(getMethodHandleJava8(method));  
		        } else {
		            return new DefaultMethodInvoker(getMethodHandleJava9(method));  
                }  
		    } catch (...) {
		        throw new RuntimeException(e);  
            }  
        });  
    } catch (RuntimeException re) {  
	    Throwable cause = re.getCause();  
        throw cause == null ? re : cause;  
    }  
}
```