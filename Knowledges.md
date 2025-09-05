

[TOC]

------



## SQS -- Amazon SQS（Simple Queue Service）

1. #### 消息生命周期

   a. **Producer** 发送消息到队列（`SendMessage`）

   b. 消息进入队列，等待被消费

   c. **Consumer** 从队列取消息（`ReceiveMessage`）

   d. 处理完成后，调用 `DeleteMessage` 删除消息

   e. 如果处理失败且未删除，消息在“可见性超时”后会再次被消费

2. ####  FIFO队列

- 在 AWS SQS 中，**FIFO 队列** 是一种 **严格保证消息顺序且不重复消费的队列类型**，不同于默认的 **Standard 队列**。
- 必须命名为 `xxx.fifo`
- 每条消息必须设置：
  - MessageGroupId：同一个组内的消息严格顺序处理，不同组之间并发处理
  - MessageDeduplicationId：用于去重，5分钟内相同ID的消息不会被重复投递

3. #### Long Polling 长轮询

- 如果队列中暂时没有消息，**不会立即返回空结果，而是等一会（最多 20 秒）再看看有没有消息来。**响应时间长，空响应概率低，成本更低

4. #### 可见性超时（Visibility Timeout）

- 当一个消费者从队列中接收到一条消息后，这条消息会进入一个“不可见”状态 —— 在设定的时间内，其他消费者**看不到这条消息**，从而避免被重复消费。

- 举例：
  - 有一条消息 `M` 在队列中。
  - 消费者 A 拉取到 `M`，SQS 会让 `M` 变成“不可见”，进入 Visibility Timeout。
  - A 需要在这个时间窗口内处理完消息，并**显式调用 DeleteMessage** 把它从队列中删掉。
  - 如果 A 在超时时间内**没有删除消息**，SQS 会让 `M` 重新变得“可见”，其他消费者（如 B）就能再次拉取它。



------

## Negative cache

1. **Negative cache（负缓存）** 是一种缓存策略，用来 **缓存查询失败的结果**，比如：

- “这个数据不存在”
- “这个域名无法解析”
- “这个用户找不到”
- “这个文件没找到”

2. **🔍 为什么要用 Negative Cache？**

很多时候，不存在的数据被频繁查询，可能会导致资源浪费，比如：

- 每次都去数据库查一个根本没有的用户 → 浪费数据库连接
- 每次都请求一个根本没有的页面 → 增加延迟和负载

**负缓存可以避免重复查无效数据，提高系统性能。**



------

## select 常量

1. `SELECT 常量` 的核心作用之一是**快速判断查询条件是否有结果**

2. ```sql
   -- 用常量判断，高效
   SELECT 1 FROM users WHERE age > 30 LIMIT 1;
   ```

   - 若查询条件存在结果，则返回常量值 1
   - 若无结果，则返回空集
   -  `LIMIT 1` 会在找到第一条匹配记录后立即终止查询，性能更优

------

## 分页查询

1. Limit-Offset

   - ```go
     tx.Offset(offset).Limit(PageSize).Find(&dtos)
     ```

   - $$
     offset=(pageNum-1) * pageSize
     $$

2. cursor 分页（游标分页）



------

## git 命令

1. 可以用千问插件辅助生成提交message，用ctrl + k 打开
2. 新建分支
   - git checkout -b refactor-query-denylist
     - 开发新功能：feat-
       - 修复bug：fix-
     - 紧急修复：hotfix
     - 发布：release
     - 测试：test
     - 代码重构、优化：refactor
     - perf : 性能优化
3. 查看修改状态
   - git status
4. 查看文件差异
   - git diff
5. 添加要提交的文件
   - git add repo/mysql.go server/grpc.go
6. 提交修改文件到本地仓库
   - git commit
   - 用vi添加提交message
7. 提交文件到远程仓库
   - git push
8. 点击分支链接，在远程仓库创建分支，将创建好的分支链接发送到群里，等待合并
9. 拉取远程仓库(更新本地)
   -  git pull origin main  # 正确：origin 是远程仓库名，main 是分支名
   - git stash      # 暂存修改但未提交的文件
   - git stash pop # 变基完成后，恢复暂存的修改（可能需要解决冲突）

10. 查看本地分支
    - git branch

11. git rebase main    # main 分支需要是最新的

12. git pull origin refactor-central-log

13. git add -u  #git 同步删除

14. git commit --amend  #将这次提交与之前提交合并

15. git checkout/switch <分支名> #切换分支



------

## MySQL + Gorm

1. explain

   - 使用explain可以模拟优化器执行SQL查询语句，从而知道MySQL怎么处理你的SQL语句的，分析你的查询语句和表结构的性能瓶颈。
   - 使用而`explain`很简单就是，在你书写的SQL语句加一个单词 -`explain`，然后将 `explain `+ SQL执行后会出现一个表，这个表会告诉你MySQL优化器是怎样执行你的SQL的。
   - 各字段表示
     - id ：select查询的序列号，包含一组数字，表示查询中执行select子句或操作表的顺序
     - select_type ：查询类型 或者是 其他操作类型
     - table ：正在访问哪个表
     - partitions ：匹配的分区
     - type ：访问的类型
     - possible_keys ：显示可能应用在这张表中的索引，一个或多个，但不一定实际使用到
     - key ：实际使用到的索引，如果为NULL，则没有使用索引
     - key_len ：表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度
     - ref ：显示索引的哪一列被使用了，如果可能的话，是一个常数，哪些列或常量被用于查找索引列上的值
     - rows ：根据表统计信息及索引选用情况，大致估算出找到所需的记录所需读取的行数
     - filtered ：查询的表行占表的百分比
     - Extra ：包含不适合在其它列中显示但十分重要的额外信息

2. gorm中终结符（**如 `First`、`Find`、`Count` 等执行查询的方法**）不会清空查询条件

3. JOIN

   - `JOIN` 的核心是通过两个表中**共同的字段**（通常是主键和外键）建立关联，例如：
     - `users` 表（用户信息）和 `orders` 表（订单信息）通过 `user_id` 字段关联。
   - 内连接语法 Inner Join（join）
     - 只返回两个表中**满足关联条件**的记录。

   ```sql
   SELECT 表1.字段, 表2.字段
   FROM 表1
   INNER JOIN 表2 ON 表1.关联字段 = 表2.关联字段;
   ```

   - 左连接 Left Join

     - **作用**：返回**左表所有记录**，以及右表中满足关联条件的记录；右表不满足条件的部分用 `NULL` 填充。

     ```sql
     SELECT 表1.字段, 表2.字段
     FROM 表1  -- 左表
     LEFT JOIN 表2 ON 表1.关联字段 = 表2.关联字段;
     ```

     

   - 右连接 Right Join

     - 返回**右表所有记录**，以及左表中满足关联条件的记录；左表不满足条件的部分用 `NULL` 填充。

     ```sql
     SELECT 表1.字段, 表2.字段
     FROM 表1
     RIGHT JOIN 表2  -- 右表
     ON 表1.关联字段 = 表2.关联字段;
     ```

     

4. 增加字段

不加after，默认放在表尾

```sql
ALTER TABLE 表名
ADD COLUMN 字段名 数据类型 [约束条件] (AFTER 字段名);


ALTER TABLE users
ADD COLUMN email VARCHAR(255) NOT NULL DEFAULT 'unknown@example.com' AFTER name;
```

5. 读从库问题：（实现礼包码、激活码等功能时需注意）

如果使用了 create 或 update 、 delete 更新了主库然后立刻读数据，如果从 从库 中读取数据，会导致数据不一致的现象。此时，应该从主库中读取数据。

6. 基于游标的分页库（gorm-cursor-paginator），它会在底层把 SQL 的 LIMIT 设为 MaxResult+1，这是刻意为之：

   - 为了判断“是否还有下一页”

   - 查询 LIMIT = MaxResult + 1 条

   - 如果实际返回条数 > MaxResult，则判定有下一页：取前 MaxResult 条返回；多出来的第 MaxResult+1 条只用于生成下一页的游标（after/next），不返回给调用方

   - 如果返回条数 ≤ MaxResult，则判定没有下一页，NextToken 置空

   - 这样做能避免再发一条 COUNT 或 OFFSET 查询，性能和正确性更稳定







