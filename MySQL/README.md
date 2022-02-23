# MySQL

mysql 主从集群的应用范围较为广泛，一般有两种同步方案，一种是传统的基于 binlog 进行主从复制，另一种则是基于 GTID 进行主从复制（也需要开启 binlog）。

## docker-compose.yml

三个节点的 `docker-compose.yml` 启动配置文件其实是完全一致的。

```yaml
version: '3'
services:
  mysql:
    image: mysql:8
    restart: always
    hostname: mysql
    container_name: mysql
    environment:
      - "MYSQL_ROOT_PASSWORD=your_password"
    ports:
      - "3306:3306"
    volumes:
      - ./mysql.cnf:/etc/mysql/my.cnf
      - ./data:/var/lib/mysql
```

如果需要添加 log 目录可自行添加。在下面的配置中，慢 SQL 日志的目录在 /var/lib/mysql 数据目录中，所以没有单独配置日志目录。

## 配置文件

配置文件主要区分主节点和从节点。

### 主节点

```cnf
# The MySQL  Client configuration file.
#
# For explanations see
# http://dev.mysql.com/doc/mysql/en/server-system-variables.html

[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
secure-file-priv= NULL

server-id=10
# 时区配置
default-time-zone='+08:00'
# 表名小写
lower-case-table-names=1

# GTID
gtid-mode=on
enforce-gtid-consistency=true
relay-log-recovery=true

# binlog
log-bin=master-bin
relay-log=relay-bin
log-slave-updates=true
slave-skip-errors=1062,1032
expire-logs-days=15

# SQL Mode setting
sql-mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES

# slow sql log
slow-query-log=ON
slow-query-log-file=/var/lib/mysql/mysql-slow.log
long-query-time=2

# 默认编码
character-set-server=utf8mb4
collation-server=utf8mb4_general_ci

[mysql]
default-character-set=utf8mb4
```

### 从节点

```cnf
# The MySQL  Client configuration file.
#
# For explanations see
# http://dev.mysql.com/doc/mysql/en/server-system-variables.html

[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
secure-file-priv= NULL

server-id=20
# 时区配置
default-time-zone='+08:00'
# 表名小写
lower-case-table-names=1

# 从库设置为只读, SUPER 权限的账号可写
read-only=1

# GTID
gtid-mode=on
enforce-gtid-consistency=true
relay-log-recovery=true

# binlog
log-bin=slave-bin
relay-log=relay-bin
log-slave-updates=true
slave-skip-errors=1062,1032
expire-logs-days=15

# SQL Mode setting
sql-mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES

# slow sql log
slow-query-log=ON
slow-query-log-file=/var/lib/mysql/mysql-slow.log
long-query-time=2

# 默认编码
character-set-server=utf8mb4
collation-server=utf8mb4_general_ci

[mysql]
default-character-set=utf8mb4
```

从节点注意 `server-id` ， `read-only` 和 `log-bin` 参数。`server-id` 要保证每个节点不一样。

## 启动

将三个节点的配置文件分别放置到三台服务器中，保证三台服务器内网 `3306` 端口可互通即可。启动命令如下：

```shell
# 启动
docker-compose up -d

# 停止
docker-compose down

# 进入容器
docker exec -it mysql bash

# 连接本地 MySQL
mysql -h127.0.0.1 -uroot -ppassword

# 检测状态
show master status;
```

`show master status;` 命令在主库执行后，会看到如下内容：

| File                    | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
| ----------------------- | -------- | ------------ | ---------------- | ----------------- |
| master-bin.000003       | 156      |              | mysql            |                   |

记录 **主数据库** `binlog` 的 **文件名称** 和 **数据同步起始位置**。

* File: mysql-master-bin.000003

* Position: 156

然后分别进入两个从库的 mysql 命令行，在从库上配置主库的信息，运行SQL如下：

## 配置主从同步

### 传统同步方式

```sql
-- 从库下运行
CHANGE MASTER TO
    MASTER_HOST='192.169.1.1',
    MASTER_USER='root',
    MASTER_PASSWORD='root_password',
    MASTER_LOG_FILE='master-bin.000003',
    MASTER_LOG_POS=156;

-- 启动 slave
start slave;

-- 检查从库状态
show slave status\G;
```

### GTID 同步方式（推荐）

```sql
-- 从库下运行
CHANGE MASTER TO
    MASTER_HOST='192.169.1.1',
    MASTER_USER='root',
    MASTER_PASSWORD='root_password',
	MASTER_PORT=3306,
    MASTER_AUTO_POSITION=1;

-- 启动 slave
start slave;

-- 检查从库状态
show slave status\G;
```

至此，主从同步就配置完成了。使用 `show slave status` 命令在从库可以检查同步状态。也可以在主库新增、修改、删除数据，然后在从库查看结果验证同步。

如果你需要对主从复制集群做读写分离的话，有很多种方案，可以在代码里实现，也可以使用中间件。这里推荐一个中间件 `maxscale` 做 MySQL 集群的读写分离。

[install maxscale with docker](../maxscale/README.md)