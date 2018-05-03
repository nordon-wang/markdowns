# 创建数据库

- Create database 数据库名字 [库选项]; 

- 创建数据库
```
    create database mydatas charset utf8;
```

- 查看数据库
```
    show databases;
```

- 若是需要中文
```  
  set names gbk;
```

- 查看数据库
```
    show databases like 'my%'; -- 查看以my开始的数据库
```

- 查看数据库创建语句
```
    show create database mydatas;
```

- 删除数据库
```
    drop dababase mydatas;
```

- 使用数据库
```
    use mydatas;
```

# 创建表

- 创建表
1. 指定数据库创建表
```
create table if not exists  mydatas.table1(
    id int not null primary key,
    name varchar(20) not null
)charset utf8;
```

2. 使用数据库 use mydatas之后
```
create table if not exists tabke2(
    id int not null primary key auto_increment,
    age int not null
)charset utf8;

```

- 查看所有表
```
show tables;
```

- 查看表的创建结构

```
show create table 表名;

show create table table1;
```

- 查看表 以 ta开始
```
show tables like 'ta%';
```

- 查看表结构
```
desc 表名
    desc table1;

describe 表名
    describe table1;
    
show columns from 表名
    show columns from table1;
```

- 重命名表
```
rename table tabke2 to table2;
```

- 修改表选项 -字符集
```
alter table table1 charset = GBK;
```

- 给table1增加一个uid放在第一个位置
```
alter table table1 add column uid int first;
```

- 增加一个字段 放到具体的字段之后
```
alter table table1 add column number char(11) after id;
```

- 修改字段的属性并放在一个字段之后
```
alter table table1 modify number int after uid;
```

- 修改表字段名称
```
alter table table1 change uid uuid varchar(50);
```

- 删除表中字段
```
alter table table1 drop uuid;
```

- 删除数据表
```
drop table table2;
```










