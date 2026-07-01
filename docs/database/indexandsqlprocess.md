---
title: 数据库索引与SQL执行流程
date: 2026-07-01
categories:
  - DataBase
tags:
  - db2
---

## 一、数据存储过程（Data Page）

数据库真正存储数据的单位不是一行，而是**数据页（Data Page）**。

当创建一张表后，数据库会为该表分配存储空间。随着数据不断插入，数据库会把数据按页写入。

例如：

```text
Page1
1
2
3

Page2
4
5
6
```

通常情况下，数据会按照插入顺序顺序写入新的 Data Page，因此物理存储也是连续的。

---

## 二、索引的创建

数据库中的索引通常采用 **B+Tree** 结构。

### 1、普通（非聚簇）索引

普通索引不会改变数据页(Data Page)的存储位置。

它维护一棵独立的 B+Tree，叶子节点保存：

```text
索引值
↓

RID（Record ID）

↓

Page号 + 行号
```

例如：

```text
20
↓

Page8 Row3

20
↓

Page15 Row8
```

真正的数据仍然存放在 Data Page 中。

---

### 2、聚簇索引（Clustered Index）

聚簇索引会尽量让**数据页按照索引键的顺序存放**。

例如：

```text
Page1
1
2
3

Page2
4
5
6
```

此时：

- 索引的逻辑顺序
    
- 数据页的物理顺序
    

基本保持一致。

需要注意的是：

> 聚簇索引保证的是"尽量按照索引顺序组织数据"，而不是永远保持完全一致。

随着数据不断更新，这种一致性会逐渐下降。

---

## 三、页分裂（Page Split）

随着系统运行多年，不断发生：

- INSERT
    
- UPDATE
    
- DELETE
    

可能导致数据页已满。

例如：

最开始：

```text
Page1
1
2
3

Page2
4
5
6
```

后来插入：

```text
3.5
3.6
```

如果 Page1 已满，数据库可能新分配一个数据页：

```text
Page1
1
2
3

Page500
3.5
3.6

Page2
4
5
6
```

这样：

**逻辑顺序仍然正确**

```
1
↓

2
↓

3
↓

3.5
↓

3.6
↓

4
```

但是：

**物理顺序已经变乱**

```
Page1

↓

Page500

↓

Page2
```

这会导致：

- 范围查询性能下降
    
- 回表随机 IO 增多
    
- 聚簇率下降
    

因此数据库会建议执行：

```text
REORG
```

REORG 会重新整理 Data Page，使物理顺序重新尽量接近索引顺序。

---

## 四、INSERT 的大致流程

一次 INSERT 并不仅仅是写一条数据。

大致流程如下：

```text
① 找到目标 Data Page

↓

② 加锁

↓

③ 修改 Data Page

↓

④ 更新所有相关索引(B+Tree)

↓

⑤ 写 Undo（支持事务回滚）

↓

⑥ 写 Redo/WAL（日志，保证崩溃恢复）

↓

⑦ 提交事务

↓

⑧ 后台异步刷盘
```

因此：

索引越多：

- INSERT 越慢
    
- UPDATE 越慢
    
- DELETE 越慢
    

---

# 五、SQL 的逻辑执行顺序

SQL 的书写顺序：

```sql
SELECT
FROM
WHERE
GROUP BY
HAVING
ORDER BY
LIMIT
```

数据库逻辑执行顺序：

```text
FROM（包含JOIN）

↓

WHERE

↓

GROUP BY

↓

HAVING

↓

SELECT

↓

ORDER BY

↓

LIMIT
```

注意：

这里只是**SQL 的逻辑执行顺序**。

真正数据库执行时，优化器会根据成本重新安排执行顺序，例如：

- Predicate Pushdown（条件下推）
    
- Join 顺序优化
    
- 索引扫描
    

因此执行计划不一定与 SQL 书写顺序一致。

---

# 六、SQL 查询的大致流程

假设：

```sql
SELECT ...
FROM ...
WHERE ...
```

数据库执行过程通常如下：

```text
SQL

↓

优化器（Optimizer）

↓

选择执行计划

↓

IXSCAN（扫描索引）

↓

得到 RID（Page + Row）

↓

FETCH（是否需要回表）

↓

Projection（SELECT）

↓

GROUP / ORDER（必要时SORT）

↓

RETURN
```

其中：

## 1、IXSCAN

扫描 B+Tree 索引。

索引叶子节点保存：

```text
索引值

↓

RID

↓

Page号 + 行号
```

---

## 2、FETCH（回表）

如果 SELECT 的字段不全部包含在索引中：

数据库根据 RID 去 Data Page 获取完整数据。

例如：

```text
IXSCAN

↓

Page1 Row3

↓

FETCH

↓

读取 Data Page

↓

返回整行数据
```

如果数据页分散：

例如：

```text
Page1

↓

Page500

↓

Page20

↓

Page800
```

随机 IO 会增加。

数据库可能会：

- 收集 RID
    
- 排序 RID
    
- 再批量回表（RIDSCN）
    

减少随机 IO。

---

## 3、覆盖索引（Covering Index）

如果：

SELECT 的所有字段都已经包含在索引中。

数据库：

```text
IXSCAN

↓

直接返回
```

无需 FETCH。

因此：

**覆盖索引可以完全避免回表。**

---

# 七、什么时候数据库不用索引？

数据库会估算：

```text
索引扫描成本

VS

全表扫描成本
```

如果：

WHERE 返回的数据比例较高：

例如：

20%、30%、50%

数据库可能直接：

```text
TBSCAN（全表扫描）
```

因为：

回表访问的数据页已经接近整张表。

这时：

顺序扫描可能比大量随机 IO 更快。

所以：

一般来说：

**数据过滤率越高（返回的数据越少），越适合建立索引。**

---

# 八、GROUP BY、ORDER BY 为什么索引很重要？

很多人认为：

索引只是为了 WHERE。

实际上：

GROUP BY 和 ORDER BY 同样非常依赖索引。

原因：

B+Tree 本身就是有序的。

例如：

```text
1
1
2
2
3
3
5
5
```

数据库可以直接：

```text
IXSCAN

↓

GROUP

↓

RETURN
```

或者：

```text
IXSCAN

↓

ORDER

↓

RETURN
```

很多情况下：

无需：

```text
SORT
```

而没有索引：

数据库通常需要：

```text
TBSCAN

↓

SORT

↓

GROUP/ORDER
```

因此：

索引真正的重要作用之一：

**避免昂贵的排序（SORT）。**

---

# 九、JOIN 的索引设计

JOIN 优化通常遵循：

**驱动表先过滤，被驱动表快速查找。**

例如：

```sql
SELECT *
FROM A
JOIN B
ON A.user_id = B.user_id
WHERE A.status='Y';
```

推荐：

A 表：

```text
(status, user_id)
```

B 表：

```text
(user_id)
```

原则：

- 驱动表：WHERE 字段 + JOIN 字段
    
- 被驱动表：JOIN 字段
    

不是简单地两边都建立单列索引。

---

# 十、索引的真正作用

索引不仅仅是为了"查找更快"。

它真正有三个核心价值：

### ① 快速定位数据

通过 B+Tree 快速找到目标数据的位置。

避免全表扫描。

---

### ② 减少回表

通过覆盖索引，直接从索引返回数据。

避免访问 Data Page。

---

### ③ 提供有序的数据访问路径

利用 B+Tree 的天然有序性：

避免：

- SORT
    
- GROUP BY 排序
    
- ORDER BY 排序
    

提高 SQL 整体执行效率。

---

# 十一、总结

数据库真正消耗性能的地方通常不是 SQL 本身，而是：

```text
大量 Data Page 访问

↓

随机 IO

↓

回表

↓

SORT

↓

JOIN

↓

GROUP
```

索引优化的本质可以总结为一句话：

> **索引的核心目的不是单纯加快查找，而是减少数据库需要访问的数据页数量，并提供有序的数据访问路径，从而减少全表扫描、回表、排序（SORT）等高成本操作。**