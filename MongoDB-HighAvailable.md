## 1[安装Docker](https://docs.docker.com/get-started/)

## 2.制做docker镜像
### 2.1拉取镜像

如果不想自己制做镜像，可以直接拉取助教做好的镜像

执行下面的指令，然后直接到 *3配置MongoDB Replica Set*
```
docker pull registry.cn-hangzhou.aliyuncs.com/ybbh/mongodb-repl:latest
```
如果自己打镜像，按照下面的做法，

Docker拉取一个镜像，这里使用ubuntu，其它发行版也可以
```
docker pull ubuntu
```

### 2.2在容器里安装MongoDB

使用docker进入Linux Shell

```
docker run -it ubuntu:latest /bin/bash
```

[使用apt-get安装MongoDB](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/)

apt-key需要gnupg gnupg1 gnupg2
```
apt-get update
apt-get install gnupg gnupg1 gnupg2
apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 9DA31620334BD75D9DCB49F368818C72E52529D4
echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/4.0 multiverse" | tee /etc/apt/sources.list.d/mongodb-org-4.0.list
apt-get update
apt-get install -y mongodb-org
```

### 2.3保存docker镜像

确认安装MongoDB成功后，就可以把镜像保存下来，方便的复制多个副本，减少了很多重复的配置工作。

```
docker commit [container-id] mongodb-repl
```
container-id可以用docker ps命令查看


## 3配置MongoDB Replica Set

### 3.1更改配置文件
改一个更短的镜像名
```
docker tag registry.cn-hangzhou.aliyuncs.com/ybbh/mongodb-repl:latest mongodb-repl:latest
```

用拉取的镜像开启一个shell
```
docker run --interactive --tty mongodb-repl bash
```

在shell中编辑配置文件/etc/mongod.conf（这里是使用写配置文件的方式，在命令行里指定对应的参数也可）
replication选项加入replSetName
```
replication:
  replSetName: replset
```

绑定本机所有IP
```
net:
  port: 27017
  bindIp: 0.0.0.0
```

### 3.2保存docker镜像
```
docker commit [container-id] mongodb-repl
```

### 3.3创建docker子网

为了让MongoDB replica set中的各个实例能互连，这里用docker创建一个虚拟的子网，每个实例都可以拥有这个子网中的一个IP
```
docker network create --subnet 192.168.10.1/25 mongodb-repl
```

### 3.4在docker中启动mongodb

如果从镜像创建一个容器，执行下面的操作：
```
docker run --detach --net=mongodb-repl --ip 192.168.10.2 mongodb-repl mongod --config /etc/mongod.conf

docker run --detach --net=mongodb-repl --ip 192.168.10.3 mongodb-repl mongod --config /etc/mongod.conf

docker run --detach --net=mongodb-repl --ip 192.168.10.4 mongodb-repl mongod --config /etc/mongod.conf

```
这样，使用前面创建的子网，启动了3个mongodb的容器，每一个容器获得一个了IP

也可以用--publish选项把容器里的端口映射到宿主机的端口，这样就可以通过宿主机访问容器

如下面的命令把容器的27017端口映射为本地宿主的27017端口

```
docker run --detach --net=mongodb-repl --ip 192.168.10.20 --publish 27017:27017 registry.cn-hangzhou.aliyuncs.com/ybbh/mongodb-repl mongod --config /etc/mongod.conf

```
这里启动了3个mongodb的容器，使用前面创建的子网，每一个容器获得一个了IP

也可以把容器里的端口映射到本机的端口，这样就可以在宿主机方问容器

对于已经关闭的容器，想重新启动（同一个容器只能执行一个实例），执行：

```
docker start [container]
```

### 3.5初始化Replica Set
开启一个同网段的一个bash容器
```
 docker run --interactive --tty --net=mongodb-repl --ip 192.168.10.10 mongodb-repl bash
```

在容器开启mongo shell，连到replica set中的任意一台机器
```
mongo 192.168.10.2/local
```
MongoDB指定了replica set name，各个MongoDB实例需要知道彼此的host/ip和port

在mongo shell中初始化replica set, 执行
```
rs.initiate(
   {
      _id: "replset",
      version: 1,
      members: [
         { _id: 0, host : "192.168.10.2:27017" },
         { _id: 1, host : "192.168.10.3:27017" },
         { _id: 2, host : "192.168.10.4:27017" }
      ]
   }
)
```

### 3.6查看replica set配置和状态
```
rs.conf()
rs.status()
```

## 4 使用MongoDB高可用集群

以上节实验课的例子为例，访问上面我们建好的高可用replica set

```
java -jar JavaDriverExample-1.0-SNAPSHOT.jar mongodb://192.168.10.2:27017,192.168.10.3:27017,192.168.10.4:27017/?replicaSet=replset
```

了解以下内容:

1）[如何部署MongoDB Replica Set](https://docs.mongodb.com/manual/tutorial/deploy-replica-set/)

2）[Replication的概念](https://docs.mongodb.com/manual/replication/)

3）[使用程序连接访问MongoDB高可用集群](http://mongodb.github.io/mongo-java-driver/3.8/driver/tutorials/connect-to-mongodb/)

4）Replication有关的参数含义，如[write concern](https://docs.mongodb.com/manual/reference/write-concern/), [read concern](https://docs.mongodb.com/manual/reference/read-concern/)
