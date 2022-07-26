# Day6 SQL高级处理

### 窗口函数

窗口函数也称为 OLAP 函数。 OLAP 是 OnLine AnalyticalProcessing 的简称，意思是对数据库数据进行实时分析处理。

```sql
<窗口函数>OVER ([PARTITION BY <列名>]
ORDER BY <排序用列名>)
```

- PARTITION BY 能够设定窗口对象范围。本例中，为了按照商品种类进行排序，我们指定了 product_type。即一个商品种类就是一个小的" 窗口"。
- ORDER BY 能够指定按照哪一列、何种顺序进行排序。为了按照销售单价的升序进行排列，我们指定了sale_price。此外，窗口函数中的 ORDER BY 与 SELECT 语句末尾的 ORDER BY 一样，可以通过关键字ASC/DESC 来指定升序/降序。省略该关键字时会默认按照 ASC

### 窗口函数种类

- 专用窗口函数
    - RANK 函数（英式排序）：计算排序时，如果存在相同位次的记录，则会跳过之后的位次
    - DENSE_RANK 函数（中式排序）：同样是计算排序，即使存在相同位次的记录，也不会跳过之后的位次。
    - ROW_NUMBER 函数：赋予唯一的连续位次
- 聚合函数在窗口函数上的使用
    - 可以看出，聚合函数结果是，按我们指定的排序，这里是 product_id， 当前所在行及之前所有的行的合计或均值。即累计到当前行的聚合。
- 窗口函数的的应用 - 计算移动平均

```sql
<窗口函数> OVER (ORDER BY <排序用列名>
ROWS n PRECEDING )

<窗口函数> OVER (ORDER BY <排序用列名>
ROWS BETWEEN n PRECEDING AND n FOLLOWING)

#PRECEDING（“之前”），将框架指定为“截止到之前 n 行”，加上自身行
#FOLLOWING（“之后”），将框架指定为“截止到之后 n 行”，加上自身行
#BETWEEN 1 PRECEDING AND 1 FOLLOWING，将框架指定为“之前 1 行” +“之后 1 行” +“自身
```

### GROUPING 运算符

- ROLLUP - 计算合计及小计

```sql
SELECT product_type
,regist_date
,SUM(sale_price) AS sum_price
FROM product
GROUP BY product_type, regist_date WITH ROLLUP
```

### 存储过程和函数

```sql
[delimiter //]($$，可以是其他特殊字符）
CREATE
[DEFINER = user]
PROCEDURE sp_name ([proc_parameter[,...]])
[characteristic ...]
[BEGIN]
routine_body
[END//]($$，可以是其他特殊字符）
```

- 参数介绍
    - IN 是入参：每个参数默认都是一个 IN 参数。如需设定一个参数为其他类型参数，请在参数名称前使用关键字 OUT或 INOUT 。一个 IN 参数将一个值传递给一个过程。存储过程可能会修改这个值，但是当存储过程返回时，调用者不会看到这个修改。
    - OUT 是出参：一个 OUT 参数将一个值从过程中传回给调用者。它的初始值在过程中是 NULL ，当过程返回时，调用者可以看到它的值
    - INOUT ：一个 INOUT 参数由调用者初始化，可以被存储过程修改，当存储过程返回时，调用者可以看到存储过程的任何改变。
- 查询

```sql
drop procedure if exists citycount;
create procedure citycount(in country char(3),out cities int)
begin
	select count(*) into cities from world.city
	where CountryCode = country;
end
```

- 创建表

```sql
use world;
DELIMITER $$
create definer=root@localhost procedure product_test()
begin
	create table product_test like shop.product;
end $$
```

- 插入数据

```sql
CREATE DEFINER=root@localhost PROCEDURE insert_product_test()
Line 2 BEGIN
declare i int;
set i=1;
while i<9 do
set @pcid = CONCAT(’000’, i);
PREPARE stmt FROM ’INSERT INTO product_test() SELECT * FROM shop.
product where product_id= ?’;
EXECUTE stmt USING @pcid;
set i=i+1;
end while;
END
```

- PREPARE Statement

MySQL PREPARE Statement 使用步骤如下：

1. PREPARE –准备需要执行的语句预处理声明。

2. EXECUTE –执行预处理声明。

3. DEALLOCATE PREPARE –释放预处理声明

```sql
prepare stmt1 from
'select
product_id,product_name
from product
where product_id=?';
set @pcid='0005';
execute stmt1 using @pcid;
DEALLOCATE PREPARE stmt1;
```