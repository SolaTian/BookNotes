# 高级SQL特性

## 约束
> 约束：管理如何插入或者处理数据库数据的规则。

### 主键

- 任意两行的主键值都不相同；
- 每行都由一个主键值，不允许`NULL`值；
- 包含主键值的列从不修改或更新；
- 主键值不能重用，如果从表中删除某一行，其主键值不分配给新行。

一种定义主键的方法是创建它
    
    CREATE TABLE 表名
    (
        列名1  CHAR(10)         NOT NULL PRIMARY KEY,
        列名2  CHAR(10)         NOT NULL,
        列名3  CHAR(254)        NOT NULL,
        列名4  DECIMAL(8,2)     NOT NULL,
        列名5  VARCHAR(1000)    NULL
    );

添加关键字`PRIMARY KEY`，使其成为主键。

也可以使用`CONSTRAINT`语法

    ALTER TABLE 表
    ADD CONSTRAINT PRIMARY KEY(列名1);

`SQL Server`只能使用第一种方法。

### 外键

外键是表中的一列，其值必须列在另一表的主键中。

    CREATE TABLE 表名
    (
        列名1  CHAR(10)         NOT NULL PRIMARY KEY,
        列名2  CHAR(10)         NOT NULL,
        列名3  CHAR(254)        NOT NULL,
        列名4  DECIMAL(8,2)     NOT NULL,
        列名5  VARCHAR(1000)    NULL REFERENCES 表2(列名5)
    );

使用`REFERENCES`关键字，表示`列名5`的任何值都必须是`表2`的`列名5`的值。

或者使用`ALTER TABLE`语句中的`CONSTRAINT`语法

    ALTER TABLE 表
    ADD CONSTRAINT
    FOREIGN KEY (列名5) REFERENCES 表名2 (列名5);

外键有助于防止以外删除。定义外键之后，`DBMS`不允许删除在另外一个表中具有关联行的行。

### 唯一约束

唯一约束用来保证一列中的数据是唯一的，类似于主键，但是有区别：

- 表中可以有多个唯一约束，但是每个表中只允许一个主键；
- 唯一约束列可以包含`NULL`；
- 唯一约束列可修改或更新；
- 唯一约束列的值可以重复使用；
- 与主键不一样，唯一约束不能用来定义外键；

唯一约束可以用`UNIQUE`关键字在表中定义，也可以用单独的`CONSTRAINT`定义

### 检查约束

检查约束用来保证一列或者一组列中的数据满足一组指定的条件。在创建表的时候使用`CHECK`语法

检查保证`列名5`大于0；

    CREATE TABLE 表名
    (
        列名1  CHAR(10)         NOT NULL,
        列名2  CHAR(10)         NOT NULL,
        列名3  CHAR(254)        NOT NULL,
        列名4  DECIMAL(8,2)     NOT NULL,
        列名5  INTEGER          NOT NULL CHECK(列名5>0)
    );

检查名为gender的列只包含`M`或`F`。

    ADD CONSTRAINT CHECK (gender LIKE '[MF]');



## 索引

索引用来排序数据以加快搜索和排序操作的速度。

创建索引

    CREATE INDEX 索引名
    ON 表(列名);

索引必须唯一命名。`ON`用来指定被索引的表，而索引中包含的列在表名后的圆括号中给出。




