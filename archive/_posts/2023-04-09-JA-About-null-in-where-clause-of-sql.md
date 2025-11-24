# 10. where a != 'sample' におけるNULLの挙動

先日アプリの不具合調査の際にコードを読んでいたが、よく理解できていなかったのでまとめてみました。
結論から言うとSQLのWHERE句で比較条件で = を使用する場合、NULLは評価されないです。

[こちら](https://docs.oracle.com/cd/E16338_01/server.112/b56299/sql_elements005.htm)の記事にもある通りですが、

> NULLはデータの欠落を表すため、任意の値や別のNULLとの関係で等号や不等号は成り立ちません。

以下でMySQLとPostgreSQLで実際にレコードを挿入して試してみました。

- MySQL

```
$ sudo docker run -e MYSQL_ROOT_PASSWORD=password mysql:latest
$ sudo docker exec -it d10510842887 bash
bash-4.4# mysql -u root -p password
Enter password: 
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES)
bash-4.4# 
bash-4.4# 
bash-4.4# 
bash-4.4# mysql -u root -ppassword
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 9
Server version: 8.0.32 MySQL Community Server - GPL

Copyright (c) 2000, 2023, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.03 sec)

mysql> create database sampledb;
Query OK, 1 row affected (0.05 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sampledb           |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

mysql> use sampledb;
Database changed
mysql> create table sample_table (
    -> id int,
    -> body varchar(50)
    -> );
Query OK, 0 rows affected (0.03 sec)

mysql> show tables;
+--------------------+
| Tables_in_sampledb |
+--------------------+
| sample_table       |
+--------------------+
1 row in set (0.01 sec)

mysql> insert into sample_table (id,body) values (1,'test1');
Query OK, 1 row affected (0.06 sec)

mysql> insert into sample_table (id,body) values (2,'test2');
Query OK, 1 row affected (0.04 sec)

mysql> insert into sample_table (id,body) values (3,'test3');
Query OK, 1 row affected (0.04 sec)

mysql> insert into sample_table (id,body) values (4,NULL);
Query OK, 1 row affected (0.04 sec)

mysql> insert into sample_table (id,body) values (5,NULL);
Query OK, 1 row affected (0.01 sec)

mysql> select * from sample_table;
+------+-------+
| id   | body  |
+------+-------+
|    1 | test1 |
|    2 | test2 |
|    3 | test3 |
|    4 | NULL  |
|    5 | NULL  |
+------+-------+
5 rows in set (0.00 sec)

mysql> select * from sample_tables where body is null;
ERROR 1146 (42S02): Table 'sampledb.sample_tables' doesn't exist
mysql> select * from sample_table where body is null;
+------+------+
| id   | body |
+------+------+
|    4 | NULL |
|    5 | NULL |
+------+------+
2 rows in set (0.01 sec)

mysql> select * from sample_table where body is not null;
+------+-------+
| id   | body  |
+------+-------+
|    1 | test1 |
|    2 | test2 |
|    3 | test3 |
+------+-------+
3 rows in set (0.00 sec)

mysql> select * from sample_table where body = null;
Empty set (0.01 sec)

mysql> select * from sample_table where body != null;
Empty set (0.00 sec)

mysql> select * from sample_table where body = 'test1';
+------+-------+
| id   | body  |
+------+-------+
|    1 | test1 |
+------+-------+
1 row in set (0.00 sec)

mysql> select * from sample_table where body != 'test1';
+------+-------+
| id   | body  |
+------+-------+
|    2 | test2 |
|    3 | test3 |
+------+-------+
2 rows in set (0.00 sec)
```

- PostgreSQl

```
$ sudo docker run -e POSTGRES_PASSWORD=password postgres:latest
$ sudo docker exec -it c8dc79e0f625 bash
root@c8dc79e0f625:/# psql -U postgres
psql (15.2 (Debian 15.2-1.pgdg110+1))
Type "help" for help.

postgres=# \l
                                                List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    | ICU Locale | Locale Provider |   Access privileges   
-----------+----------+----------+------------+------------+------------+-----------------+-----------------------
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            | 
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            | =c/postgres          +
           |          |          |            |            |            |                 | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            | =c/postgres          +
           |          |          |            |            |            |                 | postgres=CTc/postgres
(3 rows)
sampledb=# \dt
Did not find any relations.
sampledb=# create table sample_table (
sampledb(#   id serial,
sampledb(#   body varchar (50)
sampledb(# );
CREATE TABLE
sampledb=# \dt
            List of relations
 Schema |     Name     | Type  |  Owner   
--------+--------------+-------+----------
 public | sample_table | table | postgres
(1 row)

sampledb=# insert into sample_table (id,sample) values (1,'test1');
ERROR:  column "sample" of relation "sample_table" does not exist
LINE 1: insert into sample_table (id,sample) values (1,'test1');
                                     ^
sampledb=# insert into sample_table (id,body) values (1,'test1');
INSERT 0 1
sampledb=# insert into sample_table (id,body) values (2,'test2');
INSERT 0 1
sampledb=# insert into sample_table (id,body) values (3,'test3');
INSERT 0 1
sampledb=# insert into sample_table (id,body) values (4,NULL);
INSERT 0 1
sampledb=# insert into sample_table (id,body) values (5,NULL);
INSERT 0 1
sampledb=# select * from sample_table;
 id | body  
----+-------
  1 | test1
  2 | test2
  3 | test3
  4 | 
  5 | 
(5 rows)

sampledb=# select * from sample_table where body is null;
 id | body 
----+------
  4 | 
  5 | 
(2 rows)

sampledb=# select * from sample_table where body is not null;
 id | body  
----+-------
  1 | test1
  2 | test2
  3 | test3
(3 rows)

sampledb=# select * from sample_table where body = null;
 id | body 
----+------
(0 rows)

sampledb=# select * from sample_table where body != null;
 id | body 
----+------
(0 rows)

sampledb=# select * from sample_table where body = 'test1';
 id | body  
----+-------
  1 | test1
(1 row)

sampledb=# select * from sample_table where body != 'test1';
 id | body  
----+-------
  2 | test2
  3 | test3
(2 rows)
```

