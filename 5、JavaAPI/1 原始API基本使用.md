
1）Maven依赖

```xml
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>x.x.x</version>
</dependency>
```

每个Mybatis应用：
- 围绕一个 `SqlSessionFactory` 实例，能通过 `SqlSessionFactoryBuilder` 获取。
- `SqlSessionFactoryBuilder` 从xml中构建 `SqlSessionFactory` 实例。
```java
String resource = "org/mybatis/example/mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
```


）从 SqlSessionFactory 中获取SqlSession实例。
```java
try (SqlSession session = sqlSessionFactory.openSession()) {
    ...
}
```
）
```java
Blog blog = session.selectOne("org.mybatis.example.BlogMapper.selectBlog", 101);
```

```java
BlogMapper mapper = session.getMapper(BlogMapper.class);
Blog blog = mapper.selectBlog(101);
```