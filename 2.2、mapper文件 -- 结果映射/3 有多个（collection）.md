
参考文档：
- 英文参考文档：https://mybatis.org/mybatis-3/sqlmap-xml.html#collection
- 中文参考文档：https://mybatis.org/mybatis-3/zh/sqlmap-xml.html#%E9%9B%86%E5%90%88

mybatis使用 `<resultMap>子标签<collection>` 来提供“有一个”的映射。mybatis提供2种处理“有多个”的情景：
- 另外执行一次`<select>`
- 嵌套的结果处理

情景：一个博客（Blog）有多篇文章（Post）

数据库准备：
```sql
/* 创建博客(Blog)表  */
CREATE TABLE IF NOT EXISTS Blog(
	id INT AUTO_INCREMENT PRIMARY KEY,
	title VARCHAR(32) NOT NULL,
	author_id INT NOT NULL
);
/* 创建文章(Post)表  */
CREATE TABLE IF NOT EXISTS post(
	id INT AUTO_INCREMENT PRIMARY KEY,
	blog_id INT NOT NULL,
	subject VARCHAR(32) NOT NULL,
	body VARCHAR(1024) NOT NULL
);
```
数据准备：
```sql
INSERT INTO blog(title,author_id) VALUES("Java知识",1); 
INSERT INTO blog(title,author_id) VALUES("C++知识",1);

INSERT INTO post(blog_id,subject,body) VALUES(1,"Java SE","Java SE是指Java Standard Edition");
INSERT INTO post(blog_id,subject,body) VALUES(1,"Java EE","Java EE是指Java Enterprise Edition");
```

下面是blog表内容：

|id|title|author_id|
| --- | --- | --- |
|1|Java知识|1|
|2|C++知识|1|

下面是post表内容：

| id  | blog_id | subject | body                               |
| --- | ------- | ------- | ---------------------------------- |
| 1   | 1       | Java SE | Java SE是指Java Standard Edition   |
| 2   | 1       | Java EE | Java EE是指Java Enterprise Edition |



java类准备：
```java
// 省略了get和set方法
public class Blog {  
    private int id;  
    private String title;  
    private Collection<Post> posts;  
}
// 省略了get和set方法
public class Post {  
    private Integer id;  
    private String subject;  
    private String body;  
}
```

## 另外执行一次 select

以下面的文件展开说明。
```xml
<resultMap id="blogResult" type="Blog">
    <collection property="posts" javaType="ArrayList" column="id" ofType="Post" select="selectPostsForBlog"/>
</resultMap>
<!-- select 1 -->
<select id="selectBlog" resultMap="blogResult">
    SELECT * FROM BLOG WHERE ID = #{id}
</select>
<!-- select 2 -->
<select id="selectPostsForBlog" resultType="Post">
    SELECT * FROM POST WHERE BLOG_ID = #{id}
</select>
```

## 另外的 `<resultMap>`

以下面的文件展开说明。
```xml
<select id="selectBlog" resultMap="blogResult">
    select
    B.id as blog_id,
    B.title as blog_title,
    B.author_id as blog_author_id,
    P.id as post_id,
    P.subject as post_subject,
    P.body as post_body
    from Blog B
    left outer join Post P on B.id = P.blog_id
    where B.id = #{id}
</select>
```

下面是id=1的查询结果。

| blog_id | blog_title | blog_author_id | post_id | post_subject | post_body                        |
| ------- | ---------- | -------------- | ------- | ------------ | -------------------------------- |
| 1       | Java知识   | 1              | 1       | Java SE      | Java SE是指Java Standard Edition |
| 1       | Java知识   | 1              | 2       | Java EE      | Java EE是指Java Enterprise Edition |

1）下面使用内嵌`<resultMap>`将表内容映射到Java对象。
```xml
<resultMap id="blogResult" type="Blog">
    <id property="id" column="blog_id" />
    <result property="title" column="blog_title"/>
    <collection property="posts" ofType="Post">
        <id property="id" column="post_id"/>
        <result property="subject" column="post_subject"/>
        <result property="body" column="post_body"/>
    </collection>
</resultMap>
```
2）下面使用外部`<resultMap>`将表内容映射到Java对象。
```xml
<resultMap id="blogResult" type="Blog">
    <id property="id" column="blog_id" />
    <result property="title" column="blog_title"/>
    <collection property="posts" ofType="Post" resultMap="blogPostResult" columnPrefix="post_"/>
</resultMap>

<resultMap id="blogPostResult" type="Post">
    <id property="id" column="id"/>
    <result property="subject" column="subject"/>
    <result property="body" column="body"/>
</resultMap>
```

## 存储过程的映射

创建存储过程。
```sql
CREATE PROCEDURE getBlogsAndPosts(IN id int)
BEGIN
SELECT * FROM blog WHERE ID = id;
SELECT * FROM post WHERE blog_id = id;
END;
```
查询语句。
```xml
<select id="selectBlog" resultSets="blogs,posts" resultMap="blogResult">
    {call getBlogsAndPosts(#{id,jdbcType=INTEGER,mode=IN})}
</select>
```
结果映射处理。
```xml
<resultMap id="blogResult" type="Blog">
    <id property="id" column="id" />
    <result property="title" column="title"/>
    <collection property="posts" ofType="Post" resultSet="posts" column="id" foreignColumn="blog_id">
        <id property="id" column="id"/>
        <result property="subject" column="subject"/>
        <result property="body" column="body"/>
    </collection>
</resultMap>
```
