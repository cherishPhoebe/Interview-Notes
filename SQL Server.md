**触发器的作用是什么**  
触发器主要分为三类：
- 登录触发器
登录触发器降会响应`logon`事件而激发存储过程。 与 SQL Server 实例建立用户会话时将引发此事件。 登录触发器在登录的身份验证阶段完成后且用户会话建立前触发。
``` sql server
CREATE TRIGGER connection_limit_trigger 
  ON ALL SERVER
  WITH EXECUTE AS N'login_test'
FOR LOGON 
AS 
BEGIN
  IF ORIGINAL_LOGIN() = N'login_test'
    AND (
        SELECT COUNT(*)
        FROM sys.dm_exec_sessions
        WHERE is_user_process = 1
            AND original_login_name = N'login_test') > 3
    ROLLBACK;
END;
go
```
- DDL触发器  
DDL触发器响应数据定义语言(DDL)事件。这些事件主要与以关键字`CREATE`、`ALTER`、`DROP`、`GRANT`、`DENY`、`REVOKE`或`UPDATE STATISTICS 开头的 Transact-SQL `语句对应。执行DDL式操作的系统存储过程也可以触发DDL触发器。
- DML触发器  
DML触发器是一种特殊类型的存储过程，可在发生数据操作语言 (DML) 事件时自动生效，以便影响触发器中定义的表或视图。DML 事件包括 INSERT、UPDATE 或 DELETE 语句。DML 触发器可用于强制业务规则和数据完整性、查询其他表并包括复杂的 Transact-SQL 语句。 将触发器和触发它的语句作为可在触发器内回滚的单个事务对待。 如果检测到错误（例如，磁盘空间不足），则整个事务即自动回滚。  

DML触发器的优点：
- DML 触发器可以将更改通过级联方式传播给数据库中的相关表  
- DML 触发器可以防止恶意或错误的 INSERT、UPDATE 以及 DELETE 操作  
- DML 触发器可以评估数据修改前后表的状态，并根据该差异采取措施  
- 一个表中的多个同类 DML 触发器（INSERT、UPDATE 或 DELETE）允许采取多个不同的操作来响应同一个修改语句  

DML触发器的类型：
- AFTER  
适用于表，每个触发操作可以包含多个触发器，且可以指定第一个和最后一个执行  
- Instead of
适用于表和视图，每个触发操作包含一个触发器  

``` sql server
create trigger [Schema_Name].[trigger_name]
  on [Schema_Name].[table_name]
  after Insert|update|delete
as
begin
  -- 触发执行内容
end
go
```
通过使用 sp_settriggerorder 系统存储过程，可以将这些触发器中的任何一个指定为针对某事件激发的第一个或最后一个触发器。 SQL Server 不保证其余触发器的执行顺序。  

**什么是存储过程，用什么来调用**  
存储过程是一个预编译的sql语句，允许模块化设计，秩序创建一次，以后在该程序中可以多次调用。  

**存储过程的优点缺点**  

优点：
- 存储过程是预编译的，执行效率高  
- 存储过程的代码直接存放于数据库中，通过存储过程名直接调用，减少网络通讯  
- 安全性高，执行存储过程需要一定的用户权限  
- 存储过程可以重复利用，减少数据库开发人员工作量  

缺点：  
- 移植性差  

**索引**  
sql server 可用索引类型：  
|索引类型|说明|
|---|---|
|哈希|借助于哈希索引，可通过内存中的哈希表来访问数据。哈希索引的内存用量固定不变，是存储桶数量的函数|
|内存优化非聚集索引|对于内存优化的非聚集索引，内存使用量依赖于行计数以及索引键列的大小|
|聚集索引|聚集索引基于聚集索引键按顺序排序和存储表或视图中的数据行。 聚集索引按 B 树索引结构实现，B 树索引结构支持基于聚集索引键值对行进行快速检索。|
|非聚集索引|既可以使用聚集索引来为表或视图定义非聚集索引，也可以根据堆来定义非聚集索引。 非聚集索引中的每个索引行都包含非聚集键值和行定位符。 此定位符指向聚集索引或堆中包含该键值的数据行。 索引中的行按索引键值的顺序存储，但是不保证数据行按任何特定顺序存储，除非对表创建聚集索引。|
|唯一索引|唯一索引确保索引键不包含重复的值，因此，表或视图中的每一行在某种程度上是唯一的。唯一性可以是聚集索引和非聚集索引的属性。|  

- 什么是索引：  
索引是与表或试图关联的磁盘上的结构，可以加快

