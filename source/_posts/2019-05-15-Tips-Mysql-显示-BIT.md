---
title: 'Tips:Mysql 显示 BIT'
date: 2019-05-15 09:41:17
category: Other
tags: 
    - MySql
---

在数据库中创建一个表，添加一个 `is_new` 的 column，类型为 `BIT`：

``` sql
CREATE TABLE `test_table`
(
    `id`     BIGINT NOT NULL AUTO_INCREMENT PRIMARY KEY,
    `is_new` BIT    NOT NULL DEFAULT b'0'
);
```

插入一条数据库：

``` sql
INSERT INTO `test_table` SET `is_new`=1;
```

然后查询所有的数据：

``` sql
SELECT * FROM `test_table`;
```

显示结果为：

``` sh
mysql> select * from test_table;
+----+--------+
| id | is_new |
+----+--------+
|  1 |       |
+----+--------+
1 row in set (0.00 sec)
```

数据插入成功了，但是 `is_new` 的值不显示，换个图形化工具，可以查到这个值是 `1`，那么在命令行怎么查看呢？两种方法：

``` sql
SELECT `id`, bin(`is_new`) FROM `test_table`;

SELECT `id`, ord(`is_new`) FROM `test_table`;
```

```
mysql> SELECT `id`, bin(`is_new`) FROM `test_table`;
+----+---------------+
| id | bin(`is_new`) |
+----+---------------+
|  1 | 1             |
+----+---------------+
1 row in set (0.00 sec)

mysql> SELECT `id`, bin(`is_new`) FROM `test_table`;
+----+---------------+
| id | bin(`is_new`) |
+----+---------------+
|  1 | 1             |
+----+---------------+
1 row in set (0.00 sec)
```

结果都是一样的，但是这两个函数有什么区别呢？首先是 `BIN`

```
BIN(N)
返回N的二进制值的字符串表示，其中N是一个长(BIGINT)数字。这等同于CONV(N，10,2)。返回NULL，如果N为NULL。
```


``` sh
mysql> select bin(2);
+--------+
| bin(2) |
+--------+
| 10     |
+--------+
```

2 显示成了二进制形式 10。

`ORD()` 有点复杂：

```
ORD(str)
如果字符串str的最左边的字符是一个多字节字符返回该字符，用这个公式其组成字节的数值计算的代码：
  (1st byte code)
+ (2nd byte code . 256)
+ (3rd byte code . 2562) ...
如果最左边的字符不是一个多字节字符，ORD()返回相同的值如ASCII()函数。
```

***这个函数接受的是字符串*，我们就认为它的作用是返回参数的第一个字符的 ASCII，

``` sh
mysql> select ord(1);
+--------+
| ord(1) |
+--------+
|     49 |
+--------+
1 row in set (0.00 sec)

mysql> select ord(2);
+--------+
| ord(2) |
+--------+
|     50 |
+--------+
1 row in set (0.00 sec)

mysql> select ord('abc');
+------------+
| ord('abc') |
+------------+
|         97 |
+------------+
1 row in set (0.00 sec)
```

那么这个函数为什么能正常显示 `is_new` 呢，因为 `is_new` 是 BIT 类型。

``` sh
mysql> select ord(b'1');
+-----------+
| ord(b'1') |
+-----------+
|         1 |
+-----------+
1 row in set (0.01 sec)
```

就是说，`BIN` 是把 `is_new` 按二进制形式显示，`ORD` 是显示的 `is_new` 的 ASCII，或者转成了字符串？

`BIT` 对应 Java 的布尔类型，`BIT` 默认只有一个比特位，那么 0 就是 false, 其他是 true，下面的设置方式在 mysql 中都是允许的

``` sql
INSERT INTO `test_table`
SET `is_new`=1;
INSERT INTO `test_table`
SET `is_new`=b'1';
INSERT INTO `test_table`
SET `is_new`= FALSE;
INSERT INTO `test_table`
SET `is_new`= TRUE;
```



