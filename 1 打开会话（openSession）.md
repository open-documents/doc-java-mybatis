
```java
/* --------------------------- DefaultSqlSessionFactory ---------------------------*/
private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit) {  
    Transaction tx = null;  // mybatis提供的类
    try {  
	    final Environment environment = configuration.getEnvironment();  
	    final TransactionFactory transactionFactory = getTransactionFactoryFromEnvironment(environment);  
	    tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);  
	    final Executor executor = configuration.newExecutor(tx, execType);  
	    return new DefaultSqlSession(configuration, executor, autoCommit);  
    } catch (Exception e) {  
	    closeTransaction(tx); // may have fetched a connection so lets call close()  
	    throw ExceptionFactory.wrapException("Error opening session.  Cause: " + e, e);  
    } finally {  
	    ErrorContext.instance().reset();  
    }  
}
```