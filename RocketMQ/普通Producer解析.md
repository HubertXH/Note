### Producer Example    
示例代码:
```java
public class Producer {
    public static void main(String[] args) throws MQClientException {
        DefaultMQProducer producer = new DefaultMQProducer("ProducerGroupName");
        producer.setNamesrvAddr("127.0.0.1");
        producer.setInstanceName("instanceName");
        producer.start();
        try{
            Message msg = new Message("TopicTest","TagA","OrderID188",
                "Hello world".getBytes(RemotingHelper.DEFAULT_CHARSET));
            SendResult sendResult = producer.send(msg);
            System.out.printf("%s%n", sendResult);
        }catch (Exception e){
            e.printStackTrace();
        }
        producer.shutdown();
    }
}
```
1. 创建一个producer,在构造函数中设置GroupName,设置nameServer地址,设置producer实例的名称(在同一个应用中，相同ProducerGroupName，相同instanceName的实例只能启动一个。    
所以为了防止启动多个producer实例失败，应该设置instanceName亦或者 在同一个引用中将producer的实例进行统一的管理)。  
2. DefaultMQProducer默认的发送超时时间为3秒中，默认重试次数为2，
3. 一个消息默认总共会发送3次，若3次发送都失败则，认为发送消息失败。     
4. 消息的的大小大于4K时会启动消息压缩。    
5. 创建producer时会初始化DefaultMQProducerImpl实列。启动Producer时或调用DefaultMQProducerImpl的start方法，在DefaultMQProducerImpl的start方法中会进行配置检查、实例化MQClientInstance、
向MQClientInstance中注册Producer、启动MQClientInstance、向Broker发送心跳信息、启动一个线程以固定的频率删除发送超时的请求。**MQClientInstance的启动解析可以查看Consumer中的解析**。    


```
producer.send(Message msg)
```

该方法会调用DefaultMQProducerImpl的sendDefaultImpl方法。
```
TopicPublishInfo topicPublishInfo = this.tryToFindTopicPublishInfo(msg.getTopic());
```
1. 先根据消息中的TOPIC,从NameServer获取TOPIC相关的信息(调用的是MQClientInstance的updateTopicRouteInfoFromNameServer，同时更新DefaultMQProducerImpl中topicPublishInfoTable中的TOPIC信息)
```
    MessageQueue mqSelected = this.selectOneMessageQueue(topicPublishInfo, lastBrokerName);
    ......   
    public MessageQueue selectOneMessageQueue() {
        int index = this.sendWhichQueue.incrementAndGet();
        int pos = Math.abs(index) % this.messageQueueList.size();
        if (pos < 0)
            pos = 0;
        return this.messageQueueList.get(pos);
    }
```
2. 根据TOPIC返回的队列信息，选择一个消息队列。默认的选择是通过轮询的方式来选择队列。
```
String brokerAddr = this.mQClientFactory.findBrokerAddressInPublish(mq.getBrokerName());
```
3. 根据Broker名称获取当前Broker的地址。
4. 根据消息大小看是否需要进行消息压缩，执行消息检测钩子，进行远程通信前的钩子，构建消息发送请求SendMessageRequestHeader
```
    public SendResult sendMessage(
        final String addr,
        final String brokerName,
        final Message msg,
        final SendMessageRequestHeader requestHeader,
        final long timeoutMillis,
        final CommunicationMode communicationMode,
        final SendCallback sendCallback,
        final TopicPublishInfo topicPublishInfo,
        final MQClientInstance instance,
        final int retryTimesWhenSendFailed,
        final SendMessageContext context,
        final DefaultMQProducerImpl producer
    ) throws RemotingException, MQBrokerException, InterruptedException {
    ......
    request = RemotingCommand.createRequestCommand(RequestCode.SEND_MESSAGE, requestHeader);
    ......
    switch (communicationMode) {
        // 只发送，不需要送结果，可以保持最大吞吐量，但是存储在消息丢失的情况。
        case ONEWAY:
            this.remotingClient.invokeOneway(addr, request, timeoutMillis);
            return null;
        // 异步发送，发送后会执行sendCallback的方法，再该方法中需要实现发送成功和发送失败的的处理。用于事务消息的发送
        case ASYNC:
            final AtomicInteger times = new AtomicInteger();
            long costTimeAsync = System.currentTimeMillis() - beginStartTime;
            if (timeoutMillis < costTimeAsync) {
                throw new RemotingTooMuchRequestException("sendMessage call timeout");
            }
            this.sendMessageAsync(addr, brokerName, msg, timeoutMillis - costTimeAsync, request, sendCallback, topicPublishInfo, instance,
                retryTimesWhenSendFailed, times, context, producer);
            return null;
        // 同步发送，消息发送后会等待发送的结果，一般情况下都使用这种模式进行消息发送。消息发送失败后，会进行重试发送，超过最大发送次数后仍发送失败，则抛出异常，需要调用者手动处理异常
        case SYNC:
            long costTimeSync = System.currentTimeMillis() - beginStartTime;
            if (timeoutMillis < costTimeSync) {
                throw new RemotingTooMuchRequestException("sendMessage call timeout");
            }
            return this.sendMessageSync(addr, brokerName, msg, timeoutMillis - costTimeSync, request);
        default:
            assert false;
            break;
    }

}
```
5. 根据消息的发送模式，进行异步或者同步消息发送。调用MQClientAPIImpl中的sendMessage方法。根据SendMessageRequestHeader构建一个远程请求对象，
再根据发送模式进行消息发送。





