



### MySQL Configuration File for Master-1:

```
[mysqld]

server-id = 1
bind-address = 0.0.0.0
port = 3306

log-bin = /var/lib/mysql/mysql-bin-master1.log
#binlog-do-db = your_database_name

default_authentication_plugin=mysql_native_password

max_connections = 1000
max_connect_errors = 10000
```


### MySQL Configuration File for Master-2:

```
[mysqld]

server-id = 2
bind-address = 0.0.0.0
port = 3306

log-bin = /var/lib/mysql/mysql-bin-master2.log
#binlog-do-db = your_database_name

default_authentication_plugin=mysql_native_password

max_connections = 1000
max_connect_errors = 10000
```


### Run the Docker Containers:

```
docker-compose up -d
```


```
docker ps

CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS                               NAMES
7d3ac6679263   mysql:8.0.34   "docker-entrypoint.s…"   2 minutes ago   Up 2 minutes   0.0.0.0:3307->3306/tcp, 33060/tcp   mysql_master2
4f8c758bf2c2   mysql:8.0.34   "docker-entrypoint.s…"   2 minutes ago   Up 2 minutes   0.0.0.0:3306->3306/tcp, 33060/tcp   mysql_master1

```


### Login to MySQL containers:
**On Master-1:**

```
mysql -h 192.168.0.6 -P 3306 -u root -p

or,

docker exec -it <container_name> bash

mysql -u root -p
```


```
show variables like 'server_id';

+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| server_id     | 1     |
+---------------+-------+
```


**On Master-2:**

```
mysql -h 192.168.0.6 -P 3307 -u root -p

or,

docker exec -it <container_name> bash

mysql -u root -p
```


```
show variables like 'server_id';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| server_id     | 2     |
+---------------+-------+
```



### Create Replication User:
Create a dedicated MySQL user for replication with the necessary privileges.


**On Master-1:**

```
create user 'replicator'@'%' identified with mysql_native_password by 'secret';

grant replication slave on *.* to 'replicator'@'%';

FLUSH PRIVILEGES;
```

```
SHOW GRANTS FOR 'replicator'@'%';
```

```
use mysql;
select user,host,plugin from user;
```


**On Master-2:**

```
create user 'replicator'@'%' identified with mysql_native_password by 'secret';

grant replication slave on *.* to 'replicator'@'%';

FLUSH PRIVILEGES;
```

```
SHOW GRANTS FOR 'replicator'@'%';
```

```
use mysql;
select user,host,plugin from user;
```



### Get the Log File and Position:
**On Master-1:**

```
show variables like 'log_bin%';
```


```
show binary logs;
```


```
show master status;

+--------------------------+----------+--------------+------------------+-------------------+
| File                     | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+--------------------------+----------+--------------+------------------+-------------------+
| mysql-bin-master1.000003 |      831 |              |                  |                   |
+--------------------------+----------+--------------+------------------+-------------------+
```


**On Master-2:**
```
show variables like 'log_bin%';
```


```
show binary logs;
```


```
show master status;

+--------------------------+----------+--------------+------------------+-------------------+
| File                     | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+--------------------------+----------+--------------+------------------+-------------------+
| mysql-bin-master2.000003 |      831 |              |                  |                   |
+--------------------------+----------+--------------+------------------+-------------------+
```




---
---



### Set up Replication On master-1 and master-2:
After start replica ensure that "Slave_IO_Running" and "Slave_SQL_Running" are both "Yes" on both mysql instances.


**On Master-1**

```
stop replica;
```


```
CHANGE REPLICATION SOURCE TO
SOURCE_HOST='192.168.0.6',
SOURCE_PORT = 3307,
SOURCE_USER='replicator',
SOURCE_PASSWORD='secret',
SOURCE_LOG_FILE='mysql-bin-master2.000003',
SOURCE_LOG_POS=831;
```


```
start replica;
```


**On Master-2**

```
stop replica;
```


```
CHANGE REPLICATION SOURCE TO
SOURCE_HOST='192.168.0.6',
SOURCE_PORT = 3306,
SOURCE_USER='replicator',
SOURCE_PASSWORD='admin',
SOURCE_LOG_FILE='mysql-bin-master1.000003',
SOURCE_LOG_POS=831;
```


```
start replica;
```



### Verify Replication:

```
show slave status\G;
```


```
SHOW REPLICAS;
```


**On Master-1**
```
create database db1;
show databases;
```


**On Master-2**
```
create database db2;
show databases;
```



