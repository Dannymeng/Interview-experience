# MySQL常见操作


重点：
## 数据库增删改查
[具体操作](https://blog.csdn.net/qq_38345598/article/details/79416578)
[select操作](https://www.cnblogs.com/gates/p/3936974.html)

[SQL面试题](https://www.cnblogs.com/diffrent/p/8854995.html)
### SELECT(重点)
select * from tablename where 条件
### DELETE
删除一行数据
```SQL
delete from Employees
where EmployeeID=32 /*删除EmployeeID=32的数据*/
```

删除所有行数据
```SQL
delete 
from Employees
```

### UPDATE
更新单个列
```SQL
update Employees
set LastName='hello world'
where EmployeeID=30
```

更新多个列数据
```SQL
update Employees
where EmployeeID=30
set LastName='helloworld1',FirstName='ECJTU'
```

### INSERT
```SQL
INSERT INTO table_name (列1，列2，......) VALUES （值1，值2，.....）
```

## 1、数据库操作
### 创建名为demo的数据库
格式：create database demo;
使用数据库：use demo;
### 删除数据库
格式：drop database demo;
### 展示所有数据库: show databases;

## 2、表的操作
### 创建表
```SQL
CREATE TABLE student(
	id INT,
	NAME VARCHAR(20)
);
```
上面创建表的时候涉及到一个完整性约束条件，下面就列出一个完整性约束条件表：

|约束条件|说明|
|---|---|
|PRIMARY KEY|标识该属性为该表的主键，可以唯一的标识对应的元组|
| FOREIGN KEY  |  标识该属性为该表的外键，是与之联系某表的主键 |
|  NOT NULL  |  	标识该属性不能为空 |
| UNIQUE  |  标识该属性的值是唯一的  |
|AUTO_INCREMENT|自增1 标识该属性的值是自动增加，这是MySQL的SQL语句的特色 |
| DEFAULT| 	为该属性设置默认值  |

### 表的查询
#### 查看表的结构
格式： desc student;

### 修改表
#### 修改表名
格式：ALTER TABLE 旧表名 RENAME 新表名;
ALTER TABLE student RENAME student4;

#### 修改字段的数据类型
格式：ALTER TABLE 表名 MODIFY 属性名 数据类型;
ALTER TABLE student1 MODIFY name varchar(30);

#### 修改字段名
格式：ALTER TABLE 表名 CHANGE 旧属性名 新属性名 新数据类型;
ALTER TABLE student1 CHANGE name stu_name varchar(40);

#### 增加字段
格式：ALTER TABLE 表名 ADD 属性名1 数据类型 [完整性约束条件] [FIRST | AFTER 属性名2];
ALTER TABLE student1 ADD teacher_name varchar(20) NOT NULL AFTER id;
注：上面一行的意思是在id属性之后添加一个非空的名为“stu_name”的字段

### 删除操作

#### 删除表
 格式：DROP TABLE 表名;
 #### 删除字段
 格式：ALTER TABLE 表名 DROP 属性名;

#### 删除外键
格式：ALTER TABLE 表名 DROP FOREIGN KEY 外键别名;

[MYSQL常用命令](https://www.cnblogs.com/sqbk/p/5806797.html)


 

 







