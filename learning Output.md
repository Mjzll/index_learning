# TiDB索引学习分享

##索引概念
##作用
- 在数据库中处理查询请求时，如果可以尽早的将无关数据过滤掉，那么后续的算子就可以少做无用功，提升整个 SQL 的执行效率。过滤数据最常用的手段是使用索引，TiDB 的优化器也会尽量采用索引过滤的方式处理请求，利用索引有序的特点来提升查询效率。

### 索引分类

#### 类型

- 聚集索引
  - 主键索引
- 非聚集索引（辅助索引）
  - 唯一索引
  - 非唯一索引

#### 字段

- 单列索引
- 组合索引

####  依据是否有索引包含sql中所有要查询的列

- 覆盖索引
- 非覆盖索引



## TiDB

### 存储特点

- **一个全局有序的分布式 Key-Value 引擎**

### 数据存储规则

- 存取行数据

  ```
  Key： tablePrefix_rowPrefix_tableID_rowID
  Value: [col1, col2, col3, col]
  ```

- 存取唯一索引

  ```text
  Key: tablePrefix_idxPrefix_tableID_indexID_indexColumnsValue
  Value: rowID
  ```

- 存取非唯一索引

  ```text
  Key: tablePrefix_idxPrefix_tableID_indexID_ColumnsValue_rowID
  Value：null
  ```

## 总结

- 创建索引
  - 索引列的选取，从where、on、groupby、orderby 中出现的列，其次要符合一下特点:

    - 查询频繁
    - 区分度高
    - 长度小
    - 尽量覆盖常用查询字段
    - 索引列不能存在大量空值

  - 创建组合索引时，注意列顺序，一般区分度高的在前

  - 创建索引有开销，不要随便创建无用索引

  - 考虑使用覆盖索引

    

- 使用索引

  - 避免在索引列上进行运算

    > *-- 全表扫描*  
    >
    > select * from article where id  + 1 = 5
    >
    > -- id列上有索引，id为varchar，不会走索引
    > *-- 全表扫描* 
    >
    > select * from article where id = 100  -- 字符串与数值进行比较时，自动将字符串转数字
    >
    > 

  - 使用组合索引，注意列顺序，避免中间某列出现范围查询

    > -- 创建(a,b,c) 组合索引
    >
    > -- 使用了a列
    > where a = 3
    >
    > -- 使用了a b列
    > where a = 3 and b = 5
    >
    > -- 使用了a b c列
    > where a = 3 and c = 4 and b = 5
    >
    > -- 没有使用索引
    > where b = 3
    >
    > -- 使用了a列 
    > where a = 3 and c = 4
    >
    > -- 使用了a b列 

  - 避免前导模糊查询

    > *-- 全表扫描* 
    >
    > select * from user where name like '%李%'

  - 谓语下沉，避免RPC操作

## 参考文章

- [索引范围计算简介](https://segmentfault.com/a/1190000015623896)
