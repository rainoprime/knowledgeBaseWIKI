# Docker的安装和使用

该项目许多软件都安装在Linux服务器上，这里我们使用虚拟机模拟服务器软件安装，这里使用的是 VMware + CentOS7

(虚拟机系统安装过程略)

下面命令注意在root用户下运行，避免重复 sudo 省略

```bash
su - root
sudu -i
```

# Docker

## 常用命令

```bash
##列出本地images
docker images
##含中间映像层
docker images -a
##搜索仓库MySQL镜像
docker search mysql

##下载Redis官方最新镜像，相当于：docker pull redis:latest
docker pull redis
##下载仓库所有Redis镜像
docker pull -a redis
##下载私人仓库镜像
docker pull bitnami/redis

##单个镜像删除，相当于：docker rmi redis:latest
docker rmi redis

##新建并启动容器，参数：-i  以交互模式运行容器；-t  为容器重新分配一个伪输入终端；--name  为容器指定一个名称
docker run -i -t --name mycentos
##后台启动容器，参数：-d  已守护方式启动容器
docker run -d mycentos

##启动一个或多个已经被停止的容器
docker start redis
##重启容器
docker restart redis

##查看正在运行的容器
docker ps
##查看正在运行的容器的ID
docker ps -q
##查看正在运行+历史运行过的容器
docker ps -a
##显示运行容器总文件大小
docker ps -s

##停止一个运行中的容器
docker stop redis
##杀掉一个运行中的容器
docker kill redis

#进入容器
docker exec -it redis redis-cli
```



## 安装Docker

> 参考：[Docker 安装文档](https://docs.docker.com/engine/install/centos/)

### 1. 删除老版本

```bash
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

### 2. 安装工具包并设置存储库

```bash
sudo yum install -y yum-utils

sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

### 3. 安装docker引擎

```bash
sudo yum install docker-ce docker-ce-cli containerd.io
```

## Docker使用

### 1. 启动Docker

```bash
sudo systemctl start docker
```

### 2.设置开机启动docker

1. 检查docker版本

```bash
docker -v
```

2. 查看docker已有镜像

```bash
sudo docker images
```

3. 设置docker开机启动

```bash
sudo systemctl enable docker
```

### 3. 设置国内镜像仓库

>参考：[阿里云镜像加速服务](https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors)

```bash
# 创建文件
sudo mkdir -p /etc/docker
# 修改配置, 设置镜像
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://*登陆自己的阿里云账号查勘标识*.mirror.aliyuncs.com"]
}
EOF
# 重启后台线程
sudo systemctl daemon-reload
# 重启docker
sudo systemctl restart docker
```

# MySQL of Docker

## 1. Docker安装MySQL

```bash
sudo docker pull mysql:5.7
```

## 2. Docker启动MySQL

```bash
sudo docker run -p 3306:3306 --name mysql \
-v /mydata/mysql/log:/var/log/mysql \
-v /mydata/mysql/data:/var/lib/mysql \
-v /mydata/mysql/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=root \
-d mysql:5.7
```

参数：

- -p 3306:3306：将容器的3306端口映射到主机的3306端口
- --name：给容器命名

- -v /mydata/mysql/log:/var/log/mysql：将配置文件挂载到主机/mydata/..
- -e MYSQL_ROOT_PASSWORD=root：初始化root用户的密码为root

查看docker启动的容器:

```bash
docker ps
```

## 3. 配置MySQL

1. 进入挂载的MySQL配置目录

```bash
cd /mydata/mysql/conf
```

2. 修改配置文件 `my.cnf`

```bash
vi my.cnf
```

拷贝以下内容：

```bash
[client]
default-character-set=utf8
[mysql]
default-character-set=utf8
[mysqld]
init_connect='SET collation_connection = utf8_unicode_ci'
init_connect='SET NAMES utf8'
character-set-server=utf8
collation-server=utf8_unicode_ci
skip-character-set-client-handshake
skip-name-resolve
```

3. Docker重启MySQL使配置生效

```BASH
docker restart mysql
```

# Redis of Docker

## 1. Docker拉取Redis镜像

```bash
docker pull redis
```

## 2. Docker启动Redis

1. 创建Redis配置文件目录

```bash
mkdir -p /mydata/redis/conf

touch /mydata/redis/conf/redis.conf
```

2. 启动Redis容器

```bash
docker run -p 6379:6379 --name redis \
-v /mydata/redis/data:/data \
-v /mydata/redis/conf/redis.conf:/etc/redis/redis.conf \
-d redis redis-server /etc/redis/redis.conf
```

## 3. 配置Redis持久化

>更多redis配置参考：[redis配置](https://raw.githubusercontent.com/redis/redis/6.0/redis.conf)

```bash
echo "appendonly yes"  >> /mydata/redis/conf/redis.conf

# 重启生效
docker restart redis
```

# 容器随docker启动自动运行

```bash
# mysql
docker update mysql --restart=always

# redis
docker update redis --restart=always
```

# Docker说明

Docker中每一个容器都是独立运行的，相当于一个独立的Linux系统，如果想便捷地修改容器内的文件，我们就需要把容器目录挂载到主机的目录上。容器端口类似，外界无法直接访问容器内部的端口，需要先将容器端口映射Linux主机端口上才能访问。

