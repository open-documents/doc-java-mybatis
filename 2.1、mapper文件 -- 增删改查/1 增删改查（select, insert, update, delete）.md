
`<insert>`、`<update>`、`<delete>`

通过Mybatis，可以只需要简单地书写sql语句，而大量介绍重复的jdbc代码。

# select



select标签有下面的属性：
- id：
- parameterMap：
- resultMap、resultType：

## @Select

除了在`<select>`标签中定义sql语句外，Mybatis提供@Select注解。下面是@Select注解的定义（去除了不重要的部分）：
```java
@Target(ElementType.METHOD)  
@Repeatable(Select.List.class)  
public @interface Select {
	String[] value();
	String databaseId() default "";
	boolean affectData() default false;
	@Documented  
	@Retention(RetentionPolicy.RUNTIME)  
	@Target(ElementType.METHOD)  
	@interface List {  
	  Select[] value();  
	}
}
```





# insert时的主键自增

如果数据库支持主键自增（如Mysql和SQL Sever），可以通过简单地设置 useGeneratedKeys="true" 和 设置`keyProperty`为目标属性（主键）即可。
```xml
<insert id="insertAuthor" useGeneratedKeys="true" keyProperty="id">
    insert into Author (username,password,email,bio) values (#{username},#{password},#{email},#{bio})
</insert>
```


```xml
<!-- foreach标签在动态sql介绍 --> 
<insert id="insertAuthor" useGeneratedKeys="true" keyProperty="id">
    insert into Author (username, password, email, bio) values
    <foreach item="item" collection="list" separator=",">
	    (#{item.username}, #{item.password}, #{item.email}, #{item.bio})
    </foreach>
</insert>
```

-- --
如果数据库不支持自动自增，Mybatis提供`<selectKey>`标签来支持。考虑下面的例子：
```xml
<insert id="insertAuthor">
  <selectKey keyProperty="id" resultType="int" order="BEFORE">
    select CAST(RANDOM()*1000000 as INTEGER) a from SYSIBM.SYSDUMMY1
  </selectKey>
  insert into Author
    (id, username, password, email,bio, favourite_section)
  values
    (#{id}, #{username}, #{password}, #{email}, #{bio}, #{favouriteSection,jdbcType=VARCHAR})
</insert>
```
上面的例子，selectKey语句先执行，然后通过获取的值来设置Author的id属性，然后调用insert语句。