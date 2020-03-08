# 一.MySQL模糊查询

## 1.like关键字

- 通配符
  - %任意多个字符，包含0个字符
  - _任意单个字符

## 2.between and关键字

## 3.in关键字

含义：判断某字段的值是否属于in列表中的一项

## 4.is null关键字

=或<>不能用于判断null值

is null 或 is not null 可以判断null值



# 二.MySQL连接查询

## 1.内连接

### 	1.等值连接

### 	2.非等值连接

### 	3.自连接

## 2.外连接

### 	1.左外连接

### 	2.右外连接

### 	3.全外连接

## 3.交叉连接

# 三.MySQL视图

## 1.视图的创建

create view myv1

as 

select s.name,s.age,s.sex from stutent s join teacher t on s.tid = t.id;

## 2.视图的使用

select * from myv1 where last_name like '%a%';

## 3.视图的修改

- create or replace view 视图名 as 查询语句;
- alter view 视图名 as 查询语句;

## 4.视图的删除

drop view 视图名,视图名...

## 5.查看视图

- desc 视图名;
- show create view 视图名;

## 6.视图的更新

- insert into 视图名 values(...);
- update 视图名 set ...
- delete from 视图名 where...

**具备以下特点的视图不允许更新**

1. 包含以下关键字的sql语句：分组函数、distinct、group by、having、union、union all
2. 常量视图
3. select中包含子查询
4. join
5. from一个不能更新的视图
6. where子句的子查询引用了from子句中的表

## 7.视图与表的笔记

|      | 创建语法关键字 | 是否占用物理空间  | 使用                     |
| ---- | :------------- | ----------------- | ------------------------ |
| 视图 | create view    | 只是保存了sql逻辑 | 增删改查，一般不能增删改 |
| 表   | create table   | 保存了数据        | 增删改查                 |



# 四.MySQL加载顺序

1. FROM <left_table>
2. ON <join_condition>
3. <join_type> JOIN <right_type>
4. WHERE <where_condition>
5. GROUP BY <group_by_list>
6. HAVING <having_condition>
7. SELECT
8. DISTINCT <select_list>
9. ORDER BY <order_by_condition>
10. LIMIT <limit_number>

# 五.MySQL join

![sql-join](/Users/S/Desktop/LearningNotes/sql-join.png)

# 五.MySQL索引

# 六.数据库锁