
参考文档：
- 英文参考文档 -- https://mybatis.org/mybatis-3/dynamic-sql.html
- 中文参考文档 -- https://mybatis.org/mybatis-3/zh/dynamic-sql.html

# if

动态sql中使用最为频繁的就是条件性地引入条件子句。

考虑下面的情况：
```xml
<select id="findActiveBlogWithTitleLike" resultType="Blog">
    SELECT * FROM BLOG
    WHERE state = ‘ACTIVE’
    <if test="title != null">
	    AND title like #{title}
    </if>
</select>
```

# choose，when，otherwise

有时候，我们不想使用所有的条件，而只是想从多个条件中选择一个使用。针对这种情况，MyBatis 提供了 choose 元素，它有点像 Java 中的 switch 语句。

还是上面的例子，但是策略变为：传入了 “title” 就按 “title” 查找，传入了 “author” 就按 “author” 查找的情形。若两者都没有传入，就返回标记为 featured 的 BLOG（这可能是管理员认为，与其返回大量的无意义随机 Blog，还不如返回一些由管理员精选的 Blog）。


# trim，where，set


考虑下面的情况：
```xml
<select id="findActiveBlogLike" resultType="Blog">
    SELECT * FROM BLOG WHERE
    <if test="state != null">
        state = #{state}
    </if>
    <if test="title != null">
        AND title like #{title}
    </if>
    <if test="author != null and author.name != null">
        AND author_name like #{author.name}
    </if>
</select>
```
如果没有条件满足，生成的sql语句会变成下面这样。
```text
SELECT * FROM BLOG WHERE
```
如果第二个条件满足，生成的sql语句会变成下面这样。
```text
SELECT * FROM BLOG WHERE
AND title like ‘someTitle’
```
很明显上面的结果是错误的，因此mybatis提供 `<where>标签` 来应对上面的情况。
```xml
<select id="findActiveBlogLike" resultType="Blog">
    SELECT * FROM BLOG
    <where>
        <if test="state != null">
	        state = #{state}
        </if>
        <if test="title != null">
		    AND title like #{title}
        </if>
        <if test="author != null and author.name != null">
            AND author_name like #{author.name}
        </if>
    </where>
</select>
```
`<where>标签` 知道：
- 只有当内部有任何一个标签元素满足时，才会添加 WHERE。
- 如果内容以AND、OR开头，则会去掉。

where元素能够处理90％的问题，但是在某些特定的情况下，需要自定义 trim 元素。
```xml
<trim prefix="WHERE" prefixOverrides="AND |OR ">
    ...
</trim>
```

更新语句同样会出现类似的问题。考虑下面的情况：
```xml
<update id="updateAuthorIfNecessary">
    update Author
        <if test="username != null">username=#{username},</if>
        <if test="password != null">password=#{password},</if>
        <if test="email != null">email=#{email},</if>
        <if test="bio != null">bio=#{bio}</if>
    where id=#{id}
</update>
```
如果条件1、2、3只有一个满足，生成的sql语句会变成下面这样。
```text
update Author username=#{username}, where id=#{id}
```
很明显是错误的，因此mybatis同样提供 `<set>标签` 来应对上面的情况。
```xml
<update id="updateAuthorIfNecessary">
    update Author
	    <set>
	        <if test="username != null">username=#{username},</if>
	        <if test="password != null">password=#{password},</if>
	        <if test="email != null">email=#{email},</if>
	        <if test="bio != null">bio=#{bio}</if>
	    </set>
    where id=#{id}
</update>
```
`<set>标签` 知道：
- 只有当内部有任何一个标签元素满足时，才会添加 SET。
- 内部最后有 分隔符（此处是逗号）时，会去掉分隔符。

某些情况下，同样需要自定义trim 元素。
```xml
<trim prefix="SET" suffixOverrides=",">
    ...
</trim>
```

# foreach

动态 SQL 的另一个常见使用场景是对集合进行遍历，比如：
```xml
<select id="selectPostIn" resultType="domain.blog.Post">
    SELECT * FROM POST P
    <where>
	    <foreach item="item" index="index" collection="list"
	        open="ID in (" separator="," close=")" nullable="true">
            #{item}
	    </foreach>
    </where>
</select>
```

foreach元素包含如下属性：
- item、index：可以在foreach元素体内使用，分别代表集合的元素内容和元素索引（遍历Iterable和Array时，对应元素下标；遍历Map时，对应元素的键）。
- open：出现在生成的sql语句的开头，只出现一次。
- close：出现在生成的sql语句的末尾，只出现一次。
- separator：遍历集合元素时，元素与元素之间的分隔符。

上面生成的sql语句即为：
```text
SELECT * FROM POST P WHERE ID IN (1,2,3)
```

# script

要在mapper接口类的注解中使用动态 SQL，可以使用 _script_ 元素。比如：
```java
@Update({"<script>",
      "update Author",
      "  <set>",
      "    <if test='username != null'>username=#{username},</if>",
      "    <if test='password != null'>password=#{password},</if>",
      "    <if test='email != null'>email=#{email},</if>",
      "    <if test='bio != null'>bio=#{bio}</if>",
      "  </set>",
      "where id=#{id}",
      "</script>"})
    void updateAuthorValues(Author author);
```

# bind

