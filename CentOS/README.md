# CentOS

## 常用命令

1. 修改主机名、hosts

```shell
# 修改主机名
hostnamectl set-hostname [your-hostname]

# 修改/etc/hosts, 修改后最好重启服务器
172.16.211.161  iZbp12zirv9d1ix6tp3wvqZ iZbp12zirv9d1ix6tp3wvqZ
# 修改为 ->
172.16.211.161  [your-hostname] [your-hostname]
```

主机名修改之后最好重启一下。

2. 查看服务器外网 IP

```shell
# linux 命令
curl ifconfig.me

# HTTP 接口调用
GET http://ifconfig.me/ip
```

3. 上传下载文件

```shell
# 上传文件
scp /path/local_filename username@servername:/path

# 下载文件
scp username@servername:/path/filename /tmp/local_destination
```

4. 信息查看

```shell
# 查看系统性能消耗
# 大写 P 按CPU消耗排序
# 大写 M 按内存消耗排序
# 数字 1 查看每颗CPU核心使用率
top

# 查看端口占用
netstat -anp | grep 9100

# 查看目录大小，不递归子目录
du -h --max-depth=1 ./

# 查看目录大小，按大小排序
du -sh * | sort -hr

# 查看进程的可执行文件
ls -l /proc/$pid/exe

# 查看系统版本（Centos 版本）
cat /etc/redhat-release

# 查找文件
find / -name "file*"
```

5. 压缩解压

```shell
# 解压
tar -zxvf XXX.tar.gz

# 解压到指定目录
tar -zxvf XXX.tar.gz -C /xxx/xxx

# 压缩
tar -zcvf XXX.tar.gz *.jpg

# 安装 zip 解压工具
yum install -y unzip

# 解压 zip 文件
unzip xxx.zip
```

6. 安全相关

```shell
# 文件加锁解锁
lsattr filename
chattr -ia filename
chattr +ia filename

# 检索与删除开机启动项
chkconfig –list
chkconfig --del pwnrig
```

## yum

```shell
yum -y update
yum -y install vim

# 1. 安装
yum install package  // 安装指定的安装包package

# 2. 更新和升级
yum update  // 全部更新
yum update package  // 更新指定程序包package
yum check-update  // 检查可更新的程序
yum upgrade package  // 升级指定程序包package

# 3. 查找和显示
yum info // 列出所有可以安装或更新的包的信息
yum info package //显示安装包信息package
yum list // 显示所有已经安装和可以安装的程序包
yum list package  // 显示指定程序包安装情况package
yum search package // 搜索匹配特定字符的package的详细信息

# 4. 删除程序
yum remove | erase package  // 删除程序包package
yum deplist package  // 查看程序package依赖情况

# 5. 清除缓存
yum clean packages  // 清除缓存目录下的软件包
yum clean headers // 清除缓存目录下的 headers
yum clean oldheaders // 清除缓存目录下旧的 headers
yum clean, yum clean all  // (= yum clean packages; yum clean oldheaders) 清除缓存目录下的软件包及旧的headers
```

## 防火墙

```shell
# https://www.cnblogs.com/stulzq/p/9808504.html
# 启动防火墙
systemctl start firewalld

# 关闭防火墙
systemctl stop firewalld

# 重启防火墙
systemctl restart firewalld

# 查看服务状态
systemctl status firewalld

# 查看防火墙状态
firewall-cmd --state

# 重载防火墙配置
firewall-cmd --reload

# 查看默认区域的设置
firewall-cmd --list-all

# 应急命令
firewall-cmd --panic-on  # 拒绝所有流量，远程连接会立即断开，只有本地能登陆
firewall-cmd --panic-off  # 取消应急模式，但需要重启firewalld后才可以远程ssh
firewall-cmd --query-panic  # 查看是否为应急模式

# 端口操作
firewall-cmd --add-port=<port>/<protocol> #添加端口/协议（TCP/UDP）
firewall-cmd --remove-port=<port>/<protocol> #移除端口/协议（TCP/UDP）
firewall-cmd --list-ports #查看开放的端口

# 协议操作
firewall-cmd --add-protocol=<protocol> # 允许协议 (例：icmp，即允许ping)
firewall-cmd --remove-protocol=<protocol> # 取消协议
firewall-cmd --list-protocols # 查看允许的协议


# 防火墙开通端口命令
firewall-cmd --add-port=80/tcp --permanent
systemctl restart firewalld
# //查看端口防火墙是否开通
firewall-cmd --query-port=80/tcp
```

## 修改 SSH 端口

```shell
# 备份 SSH 配置文件
cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak

# 修改 SSH 配置
vim /etc/ssh/sshd_config

# 先保留 22 端口，添加一个新端口，确保新端口可连接后再禁用掉 22 端口
Port 22
Port 14444
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::

# 默认情况下，SELinux 只允许端口 22 用于 SSH，设置 SELinux 启用新创建的端口
semanage port -a -t ssh_port_t -p tcp 14444

# 验证 ssh 端口是否添加成功
semanage port -l | grep ssh

# 重启 ssh 服务
systemctl restart sshd.service

# 测试新端口
# 测试成功后删除旧端口
```

## 时间同步

参考地址: [https://zhuanlan.zhihu.com/p/156757418](https://zhuanlan.zhihu.com/p/156757418)

```shell
1、安装ntp:

yum install -y ntp

2、修改/etc/ntp.conf

# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (http://www.pool.ntp.org/join.html).

# new(新增)
server ntp1.aliyun.com prefer
server ntp2.aliyun.com

# old(原有)
server 0.centos.pool.ntp.org iburst
server 1.centos.pool.ntp.org iburst
server 2.centos.pool.ntp.org iburst
server 3.centos.pool.ntp.org iburst
3、设置时区

timedatectl set-timezone Asia/Shanghai

或者

cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

4、时间同步

手工发起同步：

ntpdate ntp1.aliyun.com

date查看时间是否已经同步

启动NTP服务：

systemctl start ntpd

设置开机启动：

systemctl enable ntpd

end 2020年7月5日11:33:50
```

## JDK 安装

前往 Oracle 官网下载最新版本的 JDK 8 的安装包，下载地址是：[https://www.oracle.com/java/technologies/downloads/#java8](https://www.oracle.com/java/technologies/downloads/#java8)

在网站中下载 linux rpm 安装包并上传到 CentOS 系统中。如当前最新版本: `jdk-8u321-linux-x64.rpm`。

按照下面的步骤和命令执行安装。

```shell
# 安装 JDK rpm
yum install -y jdk-8u321-linux-x64.rpm

# 编辑环境变量文件
vim /etc/profile

# 输入以下内容
# 此处需要注意 JAVA_HOME 所在路径的文件夹是否带有 -amd64
# JDK 8
JAVA_HOME=/usr/java/jdk1.8.0_321-amd64
CLASSPATH=.:%JAVA_HOME%/lib:%JAVA_HOME%/jre/lib
PATH=$PATH:$JAVA_HOME/bin:$JAVA_HOME/jre/bin
export PATH CLASSPATH JAVA_HOME

# 重新载入
source /etc/profile

# 版本检查
java -version
javac -version

# rpm 卸载
# 查看版本
rpm -qa|grep jdk

# 卸载对应版本
rpm -e jdk版本
```

## NodeJS 安装

前往 NodeJS 官网下载最新版本的安装包，下载地址为：[https://nodejs.org/zh-cn/download/](https://nodejs.org/zh-cn/download/)

下载 `Linux 二进制文件 (x64)` 版本安装包并上传到 CentOS 系统中。如当前最新版本: `node-v16.14.2-linux-x64.tar.xz`。

按照下面的步骤和命令执行安装。

```shell
# 解压缩
VERSION=v16.14.2
DISTRO=linux-x64
sudo mkdir -p /usr/local/lib/nodejs
sudo tar -xJvf node-$VERSION-$DISTRO.tar.xz -C /usr/local/lib/nodejs

# 编辑环境变量文件
vim /etc/profile

# 输入以下内容
# Nodejs
VERSION=v16.14.2
DISTRO=linux-x64
export PATH=/usr/local/lib/nodejs/node-$VERSION-$DISTRO/bin:$PATH

# 重新载入
source /etc/profile

# 安装后测试
node -v
npm version
npx -v
```

## Git 安装

```shell
# 安装 git
yum install -y git

# SSH RSA
ssh-keygen -t rsa
```

## Docker 安装

```shell
# 卸载老版本
yum remove docker \
                docker-client \
                docker-client-latest \
                docker-common \
                docker-latest \
                docker-latest-logrotate \
                docker-logrotate \
                docker-engine

# 设置仓库
yum install -y yum-utils

yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

# 安装 Docker ce
yum install -y docker-ce docker-ce-cli containerd.io

# 设置开机自启
systemctl enable docker

# 安装验证
docker -v

# 阿里云镜像加速
# https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors
mkdir -p /etc/docker
tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://38u6me2p.mirror.aliyuncs.com", "https://harbor.shiduai.com"]
}
EOF
systemctl daemon-reload
systemctl restart docker

# 启动 Docker
systemctl start docker
# 重启 Docker
systemctl restart docker
# 关闭 Docker
systemctl stop docker

# 更新
yum update docker-ce docker-ce-cli containerd.io

# 卸载
yum remove docker-ce docker-ce-cli containerd.io
# 手动删除配置文件
rm -rf /var/lib/docker
```

## Docker Compose 安装

```shell
# 下载 docker-compose 文件
# https://github.com/docker/compose/releases/download/1.29.2/docker-compose-Linux-x86_64
curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o cd /docker-compose

# 赋权
chmod +x /usr/local/bin/docker-compose

# 安装测试
docker-compose --version

# 卸载
rm /usr/local/bin/docker-compose
```
