
参考文档：
- 英文参考文档 -- https://mybatis.org/spring/sqlsession.html

在Mybatis中，使用 SqlSession的实现类DefaultSqlSession。
在Mybatis-Spring中，使用 SqlSessionTemplate 。

DefaultSqlSession和SqlSessionTemplate的对比。
- 线程安全：



SqlSessionTemplate实现了SqlSession接口，内部有SqlSession代理。
```java
public class SqlSessionTemplate implements SqlSession, DisposableBean {
	private final SqlSession sqlSessionProxy;
}	
```


# SqlSessionTemplate的创建

SqlSessionTemplate提供3个构造方法。其中SqlSessionFactory是必需的参数。
```java
public SqlSessionTemplate(SqlSessionFactory sqlSessionFactory) {  
    this(
	    sqlSessionFactory, 
	    sqlSessionFactory.getConfiguration().getDefaultExecutorType()
	);  
}
public SqlSessionTemplate(SqlSessionFactory sqlSessionFactory, ExecutorType executorType) {  
    this(
	    sqlSessionFactory, 
	    executorType,  
        new MyBatisExceptionTranslator(sqlSessionFactory.getConfiguration().getEnvironment().getDataSource(), true)
    );  
}
public SqlSessionTemplate(SqlSessionFactory sqlSessionFactory, ExecutorType executorType, PersistenceExceptionTranslator exceptionTranslator) {
	... // 全部创建逻辑
}
```
因此可以通过xml或@Bean的方式来向容器中注册SqlSessionTemplate组件。
1）xml方式
```xml
<bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
    <constructor-arg index="0" ref="sqlSessionFactory" />
</bean>
```
2）@Bean方式
```java
@Configuration
public class MyBatisConfig {
    @Bean
    public SqlSessionTemplate sqlSession() throws Exception {
	    return new SqlSessionTemplate(sqlSessionFactory());
    }
}
```

# 使用SqlSession

通过上面的方式向容器中注册SqlSessionTemplate组件后，可以通过xml或者自动装配的方式向DAO Bean中注入SqlSessionTemplate对象。
```java
public class UserDaoImpl implements UserDao {
    private SqlSession sqlSession;  // 通过上面的方式注入的是SqlSessionTemplate而非DefaultSqlSession
    public void setSqlSession(SqlSession sqlSession) {
	    this.sqlSession = sqlSession;
    }
    public User getUser(String userId) {  // 
	    return sqlSession.selectOne("org.mybatis.spring.sample.mapper.UserMapper.getUser", userId);
    }
}
```
通过xml向方式注入SqlSessionTemplate对象。
```xml
<bean id="userDao" class="org.mybatis.spring.sample.dao.UserDaoImpl">
    <property name="sqlSession" ref="sqlSession" />
</bean>
```

# SqlSessionTemplate的ExecutorType构造参数

SqlSessionTemplate除了必需的SqlSessionFactory参数外，还提供ExecutorType（枚举类型）可选参数。

以ExecutorType.BATCH为例。

1）通过xml的方式向容器中注册SqlSessionTemplate组件。
```xml
<bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
    <constructor-arg index="0" ref="sqlSessionFactory" />
    <constructor-arg index="1" value="BATCH" />
</bean>
```
2）通过@Bean的方式向容器中注册SqlSessionTemplate组件。
```java
@Configuration
public class MyBatisConfig {
    @Bean
    public SqlSessionTemplate sqlSession() throws Exception {
	    return new SqlSessionTemplate(sqlSessionFactory(), ExecutorType.BATCH);
    } 
}
```
现在所有的映射语句可以进行批量操作了。
```java
public class UserService {
    private final SqlSession sqlSession;
    public UserService(SqlSession sqlSession) {
        this.sqlSession = sqlSession;
    }
    public void insertUsers(List<User> users) {
	    for (User user : users) {
	        sqlSession.insert("org.mybatis.spring.sample.mapper.UserMapper.insertUser", user);
	    }
    }
}
```

注意，在希望语句执行的方法与 `SqlSessionTemplate` 中的默认设置不同时使用这种配置。

这种配置的弊端在于，当调用这个方法时，不能存在使用不同 `ExecutorType` 并且进行中的事务。要么确保对不同 `ExecutorType` 的 `SqlSessionTemplate` 的调用处在不同的事务中，要么完全不使用事务。

# SqlSessionDaoSupport

SqlSessionDaoSupport是一个抽象类，提供getSqlSession()来获取SqlSessionTemplate。
```java
public class UserDaoImpl extends SqlSessionDaoSupport implements UserDao {
    public User getUser(String userId) {
	    return getSqlSession().selectOne("org.mybatis.spring.sample.mapper.UserMapper.getUser", userId);
    }
}
```
```xml
<bean id="userDao" class="org.mybatis.spring.sample.dao.UserDaoImpl">
    <property name="sqlSessionFactory" ref="sqlSessionFactory" />
</bean>
```

SqlSessionDaoSupport不提供构造函数，而SqlSessionDaoSupport必须持有SqlSessionTemplate对象，因此必须通过set()注入的方式为提供SqlSessionTemplate实例或SqlSessionFactory实例（使用该实例去创建SqlSessionTemplate实例）。
```java
public void setSqlSessionTemplate(SqlSessionTemplate sqlSessionTemplate) {  
  this.sqlSessionTemplate = sqlSessionTemplate;  
}

public void setSqlSessionFactory(SqlSessionFactory sqlSessionFactory) {   // 方法 1
    if (this.sqlSessionTemplate == null || sqlSessionFactory != this.sqlSessionTemplate.getSqlSessionFactory()) {  
	    this.sqlSessionTemplate = createSqlSessionTemplate(sqlSessionFactory);  
    }  
}
protected SqlSessionTemplate createSqlSessionTemplate(SqlSessionFactory sqlSessionFactory) {  
    return new SqlSessionTemplate(sqlSessionFactory);  
}
```

可以方法1可以看到，当同时设置SqlSessionTemplate和SqlSessionFactory实例时，SqlSessionFactory实例会被忽略。