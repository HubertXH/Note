消息发送者发送消息后，会进入到NettyRemotingAbstract的processMessageReceived方法中进行处理
```
    // 根据请求指令判断是 request请求还是response请求，根据不同的类型进行不同的处理
    public void processMessageReceived(ChannelHandlerContext ctx, RemotingCommand msg) throws Exception {
        final RemotingCommand cmd = msg;
        if (cmd != null) {
            switch (cmd.getType()) {
                case REQUEST_COMMAND:
                    processRequestCommand(ctx, cmd);
                    break;
                case RESPONSE_COMMAND:
                    processResponseCommand(ctx, cmd);
                    break;
                default:
                    break;
            }
        }
    }



    // request请求处理
    public void processRequestCommand(final ChannelHandlerContext ctx, final RemotingCommand cmd) {
        // 先根据请求的CDOE获取处理Processor，再Broker启动的时候会通过BrokerController的registerProcessor方法将对应的处理方法加载进内存中。
        // 包含消息发送SendMessageProcessor，消息拉取PullMessageProcessor， 消息查询QueryMessageProcessor，客户端管理ClientManageProcessor，
        // 消费者管理ConsumerManageProcessor等Processor的注入。
        final Pair<NettyRequestProcessor, ExecutorService> matched = this.processorTable.get(cmd.getCode());
        final Pair<NettyRequestProcessor, ExecutorService> pair = null == matched ? this.defaultRequestProcessor : matched;
        final int opaque = cmd.getOpaque();

        if (pair != null) {
            // 创建消息写入的处理方法。
            Runnable run = new Runnable() {
                @Override
                public void run() {
                    try {
                        // 获取请求方的地址
                        String remoteAddr = RemotingHelper.parseChannelRemoteAddr(ctx.channel());
                        doBeforeRpcHooks(remoteAddr, cmd);
                        final RemotingResponseCallback callback = new RemotingResponseCallback() {
                        ......
                        };
                        // 根据处理器的类型调用处理方法，执行Message写入落盘
                        if (pair.getObject1() instanceof AsyncNettyRequestProcessor) {
                            // 异步消息发送处理
                            AsyncNettyRequestProcessor processor = (AsyncNettyRequestProcessor)pair.getObject1();
                            processor.asyncProcessRequest(ctx, cmd, callback);
                        } else {
                            // 同步消息发送处理
                            NettyRequestProcessor processor = pair.getObject1();
                            RemotingCommand response = processor.processRequest(ctx, cmd);
                            callback.callback(response);
                        }
                    } catch (Throwable e) {
                        .....
                    }
                }
            };

            // 如果当前Broker的系统的 page cache 是否繁忙。 根据page cache数据开始落盘的时间与当前时间差，是否大于配置的osPageCacheBusyTimeOutMills默认为1000ms
            // 如果大于则认为系统IO繁忙则拒绝请求。
            if (pair.getObject1().rejectRequest()) {
                final RemotingCommand response = RemotingCommand.createResponseCommand(RemotingSysResponseCode.SYSTEM_BUSY,
                    "[REJECTREQUEST]system busy, start flow control for a while");
                response.setOpaque(opaque);
                ctx.writeAndFlush(response);
                return;
            }

            try {
                // 提交运行方法至消息发送的处理线程池，进行消息处理
                final RequestTask requestTask = new RequestTask(run, ctx.channel(), cmd);
                pair.getObject2().submit(requestTask);
            } catch (RejectedExecutionException e) {
                ......
            }
        } else {
            // 获取不到处理器，则返回异常信息
            String error = " request type " + cmd.getCode() + " not supported";
            final RemotingCommand response =
                RemotingCommand.createResponseCommand(RemotingSysResponseCode.REQUEST_CODE_NOT_SUPPORTED, error);
            response.setOpaque(opaque);
            ctx.writeAndFlush(response);
            log.error(RemotingHelper.parseChannelRemoteAddr(ctx.channel()) + error);
        }
    }
```
从上面的代码可知，发送消息的指令最终会提交至BrokerController的sendMessageExecutor线程池进行消息处理，当线程池中堆积的任务满了就会直接拒绝提交。
```
    // 同步消息发送处理
    @Override
    public RemotingCommand processRequest(ChannelHandlerContext ctx,
                                          RemotingCommand request) throws RemotingCommandException {
        RemotingCommand response = null;
        try {
            // 调用asyncProcessRequest方法后，直接调用get,一直阻塞等待结果返回。
            response = asyncProcessRequest(ctx, request).get();
        } catch (InterruptedException | ExecutionException e) {
            log.error("process SendMessage error, request : " + request.toString(), e);
        }
        return response;
    }

    public CompletableFuture<RemotingCommand> asyncProcessRequest(ChannelHandlerContext ctx,
                                                                  RemotingCommand request) throws RemotingCommandException {
        final SendMessageContext mqtraceContext;
        switch (request.getCode()) {
            case RequestCode.CONSUMER_SEND_MSG_BACK:
                return this.asyncConsumerSendMsgBack(ctx, request);
            default:
                SendMessageRequestHeader requestHeader = parseRequestHeader(request);
                if (requestHeader == null) {
                    return CompletableFuture.completedFuture(null);
                }
                mqtraceContext = buildMsgContext(ctx, requestHeader);
                this.executeSendMessageHookBefore(ctx, request, mqtraceContext);
                if (requestHeader.isBatch()) {
                    return this.asyncSendBatchMessage(ctx, request, mqtraceContext, requestHeader);
                } else {
                    return this.asyncSendMessage(ctx, request, mqtraceContext, requestHeader);
                }
        }
    }
```
