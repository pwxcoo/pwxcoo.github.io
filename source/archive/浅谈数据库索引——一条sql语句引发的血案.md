---
title: 浅谈数据库索引——一条sql语句引发的血案
date: 2018-08-27 22:17:03
categories: database
tags:
- mysql
- database
---

## 起因

今天搞 [news-boom](https://github.com/pwxcoo/news-boom) 的时候，一条很奇怪的 sql 语句一直报错。

就是[这条](https://github.com/pwxcoo/news-boom/blob/master/src/main/java/com/pwxcoo/newsboom/dao/MessageDAO.java#L21)

整理一下就是这这样子:

```sql
SELECT
    from_id, to_id, content, has_read, conversation_id, created_date, count(id) as id
FROM
    (
        SELECT
            *
        FROM
            message
        WHERE
            from_id=1 OR to_id=1
        ORDER BY
            id DESC
    ) tt
    GROUP BY
        conversation_id
    ORDER BY
        id DESC
    LIMIT
        10, 10
```

message 的 schema 是这样的:

```sql
CREATE TABLE `message` (
  `id` INT NOT NULL AUTO_INCREMENT,
  `from_id` INT NULL,
  `to_id` INT NULL,
  `content` TEXT NULL,
  `created_date` DATETIME NULL,
  `has_read` INT NULL,
  `conversation_id` VARCHAR(45) NOT NULL,
  PRIMARY KEY(`id`),
  INDEX `conversation_index` (`conversation_id` ASC),
  INDEX `created_date` (`created_date` ASC)
) ENGINE=InnoDB DEFAULT CHARACTER SET=utf8mb4;
```

这条 sql 的目的是按 conversation 分类，然后把所有关于某个用户的每条 message 找出来。

然后这条 sql 就报错了。。报错信息大概就是这样子。。

```
[Err] 1055 - Expression #1 of ORDER BY clause is not in GROUP BY clause and contains nonaggregated column 'information_schema.PROFILING.SEQ' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by
```

然后就找到了官网的说法:

> SQL92 and earlier does not permit queries for which the select list, HAVING condition, or ORDER BY list refer to nonaggregated columns that are not named in the GROUP BY clause.

我当时就蒙圈了，`nonaggregated` 又是什么。。

然后这样追本溯源。。居然补了很多数据库的知识。。然后就顺便记下来了。

## MySQL索引

索引 (Index) 是用来高效获取数据的数据结构。大多的关系型数据库都是用的 B+ 树。复杂度是 $O=(n)$

### 为什么不用哈希表？

- 范围查询中毫无用处
- 没法排序
- 不支持多表联合查询和最做匹配
- 在大量重复键值的情况，因为哈希碰撞，效率也是很低

### 为什么不用红黑树？

数据库是存储在磁盘上的，最耗时间的是磁盘 I/O，要尽量减少磁盘的 I/O。

- 红黑树中树的高度明显比 B/B+ 树要大
- 逻辑上很近的节点，物理上可能很远，磁盘寻道的时间消耗大。

### B-Tree

![b-tree.png](/image/b-tree.png)

m 阶 B-Tree 满足一下条件:

1. 每个结点最多拥有 m 个子树，m-1 个 key
2. 根结点上至少有 2 个子树
3. 分支结点至少拥有 m/2 颗子树 (除根结点和叶子结点外都是分支结点)
4. 所有叶子结点都在同一层，每个结点可以有最多可以有 m-1 个 key，并且以升序排序

B-Tree 的查找:

**从根结点二分查找，找到返回对应的 data，否则继续在相应区间的指针指向的几点递归进行查找**

对于 N 个数据，一个 m 阶的 B-Tree，查找的时间复杂度为 $O(log_{m}{N})$。

### B+Tree

![b+tree](/image/b+tree.png)

m 阶 B-Tree 满足一下条件:

1. 每个结点最多拥有 m 个子树，m 个 key
2. 根结点和分支结点中不保存数据，只用于索引，所有数据都保存在叶子结点中
3. 所有分支节点和根节点都同时存在于子节点中，在子节点元素中是最大或者最小的元素
4. 叶子节点会包含所有的关键字，以及指向数据记录的指针，并且叶子节点本身是根据关键字的大小从小到大顺序链接

**一般在数据库系统或文件系统中使用的B+Tree结构都在经典B+Tree的基础上进行了优化，增加了顺序访问指针。**提高了区间访问的性能。

### 为什么用B/B+树？

- 局部性原理 (当一个数据被用到时，其附近的数据通常会被马上使用) 。预读的长度通常为页的整数倍，许多操作系统中页的大小通常为 4 K。
- 每次新建结点，直接申请一个页的空间。m 的大小通常取决于 key 和 point 和 data 的大小。 (由于 B+Tree 里内结点去掉了 data，可以拥有更大的出度)

    $$d_{max} = floor(\frac{pagesize}{keysize + datasize + pointsize})$$

## MySQL索引实现

### MyISAM

使用的是 B+Tree 作为索引结构。

MyISAM 仅仅保存数据记录的地址，就是**非聚族索引**，就是最后查出来的结果，仍然是一个指针，并不是连续的。

MyISAM 主索引和辅助索引没有任何区别，只是主索引要求 key 是唯一的，辅助索引可以重复。

### InnoDB

也是 B+Tree 作为索引结构的。

InnoDB 的数据文件本身就是索引文件。叶结点中包含了完整的数据记录，这个叫**聚族索引**。

InnoDB 本身要按主键聚集，所以要求表必须有主键，如果没有显示指定，MySQL 会自动选择第一个可以唯一标识数据 (Not NULL) 作为主键，如果没有这种列，就会自动生成一个隐含字段作为主键，字段长度 6 个字节，类型为长整型。

InnoDB 中的辅助索引最后用的是主键作为 data 域，所以辅助索引需要检索两次: 先找到主键索引，再用主键到主索引中获得记录。

所以不建议用过长的字段作为主键，这样会让辅助索引变得很大，而且非单调的主键在插入时，会导致 B+Tree 频繁地分裂。用自增主键其实是一个非常合适的选择。


## MySQL Handling of GROUP BY

上面扯远了。。跟最后的 Solution 没有关系。。只是 `aggregate` 这个单词重复出现了，我以为是同一个 `aggregate` 。。

最后文档里的意思是说在用了 `GROUP BY` 之后，是不能 select 非 `aggregate` 的 column 的，什么是 `aggregate` 呢，看一下 [Aggregate (GROUP BY) Function Descriptions](https://dev.mysql.com/doc/refman/8.0/en/group-by-handling.html) 就知道了。

另外在文档中提到，如果是 `functionally dependent on GROUP BY columns` 的 column，也是可以的。

> SQL99 and later permits such nonaggregates per optional feature T301 if they are functionally dependent on GROUP BY columns: If such a relationship exists between name and custid, the query is legal. This would be the case, for example, were custid a primary key of customers.

文档里有个例子:

> This query might be invalid with ONLY_FULL_GROUP_BY enabled because the nonaggregated address column in the select list is not named in the GROUP BY clause:

```sql
SELECT name, address, MAX(age) FROM t GROUP BY name;
```

> The query is valid if name is a primary key of t or is a unique NOT NULL column. In such cases, MySQL recognizes that the selected column is functionally dependent on a grouping column. For example, if name is a primary key, its value determines the value of address because each group has only one value of the primary key and thus only one row. As a result, there is no randomness in the choice of address value in a group and no need to reject the query.

总之就是 GROUP BY 没表达清楚意思，就不能用了，以前还能随便给你返回一个结果，现在 MySQL 改版了，遵守 `SQL92` 和更高版本的规范了，不能这么乱写了。

## Solution

如果就是想这样支持之前版本的话，可以修改 MySQL 配置。

- `select @@global.sql_mode`
- 去掉ONLY_FULL_GROUP_BY，重新设置值。`set @@global.sql_mode ='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION';`

## 总结

最近家里天气很鸡掰。。一会太阳，一会下雨。。简直有毛病。。

## References

- [12.19.3 MySQL Handling of GROUP BY](https://dev.mysql.com/doc/refman/8.0/en/group-by-handling.html)
- [12.19.1 Aggregate (GROUP BY) Function Descriptions](https://dev.mysql.com/doc/refman/8.0/en/group-by-functions.html)
- [MySQL的聚集索引和非聚集索引 - 仲夏的落叶](https://www.cnblogs.com/wyy123/p/6269875.html)
- [MySQL聚集索引和非聚集索引 - In_new](https://www.cnblogs.com/duzhentong/p/8639223.html)
- [MYSQL5.7版本sql_mode=only_full_group_by问题](https://www.cnblogs.com/zhi-leaf/p/5998820.html)
- [MySQL索引背后的数据结构及算法原理](http://blog.codinglabs.org/articles/theory-of-mysql-index.html)