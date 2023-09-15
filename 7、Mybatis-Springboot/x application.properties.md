
参考文档：
- 英文参考文档 -- http://mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/#configuration
- 

MyBatis-Spring-Boot-Application的配置参数写在`application.properties`(或 `application.yml`)。前缀为`mybatis`。



```properties
# 数据库驱动
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
# 数据库URL
spring.datasource.url=jdbc:mysql://localhost:3306/community?characterEncoding=utf-8&userSSL=false&serverTimezone=Hongkong
# 数据库连接用户名
spring.datasource.username=root
# 数据库连接密码
spring.datasource.password=xcxwrka1314
  
# 数据库连接池  
spring.datasource.type=com.zaxxer.hikari.HikariDataSource  
# 连接池最大连接数  
spring.datasource.hikari.maximum-pool-size=15  
# 连接池最小闲置连接数  
spring.datasource.hikari.minimum-idle=5  
# 连接被判定为闲置,从而释放资源的时间  
spring.datasource.hikari.idle-timeout=30000  
  
# mapper文件所在位置  
mybatis.mapper-locations=classpath:mapper/*.xml  
# mapper对应实体类对象所在位置  
mybatis.type-aliases-package=com.example.newcoder.entity  
# 插入时自动生成主键  
mybatis.configuration.use-generated-keys=true  
# 自动映射数据库下划线和java驼峰命名  
mybatis.configuration.map-underscore-to-camel-case=true
```

```java
spring:  
  datasource:  
    driver-class-name: com.mysql.cj.jdbc.Driver  
    url: jdbc:mysql://localhost:3306/community?characterEncoding=utf-8&userSSL=false&serverTimezone=Hongkong  
    username: root  
    password: xcxwrka1314  
    type: com.zaxxer.hikari.HikariDataSource  
    hikari:  
      maximum-pool-size: 15  
      minimum-idle: 5  
      idle-timeout: 30000  
  
mybatis:  
  mapper-locations: classpath:mapper/*.xml  
  type-aliases-package: com.example.newcoder.entity  
  configuration:  
    use-generated-keys: true  
    map-underscore-to-camel-case: true
```