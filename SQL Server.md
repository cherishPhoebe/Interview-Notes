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
存储过程是一个预编译的sql语句，
