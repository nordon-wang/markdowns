# 数据类型(列类型)

## 数值类型

- 数值型数据都是数值
- 系统将数值型分为整数型和小数型

### 整数型

- 因为sql需要考虑如何节省磁盘空间，因为系统将整型又细分为5类
  1. tinyint:迷你整型,使用一个字节存储,表示的状态为256种(常用)
  2. smallint:小整型,使用2个字节存储,表示的状态最多为65536种
  3. mediumint:中整型, 使用3个字节存储
  4. int: 标准整型, 使用4个字节存储(常用)
  5. bigint: 大整型,使用8个字节存储
- 创建一个整型的表

```sql
create table if not exists my_int(
	int_1 tinyint,
	int_2 int
)charset utf8;
```

- 插入数据

```sql
-- 有效数据
-- 查看数据为正确数据100,100
insert into my_int values(100,100);

-- 无效数据: 类型限定
-- 因为只能存储数值型，类型限制
insert into my_int values('a','188');	 

-- 错误: 超出范围
-- 因为tinyint是有符号的，存储范围是-128-127，若是无符号则存储范围是0-255
insert into my_int values(255,10000);
```

- sql中的数值类型全部都是默认有符号的: 就是分正负
- 在需要无符号数据的时候，需要给数据类型进行限定-unsigned，无符号从0开始
- 增加一个无符号的字段

```sql
alter table my_int add int_3 tinyint unsigned;

-- 增加一条数据,此时的255可以存储进去
insert into my_int values(127,222,255);
```

- 产看数据结构的时候，会在type这列发现列类型后面会存在int(11),tinyint(3) unsigned,tinyint(4)的结构
- tinyint(3)  unsigned 代表无符号的数据，3代表显示宽度，255是3位的宽度，正号省略
- tinyint(4)代表有符号的数据，4代表显示宽度(包括符号)，-111是一个4位的显示宽度(包括负号)
- 显示宽度没有什么特殊的意义，仅仅只是默认的告诉用户可以显示的形式而已，虽然可以自定义显示宽度的长度，但是并不能控制其本身存储的数据大小，即显示宽度修改并不能影响存储的数据大小

### 浮点型

#### float

#### double

## 字符串类型

#### char

#### varchar

#### text

#### set

#### enum

#### blob



## 时间日期类型





