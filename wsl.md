### WSL操作

- #### 查看已安装的linux

- `wsl -l -v`

- #### 安装

- `wsl --install Ubuntu`

- #### 启动

- `wsl.exe -d Ubuntu`

- #### 关闭

- `wsl --shutdown`





------

### docker 操作

| 启动docker                                                   |
| ------------------------------------------------------------ |
| sudo service docker start                                    |
| **查看docker状态**                                           |
| systemctl status docker                                      |
| **开机自启动**                                               |
| systemctl enable docker                                      |
| **拉取镜像**                                                 |
| sudo docker pull mysql:8.0.28                                |
| **查看已有镜像**                                             |
| docker images                                                |
| **创建数据存储目录**                                         |
| # 创建一个 volume（只需创建一次）<br/>docker volume create mysql-data |
| **运行时挂载数据目录**                                       |
| sudo docker run -d  --name my-mysql -e MYSQL_ROOT_PASSWORD=123456 -v mysql-data:/var/lib/mysql mysql:8.0.28 |
| **停止docker**                                               |
| sudo systemctl stop docker   sudo systemctl stop docker.socket       sudo systemctl stop containerd  # 停止容器运行时 |
|                                                              |
| docker run -d --name mysql-container -e MYSQL_ROOT_PASSWORD=root -v  /mnt/e/dockerVolume/mysql:/var/lib/mysql -p 3306:3306 mysql:8.0.28 |
| 进入容器                                                     |
| docker exec -it redis-7.2 sh                                 |

docker run -d --name mysql-container -e MYSQL_ROOT_PASSWORD=root -v /mnt/e/dockerVolume/mysql:/var/lib/mysql -p 3306:3306 mysql:8.0.28

\#######

\# -d 后台运行容器

\#--name 容器名称

\# -e mysql密码

\# -v 将/var/lib/mysql挂载在windows中 /mnt/e/dockerVolume/mysql

\#-p端口号

\#mysql:8.0.28 使用的镜像



sudo docker start mysql-container \# 启动已存在的容器 

------

### Redis

1. **快速创建集群，start 命令将创建端口范围为 30001 到 30006 的集群**

```bash
cd ~/redis-stable/utils/create-cluster
./create-cluster create

```

2. **启动上述集群**

```bash
./create-cluster start
```

3. **连接到集群**

```bash
cd ~/redis-stable/src
./redis-cli -c -p 30001
```

4. **停止集群**

```bash
./create-cluster stop
```

5. **删除集群**

```bash
./create-cluster stop
./create-cluster clean
```

```bash
sudo docker run -d --name redis-cluster \

-e "IP=0.0.0.0" \

-e "ALLOW_EMPTY_PASSWORD=yes" \

-e "REDIS_NODES=redis" \

-e "REDIS_CLUSTER_CREATOR=yes" \

-p 7000-7005:7000-7005 grokzen/redis-cluster:7.2
```



\# -p 前一个7000-7005是外部端口，go程序中使用的这个端口，后面的是容器内部端口



- docker-compose:

```yml
version: '3.8'

services:
  localstack:
    image: localstack/localstack
    container_name: localstack
    ports:
      - "4566:4566"
      - "4571:4571"
    environment:
      - SERVICES=kms,s3,lambda,sqs
      - DEFAULT_REGION=us-east-1
    volumes:
      - "E:/dockerVolume/localstack:/var/lib/localstack"#容器持久化位置
      - "/var/run/docker.sock:/var/run/docker.sock"
      - "C:/Users/Administrator/.aws:/root/.aws" #本地aws配置


  redis-cluster:
  image: grokzen/redis-cluster:7.0.11
  container_name: redis-cluster
  environment:
    - IP=0.0.0.0
  ports:
    - "7000-7005:7000-7005"
    - "17000-17005:17000-17005"
  volumes:
    - /mnt/e/dockerVolume/redis/data/node-1:/data/7000
    - /mnt/e/dockerVolume/redis/data/node-2:/data/7001
    - /mnt/e/dockerVolume/redis/data/node-3:/data/7002
    - /mnt/e/dockerVolume/redis/data/node-4:/data/7003
    - /mnt/e/dockerVolume/redis/data/node-5:/data/7004
    - /mnt/e/dockerVolume/redis/data/node-6:/data/7005
  restart: always


```

- docker-compose运行：

```bash
docker-compose up -d
```

- 删除：

```
docker-compose down -d
```

- 创建单节点redis

```bash
sudo docker run -d --name redis-7.2 -p 6379:6379 -v /mnt/e/dockerVolume/redis-data:/data redis:7.2.0 redis-server --appendonly yes
```

- 进入容器：

```bash
docker exec -it redis-7.2 sh
```

------

### localstack

1. **启动localstack(后台)**

   ```bash
   localstack start (-d)
   ```

2. 创建本地SQS队列

   ```bash
   
   
   # 创建第一个 FIFO 队列
   awslocal sqs create-queue \
     --queue-name seayoo-order-payment.fifo \
     --attributes FifoQueue=true,ContentBasedDeduplication=true
   
   # 创建第二个 FIFO 队列
   awslocal sqs create-queue \
     --queue-name seayoo-payment-weixin-transfer.fifo \
     --attributes FifoQueue=true,ContentBasedDeduplication=true
   
   ```

   




------

### Linux设置环境变量

```bash
vim ~/.profile

#在 .profile中插入
export PATH=$PATH:/home/ws/go/bin

source ~/.profile 
```



------

### new一个grpc服务

1. 设置grpcRecovery参数

   ```go
   recoveryOpts := []grpcRecovery.Option{
   	...
   }
   ```

   

2. 设置grpc.Server参数

   ```go
   opts := []grpc.ServerOption{
   	...
   }
   ```

   

3. 获取grpcSever对象

   ```go
   grpcSrv := grpc.NewServer(opts...)
   
   ```

4. 新建指定Server对象

   ```go
   s:=&AccountServer{
   	...
   }
   ```

5. 注册服务

   ```go
   pb.RegisterAccountServer(grpcSrv,s)
   ```

   

6. 返回s

   ```go
   return s
   ```

------

### 数数分析埋点

1. 创建核心事件数据对象

   ```go
   eventData := &CoreEventData{}
   ```

   

2. 通过defer上传最终数据

   ```go
   defer s.eventCollector.Collect(ctx, eventData)
   ```

   

3. 构建特定事件收集

   ```go
   eventData.VerifyTokenEventBuilder(request.Device)
   eventData.WithOrigin(request.Origin)
   ```

   

4. 收集指定数据,如错误、数据来源、用户信息等

   ```go
   eventData.With...(...)
   ```




## 

### 生成ssh秘钥

1. wsl 生成 SSH KEY

```bash
cd ~
mkdir .ssh
cd .ssh
ssh-keygen -t rsa -C 'yourname@kingsoft.com'
```

2. 将上一步生成的 SSH 公私钥对复制到 windows 用户目录下的 .ssh 目录下

​	2.1 可以通过` explorer.exe .`  命令打开当前wsl目录在winows中文件夹



------

### 创建数据库

```sql
CREATE DATABASE account DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
use account;
```



------

### Ubuntu用apt安装Go最新版本

```bash
sudo add-apt-repository ppa:longsleep/golang-backports
sudo apt update
sudo apt install golang-go

echo 'export PATH="$PATH:$(go env GOPATH)/bin"' >> ~/.bashrc
source ~/.bashrc

```

------

### Jenkins -- jenkinsfile

1. 世游通行证：更新构建新代码
2. 世游平台：部署新代码 



------

### Nomad

