---
layout: post
title:  mybatis-注意事项
categories: [java]
tags: [mybatis]
comments: true
---

工作中遇见的一些mybatis小点，怕忘记了，总结一下

<!--more-->

#### 1、Mapper文件预编译sql中特殊字符的处理

| <    | <=   | >    | >=    | &     | '      | "      |
| ---- | ---- | ---- | ----- | ----- | ------ | ------ |
| &lt; | &lt; | &gt; | &gt;= | &amp; | &apos; | &quot; |

大于小于等符合在动态sql预编译时属于特殊字符，因此在写mapper文件时需要使用对应的表达式来代替，同样的之前关于[mybatis注解版](https://silentself.github.io/articles/2018-11/mybatis-annotation-sql-1)的同样也需要用上述方式处理

#### 2、Mybatis在做查询Oracle数据库时

##### 1）需要加上jdbcType指定一下数据库类型否侧会出错（***类型错误***）,加上之后启动正常

```java
@Select({"<script>","select * from tbl_varys_m_export_file_task e ",
         "where e.eft_uuid = #{uuid,javaType=long,jdbcType=NUMERIC} ",
         "<if test = ' startDate != null and startDate != \"\" '> and e.eft_time &gt; to_date(#{startDate,jdbcType=DATE}, 'YYYY-MM-DD') </if>",
         "<if test = ' endDate != null and startDate != \"\" ' > and e.eft_time &lt; to_date(#{endDate,jdbcType=DATE}, 'YYYY-MM-DD') </if>",
         " ORDER BY e.EFT_TIME DESC","</script>"
        })
@Results({
    @Result(column = "eft_id",property = "id",id= true),
    @Result(column = "EFT_URL",property = "url"),
    @Result(column = "EFT_TIME",property = "time"),
    @Result(column = "EFT_STATUS",property = "status"),
    @Result(column = "EFT_uuid",property = "uuid"),
    @Result(column = "EFT_DESC",property = "desc")
})
public List<ExportTaskModel> getExportList(ExportDateSearchForm exportDateSearchForm);
```

##### 2）关于Oracle和MySQL对应的Mapper.xml文件中jdbcType设置类型，如果报错类型错误可以尝试加上

[原文](https://blog.csdn.net/loongshawn/article/details/50496460)

| Mybatis  | JdbcType      | Oracle         | MySql              |
| -------- | ------------- | -------------- | ------------------ |
| JdbcType | ARRAY         |                |                    |
| JdbcType | BIGINT        |                | BIGINT             |
| JdbcType | BINARY        |                |                    |
| JdbcType | BIT           |                | BIT                |
| JdbcType | BLOB          | BLOB           | BLOB               |
| JdbcType | BOOLEAN       |                |                    |
| JdbcType | CHAR          | CHAR           | CHAR               |
| JdbcType | CLOB          | CLOB           | CLOB–>修改为TEXT   |
| JdbcType | CURSOR        |                |                    |
| JdbcType | DATE          | DATE           | DATE               |
| JdbcType | DECIMAL       | DECIMAL        | DECIMAL            |
| JdbcType | DOUBLE        | NUMBER         | DOUBLE             |
| JdbcType | FLOAT         | FLOAT          | FLOAT              |
| JdbcType | INTEGER       | INTEGER        | INTEGER            |
| JdbcType | LONGVARBINARY |                |                    |
| JdbcType | LONGVARCHAR   | LONG VARCHAR   |                    |
| JdbcType | NCHAR         | NCHAR          |                    |
| JdbcType | NCLOB         | NCLOB          |                    |
| JdbcType | NULL          |                |                    |
| JdbcType | NUMERIC       | NUMERIC/NUMBER | NUMERIC/           |
| JdbcType | NVARCHAR      |                |                    |
| JdbcType | OTHER         |                |                    |
| JdbcType | REAL          | REAL           | REAL               |
| JdbcType | SMALLINT      | SMALLINT       | SMALLINT           |
| JdbcType | STRUCT        |                |                    |
| JdbcType | TIME          |                | TIME               |
| JdbcType | TIMESTAMP     | TIMESTAMP      | TIMESTAMP/DATETIME |
| JdbcType | TINYINT       |                | TINYINT            |
| JdbcType | UNDEFINED     |                |                    |
| JdbcType | VARBINARY     |                |                    |
| JdbcType | VARCHAR       | VARCHAR        | VARCHAR            |

