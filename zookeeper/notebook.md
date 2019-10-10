##什么是zookeeper？

简单描述：

--Zookeeper 可以被用作注册中心。

--Zookeeper 是 Hadoop 生态系统的一员；

--构建 Zookeeper 集群的时候，使用的服务器最好是奇数台。

ZooKeeper 概览：

ZooKeeper 的设计目标是将那些复杂且容易出错的分布式一致性服务封装起来，构成一个高效可靠的原语集，并以一系列简单易用的接口提供给用户使用。

ZooKeeper 是一个典型的分布式数据一致性解决方案，分布式应用程序可以基于 ZooKeeper 实现诸如数据发布/订阅、负载均衡、命名服务、分布式协调/通知、集群管理、Master 选举、分布式锁和分布式队列等功能。

Zookeeper 一个最常用的使用场景就是用于担任服务生产者和服务消费者的注册中心。

服务生产者将自己提供的服务注册到Zookeeper中心，服务的消费者在进行服务调用的时候先到Zookeeper中查找服务，获取到服务生产者的详细信息之后，再去调用服务生产者的内容与数据。如下图所示，在 Dubbo架构中 Zookeeper 就担任了注册中心这一角色。

https://github.com/jiaming9844/hello-world/blob/master/zookeeper/img/1063359856-5b973176b0f9c_articlex.png
