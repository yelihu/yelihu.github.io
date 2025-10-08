---
title: MacOS本地如何搭建一个双主双从的RocketMQ集群？
date: 2024-07-12 14:12:23
updated: 2024-07-12 14:30:29
toc: true
categories:
  - 分布式中间件
tags:
  - 分布式技术
  - 消息队列
  - RocketMQ
author: 虎太郎
excerpt: 介绍了如何在 MacOS 本地安装和配置RocketMQ集群，用于本地调试或者源码学习的时候非常棒。全过程踩了很多坑（包括版本选择、JVM的配置修改等），现在记录下来方便日后进行环境配置。
---

## 下载和解压

不多赘述，我选择的是RocketMQ 4.7.0版本。去官网下载即可。



## 创建NameServer集群

### 复制原RocketMQ包

原有RocketMQ包如下位置

![image](https://hexo-yelihu.oss-cn-hangzhou.aliyuncs.com/hexo/img/2427414-20220325220346710-423135779.png)


复制一份在原地，并且修改名字为 **rocketmq-all-4.7.0-bin-release-nameserver**



### 新增两个配置文件

进入刚刚创建的 **rocketmq-all-4.7.0-bin-release-nameserver** 包，在其conf包下面创建如下两个文件：

* namesrv1.properties
* namesrv2.properties

**分别**在这两个.properties文件里面输入 listenPort=19876和listenPort=29876。



### 编写并启动start.sh脚本

返回conf包上一层，创建启动脚本文件**start.sh**。在这个脚本里面编写如下启动语句：

```bash
echo "starting nameserver1"
nohup sh bin/mqnamesrv -c conf/namesrv1.properties &
echo "started nameserver1"
echo "starting nameserver2"
nohup sh bin/mqnamesrv -c conf/namesrv2.properties &
echo "started nameserver2"
```

接着就可以启动两个运行在端口号是19876和29876的两个NameServer了。效果如下：

![image](https://hexo-yelihu.oss-cn-hangzhou.aliyuncs.com/hexo/img/2427414-20220325220419637-871617138.png)


## 搭建Broker集群

### 配置文件修改

首先将还是将原来的RocketMQ包复制并命名为 **rocketmq-all-4.7.0-bin-release-sync** 用于搭建Broker集群。在/Users/apple/dev/rocketmq-all-4.7.0-bin-release-sync/conf/2m-2s-sync包下面，存在这样几个配置文件。

1. broker-a-s.properties
1. broker-a.properties
1. broker-b-s.properties
1. broker-b.properties

#### 配置文件的要点

> 这一小节是在在配置完成之后，总结出来写的这一部分内容。

**brokerName配置错误：**

在第一次配置配置完成之后，运行demo，但是发现使用任何消息发送的方式发送的消息，都会出现返回的发送状态SendResult对象当中出现的是SLAVE_NOT_AVAILABLE的错误信息（*正确应该是SEND_OK*）。

直觉告诉原因就是Master找不到对应的Slave，参考资料和回查配置文件的时候，发现下面配置的四个文件中，每个 **brokerName**配置的brokerName都不一样，导致了每个broker都是孤立的主从节点。

正确的方式应该是broker-a、broker-a-s互为主从节点，那么就设定为同一名字“broker-a”，这样brokerRole=SYNC_MASTER的主节点在接受到消息的时候，如果需要同步消息，就能找到作为brokerRole=SLAVE的从节点。

**总结：互为主从的Broker的brokerName=配置项必须一样！**



**brokerRole属性解析**

brokerRole包含三种属性，其中Master相关的属性包含SYNC_MASTER和ASYNC_MASTER，SLAVE相关的只有一个。SYNC_MASTER这种属性既包含了同步机制，也就是**主从是否同步完消息再返回OK状态（如果是就为SYNC，否则为ASYNC）**。



**关于刷盘：**

什么是刷盘，就是将消息写入存储物质，这个存储物质可以是磁盘、也可以是Page Cache *页高速缓冲存储器*。

> Page Cache *页高速缓冲存储器*
>
> 用来缓存一些磁盘上文件内容的内存，一页为4K。用户进程调用read()系统调用，内核先去内存查看是否命中，否则就去磁盘读取（速度很慢）。
>
> [Linux中的Page Cache](https://zhuanlan.zhihu.com/p/68071761)





#### Broker-a的配置

配置其中一个Slave，设置监听消息的端口为10211。

```properties
#所属集群名字
brokerClusterName=rocketmq-cluster_2m-2s-sync
#broker名字，注意此处不同的配置文件填写的不一样 例如：在a.properties 文件中写 broker-a 在b.properties 文件中写 broker-b
brokerName=broker-a
#0 表示 Master，>0 表示 Slave
brokerId=0 
#两个nameserver的对应的ip地址。
namesrvAddr=127.0.0.1:19876;127.0.0.1:29876
#Broker 对外服务的监听端口
listenPort=10211
#删除文件时间点，默认凌晨 4点
deleteWhen=04
#- Broker 的角色；SLAVE；SYNC_MASTER 同步双写Master；ASYNC_MASTER 异步复制Master
brokerRole=SYNC_MASTER
#- 刷盘方式：SYNC_FLUSH 同步刷盘； ASYNC_FLUSH 异步刷盘
flushDiskType=SYNC_FLUSH



#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=4
#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true
#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true
#文件保留时间，默认 48 小时
fileReservedTime=120
#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存30W条，根据业务情况调整
mapedFileSizeConsumeQueue=300000
#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=88
#存储路径和一些配置消息的根目录（应该是只用配置此路径，二下面的几条路径配置可以省略）
storePathRootDir=/Users/apple/dev/rocketmq-all-4.7.0-bin-release-sync_broker-a/store

#commitLog 存储路径
storePathCommitLog=/Users/apple/dev/rocketmq-all-4.7.0-bin-release-sync_broker-a/store/commitlog
#消费队列存储路径存储路径
storePathConsumeQueue=/Users/apple/dev/rocketmq-all-4.7.0-bin-release-sync_broker-a/store/consumequeue
#消息索引存储路径
storePathIndex=/Users/apple/dev/rocketmq-all-4.7.0-bin-release-sync_broker-a/store/index
#checkpoint 文件存储路径
storeCheckpoint=/Users/apple/dev/rocketmq-all-4.7.0-bin-release-sync_broker-a/store/checkpoint
#abort 文件存储路径
abortFile=/Users/apple/dev/rocketmq-all-4.7.0-bin-release-sync_broker-a/store/abort

#限制的消息大小
maxMessageSize=65536
```

 **需要额外注意的是，**RocketMQ会自动将接受到的消息，自动的持久化，持久化的位置默认是root的home路径下，上面的如下部分就是将这个持久化的位置修改为了指定路径/Users/apple/dev/rocketmq-all-4.7.0-bin-release-sync_broker-a下面。这样命名和指定的好处就是每一个Broker的消息持久化的路径都不不同。

```properties
#存储路径
storePathRootDir=/Users/apple/dev/rocketmq-all-4.7.0-bin-release-sync_broker-a/store
#commitLog 存储路径
storePathCommitLog=/Users/apple/dev/rocketmq-all-4.7.0-bin-release-sync_broker-a/store/commitlog
#消费队列存储路径存储路径
storePathConsumeQueue=/Users/apple/dev/rocketmq-all-4.7.0-bin-release-sync_broker-a/store/consumequeue
#消息索引存储路径
storePathIndex=/Users/apple/dev/rocketmq-all-4.7.0-bin-release-sync_broker-a/store/index
#checkpoint 文件存储路径
storeCheckpoint=/Users/apple/dev/rocketmq-all-4.7.0-bin-release-sync_broker-a/store/checkpoint
#abort 文件存储路径
abortFile=/Users/apple/dev/rocketmq-all-4.7.0-bin-release-sync_broker-a/store/abort
```

全部Broker启动之后，会在指定目录下出现如下文件夹：

![image](https://hexo-yelihu.oss-cn-hangzhou.aliyuncs.com/hexo/img/2427414-20220325220457692-922882257.png)


另外，集群的名称是四个角色都要归属于同一个集群的，所以在brokerClusterName这一项的配置全部为rocketmq-cluster_2m-2s-sync。

**brokerRole=SYNC_MASTER** 首先标注了当前这个节点的角色是Slave，另外也说明了主从复制方式是同步，表示：Broker和Slave在完成消息同步之后，才会给消息的发送者一个响应。



>启动之后可以打开jps查看当前运行的Java进程，可以看到两个NameServer和Broker启动在本机上。
>
>![image](https://hexo-yelihu.oss-cn-hangzhou.aliyuncs.com/hexo/img/2427414-20220325220521001-726576188.png)




#### Broker-a-s的配置

作为一个Slave，首先明显的标识就是brokerId = 1，另外注意的是这个Slave的对外消息的监听端口是20111。

```properties
brokerClusterName=rocketmq-cluster_2m-2s-sync
brokerName=broker-a
brokerId=1
namesrvAddr=127.0.0.1:19876;127.0.0.1:29876
defaultTopicQueueNums=4
autoCreateTopicEnable=true
autoCreateSubscriptionGroup=true
listenPort=20111
deleteWhen=04
fileReservedTime=120
mapedFileSizeCommitLog=1073741824
mapedFileSizeConsumeQueue=300000
diskMaxUsedSpaceRatio=88
storePathRootDir=/Users/apple/dev/rocketmq-all-4.7.0-bin-release-sync_broker-a-s/store
storePathCommitLog=/Users/apple/dev/rocketmq-all-4.7.0-bin-release-sync_broker-a-s/store/commitlog
storePathConsumeQueue=/Users/apple/dev/rocketmq-all-4.7.0-bin-release-sync_broker-a-s/store/consumequeue
storePathIndex=/Users/apple/dev/rocketmq-all-4.7.0-bin-release-sync_broker-a-s/store/index
storeCheckpoint=/Users/apple/dev/rocketmq-all-4.7.0-bin-release-sync_broker-a-s/store/checkpoint
abortFile=/Users/apple/dev/rocketmq-all-4.7.0-bin-release-sync_broker-a-s/store/abort
maxMessageSize=65536
brokerRole=SLAVE
flushDiskType=SYNC_FLUSH
```



#### Broker-b的配置

作为Broker-b的主节点，brokerRole=SYNC_MASTER和brokerId=0都必须标注上去。监听消息的端口为10311

```properties
brokerClusterName=rocketmq-cluster_2m-2s-sync
brokerName=broker-b
brokerId=0
namesrvAddr=127.0.0.1:19876;127.0.0.1:29876
defaultTopicQueueNums=4
autoCreateTopicEnable=true
autoCreateSubscriptionGroup=true
listenPort=10311
deleteWhen=04
fileReservedTime=120
mapedFileSizeCommitLog=1073741824
mapedFileSizeConsumeQueue=300000
diskMaxUsedSpaceRatio=88
storePathRootDir=/Users/apple/dev/rocketmq-all-4.7.0-bin-release-sync_broker-b/store
storePathCommitLog=/Users/apple/dev/rocketmq-all-4.7.0-bin-release-sync_broker-b/store/commitlog
storePathConsumeQueue=/Users/apple/dev/rocketmq-all-4.7.0-bin-release-sync_broker-b/store/consumequeue
storePathIndex=/Users/apple/dev/rocketmq-all-4.7.0-bin-release-sync_broker-b/store/index
storeCheckpoint=/Users/apple/dev/rocketmq-all-4.7.0-bin-release-sync_broker-b/store/checkpoint
abortFile=/Users/apple/dev/rocketmq-all-4.7.0-bin-release-sync_broker-b/store/abort
maxMessageSize=65536
brokerRole=SYNC_MASTER
flushDiskType=SYNC_FLUSH
```



#### Broker-b-s的配置

同样是Broker-b，但是-s的后缀、brokerRole和brokerId的配置项都表明其是一个Slave。监听消息的端口号是10411

```properties
brokerClusterName=rocketmq-cluster_2m-2s-sync
brokerName=broker-b
brokerId=1
namesrvAddr=127.0.0.1:19876;127.0.0.1:29876
defaultTopicQueueNums=4
autoCreateTopicEnable=true
autoCreateSubscriptionGroup=true
listenPort=10411
deleteWhen=04
fileReservedTime=120
mapedFileSizeCommitLog=1073741824
mapedFileSizeConsumeQueue=300000
diskMaxUsedSpaceRatio=88
storePathRootDir=/Users/apple/dev/rocketmq-all-4.7.0-bin-release-sync_broker-b-s/store
storePathCommitLog=/Users/apple/dev/rocketmq-all-4.7.0-bin-release-sync_broker-b-s/store/commitlog
storePathConsumeQueue=/Users/apple/dev/rocketmq-all-4.7.0-bin-release-sync_broker-b-s/store/consumequeue
storePathIndex=/Users/apple/dev/rocketmq-all-4.7.0-bin-release-sync_broker-b-s/store/index
storeCheckpoint=/Users/apple/dev/rocketmq-all-4.7.0-bin-release-sync_broker-b-s/store/checkpoint
abortFile=/Users/apple/dev/rocketmq-all-4.7.0-bin-release-sync_broker-b-s/store/abort
maxMessageSize=65536 
brokerRole=SLAVE
flushDiskType=SYNC_FLUSH
```



> 到目前为止，Broker-a监听10211、Broker-a-s监听20211、Broker-b监听10311、Broker-b-s监听10411。2M2S的集群的配置文件修改就到此配置完毕，主要的一些配置项分别如下：
>
> 1. namesrvAddr
> 1. listenPort
> 1. brokerId
> 1. brokerRole





### 编写并启动Broker集群

编写

```properties
echo "starting broker-a"
nohup sh bin/mqbroker -c conf/2m-2s-sync/broker-a.properties &
echo "started broker-a"
echo "starting broker-a-s"
nohup sh bin/mqbroker -c conf/2m-2s-sync/broker-a-s.properties &
echo "started broker-a-s"
echo "starting broker-b"
nohup sh bin/mqbroker -c conf/2m-2s-sync/broker-b.properties &
echo "started broker-b"
echo "starting broker-b-s"
nohup sh bin/mqbroker -c conf/2m-2s-sync/broker-b-s.properties &
echo "started broker-b-s"
```

在 **rocketmq-all-4.7.0-bin-release-sync** 下创建start.sh脚本文件。

```properties
nohup sh bin/mqbroker -c conf/2m-2s-sync/broker-a.properties &
```

**-c** 参数的作用是指定配置文件来启动Broker程序，指定的配置文件路径紧跟在后面。





### 验证搭建成功

编写并保存之后，启动此sh脚本文件即可启动Broker集群。使用jps命令验证。

![image](https://hexo-yelihu.oss-cn-hangzhou.aliyuncs.com/hexo/img/2427414-20220326104314143-630799320.png)


可见，四个Broker= 两个Master + 两个Slave，配合两个NameServer就构成了本次想要搭建的集群。*如果是在四台机器上面搭建此集群，则不会出现这种输出情况。*

使用如下👇命令，查看namesrv、broker的日志，观察是否

```bash
# 查看nameServer日志
tail -500f ~/logs/rocketmqlogs/namesrv.log
# 查看broker日志
tail -500f ~/logs/rocketmqlogs/broker.log
```


![image](https://hexo-yelihu.oss-cn-hangzhou.aliyuncs.com/hexo/img/2427414-20220326104253750-1563434376.png)










## 使用RocketMQ-dashboard可视化

### 下载RocketMQ-dashboard

点击链接，克隆下来如下这个项目[rocketmq-dashboard](https://github.com/apache/rocketmq-dashboard)吗。将maven加载完毕。



### 修改RocketMQ-dashboard配置

项目的运行需要制定一些配置，方便我们第一次启动。找到配置文件：

![image](https://hexo-yelihu.oss-cn-hangzhou.aliyuncs.com/hexo/img/2427414-20220326104340681-753411551.png)

修改其中关于nameServer运行端口的配置：

```properties
#if this value is empty,use env value rocketmq.config.namesrvAddr  NAMESRV_ADDR | now, you can set it in ops page.default localhost:9876
rocketmq.config.namesrvAddr=localhost:19876;localhost:29876;
```

*原来此处配置是空的，如果不加设置，需要咋jar包运行的时候给定参数。*

另外，将RocketMQ-dashboard产生的数据的输出目录也修改为自己定义的目录：

```properties
rocketmq.config.dataPath=/Users/apple/dev/rocketmq-console-data
```



> 配置文件当中还有几项是不加配置的配置项，选用默认的即可。



### 打包和运行

使用Maven打包这个项目，得到可以执行的jar包。

按照下图所示：

1. 首先将test模块忽略掉，点击上面的蓝底闪电圆形按钮即可。
1. 按照先后 选中clean+package打包，然后等待其输出构建成功标识即可



![image](https://hexo-yelihu.oss-cn-hangzhou.aliyuncs.com/hexo/img/2427414-20220326104450601-1091774754.png)


*构建成功标识如下:*

![image](https://hexo-yelihu.oss-cn-hangzhou.aliyuncs.com/hexo/img/2427414-20220326104525994-1842873639.png)


在target包下面可以得到一个jar包：

![image](https://hexo-yelihu.oss-cn-hangzhou.aliyuncs.com/hexo/img/2427414-20220326104546411-1285661039.png)



在terminal里找到这个jar包，然后输入如下命令即可运行此jar包：

```bash
java -jar rocketmq-dashboard-1.0.1-SNAPSHOT.jar
```



RocketMQ-dashboard是一个spring-boot项目，运行成功后会输出日志提示此项目运行在8080（*默认*）端口上。点击 [http://localhost:8080/](http://localhost:8080/)即可访问管理台页面：

![image](https://hexo-yelihu.oss-cn-hangzhou.aliyuncs.com/hexo/img/2427414-20220326104429273-1695232268.png)




### 证集群是否搭建成功

首先确保前面说述的2M2S的Broker集群搭建完毕，使用jps能看见四个Broker进程。

点击管理台的“集群”选项卡，能够看见如下：

![image](https://hexo-yelihu.oss-cn-hangzhou.aliyuncs.com/hexo/img/2427414-20220326104609704-2139411535.png)

在下拉框可以看到，之前配置文件中设定的集群名字在这里显示出来：**rocketmq-cluster_2m-2s-sync**

下面的表格也能展现四个Broker的信息：

![image](https://hexo-yelihu.oss-cn-hangzhou.aliyuncs.com/hexo/img/2427414-20220326104631562-843709107.png)


> **🎉 到此为止，在Mac本地搭建2M2S的集群完成！**
