# kafka

## 理论    
###### 消息队列模型:    
是基于队列提供消息传输服务的，多用于进程间通信以及线程间通信。提供一种P2P的消息传递模式，即发送者发送每条消息到队列指定的位置，接收者可以从指定的位置获取消息。一旦消息被消费，就会从队列中被移除。每条消息由一个生产者生产出来，且只被一个接收者处理。    
###### 发布/订阅模型：    
发布者将消息生产出来发送至指定的topic中，所有订阅了该topic的订阅着都可以收到该topic下的所有消息。    
###### 数据写入磁盘方式：    
Kafka每次本质上只是把数据写入到操作系统的页缓存中，然后由操作系统自行决定是什么时间把缓存页中的数据写回磁盘中。这样设计的优势如下：
- 操作系统缓存页实在内存中分配的，所以消息的写入是非常块的。
- Kafka不必直接与底层的文件系统打交道。所有繁琐的I/O操作都交由操作系统来处理
- Kafka写入操作采用追加的方式，避免了磁盘随机写入的操作。

###### 相关概念：
- topic：逻辑概念，代表一类消息，也可以认为是消息被发送到的地方。
- partition：分区，消息topic的分区。以提高并发量
- replication：副本，保证数据不会丢失的机制，数据冗余存储。(leader，follower)
- ISR(in-sync replica) :与leader replication保持同步的replication集合。只有该集合中的replication才可以被选举成为leader，也只有该集中的所有replication都收到了同一条消息，Kafka才会把这条消息置为以提交的状态。ISR 集合中只少有一个活着的replication。

###### Producer 工作流程：    
1. 封装一个ProducerRecord类实例。
2. 将封装好的实例进行序列化后发送给分区器。
3. 分区器确定分区后将其发送至producer的页内存缓冲区。    
4. producer的sender线程实时的从该缓冲区提取出准备就绪的消息封装进一个批次，然后发送到broker。 

##### Consumer    
###### consumer group:    
Consumers label themselves with a consumer group name,and each record published to a topic is delivered to one consumer instance within each subscribing consumer group.
- 以个consumer group 可以有多个consumer
- 对于同一个group而言，topic的每天消息只能被发送到group中的一个consumer实列上。
- topic消息可以被发送到多个group中。
 
++*每个consumer group实例都会为消费这分区维护属于自己的位置信心偏移量(offset)。P103*++    
++*Kafka引入点检查机制顶起对offset进行持久化。*++
 
###### consumer rebalance:    
coordinator:新版本中一个内置的全新的brokder会被选择为group coordinator，负责对组的状态进行管理。主要指责就是当新的成员到达时促成组内成员达成新的分配方案。    

###### 副本与ISR设计：    
ISR：一组同步副本集合(in-sync replicas) 每个topic分区都有自己ISR列表。只有在ISR列表中的副本才有可能被选举为leader，且ISR中的副本都与leader保持同步。    
base offset：起始位移，表示该副本当前所含第一条消息的offset    
high watermark(HW)：副本高水印值，保存了该副本最新一条已经提交消息位移。    
log end offset(LEO)：日志末端位移，副本日志中吓一条待写入消息的offset


## 启动程序
##### 首先启动zookeeper    
bin/zkServer.sh start conf/zoo.cfg   
##### zookeeper 状态查看命令:
bin/zkServer.sh status    
若状态显示失败则查看2888和3888端口是否开放
在查zoo.cfg配置文件，再查看myid文件中的数字是否与zoo.cfg中的配置相同    
##### 启动kafka    
bin/kafka-server-start.sh -daemon config/server.properties    

## Zookeeper 配置    
zoo.cfg文件中的配置:    
1. tickTime：zookeeper中的最小时间单位，用于丈量心跳和超时时间，默认值为2秒    
1. dataDir：zookeeper系统快照的保存路径    
1. clientPort：Zookeeper监听客户端连接的端口，一般默认值为2181    
1. initLimit：制定follower节点初始连接leader节点时最大的tick数。假定initLimit的数是3则follower节点必须在3*2的时间内接连上Leader否则会被认为是超时    
1. syncLimit：follower与Leader同步的最大时间限制
1. server.X=host:port1:port2 X 数字，全局唯一，且必须与myid文件中的数字保持一致，端口port1：用于follower节点连接leader节点，port2:用于leader节点的选举    

## Kafka 配置    
#### broker端参数配置：    
 
```
broker.id
```

 非常重要，全局唯一，默认参数为-1，若用户不指定则kafka会生成一个唯一值。    

```
log.dir
```

非常重要，该参数指定了kafka持久化消息的目录,该参数可以设置为多个目录以逗号分隔例如：data/kafka1,data/kafka2,data/kafka3    

```
zookeeper.connect
```
 非常重要的参数 也可以配置为一个列表 用于kafka想zookeeper的注册。格式为：host:port,host:port,host:port/chroot。最后的chroot可配可不配，若使用一套zookeeper集群管理多套kafka集群则推荐配置该参数，可以起到很好的隔离作用。    

```
listeners：
```
broke监听的列表，格式：[协议]://[主机名]:[端口号],[协议]://[主机名]:[端口号],[协议]://[主机名]:[端口号]。用于客户端连接broker使用。若该参数不配置则表示绑定默认的网卡地址。若配置为0.0.0.0则表示绑定所有的网卡地址。    
 协议类型：PLAINTEXT、SSL、SASL_SSL。对于未启用安全验证的集群使用PLAINTEXT协议就好。如果启用了安全验证的可以考虑使用SSL或者SASL_SSL

```
advertised.listeners:
```
该参数也是用与给client的监听器的，不过该参数多用于Iaas环境下，因为云主机一般配有多块网卡，私有和共有网卡。一般建议用户可以设置该参数为公网Ip共外部的client使用，而配置listeners为私有Ip共broker之间通信使用。    

```
unclean.leader.election.enable：
```
是否开启unclean.leader    

```
delete.topic.enable：
```
是否允许删除topic，推荐设置为true。    

```
log.retention.{hour|minutes|ms}：
```
用于控制消息的留存时间，优先取ms，minutes次之，最后取hour。kafka默认的消息留存时间为7天，超过7天的数据会自动删除。    

```
log.retention.bytes:
```
用于控制留存消息的大小。指出了kafka集群为每个消息日志保存多大的数据，若大小超过该参数的分区日志，kafka会自动清理该分区的过期数据。默认数值为-1表示永远不会根据这个参数来删除数据。    

```
min.insync.replicas:
```
最小同步副本数量，配合client端的acks使用。    

```
num.networks.threads:
```
表示一个broker在后台用于处理网络请求的线程数量，默认为3。broket会在启动的时候创建多个线程来处理其他broker或client发送过来的请求。使用这些线程将发送过来的网络请求转发给后台处理线程中。在实际的环境中需要实时的监控NetworkProcessorAvgIdlePercent JMX指标若该指标低于0.3则可以适当增加该参数的值。    
 
```
num.io.threads:
```
表示broker实际后台处理网络请求的县城数，默认值为8。同理kafka提供可监控的参数 RequestHandlerAvglePercent JMX 若低于0.3则可以适当的增加该参数的值。    

```
message.max.bytes：
```
broker能够接受的最大的消息大小。默认为977kb。    
#### topic参数    

```
delete.retention.ms
```
每个topic下可以设置的日志的留存时间，可以覆盖全部默认值。    

```
max.message.bytes
```
每个topic下每个message最大的尺寸，可以覆盖全局的设置。    

```
retention.bytes
```
topic下设置不同的日志留存尺寸。    

1. 