

参考文档：
- 英文文档：`https://mybatis.org/mybatis-3/sqlmap-xml.html#association`
- 中文文档：`https://mybatis.org/mybatis-3/zh/sqlmap-xml.html#%E5%85%B3%E8%81%94`

mybatis使用 `<resultMap>子标签<association>` 来提供“有一个”的映射。mybatis提供2种处理“有1个”的情景：
- 嵌套的`<select>`
- 嵌套的结果处理

情景：一个博客（Blog）对应一个作者（Author）

数据库准备：
```sql
/* 创建博客(Blog)表  */
CREATE TABLE IF NOT EXISTS Blog(
	id INT AUTO_INCREMENT PRIMARY KEY,
	title VARCHAR(32) NOT NULL,
	author_id INT NOT NULL
);
/* 创建作者(Author)表  */
CREATE TABLE IF NOT EXISTS Author(
	id INT AUTO_INCREMENT PRIMARY KEY,
	username VARCHAR(64) NOT NULL,
	password VARCHAR(64) NOT NULL,
	email VARCHAR(32) NOT NULL,
	bio VARCHAR(32) NOT NULL,
	favourite_section VARCHAR(32) NOT NULL
);
```
插入数据：
```sql
	INSERT INTO author(username,password,email,bio,favourite_section) VALUES("小明",123,"123456@qq.com","bio_0","favourite_section_0")
```
Java类准备
```java
// 省略了get和set方法
public class Blog {  
    private int id;  
    private String title;  
    private Author author;  
}
// 省略了get和set方法
public class Author {  
    private Integer id;  
    private String username;  
    private String password;  
    private String email;  
    private String bio;  
    private String favouriteSection;  
}
```

## 嵌套的 `<select>`

以下面的文件展开说明。
```xml
<resultMap id="blogResult" type="Blog">
    <association property="author" column="author_id" javaType="Author" select="selectAuthor"/>
</resultMap>

<!-- select 1 -->
<select id="selectBlog" resultMap="blogResult">
    SELECT * FROM BLOG WHERE ID = #{id}
</select>
<!-- select 2 -->
<select id="selectAuthor" resultType="Author">
    SELECT * FROM AUTHOR WHERE ID = #{id}
</select>
```

1）执行 select 1 后，得到的表的列名包含：id、title、author_id。
2）根据得到的 author_id 再做一次 select 2 查询，得到的表的列名包含：id、username、password、email、bio、favourite_section。然后将其映射到Author实例属性字段，将该Author实例赋值给Blog实例。

## 嵌套的结果处理

以下面的文件展开说明。
```xml
<select id="selectBlog" resultMap="blogResult">
    select
	    B.id            as blog_id,
	    B.title         as blog_title,
	    B.author_id     as blog_author_id,
	    A.id            as author_id,
	    A.username      as author_username,
	    A.password      as author_password,
	    A.email         as author_email,
	    A.bio           as author_bio
    from Blog B left outer join Author A on B.author_id = A.id
    where B.id = #{id}
</select>
```
使用的`<resultMap>`如下也有2种格式：
- 外部
```xml
<resultMap id="blogResult" type="Blog">
    <id property="id" column="blog_id" />
    <result property="title" column="blog_title"/>
    <association property="author" resultMap="authorResult" />
</resultMap>
<resultMap id="authorResult" type="Author">
    <id property="id" column="author_id"/>
    <result property="username" column="author_username"/>
    <result property="password" column="author_password"/>
    <result property="email" column="author_email"/>
    <result property="bio" column="author_bio"/>
</resultMap>
```
- 内部
```xml
<resultMap id="blogResult" type="Blog">
    <id property="id" column="blog_id" />
    <result property="title" column="blog_title"/>
    <association property="author" javaType="Author">
        <id property="id" column="author_id"/>
        <result property="username" column="author_username"/>
        <result property="password" column="author_password"/>
        <result property="email" column="author_email"/>
        <result property="bio" column="author_bio"/>
    </association>
</resultMap>
```

可以看到与之前明显不同的地方在于只进行了一次select查询，一次查询就将Blog实例应该包含的id属性、title属性、author（Author类型）属性所需要的id、username、password、email、bio全部包含在内，然后按需进行映射即可。

另外补充 `<resultMap>的columnPrefix属性` 的作用。

## @One

下面是@One的定义。
```java
@Target({})  
public @interface One {
	String select() default "";
	FetchType fetchType() default FetchType.DEFAULT;
}
```

## 存储过程的映射

参考文档：
- 英文参考文档：https://mybatis.org/mybatis-3/sqlmap-xml.html#multiple-resultsets-for-association
- 中文参考文档：https://mybatis.org/mybatis-3/zh/sqlmap-xml.html#%E5%85%B3%E8%81%94%E7%9A%84%E5%A4%9A%E7%BB%93%E6%9E%9C%E9%9B%86%EF%BC%88resultset%EF%BC%89

存储过程准备：
```sql
CREATE PROCEDURE getBlogsAndAuthors(IN id int)
BEGIN
SELECT * FROM blog WHERE id = id;
SELECT * FROM author WHERE id = id;
END;
```
通过存储过程得到的两个结果集：blogs和authors，使用resultSets属性进行指定。
```xml
<select id="selectBlog" resultSets="blogs,authors" resultMap="blogResult" statementType="CALLABLE">
    {call getBlogsAndAuthors(#{id,jdbcType=INTEGER,mode=IN})}
</select>
```
下面是处理结果映射。
```xml
<resultMap id="blogResult" type="Blog">
    <id property="id" column="id" />
    <result property="title" column="title"/>
    <association property="author" javaType="Author" resultSet="authors" column="id" foreignColumn="author_id">
        <id property="id" column="id"/>
        <result property="username" column="username"/>
        <result property="password" column="password"/>
        <result property="email" column="email"/>
        <result property="bio" column="bio"/>
    </association>
</resultMap>
```
