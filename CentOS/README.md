# CentOS

## yum

```shell
yum -y update
yum -y install vim

# 1 安装 
yum install package  // 安装指定的安装包package 

# 2 更新和升级 
yum update  // 全部更新 
yum update package  // 更新指定程序包package
yum check-update  // 检查可更新的程序 
yum upgrade package  // 升级指定程序包package 

# 3 查找和显示 
yum info // 列出所有可以安装或更新的包的信息
yum info package //显示安装包信息package 
yum list // 显示所有已经安装和可以安装的程序包 
yum list package  // 显示指定程序包安装情况package
yum search package // 搜索匹配特定字符的package的详细信息

# 4 删除程序 
yum remove | erase package  // 删除程序包package
yum deplist package  // 查看程序package依赖情况

# 5 清除缓存 
yum clean packages  // 清除缓存目录下的软件包 
yum clean headers // 清除缓存目录下的 headers 
yum clean oldheaders // 清除缓存目录下旧的 headers 
yum clean, yum clean all  // (= yum clean packages; yum clean oldheaders) 清除缓存目录下的软件包及旧的headers
```

## 常用命令

1. 修改主机名、Hosts

```shell
## 修改主机名
hostnamectl set-hostname [your-hostname]

## 修改/etc/hosts, 修改后最好重启服务器
172.16.211.161  iZbp12zirv9d1ix6tp3wvqZ iZbp12zirv9d1ix6tp3wvqZ
# 修改为 ->
172.16.211.161  [your-hostname] [your-hostname]
```