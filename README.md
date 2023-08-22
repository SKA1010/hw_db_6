# Домашнее задание к занятию «Репликация и масштабирование. Часть 1»

---

### Задание 1

На лекции рассматривались режимы репликации master-slave, master-master, опишите их различия.

*Ответить в свободной форме.*

---

### Задание 2

Выполните конфигурацию master-slave репликации, примером можно пользоваться из лекции.

*Приложите скриншоты конфигурации, выполнения работы: состояния и режимы работы серверов.*

### Решение 2

Прописываем конфиг в файле /etc/mysql/conf.d/mysql.cnf на Мастере: 
```
[mysqld]
bind-address=0.0.0.0
server-id = 1
log_bin = /var/log/mysql/mysql-bin.log
```
Прописываем конфиг в файле /etc/mysql/conf.d/mysql.cnf на Слейве: 
```
[mysqld]
bind-address=0.0.0.0
server-id = 2
log_bin = /var/log/mysql/mysql-bin.log
```
Создаём пользователя на Мастере, смотрим статус:
```sql
CREATE USER 'replica_user'@'%' IDENTIFIED WITH mysql_native_password BY '12345';
GRANT REPLICATION SLAVE ON *.* TO 'replica_user'@'%';
FLUSH PRIVILEGES;
show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000001 |      843 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```
Переходим к конфигурированию Slave:
```sql
CHANGE MASTER TO MASTER_HOST = '10.44.44.201', MASTER_USER = 'replica_user', MASTER_PASSWORD = '12345', MASTER_LOG_FILE = 'mysql-bin.00001', MASTER_LOG_POS = 843;
start slave;
```
Смотрим статус и текущие таблицы:
```sql
show slave status \G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for source to send event
                  Master_Host: 10.44.44.201
                  Master_User: replica_user
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000001
          Read_Master_Log_Pos: 843
               Relay_Log_File: test1-relay-bin.000002
                Relay_Log_Pos: 326
        Relay_Master_Log_File: mysql-bin.000001
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 843
              Relay_Log_Space: 536
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 1
                  Master_UUID: 25fd03b0-3601-11ee-8e30-00155d010813
             Master_Info_File: mysql.slave_master_info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Replica has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
         Replicate_Rewrite_DB: 
                 Channel_Name: 
           Master_TLS_Version: 
       Master_public_key_path: 
        Get_master_public_key: 0
            Network_Namespace: 
1 row in set, 1 warning (0.00 sec)

ERROR: 
No query specified

show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)
```
Переходим на Мастер и создаём таблицу:
```sql
create database test_hw;
```
Смотрим список таблиц на Slave и видим нашу таблицу:
```sql
show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| test_hw            |
+--------------------+
5 rows in set (0.01 sec)
```
Скриншоты:
Master:

![image](https://github.com/SKA1010/hw_db_6/assets/125235217/154d4d8c-411c-4d76-b9fb-065873556a5b)

Slave:

![image](https://github.com/SKA1010/hw_db_6/assets/125235217/5ba387c0-4ed3-46c7-a73f-9f5f08b07f36)

![image](https://github.com/SKA1010/hw_db_6/assets/125235217/a634b8b8-f7d5-4d41-a07c-2233816d949d)


