# mysql

## mysql 查连接 IP

```
SELECT * FROM performance_schema.hosts;
show full processlist;
```

## mysql 查最大数量表

```
select table_name,table_rows,table_schema,table_comment from  information_schema.tables order by table_rows desc;
```

## 查询 user 字段在哪个库哪个表

```sql
SELECT 
    TABLE_SCHEMA AS database_name,
    TABLE_NAME AS table_name,
    COLUMN_NAME AS column_name
FROM 
    INFORMATION_SCHEMA.COLUMNS
WHERE 
    COLUMN_NAME LIKE '%user%';
```

## 统计访问过的表次数

```
//库名,表名,访问次数
select table_schema,table_name,sum(io_read_requests+io_write_requests) io from sys.schema_table_statistics group by table_schema,table_name order by io desc; 
```

## 查看写入权限

```
mysql> show global variables like '%secure%';
+------------------+-------+
| Variable_name    | Value |
+------------------+-------+
| secure_auth      | ON    |
| secure_file_priv |          |    可写入
| secure_file_priv | NULL |   不可写入
+------------------+-------+
```

```
SHOW VARIABLES LIKE "secure_file_priv";
```

- NULL，表示禁止。
- 如果value值有文件夹目录，则表示只允许该目录下文件，测试子目录也不行。
- 如果为空，则表示不限制目录。

## 不登录执行 sql

```
mysql -uaHmin -proot test -e "select now()" -N >H:/work/target1.txt
mysql -uroot -e "show databases;" >1.txt
```


## 基础命令

```
显示版本: select version();
显示字符集: select @@character_set_database;
显示数据库: show databases;
显示表名: show tables;
显示字段: show columns from table_name;
显示计算机名: select @@hostname;
系统版本: select @@version_compile_os;
mysql路径: select @@basedir;
数据库路径: select @@datadir;
describe describe table_name;
显示root密码: select User,Password from mysql.user;
导入文件: select load_fie(0x633A5C5C77696E646F77735C73797374656D33325C5C696E65747372765C5C6D657461626173652E786D6C);
导出文件: select 'testtest' into outfile '/var/www/html/test.txt' from mysql.user;
开启外连: GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'root' WITH GRANT OPTION;
mysql安装路径: show variables;   
更新数据库: UPDATE `DX15`.`dx15_common_member` SET `uid` = '1' WHERE `dx15_common_member`.`uid` =40407;更新40407uid变成uid1
mysql更改root密码: mysqladmin -u root password "newpwd";
查询表: select concat(User,0x3a,Password) from mysql.user; 
获取数据库所有表: SHOW TABLES FROM `databases`;
获取列前20行: SELECT * FROM `admin_bbs` ORDER BY 1 DESC LIMIT 0,20;
获取表行数: SELECT COUNT(*) AS CNT FROM `dede_admin`;
```