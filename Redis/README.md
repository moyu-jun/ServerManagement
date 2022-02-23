# Redis

## 下载 Redis

在[Redis 官网](https://redis.io/download)下载 `Redis 6.2 Stable` 版本文件 `redis-6.2.6.tar.gz`。

然后将其解压，解压后复制 `redis.conf` 文件作为 Docker 安装时的 Redis 配置文件。

## 编写 docker-compose.yml

建议使用 Docker Compose 来管理应用，便于应用的启动、暂停、关闭等操作。比直接使用 docker run 更好。

`docker-compose.yml` 文件如下：

```yaml
version: '3'
services:
  redis:
    image: redis:6
    container_name: redis
    hostname: redis
    privileged: true
    restart: always
    environment:
      TZ: Asia/Shanghai
    ports:
      - 6379:6379
    volumes:
      - ./redis.conf:/etc/redis/redis.conf
      - ./data:/data
    command: [ "redis-server", "/etc/redis/redis.conf"]
```

复制 `redis.conf` 文件到 `docker-compose.yml` 同级目录下，并按照自己的需求修改配置即可。具体的配置项与常规安装一致。

`./data` 目录为数据目录，Redis 的 RDB 和 AOF 数据文件均在此目录下。

## 修改 redis.conf 

redis 配置需要修改部分配置项。修改内容如下：

```conf
# 开启外网访问
# bind 127.0.0.1 -::1

# 设置 RDB 数据持久化规则
save 3600 1
save 300 100
save 60 10000

# 设置密码
requirepass your_password

# 开启 AOF 持久化
appendonly yes
```

其他配置项按需修改即可。

## 安装

配置完成之后，将 `docker-compose.yml` 与 `redis.conf` 文件复制到服务器上。然后执行启动命令即可。

```shell
# 启动
docker-compose up -d

# 停止
docker-compose down

# 进入容器
docker exec -it redis bash
```
