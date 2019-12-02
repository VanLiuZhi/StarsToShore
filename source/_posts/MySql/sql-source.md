---
title: MySQL Sql sentence
date: 2019-3-07 00:00:00
tags: [database, note]
categories: database
---

原生语句的使用与学习

<!-- more -->

## 常见join

```
select * from table_a a left join table_b b on a.key = b.key

select * from table_a a inner join table_b b on a.key = b.key

select * from table_a a right join table_b b on a.key = b.key

select * from table_a a left join table_b b on a.key = b.key where b.key is NULL

select * from table_a a right join table_b b on a.key = b.key where a.key is NULL

select * from table_a a full outer join table_b b on a.key = b.key

select * from table_a a full outer join table_b b on a.key = b.key where a.key is NULL or b.key is NULL
```
查询的效果就是取两张表的join结果，有不同的join方法，字段会被组合在一行中，也就是笛卡尔积。

### 一些例子

select a.id from app_info a right join agents_profit_setting b on a.id = b.id;
注意使用了别名a为表app_info取别名，所以在其它需要用到表app_info都使用别名访问

select a.id, b.id as b_id from app_info a right join agents_profit_setting b on a.id = b.id;
返回a，b的id，并对b的id取别名b_id

## 函数

系统提供了一些通用函数，可以在语句中使用

### 时间函数

```
mysql> select now(), curdate(), sysdate(), curtime();
+---------------------+------------+---------------------+-----------+
| now()               | curdate()  | sysdate()           | curtime() |
+---------------------+------------+---------------------+-----------+
| 2019-03-07 17:17:44 | 2019-03-07 | 2019-03-07 17:17:44 | 17:17:44  |
+---------------------+------------+---------------------+-----------+
```

### concat 和 contcat_ws

concat用来连接字符串，contcat_ws用来以分隔符参数来连接字符串
`contcat_ws(',', 'a', 'b')  输出为：a,b`

### case 排序把特定字段放在结果前面

```sql
select * from `equipment` order by case when (id='ZJXCA007804' or id='ZJXCA000695') then 0 else 1 end ,ismonitor desc
```

```py
equipments.order_by(Case(When(id__in=[item.get('eqid', 0) for item in equipment_ids], then=0), default=1), '-ismonitor', 'id')
```

## 分页

limit y 分句表示: 读取 y 条数据
limit x, y 分句表示: 跳过 x 条数据，读取 y 条数据
limit y offset x 分句表示: 跳过 x 条数据，读取 y 条数据

第1页： 从第0个开始，获取20条数据
```sql
selete * from testtable limit 0, 20; 
selete * from testtable limit 20 offset 0;  
```

第2页： 从第20个开始，获取30条数据
```sql
selete * from testtable limit 20, 30; 
selete * from testtable limit 30 offset 20;  
```

