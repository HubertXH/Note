##### common argument
```properties
# canal server绑定的本地IP信息，如果不配置，默认选择一个本机IP进行启动服务	
canal.ip =
# canal server绑定的本地IP信息，如果不配置，默认选择一个本机IP进行启动服务	
canal.register.ip =
# canal server提供socket服务的端口	
canal.port = 11111
# canal监控信息获取端口 可以使用prometheus,AlertManager做报警管理使用。
canal.metrics.pull.port = 11112
# canal instance user/passwd
# canal数据端口订阅的ACL配置(v1.1.4新增) 如果为空，代表不开启 
canal.user = canal
canal.passwd = E3619321C1A937C46A0D8BD1DAC39F93B27D4458
```

##### canal admin config
```properties

canal.admin.manager = 127.0.0.1:8089
canal.admin.port = 11110
canal.admin.user = admin
canal.admin.passwd = 4ACFE3202A5FF5CF467898FC58AAB1D615029441
# admin auto register
canal.admin.register.auto = true
canal.admin.register.cluster =
canal.admin.register.name =
```

```properties
# canal 集群注册ZK地址及端口号
canal.zkServers = zookeeper ip:port
# 像ZK刷新数据的周期单位毫秒
canal.zookeeper.flush.period = 1000
# 
canal.withoutNetty = false
# 运行模式：tcp, kafka, rocketMQ, rabbitMQ
canal.serverMode = rocketMQ
# 
canal.file.data.dir = ${canal.conf.dir}
canal.file.flush.period = 1000
# canal内存store中可缓存buffer记录数，需要为2的指数
canal.instance.memory.buffer.size = 16384
# 内存记录的单位大小，默认1KB，和buffer.size组合决定最终的内存使用大小
canal.instance.memory.buffer.memunit = 16384
# canal内存store中数据缓存模式 
# 1. ITEMSIZE : 根据buffer.size进行限制，只限制记录的数量
# 2. MEMSIZE : 根据buffer.size  * buffer.memunit的大小，限制缓存记录的大小
canal.instance.memory.batch.mode = MEMSIZE
# 针对entry是否开启raw模式 默认为true,涉及到消息的序列化，不建议修改。
canal.instance.memory.rawEntry = true
```
#### 健康检测
```properties
# detecing config;是否开启心跳检测
canal.instance.detecting.enable = false
#心跳检测SQL,默认为： insert into retl.xdual values(1,now()) on duplicate key update x=now()
canal.instance.detecting.sql = select 1
# 心跳检查频率，单位秒	
canal.instance.detecting.interval.time = 3
# 心跳检查失败重试次数	
canal.instance.detecting.retry.threshold = 3
# 心跳检查失败后，是否开启自动mysql自动切换
# 说明：比如心跳检查失败超过阀值后，如果该配置为true，canal就会自动链到mysql备库获取binlog数据
canal.instance.detecting.heartbeatHaEnable = false
```

```properties
# 大事务解析大小，超过该大小后事务将被切分为多个事务投递
canal.instance.transaction.size =  1024
# mysql fallback 切换时回退的时间默认60秒
canal.instance.fallbackIntervalInSeconds = 60

# network config
#canal.instance.network.receiveBufferSize = 16384
canal.instance.network.receiveBufferSize = 131072
#canal.instance.network.sendBufferSize = 16384
canal.instance.network.sendBufferSize = 131072
canal.instance.network.soTimeout = 30

# binlog filter config
# 是否使用druid处理所有的ddl解析来获取库和表名
canal.instance.filter.druid.ddl = true
# 是否忽略dcl语句	
canal.instance.filter.query.dcl = false
# 是否忽略dml语句
canal.instance.filter.query.dml = false
# 是否忽略ddl语句	
canal.instance.filter.query.ddl = false
# 是否忽略binlog表结构获取失败的异常 主要解决回溯binlog时,对应表已被删除或者表结构和binlog不一致的情况
canal.instance.filter.table.error = false
# 是否dml的数据变更事件 (主要针对用户只订阅ddl/dcl的操作)
canal.instance.filter.rows = false
# 是否忽略事务头和尾,比如针对写入kakfa的消息时，不需要写入TransactionBegin/Transactionend事件
canal.instance.filter.transaction.entry = false

canal.instance.filter.dml.insert = false
canal.instance.filter.dml.update = false
canal.instance.filter.dml.delete = false

# 支持的binlog format格式列表
canal.instance.binlog.format = ROW,STATEMENT,MIXED
# 支持的binlog image格式列表
canal.instance.binlog.image = FULL,MINIMAL,NOBLOB

# ddl语句是否单独一个batch返回 默认false 下游dml/ddl如果做batch内无序并发处理,会导致结构不一致
canal.instance.get.ddl.isolation = false

# 是否开启binlog并行解析模式 (串行解析资源占用少,但性能有瓶颈, 并行解析可以提升近2.5倍+)
canal.instance.parser.parallel = true
# binlog并行解析的线程数，默认为当前机器可用处理器的60%，建议不要超过当前机器的可用处理器数量
canal.instance.parser.parallelThreadSize = 20
## binlog并行解析的异步ringbuffer队列必须为2的指数-默认为256
canal.instance.parser.parallelBufferSize = 8192
```
#### table meta tsdb info [TableMetaTSDB详细介绍](https://github.com/alibaba/canal/wiki/TableMetaTSDB)
```properties
# 是否开启TableMetaTSDB 用于兼容表结构发生变化时的数据同步
canal.instance.tsdb.enable = true
canal.instance.tsdb.dir = ${canal.file.data.dir:../conf}/${canal.instance.destination:}
canal.instance.tsdb.url = jdbc:h2:${canal.instance.tsdb.dir}/h2;CACHE_SIZE=1000;MODE=MYSQL;
canal.instance.tsdb.dbUsername = canal
canal.instance.tsdb.dbPassword = canal
# dump snapshot interval, default 24 hour
canal.instance.tsdb.snapshot.interval = 24
# purge snapshot expire , default 360 hour(15 days)
canal.instance.tsdb.snapshot.expire = 360
```

##### Destinations Config
```properties
# 当前server上需要启动的实例
canal.destinations = instance01,instance02
# canal配置目录
canal.conf.dir = ../conf
# 是否开启Instance自动扫描，开启后可监听配置的变化对，对instance做出更新。监听的目录为 canal.conf.dir 配置的目录
# a. instance目录新增： 触发instance配置载入，lazy为true时则自动启动
# b. instance目录删除：卸载对应instance配置，如已启动则进行关闭
# c. instance.properties文件变化：reload instance配置，如已启动自动进行重启操作
canal.auto.scan = true
# 自动扫描间隔时间-单位秒
canal.auto.scan.interval = 5
# set this value to 'true' means that when binlog pos not found, skip to latest.
# WARN: pls keep 'false' in production env, or if you know what you want.
canal.auto.reset.latest.pos.mode = false

canal.instance.tsdb.spring.xml = classpath:spring/tsdb/h2-tsdb.xml
#canal.instance.tsdb.spring.xml = classpath:spring/tsdb/mysql-tsdb.xml
# 全局配置加载方式	spring,manager
# 1 spring 为本地配置 2 manager为通过canal管理后台进行配置管理
canal.instance.global.mode = spring
# 全局lazy模式	
canal.instance.global.lazy = false
# 在mode为manager的模式下必须配置 配置管理后台地址
canal.instance.global.manager.address = ${canal.admin.manager}
#canal.instance.global.spring.xml = classpath:spring/memory-instance.xml
#canal.instance.global.spring.xml = classpath:spring/file-instance.xml
# 在mode为spring的模式下必须配置
canal.instance.global.spring.xml = classpath:spring/default-instance.xml
```

##### MQ Properties
````properties
# aliyun ak/sk , support rds/mq
canal.aliyun.accessKey =
canal.aliyun.secretKey =
canal.aliyun.uid=

# 是否为json格式 如果设置为false,对应MQ收到的消息为protobuf格式 需要通过CanalMessageDeserializer进行解码
canal.mq.flatMessage = true
# 获取canal数据的批次大小	
canal.mq.canalBatchSize = 50
# 获取canal数据的超时时间
canal.mq.canalGetTimeout = 100
# Set this value to "cloud", if you want open message trace feature in aliyun.
# canal.mq.accessChannel = local

#是否开启database混淆hash，确保不同库的数据可以均匀分散，如果关闭可以确保只按照业务字段做MQ分区计算
canal.mq.database.hash = true
# MQ消息发送并行度	
canal.mq.send.thread.size = 30
# MQ消息构建并行度	
canal.mq.build.thread.size = 8
````

##### Kafka Config
```properties
kafka.bootstrap.servers = 127.0.0.1:9092
kafka.acks = all
kafka.compression.type = none
kafka.batch.size = 16384
kafka.linger.ms = 1
kafka.max.request.size = 1048576
kafka.buffer.memory = 33554432
kafka.max.in.flight.requests.per.connection = 1
kafka.retries = 0

kafka.kerberos.enable = false
kafka.kerberos.krb5.file = "../conf/kerberos/krb5.conf"
kafka.kerberos.jaas.file = "../conf/kerberos/jaas.conf"
```

##### RocketMQ Config
```properties
# 生产者组名
rocketmq.producer.group = topic-producer
# 是否开启消息追踪
rocketmq.enable.message.trace = false
rocketmq.customized.trace.topic =
# namespace
rocketmq.namespace =
# nameserver 地址
rocketmq.namesrv.addr = 127.0.0.1:9876;
# 发送时候重试次数
rocketmq.retry.times.when.send.failed = 3
# 是否开启vip通道
rocketmq.vip.channel.enabled = false
# 消息tag,可以用于过滤消息
rocketmq.tag =
# 当前开源版本无该配置，发送的MQ均没有key,该字段为定制化开发字段。
rocketmq.message.keys = schema1.tableName1:field1;
```

---
##### RabbitMQ Config
```properties
rabbitmq.host =
rabbitmq.virtual.host =
rabbitmq.exchange =
rabbitmq.username =
rabbitmq.password =
rabbitmq.deliveryMode =
```
