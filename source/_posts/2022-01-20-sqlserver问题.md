---
title: sqlserver问题
catalog: true
comments: true
indexing: true
header-img: ../../../../img/default.jpg
top: false
tocnum: true
date: 2022-01-20 10:52:09
subtitle:
tags:
- 技术归纳
categories:
- 技术归纳
---

在执行create database test后，报错Could not obtain exclusive lock on database 'model'. Retry the operation later.。
通过在查看SQL Server的官方说明，显示引起该问题的原因是：
SQL Server在创建数据库时，会使用model数据库的副本来初始化数据库和元数据，此时必须以独占方式锁定model数据库防止从model数据库复制更改的数据，否则无法保证从数据一致性。
因此可以检查下，该model是否被其他会话占用：

```sql
IF
EXISTS(SELECT request_session_id FROM
sys.dm_tran_locks
WHERE resource_database_id =
DB_ID('Model'))
PRINT
'Model Database being used by some other session'
ELSE
PRINT
'Model Database not used by other session'
```
当model被其他会话占用时，可使用下面的语句查询是哪些会话占用
```sql
SELECT request_session_id FROM
sys.dm_tran_locks WHERE resource_database_id =DB_ID('Model');
DBCC INPUTBUFFER(spid)
```
结果显示有51占用，这时只需要移除掉即可创建数据库

```sql
kill 51
```