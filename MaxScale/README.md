# MaxScale

`MaxScale` 是 MariaDB 开发的一个中间件产品。它是一个智能的数据库代理中间件。在这里主要是用到了它的读写分离功能，实现 MySQL 主从集群的智能读写分离。

在安装 `MaxScale` 之前，请先安装一个 MySQL 集群。[install mysql cluster with docker](../mysql/README.md)

## docker-compose.yml

```yaml
version: "3"
services:
  maxscale:
    image: mariadb/maxscale:latest
    restart: always
    hostname: mxs
    container_name: mxs
    environment:
      - TZ=Asia/Shanghai
    volumes:
      - ./maxscale.cnf:/etc/maxscale.cnf
    ports:
      - 8989:8989
      - 4006:4006
```

`docker-compose.yml` 文件如上所示，需要添加一个配置文件 `maxscale.cnf`。另外开放两个端口。

* 8989: 此端口是 `MaxScale` 的 Web 管理端入口
* 4006: 此端口是 `MaxScale` MySQL 集群读写分离时访问的端口

## 准备工作

### 创建账号

### 准备默认配置文件

### 生成密码密钥

## 配置文件

`maxscale.cnf` 配置文件内容如下：

```cnf
# MaxScale documentation:
# https://mariadb.com/kb/en/mariadb-maxscale-25/

# MaxScale 自身的配置信息
[maxscale]
threads=auto
admin_host=0.0.0.0
admin_secure_gui=false

# MySQL 集群信息
# 主库
[mysql-master]
type=server
address=192.168.239.128
port=3306
protocol=MySQLBackend

# 从库 1
[mysql-slave1]
type=server
address=192.168.239.129
port=3306
protocol=MySQLBackend

# 从库 2
[mysql-slave2]
type=server
address=192.168.239.130
port=3306
protocol=MySQLBackend

# MySQL 集群监控服务

[MySQL-Monitor]
type=monitor
module=mysqlmon
servers=mysql-master,mysql-slave1,mysql-slave2
user=maxscale
password=4B6529D3E3353C59B4172CB3E903859E522E422BC2B0C6CC1550310F769A6DDAAEFD89D98566BB961E064C479A88B372
monitor_interval=2000

# 读写分离服务配置

[Read-Write-Service]
type=service
router=readwritesplit
servers=mysql-master,mysql-slave1,mysql-slave2
user=maxscale
password=4B6529D3E3353C59B4172CB3E903859E522E422BC2B0C6CC1550310F769A6DDAAEFD89D98566BB961E064C479A88B372

# 读写分离监听器配置

[Read-Write-Listener]
type=listener
service=Read-Write-Service
protocol=MySQLClient
# 外部访问集群的端口
port=4006
# 默认访问方式是 ipv6，配置此项修改为 ipv4，不然无法访问数据库
address=0.0.0.0
```