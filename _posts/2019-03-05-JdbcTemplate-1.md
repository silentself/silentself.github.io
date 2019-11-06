---
layout: post
title:  JdbcTemplate一次查询多个sql并返回结果
categories: [java]
tags: [JdbcTemplate]
comments: true
---

#### 应用场景

项目中是通过oracle存储过程触发器控制用户查询权限，所以每次都需要先查询一次固定的sql，但并不需要取回结果集

##### 解决办法1.

连续同步查询两次

```java
public synchronized static List<Map<String, Object>> getResults(JdbcTemplate jdbcTemplate, StringBuilder sb,
			Long uuid,int flag) throws SQLException {
		final String callFunctionSql = String.valueOf("SELECT FUNC_SET_IDENTIFIER('" + uuid + "') FROM DUAL");
		jdbcTemplate.queryForList(callFunctionSql);
		List<Map<String, Object>> list = jdbcTemplate.queryForList(sb.toString());
		DataSource dataSource = jdbcTemplate.getDataSource();
		Connection con = dataSource.getConnection();
		DataSourceUtils.doCloseConnection(con, dataSource);
		return list;
	}
```

##### 解决办法2.

利用JdbcTemplate可以一次连接执行多条sql的特点，一次执行两条sql，取出第二条sql的结果集

```java
/**
	 * 
	 * @param jdbcTemplate
	 * @param sb
	 *            sql语句
	 * @param uuid
	 *            用户id
	 * @param flag
	 *            标识，扩展字段，暂未涉及
	 * @return
	 */
	public static List<Map<String, Object>> getResults(JdbcTemplate jdbcTemplate, StringBuilder sb, Long uuid,
			int flag) {
		final String callFunctionSql = String.valueOf("SELECT FUNC_SET_IDENTIFIER('" + uuid + "') FROM DUAL");
		List<Map<String, Object>> result = jdbcTemplate.execute((Statement stmt) -> {
			// 0.权限
			stmt.executeQuery(callFunctionSql);
			// 1.查询数据列表
			List<Map<String, Object>> listMap = new ArrayList<>();
			ResultSet listMapRs = stmt.executeQuery(sb.toString());
			ResultSetMetaData listMapMetaData = listMapRs.getMetaData();
			int columnCount = listMapMetaData.getColumnCount();
			while (listMapRs.next()) {
				Map<String, Object> map = new HashMap<>();
				for (int i = 1; i <= columnCount; i++) {
					map.put(listMapMetaData.getColumnName(i), listMapRs.getObject(i));
				}
				listMap.add(map);
			}
			return listMap;
		});
		return result;
	}
```

