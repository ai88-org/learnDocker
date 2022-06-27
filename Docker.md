# Docker

## 1、安装步骤

### 1.1、环境要求

- 64位

- CentOS 7, CentOS 8 (stream), or CentOS 9 (stream)（本次以Centos为例）

### 1.2、卸载旧版本docker

```sh
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

### 1.3、安装gcc

```sh
sudo yum -y install gcc
sudo yum -y install gcc-c++
```

### 1.4、设置仓库

```sh
sudo yum install -y yum-utils

sudo yum-config-manager \
--add-repo \
http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

### 1.5、更新一下yum索引

```sh
sudo yum makecache
```

### 1.6、安装docker引擎

```sh
sudo yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

### 1.7、启动docker

```sh
sudo systemctl start docker
```

### 1.8、测试一下

```sh
sudo docker run hello-world
```

## 2、阿里云容器镜像加速

https://cr.console.aliyun.com/cn-hangzhou/instances

写到这里的时候，还没有感觉配置这个有什么用处。

```sh
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://ncvr7xk8.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## 3、镜像命令

```sh
# 本地镜像
sudo docker images 
-a 列出本地所有镜像（含历史镜像层）
-q 只显示镜像id

# 查镜像
sudo docker search xxx镜像名称

# 下载镜像
sudo docker pull 镜像名称

sudo docker pull 镜像名称:Tag

# 查看镜像/容器/数据卷占的空间
sudo docker system df

# 删除镜像
docker rmi xxx镜像id
```

## 4、面试题：谈谈docker的虚悬镜像

仓库名和标签名都是<none>的镜像叫虚悬镜像。

## 5、容器命令

### 5.1、新建一个容器

容器由镜像中而来。

```sh
sudo docker run [options] image [command][arg...]
# options
--name="容器新名字"
-d 后台运行容器
-i 以交互模式运行容器，通常与-t联用
-t 为容器分配一个伪输入终端，通常与-i联用
-P 随机端口
-p 指定端口 -p 宿主机端口:容器端口
```

### 5.2、启动交互式容器(前台运行)

```sh
sudo docker run --name pen -it ubuntu /bin/bash
```

### 5.3、显示容器

```sh
# 或者的容器
sudo docker ps

# 所有容器
sudo docker ps -a

# 最近的容器
sudo docker ps -l

# 最近的n个容器
sudo docker ps -n 数字

# 容器编号
sudo docker ps -q
```

### 1.5.4、停止容器

```sh
# exit退出，当我们使用交互式启动容器之后，想要退出，使用exit命令的话，容器也会停止

# ctrl+p+q退出，容器不停止
```

### 1.5.5、启动已经停止的容器

```sh
sudo docker start 容器id或者容器名称
```

### 1.5.6、停止容器

```sh
sudo docker stop 容器id或者容器名
```

### 1.5.7、重启容器

```sh
sudo docker restart 容器id或者容器名
```

### 1.5.8、删除容器

```sh
 sudo docker rm 容器id或者容器名
```

### 1.5.9、删除正在运行的容器

```sh
sudo docker rm -f 容器id或者容器名
```

### 1.5.10、后台运行容器

```sh
# 挂起为后台进程(以redis为例)
sudo docker run -d redis:6.0.8
# 查看一下是否挂起来了
sudo docker ps
```

### 1.5.11、查看容器日志

```sh
sudo docker logs 容器id
```

### 1.5.12、查看容器进程

```sh
sudo docker top 容器id
```

### 1.5.13、查看容器内细节

```sh
sudo docker inspect 容器id
```

### 1.5.14、面试题：docker exec和docker attach的区别

```sh
docker exec进入容器之后，使用exit退出，容器不会停止，相当于新开了一个会话。
docker attach进入容器之后，使用exit退出，容器会停止。
工作中使用docker exec多。避免误关闭容器，造成影响。
```

### 1.5.15、容器文件到主机

```sh
docker cp 容器id:容器内文件地址 宿主机地址
```

### 1.5.16、导出容器

```sh
docker export 容器id > xxx.tar
```

### 1.5.17、导入容器

```sh
# 就是把上面的tar包打包成镜
# cat 文件名.tar | docker import - 镜像用户/镜像名:镜像版本号
cat aaa.tar | docker import - ai88/ubuntu:1.0
```

## 6、镜像分层

联合文件系统，bootfs共用宿主机的，rootfs用自己的。

容器层是可写的，容器层下面都是镜像层。目的就是为了复用。

### 6.1、commit案例

给原生的ubuntu增加vim功能

### 6.2、原生ubuntu镜像生成的容器没有vim功能

![image-20220621172658925](/Users/ai88/Library/Application Support/typora-user-images/image-20220621172658925.png)

### 6.3、在此基础上安装vim

![image-20220621173005285](/Users/ai88/Library/Application Support/typora-user-images/image-20220621173005285.png)

### 6.4、记录下容器id

退出容器，开始对记录下的容器id进行生成镜像的操作。

### 6.5、提交的命令

```sh
docker commit -m ="提交的描述信息" -a="作者" 容器id 要创建的目标镜像名称:TAG号
```

![image-20220621173855759](/Users/ai88/Library/Application Support/typora-user-images/image-20220621173855759.png)

### 6.6、新的镜像里面容器是否有vim命令

![image-20220621173954692](/Users/ai88/Library/Application Support/typora-user-images/image-20220621173954692.png)

### 6.7、提交镜像到阿里云仓库

先登录

![image-20220621184254993](/Users/ai88/Library/Application Support/typora-user-images/image-20220621184254993.png)

给需要推送的本地镜像打tag(类似于重命名)

```sh
docker tag [ImageId] registry.cn-hangzhou.aliyuncs.com/ai88images/justforlearn:[镜像版本号]
```

推送本地镜像到镜像仓库(感觉这个镜像仓库名字就是镜像名字)

```sh
docker push registry.cn-hangzhou.aliyuncs.com/ai88images/justforlearn:[镜像版本号]
```

## 7、搭建私有仓库

```sh
docker pull registry
```

### 7.1、启动私有仓库这个容器

```sh
# -p 宿主机端口:容器内端口
docker run -d -p 5000:5000  -v /zzyyuse/myregistry/:/tmp/registry --privileged=true registry
```

### 7.2、查看私有库上面的当前镜像

```sh
curl -XGET http://106.13.193.149:5188/v2/_catalog
```

### 7.3、重命名

```sh
docker tag ai88/ifconfigubuntu:1.1 172.17.0.1:5000/ifconfigubuntu:1.1
```

### 7.4、推送到远程仓库

```sh
docker push 172.17.0.1:5000/ifconfigubuntu:1.1
```

### 7.5、从远程仓库拉取到本地

```sh
 docker pull 172.17.0.1:5000/myubuntu:1.3
 docker pull 172.17.0.1:5000/ifconfigubuntu:1.1
```

## 8、容器卷

- 数据持久化
- 独立于容器生存周期

### 8.1、容器卷与主机互通互联

```sh
# 启动一个新的ubuntu容器实例，并加上容器卷
docker run -it --name u1 --privileged=true -v /root/host/u1:/root/docker/u1 ubuntu /bin/bash

# 在宿主机对应目录下面新建一个文件
touch a.txt
ls
a.txt
# 去容器内对应目录下面看看有没有a.txt
ls 
a.txt
```

宿主机内的文件

![image-20220622205743250](/Users/ai88/GoLearnNotes/docker/image-20220622205743250.png)

容器内生成文件

![image-20220622205641703](/Users/ai88/Library/Application Support/typora-user-images/image-20220622205641703.png)

### 8.2、容器卷的读写规则说明

不加规则默认就是rw，可读可写，例如：

```sh
docker run -it --name u1 --privileged=true -v /root/host/u1:/root/docker/u1 ubuntu /bin/bash
```

这个时候，我们在容器内修改文件也是会自动在宿主机生成。

### 8.3、容器内部只有只读权限

```sh
# ro代表read only
docker run -it --name u1 --privileged=true -v /root/host/u1:/root/docker/u1:ro ubuntu /bin/bash
```

此时我们再次尝试往容器里面挂载的那个目录里面文件写数据的话，就会提示如下：

![image-20220622210833448](/Users/ai88/Library/Application Support/typora-user-images/image-20220622210833448.png)

### 8.4、卷的继承和共享

```sh
# 先启动宿主机和第一个容器之间的卷共享
docker run -it --name u1 --privileged=true -v /root/host/u1:/root/docker/u1 ubuntu /bin/bash

# 启动第二个容器，继承第一个容器
docker run -it --name u2 --privileged=true --volumes-from u1 --name u2 ubuntu
```

停止u1或者直接删掉u1的容器实例，再启动或者再生成u1之后，u2和宿主机上所做的改动依旧会被自动同步。

那么假如我把u1删掉了，然后run的时候改掉容器内部的路径呢？还是一样。

假如我换掉宿主机的路径呢？那就完全不是一回事了。相当于这个u1已经挂掉了。

## 9、Docker安装tomcat

```sh
docker run -it -p 8080:8080 tomcat /bin/bash
```

## 10、Docker安装MySQL

```sh
# 启动一个MySQL实例，如果没有也能去仓库下载
docker run --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql:tag
```

## 11、Docker安装MySQL实战版

容器一旦被删除，数据库里面的数据就没了，这显然是不合适的。所以我们需要用到我们上面的容器卷的功能，将数据在宿主机持久化就行。

```sh
docker run -d -p 3306:3306 --privileged=true -v /root/mysql/log:/var/log/mysql -v /root/mysql/data:/var/lib/mysql -v /root/mysql/conf:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=123456  --name mysql mysql:latest
```

我们新建了一些数据，然后我们退出容器，并且删除容器，之后再创建容器，数据是否还在呢？

经过测试，还在！！！

## 12、Docker安装redis

```sh
# 进入到宿主机的目录下面，新建一个redis.conf文件

# 守护进程模式运行镜像，生成容器实例，记得要把上面一部的文件挂载到容器卷
# redis-server启动是读取的容器内被挂载的文件
docker run  -p 6379:6379 --name myr3 --privileged=true -v /root/host/redis/redis.conf:/etc/redis/redis.conf -v /root/host/redis/data:/data -d redis:6.0.8 redis-server /etc/redis/redis.conf
```

```sh
# 进入到容器执行一下命令
docker exec -it myr3 /bin/bash

# 连接一下redis
redis-cli

# 切换一下数据库
select 15

# 退出一下容器，修改一下宿主机的/root/host/redis/redis.conf的数据库数目
vim /root/host/redis/redis.conf

# 从16个改为10个

# 重启容器
docker restart myr3

# 再次进入容器，尝试select 15
# 发现直接报错
(error) ERR DB index is out of range
# 说明在宿主机修改的redis.conf文件生效了
```

## 13、MySQL主从复制Docker版本

主从复制的意义，一个主只负责写，从只负责读取。

这样假如主库中有锁表，也不会影响从库的读取。

### 13.1、新建主服务器容器实例

```sh
docker run -p 3307:3306 --name mysql-master \

--privileged=true \

-v /root/mysql-master/log:/var/log/mysql \

-v /root/mysql-master/data:/var/lib/mysql \

-v /root/mysql-master/conf:/etc/mysql/conf.d \

-e MYSQL_ROOT_PASSWORD=123456  \ 错误，写密码这里不要有其他乱七八糟的符号

-d mysql:latest

# 正确写法
docker run -p 3307:3306 --name mysql-master \
--privileged=true \
-v /root/mysql-master/log:/var/log/mysql \
-v /root/mysql-master/data:/var/lib/mysql \
-v /root/mysql-master/conf:/etc/mysql/conf.d \
-e MYSQL_ROOT_PASSWORD=123456  -d mysql
```

### 13.2、进入/root/mysql-master/conf目录创建my.cnf

```sh
[mysqld]

## 设置server_id，同一局域网中需要唯一

server_id=101 

## 指定不需要同步的数据库名称

binlog-ignore-db=mysql  

## 开启二进制日志功能

log-bin=mall-mysql-bin  

## 设置二进制日志使用内存大小（事务）

binlog_cache_size=1M  

## 设置使用的二进制日志格式（mixed,statement,row）

binlog_format=mixed  

## 二进制日志过期清理时间。默认值为0，表示不自动清理。

expire_logs_days=7  

## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。

## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致

slave_skip_errors=1062
```

### 13.3、重现启动master容器

```sh
docker restart mysql-master
```

### 13.4、进入mysql-master容器

```sh
docker exec -it mysql-master /bin/bash

mysql -uroot -p
```

### 13.5、在master容器实例内创建专门用于数据同步的用户slave

```sh
# 创建用户
CREATE USER 'slave'@'%' IDENTIFIED BY '123456';

# 赋予权限
GRANT REPLICATION SLAVE,REPLICATION CLIENT ON *.* TO ‘slave’@'%';
```

### 13.6、新建从服务器实例

```sh
docker run -p 3308:3306 --name mysql-slave \
-v /root/mysql-slave/log:/var/log/mysql \
-v /root/mysql-slave/data:/var/lib/mysql \
-v /root/mysql-slave/conf:/etc/mysql/conf.d \
-e MYSQL_ROOT_PASSWORD=root  -d mysql
```

### 13.7、新建从数据库的my.cnf

```sh
[mysqld]

## 设置server_id，同一局域网中需要唯一

server_id=102

## 指定不需要同步的数据库名称

binlog-ignore-db=mysql  

## 开启二进制日志功能，以备Slave作为其它数据库实例的Master时使用

log-bin=mall-mysql-slave1-bin  

## 设置二进制日志使用内存大小（事务）

binlog_cache_size=1M  

## 设置使用的二进制日志格式（mixed,statement,row）

binlog_format=mixed  

## 二进制日志过期清理时间。默认值为0，表示不自动清理。

expire_logs_days=7  

## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。

## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致

slave_skip_errors=1062  

## relay_log配置中继日志

relay_log=mall-mysql-relay-bin  

## log_slave_updates表示slave将复制事件写进自己的二进制日志

log_slave_updates=1  

## slave设置为只读（具有super权限的用户除外）

read_only=1
```

### 13.8、重启从实例

```sh
docker restart mysql-slave
```

### 13.9、在主数据库查看主从同步状态

```sh
show master status;
+-----------------------+----------+--------------+------------------+-------------------+
| File                  | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+-----------------------+----------+--------------+------------------+-------------------+
| mall-mysql-bin.000003 |      397 |              | mysql            |                   |
+-----------------------+----------+--------------+------------------+-------------------+
```

### 13.10、进入从容器配置主从复制

```sh
docker exec -it mysql-slave /bin/bash

mysql -uroot -p

change master to master_host='172.17.0.1',master_user='slave',master_password='123456',master_port=3307,master_log_file='mall-mysql-bin.000003',master_log_pos=397,master_connect_retry=30;

master_host：主数据库的IP地址；

master_port：主数据库的运行端口；

master_user：在主数据库创建的用于同步数据的用户账号；

master_password：在主数据库创建的用于同步数据的用户密码；

master_log_file：指定从数据库要复制数据的日志文件，通过查看主数据的状态，获取File参数；

master_log_pos：指定从数据库从哪个位置开始复制数据，通过查看主数据的状态，获取Position参数；

master_connect_retry：连接失败重试的时间间隔，单位为秒。
```

### 13.11、从数据库查看主从复制状态

```sh
show slave status \G;
```

### 13.12、从数据库开启主从同步

```sh
start slave;
```

### 13.13、再次查看从数据库的主从同步状态

```sh
show slave status \G;
```

### 13.14、主从复制测试

```sh
# 主数据库中写入数据，在从数据库中查看是否存在
```

## 14、安装Redis集群（大厂分布式面试题）

### 14.1、哈希取余算法

hash(key) % N，N就是节点数目

- 优点：负载均衡
- 缺点：
  - 如果节点数量不变化倒是没有什么问题，节点数量一旦发生了变化，由于公式是hash(key)/N，当N的数量发生变化后，结果也会发生变化。原先一个key经过运算之后是放在1号节点的，在我们扩容或者缩容之后，说不定就放在其他节点了。
  - 基本上只要有节点变动，所有的机器与节点之间的映射关系将全部重新洗牌。

### 14.2、一致性哈希算法

- 就是为了当服务器节点发生变化时候，尽量减少节点与数据直接映射关系变动

#### 14.2.1、步骤

- 算法构建一致性哈希环
- 将集群的IP或者主机名称经过hash算法，投射到上面的哈希环上。
- key在服务器上的落点规则：
  - 将key用上面IP或者主机名称一致的hash算法，算出值，落到哈希环上。
  - 上面的值，顺时针环绕，遇到的第一个节点就是该key落的节点。

#### 14.2.2、优点

- 容错性：假设Node C宕机，可以看到此时对象A、B、D不会受到影响，只有C对象被重定位到Node D。一般的，在一致性Hash算法中，如果一台服务器不可用，则受影响的数据仅仅是此服务器到其环空间中前一台服务器（即沿着逆时针方向行走遇到的第一台服务器）之间数据，其它不会受到影响。简单说，就是C挂了，受到影响的只是B、C之间的数据，并且这些数据会转移到D进行存储。

- 扩展性：数据量增加了，需要增加一台节点NodeX，X的位置在A和B之间，那收到影响的也就是A到X之间的数据，重新把A到X的数据录入到X上即可，

  不会导致hash取余全部数据重新洗牌。

#### 14.2.3、缺点

- 数据倾斜：一致性Hash算法在服务**节点太少时**，容易因为节点分布不均匀而造成**数据倾斜**（被缓存的对象大部分集中缓存在某一台服务器上）问题

### 14.3、哈希槽分区

- 哈希槽实质就是一个数组，数组[0,2^14 -1]形成hash slot空间
- 在数据和节点之间又加了一层，这层称为槽。相当于节点上放的槽，槽里面是数据，也就是key值。
- 槽解决的是粒度问题，相当于把粒度变大了，这样便于数据移动。
- 一个集群只能有16384个槽，编号0-16383（0-2^14-1）。这些槽会分配给集群中的所有主节点，分配策略没有要求。可以指定哪些编号的槽分配给哪个主节点。集群会记录节点和槽的对应关系。解决了节点和槽的关系后，接下来就需要对key求哈希值，然后对16384取余，余数是几key就落入对应的槽里。slot = CRC16(key) % 16384。以槽为单位移动数据，因为槽的数目是固定的，处理起来比较容易，这样数据移动问题就解决了。

### 14.4、 三主三从redis集群配置



#### 14.4.1、 步骤

##### 14.4.2、 新建6个docker容器

```sh
docker run -d --name redis-node-1 --net host --privileged=true -v /root/data/redis/redis-node-1:/data redis --cluster-enabled yes --appendonly yes --port 6381

docker run -d --name redis-node-2 --net host --privileged=true -v /root/data/redis/redis-node-2:/data redis --cluster-enabled yes --appendonly yes --port 6382

docker run -d --name redis-node-3 --net host --privileged=true -v /root/data/redis/redis-node-3:/data redis --cluster-enabled yes --appendonly yes --port 6383

docker run -d --name redis-node-4 --net host --privileged=true -v /root/data/redis/redis-node-4:/data redis --cluster-enabled yes --appendonly yes --port 6384

docker run -d --name redis-node-5 --net host --privileged=true -v /root/data/redis/redis-node-5:/data redis --cluster-enabled yes --appendonly yes --port 6385

docker run -d --name redis-node-6 --net host --privileged=true -v /root/data/redis/redis-node-6:/data redis --cluster-enabled yes --appendonly yes --port 6386
```

```sh
# 
--net host 的意思
1、加了--net=host以后就不需要再做端口映射了.比如docker容器内在8080端口起了一个web server.不加的话需要把本机的某个port比如7979和docker内的8080做一个映射关系,访问的时候访问7979. 加了net=host则直接访问8080

2、加了net=host后会使得创建的容器进入命令行好名称显示为主机的名称而不是一串id

3、容器中的app1无法访问到宿主机的app2，因为两者不在一个网络内。最简单的方式是在启动docker时增加–net=host选项，这样容器就和宿主机共用网络，容器中的app1也就能访问app2了
```

#### 14.4.3、进入docker容器构建主从关系

```sh
docker exec -it redis-node-1 /bin/bash

redis-cli --cluster create 192.168.0.4:6381 192.168.0.4:6382 192.168.0.4:6383 192.168.0.4:6384 192.168.0.4:6385 192.168.0.4:6386 --cluster-replicas 1
```

![image-20220627095330516](C:\Users\pythonbug\AppData\Roaming\Typora\typora-user-images\image-20220627095330516.png)

#### 14.4.4、以6381为切入点，查看集群状态

```sh
docker exec -it redis-node-1 /bin/bash

redis-cli -p 6381

cluster info

# 查看各个节点信息
cluster nodes
```

![image-20220627103413119](C:\Users\pythonbug\AppData\Roaming\Typora\typora-user-images\image-20220627103413119.png)

#### 14.4.5、 数据读写存储

```sh
docker exec -it redis-node-1 /bin/bash 
redis-cli -p 6381

set k1 v1

# 报错，12706槽位不在6381
(error) MOVED 12706 192.168.0.4:6383 

set k2 v2
ok

# 加入参数-c 优化路由
redis-cli -p 6381 -c

set k1 v1

-> Redirected to slot [12706] located at 192.168.0.4:6383
OK

# 查看集群数据信息
redis-cli --cluster check 192.168.0.4:6381
```

#### 14.4.6、 模拟机器宕机

让一台master机器发生宕机，看看从机是否会自动切换上来，并提供服务。

```sh
# 停止6382这台机器
docker stop redis-node-2

# 查看集群状态
docker exec -it redis-node-1 /bin/bash

redis-cli -p 6381 -c

cluster nodes
## output start 6382已经停止了，其从机6386变成新的master
1dbe18a4d676ec4ff9dc8d2bf5766060f8476801 192.168.0.4:6382@16382 master,fail - 1656296852978 1656296850000 2 disconnected
1f532d2d50706292f45b38d35cffd80aadee498c 192.168.0.4:6385@16385 slave d22f044f6e18659d554ba7df9fc14b8d6a279383 0 1656296877061 1 connected
420040a0fbf94995f5458b94cd650e3c533d8c4b 192.168.0.4:6383@16383 master - 0 1656296876058 3 connected 10923-16383
e33ed0261b23c912714017bf1dfb885bf06fc5b1 192.168.0.4:6384@16384 slave 420040a0fbf94995f5458b94cd650e3c533d8c4b 0 1656296874000 3 connected
d22f044f6e18659d554ba7df9fc14b8d6a279383 192.168.0.4:6381@16381 myself,master - 0 1656296875000 1 connected 0-5460
b39b7341dd887deaee13b5a150d61e256b042436 192.168.0.4:6386@16386 master - 0 1656296875055 7 connected 5461-10922
## output end

# 查看能否提供服务
set k4 v4
-> Redirected to slot [8455] located at 192.168.0.4:6386
OK

# 查看集群数据
redis-cli --cluster check 192.168.0.4:6381
Could not connect to Redis at 192.168.0.4:6382: Connection refused
192.168.0.4:6381 (d22f044f...) -> 2 keys | 5461 slots | 1 slaves.
192.168.0.4:6383 (420040a0...) -> 1 keys | 5461 slots | 1 slaves.
192.168.0.4:6386 (b39b7341...) -> 1 keys | 5462 slots | 0 slaves.
```

#### 14.4.7、 还原一下三主三从

```sh
# 启动6382
docker start redis-node-2

# 查看一下集群节点
docker exec -it redis-node-1 /bin/bash

redis-cli -p 6381 -c

cluster nodes

## output start 6382 变成了从，是6386的从
1dbe18a4d676ec4ff9dc8d2bf5766060f8476801 192.168.0.4:6382@16382 slave b39b7341dd887deaee13b5a150d61e256b042436 0 1656298476449 7 connected
1f532d2d50706292f45b38d35cffd80aadee498c 192.168.0.4:6385@16385 slave d22f044f6e18659d554ba7df9fc14b8d6a279383 0 1656298474444 1 connected
420040a0fbf94995f5458b94cd650e3c533d8c4b 192.168.0.4:6383@16383 master - 0 1656298475000 3 connected 10923-16383
e33ed0261b23c912714017bf1dfb885bf06fc5b1 192.168.0.4:6384@16384 slave 420040a0fbf94995f5458b94cd650e3c533d8c4b 0 1656298474000 3 connected
d22f044f6e18659d554ba7df9fc14b8d6a279383 192.168.0.4:6381@16381 myself,master - 0 1656298473000 1 connected 0-5460
b39b7341dd887deaee13b5a150d61e256b042436 192.168.0.4:6386@16386 master - 0 1656298475447 7 connected 5461-10922
## output end

# 停止6386
docker stop  redis-node-6

# 再启动6386
docker start  redis-node-6

# 查看一下
docker ps

# 查看集群节点
docker exec -it  redis-node-1 /bin/bash

redis-cli -p 6381 -c

cluster nodes

## output start 恢复成了初始状态 6382为master，6386为其slave
1dbe18a4d676ec4ff9dc8d2bf5766060f8476801 192.168.0.4:6382@16382 master - 0 1656298693087 8 connected 5461-10922
1f532d2d50706292f45b38d35cffd80aadee498c 192.168.0.4:6385@16385 slave d22f044f6e18659d554ba7df9fc14b8d6a279383 0 1656298694089 1 connected
420040a0fbf94995f5458b94cd650e3c533d8c4b 192.168.0.4:6383@16383 master - 0 1656298692000 3 connected 10923-16383
e33ed0261b23c912714017bf1dfb885bf06fc5b1 192.168.0.4:6384@16384 slave 420040a0fbf94995f5458b94cd650e3c533d8c4b 0 1656298693000 3 connected
d22f044f6e18659d554ba7df9fc14b8d6a279383 192.168.0.4:6381@16381 myself,master - 0 1656298690000 1 connected 0-5460
b39b7341dd887deaee13b5a150d61e256b042436 192.168.0.4:6386@16386 slave 1dbe18a4d676ec4ff9dc8d2bf5766060f8476801 0 1656298692000 8 connected
## output end
```

#### 14.4.8、额外搞事：把master及其slave都搞挂掉会发生什么

```sh
docker ps

docker stop redis-node-2

docker stop redis-node-6

docker ps

docker exec -it  redis-node-1 /bin/bash

redis-cli -p 6381 -c

cluster nodes
## output start
1dbe18a4d676ec4ff9dc8d2bf5766060f8476801 192.168.0.4:6382@16382 master,fail - 1656298863516 1656298859506 8 disconnected 5461-10922
1f532d2d50706292f45b38d35cffd80aadee498c 192.168.0.4:6385@16385 slave d22f044f6e18659d554ba7df9fc14b8d6a279383 0 1656298972000 1 connected
420040a0fbf94995f5458b94cd650e3c533d8c4b 192.168.0.4:6383@16383 master - 0 1656298973922 3 connected 10923-16383
e33ed0261b23c912714017bf1dfb885bf06fc5b1 192.168.0.4:6384@16384 slave 420040a0fbf94995f5458b94cd650e3c533d8c4b 0 1656298973000 3 connected
d22f044f6e18659d554ba7df9fc14b8d6a279383 192.168.0.4:6381@16381 myself,master - 0 1656298970000 1 connected 0-5460
b39b7341dd887deaee13b5a150d61e256b042436 192.168.0.4:6386@16386 slave,fail 1dbe18a4d676ec4ff9dc8d2bf5766060f8476801 1656298869533 1656298867526 8 disconnected
## output end

redis-cli --cluster check 192.168.0.4:6381

set k4 v44
## output start
127.0.0.1:6381> set k4 v44
(error) CLUSTERDOWN The cluster is down
## output end

## 再恢复一下
docker start redis-node-2

docker start redis-node-6

docker exec -it  redis-node-1 /bin/bash

redis-cli -p 6381 -c

redis-cli --cluster check 192.168.0.4:6381

set k4 v44
## output start
-> Redirected to slot [8455] located at 192.168.0.4:6382
OK
## output end
```

#### 14.4.9、 主从扩容

```sh
# 新建两个redis的docker容器 6387(master)、6388(slave)
docker run -d --name redis-node-7 --net host --privileged=true -v /root/data/redis/redis-node-7:/data redis --cluster-enabled yes --appendonly yes --port 6387

docker run -d --name redis-node-8 --net host --privileged=true -v /root/data/redis/redis-node-8:/data redis --cluster-enabled yes --appendonly yes --port 6388

# 进入6387容器实例里面
docker exec -it redis-node-7 /bin/bash

# 将6387作为master节点加入原先集群
redis-cli --cluster add-node 192.168.0.4:6387 192.168.0.4:6381

# 再次检查集群
redis-cli --cluster check 192.168.0.4:6381

# 已经加入节点，但是没有分配槽位
192.168.0.4:6387 (7554ee22...) -> 0 keys | 0 slots | 0 slaves. 

# 重新分配槽位
redis-cli --cluster reshard 192.168.0.4:6381

# 重新检查集群
redis-cli --cluster check 192.168.0.4:6381

## outpu start
192.168.0.4:6381 (d22f044f...) -> 1 keys | 4096 slots | 1 slaves.
192.168.0.4:6387 (7554ee22...) -> 1 keys | 4096 slots | 0 slaves.
192.168.0.4:6382 (1dbe18a4...) -> 1 keys | 4096 slots | 1 slaves.
192.168.0.4:6383 (420040a0...) -> 1 keys | 4096 slots | 1 slaves.
[OK] 4 keys in 4 masters.
0.00 keys per slot on average.
>>> Performing Cluster Check (using node 192.168.0.4:6381)
M: d22f044f6e18659d554ba7df9fc14b8d6a279383 192.168.0.4:6381
   slots:[1365-5460] (4096 slots) master
   1 additional replica(s)
M: 7554ee22a753a0640c652035f5bf6abd5547dbeb 192.168.0.4:6387
   slots:[0-1364],[5461-6826],[10923-12287] (4096 slots) master
M: 1dbe18a4d676ec4ff9dc8d2bf5766060f8476801 192.168.0.4:6382
   slots:[6827-10922] (4096 slots) master
   1 additional replica(s)
S: 1f532d2d50706292f45b38d35cffd80aadee498c 192.168.0.4:6385
   slots: (0 slots) slave
   replicates d22f044f6e18659d554ba7df9fc14b8d6a279383
M: 420040a0fbf94995f5458b94cd650e3c533d8c4b 192.168.0.4:6383
   slots:[12288-16383] (4096 slots) master
   1 additional replica(s)
S: e33ed0261b23c912714017bf1dfb885bf06fc5b1 192.168.0.4:6384
   slots: (0 slots) slave
   replicates 420040a0fbf94995f5458b94cd650e3c533d8c4b
S: b39b7341dd887deaee13b5a150d61e256b042436 192.168.0.4:6386
   slots: (0 slots) slave
   replicates 1dbe18a4d676ec4ff9dc8d2bf5766060f8476801
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
## output end

# 为主节点6387分配从节点6388
# redis-cli --cluster add-node ip:新slave端口 ip:新master端口 --cluster-slave --cluster-master-id 新主机节点ID
redis-cli --cluster add-node 192.168.0.4:6388 192.168.0.4:6387 --cluster-slave --cluster-master-id 7554ee22a753a0640c652035f5bf6abd5547dbeb

# 再次检查
127.0.0.1:6381> cluster nodes
## output start
7554ee22a753a0640c652035f5bf6abd5547dbeb 192.168.0.4:6387@16387 master - 0 1656315630000 9 connected 0-1364 5461-6826 10923-12287
1dbe18a4d676ec4ff9dc8d2bf5766060f8476801 192.168.0.4:6382@16382 master - 0 1656315630746 8 connected 6827-10922
1f532d2d50706292f45b38d35cffd80aadee498c 192.168.0.4:6385@16385 slave d22f044f6e18659d554ba7df9fc14b8d6a279383 0 1656315631749 1 connected
420040a0fbf94995f5458b94cd650e3c533d8c4b 192.168.0.4:6383@16383 master - 0 1656315630000 3 connected 12288-16383
e33ed0261b23c912714017bf1dfb885bf06fc5b1 192.168.0.4:6384@16384 slave 420040a0fbf94995f5458b94cd650e3c533d8c4b 0 1656315632752 3 connected
d22f044f6e18659d554ba7df9fc14b8d6a279383 192.168.0.4:6381@16381 myself,master - 0 1656315628000 1 connected 1365-5460
4fc9e233f9d41e1c9857a7b18a489167b7ed4776 192.168.0.4:6388@16388 slave 7554ee22a753a0640c652035f5bf6abd5547dbeb 0 1656315629000 9 connected
b39b7341dd887deaee13b5a150d61e256b042436 192.168.0.4:6386@16386 slave 1dbe18a4d676ec4ff9dc8d2bf5766060f8476801 0 1656315630000 8 connected
## output end
```

#### 14.4.10、 主从缩容

6387和6388下线，变成三主三从，槽位归还原来集群的节点

```sh
# 获得6387和6388的节点id
7554ee22a753a0640c652035f5bf6abd5547dbeb
4fc9e233f9d41e1c9857a7b18a489167b7ed4776

# 先下线6388节点 redis-cli --cluster del-node ip:从机端口 从机6388节点ID
docker exec -it redis-node-1 /bin/bash 
redis-cli --cluster del-node 192.168.0.4:6388 4fc9e233f9d41e1c9857a7b18a489167b7ed4776

# 再次检查 还剩下7台
redis-cli --cluster check 192.168.0.4:6381

# 再次分配槽位 将多余的槽位给6381
redis-cli --cluster reshard 192.168.0.4:6381

# 再次检查槽位 发现6387已经没有槽位分配了
redis-cli --cluster check 192.168.0.4:6381
## output start
192.168.0.4:6381 (d22f044f...) -> 2 keys | 8192 slots | 1 slaves.
192.168.0.4:6387 (7554ee22...) -> 0 keys | 0 slots | 0 slaves.
192.168.0.4:6382 (1dbe18a4...) -> 1 keys | 4096 slots | 1 slaves.
192.168.0.4:6383 (420040a0...) -> 1 keys | 4096 slots | 1 slaves.
[OK] 4 keys in 4 masters.
0.00 keys per slot on average.
>>> Performing Cluster Check (using node 192.168.0.4:6381)
M: d22f044f6e18659d554ba7df9fc14b8d6a279383 192.168.0.4:6381
   slots:[0-6826],[10923-12287] (8192 slots) master
   1 additional replica(s)
M: 7554ee22a753a0640c652035f5bf6abd5547dbeb 192.168.0.4:6387
   slots: (0 slots) master
M: 1dbe18a4d676ec4ff9dc8d2bf5766060f8476801 192.168.0.4:6382
   slots:[6827-10922] (4096 slots) master
   1 additional replica(s)
S: 1f532d2d50706292f45b38d35cffd80aadee498c 192.168.0.4:6385
   slots: (0 slots) slave
   replicates d22f044f6e18659d554ba7df9fc14b8d6a279383
M: 420040a0fbf94995f5458b94cd650e3c533d8c4b 192.168.0.4:6383
   slots:[12288-16383] (4096 slots) master
   1 additional replica(s)
S: e33ed0261b23c912714017bf1dfb885bf06fc5b1 192.168.0.4:6384
   slots: (0 slots) slave
   replicates 420040a0fbf94995f5458b94cd650e3c533d8c4b
S: b39b7341dd887deaee13b5a150d61e256b042436 192.168.0.4:6386
   slots: (0 slots) slave
   replicates 1dbe18a4d676ec4ff9dc8d2bf5766060f8476801
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
## output end

# 再将6387节点删除
redis-cli --cluster del-node 192.168.0.4:6381 7554ee22a753a0640c652035f5bf6abd5547dbeb

# 再次检查
```







