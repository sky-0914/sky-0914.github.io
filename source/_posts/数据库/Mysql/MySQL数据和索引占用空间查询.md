---
title: MySQL数据和索引占用空间查询
tags:
  - Mysql
  - 数据库
categories:
  - 数据库
---
# MySQL数据和索引占用空间查询

## 查询所有数据库占用磁盘空间大小的SQL语句

```mysql
SELECT
	table_schema, -- 数据库名称
	concat( TRUNCATE ( sum( data_length ) / 1024 / 1024, 2 ), 'MB' ) AS data_size, -- 数据占用空间
	concat( TRUNCATE ( sum( index_length ) / 1024 / 1024, 2 ), 'MB' ) AS index_size -- 索引占用空间
FROM
	information_schema.TABLES 
GROUP BY
	table_schema 
ORDER BY
	sum( data_length ) DESC;
```

## 查询单个库中所有表磁盘占用大小的SQL语句

```mysql
SELECT
	table_name, -- 表名称
	concat( TRUNCATE ( data_length / 1024 / 1024, 2 ), 'MB' ) AS data_size, -- 数据占用空间
	concat( TRUNCATE ( index_length / 1024 / 1024, 2 ), 'MB' ) AS index_size -- 索引占用空间
FROM
	information_schema.TABLES 
WHERE
	table_schema = '数据库名称' 
ORDER BY
	data_length DESC;
```

