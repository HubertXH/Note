## Basic Simple

以PushConsumer官网消费者代码为例，代码如下所示:

```java
public class Consumer {
  public static void main(String[] args) throws InterruptedException, MQClientException {
    // 1.Instantiate with specified consumer group name
    DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("please_rename_unique_group_name");
  
    // 2.Specify name server addresses
    consumer.setNamesrvAddr("localhost:9876");

    // 3.Subscribe one or more topics and tags for finding those messages need to be consumed
    consumer.subscribe("TopicTest", "*");
    // 4.Register callback to execute on arrival of messages fetched from brokers
    consumer.registerMessageListener(new MessageListenerConcurrently() {
      @Override
      public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {
        System.out.printf("%s Receive New Messages: %s %n", Thread.currentThread().getName(), msgs);
        // Mark the message that have been consumed successfully
        return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
      }
    });
    // 5.Launch the consumer instance
    consumer.start();
    System.out.printf("Consumer Started.%n");
  }
}
```

- 标记点1，初始化MQ实例其调用的方法如下所示：
  设置ConsumerGroup,
  AllocateMessageQueueStrategy赋默认值-即在Consumer进行rebalance 时采取的策略，默认为平均策略(根据整个Consumer组中consumer的实例数量与所订阅的TOPIC的队列数量金进行平均分配)
  这个策略在后续的流程中会用到，RocketMQ提供了几种默认的策略，包含平均策略，HASH值策略，同一机房策略等可以查看 org.apache.rocketmq.client.consumer.AllocateMessageQueueStrategy的实现类，
  同时若以上均不能满足，可以自己写一个在，实例化消费者对象时注入即可。
  在调用构造函数的同时，初始化DefaultMQPushConsumerImpl，同时将consumer注入至IMPL类中。

```java
public class DefaultMQPushConsumer extends ClientConfig implements MQPushConsumer {

    public DefaultMQPushConsumer(final String consumerGroup) {
        this(null, consumerGroup, null, new AllocateMessageQueueAveragely());
    }

    public DefaultMQPushConsumer(final String namespace, final String consumerGroup, RPCHook rpcHook,
        AllocateMessageQueueStrategy allocateMessageQueueStrategy) {
        this.consumerGroup = consumerGroup;
        this.namespace = namespace;
        this.allocateMessageQueueStrategy = allocateMessageQueueStrategy;
        defaultMQPushConsumerImpl = new DefaultMQPushConsumerImpl(this, rpcHook);
    }
}
```

- 标记点2，设置NameServer的地址
- 标记点3，设置订阅TOPIC和TAGS;TAGS可以是多个，使用"||"进行分隔。

```java
public class DefaultMQPushConsumer extends ClientConfig implements MQPushConsumer {
    @Override
    public void subscribe(String topic, String subExpression) throws MQClientException {
        this.defaultMQPushConsumerImpl.subscribe(withNamespace(topic), subExpression);
    }
}
public class DefaultMQPushConsumerImpl implements MQConsumerInner {

    public void subscribe(String topic, String subExpression) throws MQClientException {
        try {
            SubscriptionData subscriptionData = FilterAPI.buildSubscriptionData(topic, subExpression);
            this.rebalanceImpl.getSubscriptionInner().put(topic, subscriptionData);
            if (this.mQClientFactory != null) {
                this.mQClientFactory.sendHeartbeatToAllBrokerWithLock();
            }
        } catch (Exception e) {
            throw new MQClientException("subscription exception", e);
        }
    }
}

public class FilterAPI {
    public static SubscriptionData buildSubscriptionData(String topic, String subString) throws Exception {
        SubscriptionData subscriptionData = new SubscriptionData();
        subscriptionData.setTopic(topic);
        subscriptionData.setSubString(subString);

        if (null == subString || subString.equals(SubscriptionData.SUB_ALL) || subString.length() == 0) {
            subscriptionData.setSubString(SubscriptionData.SUB_ALL);
        } else {
            String[] tags = subString.split("\\|\\|");
            if (tags.length > 0) {
                for (String tag : tags) {
                    if (tag.length() > 0) {
                        String trimString = tag.trim();
                        if (trimString.length() > 0) {
                            subscriptionData.getTagsSet().add(trimString);
                            subscriptionData.getCodeSet().add(trimString.hashCode());
                        }
                    }
                }
            } else {
                throw new Exception("subString split error");
            }
        }

        return subscriptionData;
    }
}
```

如上代码所示，DefaultMQPushConsumer.subscribe调用了DefaultMQPushConsumerImpl的subscribe方法，
首先根据TOPIC及TAG构建出SubscriptionData对象，然后将其放入rebalance的实现类的中变量subscriptionInner中。
subscriptionInner变量是一个ConcurrentMap如下所示:

```
protected final ConcurrentMap<String /* topic */, SubscriptionData> subscriptionInner = new ConcurrentHashMap<String, SubscriptionData>();
```

从这个方法buildSubscriptionData中可以看到消费者的订阅信息中也对TAG进行了哈希计算并存储起来了。
==顺便说以下，如果在消费者集群中使用TAG进行消息过滤，那么整个消费者集群的实例中配置的TAG应该是相同的==，
RocketMQ的TAG过滤是发生在Broker端的，即被过滤调的消息一定不会被其实例所消费(详请可以查看消息的存储设计及消息拉取机制)。
==同时建议在消费者中使用TAG进行二次过滤，Broker端过虑消息是根据TAG的HASH值进行的过滤，为了防止出现HASH碰撞，所以需要在消费者实例里使用TAG进行过滤==

- 标记点4，注册并发消费回调函数
- 标记点5，启动消费者。

```java
public class DefaultMQPushConsumer extends ClientConfig implements MQPushConsumer {
    public void start() throws MQClientException {
        setConsumerGroup(NamespaceUtil.wrapNamespace(this.getNamespace(), this.consumerGroup));
        this.defaultMQPushConsumerImpl.start();
        if (null != traceDispatcher) {
            try {
                traceDispatcher.start(this.getNamesrvAddr(), this.getAccessChannel());
            } catch (MQClientException e) {
                log.warn("trace dispatcher start failed ", e);
            }
        }
    }
}
```

第一行setConsumerGroup就是将NameSpace和Group组装起来重新赋值给consumerGroup,但起始的时候我们并没有设置NameSpace会直接赋值consumerGroup原值
第二行开始正式启动consumer,consumer的启动工作都是在这个类defaultMQPushConsumerImpl里面进行的。下面看下这个类的代码

```java
public class DefaultMQPushConsumerImpl implements MQConsumerInner {
    public synchronized void start() throws MQClientException {
        switch (this.serviceState) {
            case CREATE_JUST:
                log.info("the consumer [{}] start beginning. messageModel={}, isUnitMode={}", this.defaultMQPushConsumer.getConsumerGroup(),
                    this.defaultMQPushConsumer.getMessageModel(), this.defaultMQPushConsumer.isUnitMode());
                this.serviceState = ServiceState.START_FAILED;
                // 6.配置检查
                this.checkConfig();
                // 7.存储订阅关系
                this.copySubscription();

                // 8.修改实例名称为具体部署机器的IP相关连上，方便问查问题
                if (this.defaultMQPushConsumer.getMessageModel() == MessageModel.CLUSTERING) {
                    this.defaultMQPushConsumer.changeInstanceNameToPID();
                }
                // 9.实例化MQClientInstance
                this.mQClientFactory = MQClientManager.getInstance().getOrCreateMQClientInstance(this.defaultMQPushConsumer, this.rpcHook);

                // 10.设置消费者Rebalance相关属性
                this.rebalanceImpl.setConsumerGroup(this.defaultMQPushConsumer.getConsumerGroup());
                this.rebalanceImpl.setMessageModel(this.defaultMQPushConsumer.getMessageModel());
                this.rebalanceImpl.setAllocateMessageQueueStrategy(this.defaultMQPushConsumer.getAllocateMessageQueueStrategy());
                this.rebalanceImpl.setmQClientFactory(this.mQClientFactory);

                this.pullAPIWrapper = new PullAPIWrapper(
                    mQClientFactory,
                    this.defaultMQPushConsumer.getConsumerGroup(), isUnitMode());
                this.pullAPIWrapper.registerFilterMessageHook(filterMessageHookList);
                // 11.获取消费的消费位点
                if (this.defaultMQPushConsumer.getOffsetStore() != null) {
                    this.offsetStore = this.defaultMQPushConsumer.getOffsetStore();
                } else {
                    switch (this.defaultMQPushConsumer.getMessageModel()) {
                        case BROADCASTING:
                            this.offsetStore = new LocalFileOffsetStore(this.mQClientFactory, this.defaultMQPushConsumer.getConsumerGroup());
                            break;
                        case CLUSTERING:
                            this.offsetStore = new RemoteBrokerOffsetStore(this.mQClientFactory, this.defaultMQPushConsumer.getConsumerGroup());
                            break;
                        default:
                            break;
                    }
                    this.defaultMQPushConsumer.setOffsetStore(this.offsetStore);
                }
                this.offsetStore.load();
                // 12.设置具体的消费逻辑-业务逻辑执行方法
                if (this.getMessageListenerInner() instanceof MessageListenerOrderly) {
                    this.consumeOrderly = true;
                    this.consumeMessageService =
                        new ConsumeMessageOrderlyService(this, (MessageListenerOrderly) this.getMessageListenerInner());
                } else if (this.getMessageListenerInner() instanceof MessageListenerConcurrently) {
                    this.consumeOrderly = false;
                    this.consumeMessageService =
                        new ConsumeMessageConcurrentlyService(this, (MessageListenerConcurrently) this.getMessageListenerInner());
                }
                // 13.修改消费者状态为开始消费
                this.consumeMessageService.start();
                // 14.注册
                boolean registerOK = mQClientFactory.registerConsumer(this.defaultMQPushConsumer.getConsumerGroup(), this);
                if (!registerOK) {
                    this.serviceState = ServiceState.CREATE_JUST;
                    this.consumeMessageService.shutdown(defaultMQPushConsumer.getAwaitTerminationMillisWhenShutdown());
                    throw new MQClientException("The consumer group[" + this.defaultMQPushConsumer.getConsumerGroup()
                        + "] has been created before, specify another name please." + FAQUrl.suggestTodo(FAQUrl.GROUP_NAME_DUPLICATE_URL),
                        null);
                }
                // 15.
                mQClientFactory.start();
                log.info("the consumer [{}] start OK.", this.defaultMQPushConsumer.getConsumerGroup());
                this.serviceState = ServiceState.RUNNING;
                break;
            case RUNNING:
            case START_FAILED:
            case SHUTDOWN_ALREADY:
                throw new MQClientException("The PushConsumer service state not OK, maybe started once, "
                    + this.serviceState
                    + FAQUrl.suggestTodo(FAQUrl.CLIENT_SERVICE_NOT_OK),
                    null);
            default:
                break;
        }
        // 16.更新订阅消息同时更新消费位点
        this.updateTopicSubscribeInfoWhenSubscriptionChanged();
        // 17.检测
        this.mQClientFactory.checkClientInBroker();
        // 18.心跳检测
        this.mQClientFactory.sendHeartbeatToAllBrokerWithLock();
        // 19.立即进行消费者Rebalance重新分消费队列
        this.mQClientFactory.rebalanceImmediately();
    }

}
```

DefaultMQPushConsumerImpl类是在DefaultMQPushConsumer中进行实例化的，在初始化时默认设置其ServiceState为CREATE_JUST。

- 标记点6:配置验证，检测consumer的配饰是否都正确，异常则报错。
- 标记点7:1、复制通过DefaultMQPushConsumer.setSubscription方法设置的订阅关系给DefaultMQPushConsumerImpl的RebalanceImpl字段中。2、增加重试队列的订阅关系到Consumer中。
- 标记点8:如果消费模式为集群消费则，将当前实例的名称修改为具体的IP和进程号
- 标记点9:实例化MQClientManager对象，先从缓存中获取，如果缓存中没有则new出一个新的对象，并放入缓存中。
- 标记点10:设置消费者Rebalance相关属性,同时创建PullAPIWrapper对象，后期从broker拉取消息就是通过这个类来获取的。
- 标记点11:获取消费位点，集群消费模式从Broker获取，广播消费模式则读取本地文件
- 标记点12:创建消息的消费服务，后面消息的业务逻辑执行依赖此服务
- 标记点13:设置消息消费服务状态为开始即在后台开启一个固定频率的线程定时清理过滤的消息(即消费超时的消息，当前时间减去消费开始时间大于消费超时时间的消息)
- 标记点14:将ConsumerImpl放入到MQClientInstance的缓存中。

```java
public class PullMessageService extends ServiceThread {
    private final LinkedBlockingQueue<PullRequest> pullRequestQueue = new LinkedBlockingQueue<PullRequest>();
}
```

- 标记点15:设置MQClientInstance状态为起始状态，1、获取NameServer地址，2、初始化Netty客户端，3、启动一系列的定时服务包含(NameServer地址查找，更新TOPIC订阅关系，
  清理下线的Broker，给Broker发送心跳信息，将消费位点持久化至Broker,调整并发消费的线程池中的线程数量参数)。
  4、启动消息拉取服务-修改状态为启动，消息拉取服务的线程启动,
  启动后线程处于阻塞状态，等待获取PullRequest，如上代码所示,PullRequest是存放在一个阻塞队列里面的，队列里面没有数据的情况下会会一直等待获取。该类的Run方法如下所示,
  启动后会循环的从队列中获取PullRequest，然后从Broker拉取消息。

```java
public class PullMessageService extends ServiceThread {
        @Override
        public void run() {
            log.info(this.getServiceName() + " service started");
  
            while (!this.isStopped()) {
                try {
                    PullRequest pullRequest = this.pullRequestQueue.take();
                    this.pullMessage(pullRequest);
                } catch (InterruptedException ignored) {
                } catch (Exception e) {
                    log.error("Pull Message Service Run Method exception", e);
                }
            }
  
            log.info(this.getServiceName() + " service end");
        }
}
```

5、启动消费者Rebalance服务，修改服务状态为运行中，启动一个线程线程执行Rebalance任务(每隔20秒执行一次)或者当该类的CountDown为0是执行。代码如下:

```java
public class RebalanceService extends ServiceThread {
    public void run() {
        log.info(this.getServiceName() + " service started");

        while (!this.isStopped()) {
            this.waitForRunning(waitInterval);
            this.mqClientFactory.doRebalance();
        }

        log.info(this.getServiceName() + " service end");
    }
}
```

- 标记点16-18:从Broker更新订阅关系，检测通信状态，给所有的Broker发送心跳信息。
- 标记点19:立即进行Consumer的rebalance,通过将CountDown减为0，通知Rebalance立即执行，会调用MQClientInstance的doRebalance方法。
  这个方法循环遍历consumerTable的所有值，执行rebalance。consumerTable这个Map的值在14标记位的时候放入，key位消费者Group,Value为DefaultMQPushConsumerImpl。

```
    public void doRebalance() {
        for (Map.Entry<String, MQConsumerInner> entry : this.consumerTable.entrySet()) {
            MQConsumerInner impl = entry.getValue();
            if (impl != null) {
                try {
                    impl.doRebalance();
                } catch (Throwable e) {
                    log.error("doRebalance exception", e);
                }
            }
        }
    }
```

Value为DefaultMQPushConsumerImpl的doRebalance方法如下所示,调用RebalanceImpl的doRebalance方法

```
    @Override
    public void doRebalance() {
        if (!this.pause) {
            this.rebalanceImpl.doRebalance(this.isConsumeOrderly());
        }
    }

```

RebalanceImpl的doRebalance方法如下所示，rebalanceByTopic方法中，
先获取该TOPIC下所有的消息队列，再获取所有的消费者ID，然后根据设置的均衡策略重新分配消息队列与消费者的对应关系，并更新本地缓存。
更新完缓存后，构造PullRequest对象，并将构建的对象放入到PullMessageService的pullRequestQueue队列中去，
放入后PullMessageService就可以从中获取到对象，执行pullMessage方法，该方法又调用了DefaultMQPushConsumerImpl的pullMessage方法

```
    public void doRebalance(final boolean isOrder) {
        Map<String, SubscriptionData> subTable = this.getSubscriptionInner();
        if (subTable != null) {
            for (final Map.Entry<String, SubscriptionData> entry : subTable.entrySet()) {
                final String topic = entry.getKey();
                try {
                    this.rebalanceByTopic(topic, isOrder);
                } catch (Throwable e) {
                    if (!topic.startsWith(MixAll.RETRY_GROUP_TOPIC_PREFIX)) {
                        log.warn("rebalanceByTopic Exception", e);
                    }
                }
            }
        }

        this.truncateMessageQueueNotMyTopic();
    }
```

DefaultMQPushConsumerImpl的pullMessage方法，在该方法中首先会对本地拉取回来缓存的，未消费的消息的数量及大小进行判断，如果超过上限则不进行拉取，等待一段时间后再拉取。
从Broker拉取消息成功后，会执行CallBack方法，在回调函数中执行消息的消费以及将新的PullRequest对象放入到PullMessageService的pullRequestQueue队列中去。这样就会进行持续的拉取消费，
下面为并发消费的代码：
先判断每次消费的数量如果大于待消费的消息，则直接放入线程池进行消费。如果小于则将消息按消费数量分批次提交给线程池进行消费。
消费完成后会更新当前的消费者的Offset,同时更新PullRequest的信息，并将其压入队列，在PullMessageService拿到该对象后就会重新拉取消息，然后继续消费。

```
    public void submitConsumeRequest(
        final List<MessageExt> msgs,
        final ProcessQueue processQueue,
        final MessageQueue messageQueue,
        final boolean dispatchToConsume) {
        final int consumeBatchSize = this.defaultMQPushConsumer.getConsumeMessageBatchMaxSize();
        if (msgs.size() <= consumeBatchSize) {
            ConsumeRequest consumeRequest = new ConsumeRequest(msgs, processQueue, messageQueue);
            try {
                this.consumeExecutor.submit(consumeRequest);
            } catch (RejectedExecutionException e) {
                this.submitConsumeRequestLater(consumeRequest);
            }
        } else {
            for (int total = 0; total < msgs.size(); ) {
                List<MessageExt> msgThis = new ArrayList<MessageExt>(consumeBatchSize);
                for (int i = 0; i < consumeBatchSize; i++, total++) {
                    if (total < msgs.size()) {
                        msgThis.add(msgs.get(total));
                    } else {
                        break;
                    }
                }

                ConsumeRequest consumeRequest = new ConsumeRequest(msgThis, processQueue, messageQueue);
                try {
                    this.consumeExecutor.submit(consumeRequest);
                } catch (RejectedExecutionException e) {
                    for (; total < msgs.size(); total++) {
                        msgThis.add(msgs.get(total));
                    }

                    this.submitConsumeRequestLater(consumeRequest);
                }
            }
        }
    }
```

#### 注意事项：

1. 集群消费时，Broker上消息队列都有唯一的对象的消费者实例。所以，在集群消费模式下，同一个消费者集群内一条消息只会被一个消费者所消费。
2. 一个消费者集群内，消费者的实例数不应该大于TOPIC在Broker上的消息队列数量，如果大于则有部分消费者实例是会空闲的，造成资源浪费。如果小于一个消费者实例可以对应多个消息队列。
3. 同一个消费者集群中每个实例配置的TAG应该保持统一，防止漏掉消息
4. 如果需要时使用TAG对消息进行过滤，则需要在业务的consumerListener对消息TAG进行二次比较过滤(具体原因可以参见消息拉取流程分析)。
