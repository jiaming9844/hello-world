# 什么是zookeeper？

简单描述：

- Zookeeper 可以被用作注册中心。

- Zookeeper 是 Hadoop 生态系统的一员；

* 构建 Zookeeper 集群的时候，使用的服务器最好是奇数台。

ZooKeeper 概览：

ZooKeeper 的设计目标是将那些复杂且容易出错的分布式一致性服务封装起来，构成一个高效可靠的原语集，并以一系列简单易用的接口提供给用户使用。

ZooKeeper 是一个典型的分布式数据一致性解决方案，分布式应用程序可以基于 ZooKeeper 实现诸如数据发布/订阅、负载均衡、命名服务、分布式协调/通知、集群管理、Master 选举、分布式锁和分布式队列等功能。

Zookeeper 一个最常用的使用场景就是用于担任服务生产者和服务消费者的注册中心。

服务生产者将自己提供的服务注册到Zookeeper中心，服务的消费者在进行服务调用的时候先到Zookeeper中查找服务，获取到服务生产者的详细信息之后，再去调用服务生产者的内容与数据。如下图所示，在 Dubbo架构中 Zookeeper 就担任了注册中心这一角色。


<img src="https://github.com/jiaming9844/hello-world/blob/master/zookeeper/img/1063359856-5b973176b0f9c_articlex.png"/>

# 重要概念总结

ZooKeeper 本身就是一个分布式程序（只要半数以上节点存活，ZooKeeper 就能正常服务）。

为了保证高可用，最好是以集群形态来部署 ZooKeeper，这样只要集群中大部分机器是可用的（能够容忍一定的机器故障），那么 ZooKeeper 本身仍然是可用的。

ZooKeeper 将数据保存在内存中，这也就保证了 高吞吐量和低延迟（但是内存限制了能够存储的容量不太大，此限制也是保持znode中存储的数据量较小的进一步原因）。

ZooKeeper 是高性能的。 在“读”多于“写”的应用程序中尤其地高性能，因为“写”会导致所有的服务器间同步状态。（“读”多于“写”是协调服务的典型场景。）

ZooKeeper有临时节点的概念。 当创建临时节点的客户端会话一直保持活动，瞬时节点就一直存在。而当会话终结时，瞬时节点被删除。持久节点是指一旦这个ZNode被创建了，除非主动进行ZNode的移除操作，否则这个ZNode将一直保存在Zookeeper上。

ZooKeeper 底层其实只提供了两个功能：①管理（存储、读取）用户程序提交的数据；②为用户程序提交数据节点监听服务。


# zookeeper 集群环境搭建

1、目前有三台服务器分别为231，232，233。下载并解压 zookeeper-3.4.10.tar.gz （jdk已安装）

2、将conf目录下zoo_sample.cfg配置文件修改为zoo.conf。

3、修改zoo.conf。

````
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just
# example sakes.
#修改这部分路径，修改之前必须新建好相应的文件夹。（环境变量的方式不好用，以后再尝试）
dataDir=/usr/local/druid/apache-druid-0.16.0-incubating/zk/data
# the port at which the clients will connect
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1

#新加三台服务器信息
server.1=.233:2888:3888
server.2=.232:2888:3888
server.3=.231:2888:3888
````
　　①、tickTime：基本事件单元，这个时间是作为Zookeeper服务器之间或客户端与服务器之间维持心跳的时间间隔，每隔tickTime时间就会发送一个心跳；最小 的session过期时间为2倍tickTime

　　②、dataDir：存储内存中数据库快照的位置，除非另有说明，否则指向数据库更新的事务日志。注意：应该谨慎的选择日志存放的位置，使用专用的日志存储设备能够大大提高系统的性能，如果将日志存储在比较繁忙的存储设备上，那么将会很大程度上影像系统性能。

　　③、client：监听客户端连接的端口。

　　④、initLimit：允许follower连接并同步到Leader的初始化连接时间，以tickTime为单位。当初始化连接时间超过该值，则表示连接失败。

　　⑤、syncLimit：表示Leader与Follower之间发送消息时，请求和应答时间长度。如果follower在设置时间内不能与leader通信，那么此follower将会被丢弃。

　　⑥、server.A=B:C:D

　　　　A：其中 A 是一个数字，表示这个是服务器的编号；

　　　　B：是这个服务器的 ip 地址；

　　　　C：Leader选举的端口；

　　　　D：Zookeeper服务器之间的通信端口。

　　我们需要修改的第一个是 dataDir ,在指定的位置处创建好目录。

　　第二个需要新增的是 server.A=B:C:D 配置，其中 A 对应下面我们即将介绍的myid 文件。B是集群的各个IP地址，C:D 是端口配置。

4、创建myid文件，然后在该文件添加上一步 server 配置的对应 A 数字。
　　下面配置是：
    server.1=.233:2888:3888
    server.2=.232:2888:3888
    server.3=.231:2888:3888
　　那么就必须在 .233 机器的的 dataDir 目录下创建 myid 文件，然后在该文件中写上 1 即可。同理232机器myid为2。
  
5、配置环境变量

　　为了能够在任意目录启动zookeeper集群，我们需要配置环境变量。

　　ps:你也可以不配，这不是搭建集群的必要操作，只不过如果你不配置环境变量，那么每次启动zookeeper需要到安装文件的 bin 目录下去启动。

　　首先进入到 /etc/profile 目录，添加相应的配置信息：

    #set zookeeper environment
    export ZK_HOME=/usr/local/software/zookeeper-3.3.6
    export PATH=$PATH:$ZK_HOME/bin
　　然后通过如下命令使得环境变量生效：

    source /etc/profle

6、启动zookeeper服务

　　启动命令：

    zkServer.sh start
　　停止命令：

    zkServer.sh stop
　　重启命令：

    zkServer.sh restart
　　查看集群节点状态：

    zkServer.sh status
　　我们分别对集群三台机器执行启动命令。执行完毕后，分别查看集群节点状态：
  <img src="https://github.com/jiaming9844/hello-world/blob/master/zookeeper/img/1120165-20181027125808915-966913713.png"/>
  
  
  7、搭建问题
  
  　如果没有出现上面的状态，说明搭建过程出了问题，那么解决问题的首先就是查看日志文件：

　　zookeeper 日志文件目录在：

　　dataDir 配置的目录下，文件名称为：zookeeper.out。通过查看日志来解决相应的问题。下面是两种常见的问题：

　　①、防火墙为关闭

　　查看防火墙状态：

    service iptables status
　　关闭防火墙：

    chkconfig iptables off
　　②、dataDir 配置的目录没有创建

　　在 zoo.cfg 文件中，会有对 dataDir 的一项配置，需要创建该目录，并且注意要在该目录下创建 myid 文件，里面的配置和 zoo.cfg 的server.x 配置保持一致
