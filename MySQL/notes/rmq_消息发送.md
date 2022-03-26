# rocketmq消息发送
- 方法地址:org.apache.rocketmq.client.impl.producer.DefaultMQProducerImpl#sendDefaultImpl


## 一、消息长度验证
## 二、查找主题路由
- 方法地址:org.apache.rocketmq.client.impl.producer.DefaultMQProducerImpl#tryToFindTopicPublishInfo
- 说明:查找主题的路由信息的方法。如果生产者中缓存了 topic 的路由信息，如果该路由信息中包含了消息队列，则直接返回该路由信息，如果没有缓存 或没有包含消息队列， 则向 NameServer 查询该 topic 的路由信息。如果最终未找到路由信息，则抛出异常： 无法找到主题相关路由信息异常

![20210711222323](https://xiongshengyu-1256692535.cos.ap-beijing.myqcloud.com/photos/20210711222323.png)

#### 向NameServer查询topic对应的路由信息,可能会更新和维护缓存

##### step1:
- 如果isDefault为true,使用默认主题去查
  - 如果查询到路由信息,替换路由信息中读写队列个数 ← min(消息生产者默认的的队列个数,原读队列个数_

- 否则,使用topic去查询
- 如果未查询到路由信息,返回false,表示路由信息未变化
  
![20210711223227](https://xiongshengyu-1256692535.cos.ap-beijing.myqcloud.com/photos/20210711223227.png)


![20210711223802](https://xiongshengyu-1256692535.cos.ap-beijing.myqcloud.com/photos/20210711223802.png)

##### step2:
- 根据topic查询到的路由信息找到,与本地缓存中的路由信息进行**对比**
- 如果未发生变化,直接返回false
![20210713204832](https://xiongshengyu-1256692535.cos.ap-beijing.myqcloud.com/photos/20210713204832.png)

- 1 **对比**
  - 如果其一为null,返回true
  - 本地保留oldData 和 nowData的拷贝
  - 根据queueData和BrokerData排序
比较是否相等
  - 相等返回 false,否则返回false
![20210713204644](https://xiongshengyu-1256692535.cos.ap-beijing.myqcloud.com/photos/20210713204644.png)

- 2 **对比结果没有发生变化,判断topic对应的路由信息 是否需要更新**
  - 检查MQClientInstance管理的有关topic的消息**发送**路由信息是否需要更新,如需要,则返回
  - 检查MQClientInstance管理的有关topic的消息**订阅**路由信息是否需要更新,如需要,则返回
  
![20210713205856](https://xiongshengyu-1256692535.cos.ap-beijing.myqcloud.com/photos/20210713205856.png)

- 2.1 检查MQClientInstance管理的有关topic的消息发送路由信息是否需要更新

![20210713210059](https://xiongshengyu-1256692535.cos.ap-beijing.myqcloud.com/photos/20210713210059.png)

![20210713210233](https://xiongshengyu-1256692535.cos.ap-beijing.myqcloud.com/photos/20210713210233.png)

- 2.2 检查MQClientInstance管理的有关topic的消息订阅路由信息是否需要更新
- 方法地址:org.apache.rocketmq.client.impl.consumer.MQConsumerInner#isSubscribeTopicNeedUpdate
- 三个实现类(策略模式)

![20210713210544](https://xiongshengyu-1256692535.cos.ap-beijing.myqcloud.com/photos/20210713210544.png)


##### step3: 如果发生变化,更新MQClientInstanceBroker地址缓存表 + 更新MQClientInstance所管辖的所有消息发送,消息订阅关于topic的路由信息,完成更新后,返回true
  
![20210713212617](https://xiongshengyu-1256692535.cos.ap-beijing.myqcloud.com/photos/20210713212617.png)

## 三、选择消息队列

## 四、消息发送
- 方法地址:org.apache.rocketmq.client.impl.producer.DefaultMQProducerImpl#sendKernelImpl

### step1:根据mq获得broker的网络地址
![20210713214731](https://xiongshengyu-1256692535.cos.ap-beijing.myqcloud.com/photos/20210713214731.png)

![20210713214904](https://xiongshengyu-1256692535.cos.ap-beijing.myqcloud.com/photos/20210713214904.png)


### step2:参数设置 && step3:执行钩子函数的增强逻辑
![20210713215003](https://xiongshengyu-1256692535.cos.ap-beijing.myqcloud.com/photos/20210713215003.png)

![20210713215102](https://xiongshengyu-1256692535.cos.ap-beijing.myqcloud.com/photos/20210713215102.png)

### step4:构造发送请求包
![20210713215244](https://xiongshengyu-1256692535.cos.ap-beijing.myqcloud.com/photos/20210713215244.png)

![20210713215316](https://xiongshengyu-1256692535.cos.ap-beijing.myqcloud.com/photos/20210713215316.png)

### step5:根据消息发送方式: 同步, 异步, 单向方式进行网络传输
#### 同步
![20210713220023](https://xiongshengyu-1256692535.cos.ap-beijing.myqcloud.com/photos/20210713220023.png)

![20210713220215](https://xiongshengyu-1256692535.cos.ap-beijing.myqcloud.com/photos/20210713220215.png)
#### 异步/单向
![20210713220234](https://xiongshengyu-1256692535.cos.ap-beijing.myqcloud.com/photos/20210713220234.png)

- 不管是同步/异步/单向发送,都会调用org.apache.rocketmq.client.impl.MQClientAPIImpl#sendMessage

![20210713221050](https://xiongshengyu-1256692535.cos.ap-beijing.myqcloud.com/photos/20210713221050.png)

##### ONEWAY
- org.apache.rocketmq.remoting.RemotingClient#invokeOneway

![20210713223321](https://xiongshengyu-1256692535.cos.ap-beijing.myqcloud.com/photos/20210713223321.png)

- 在invokeOneway方法中
  - (1)获得并创建一个channel,如果channel被激活
  - (2)调用org.apache.rocketmq.remoting.netty.NettyRemotingAbstract#invokeOnewayImpl
  - (3)关闭channel


 ![20210715212605](https://xiongshengyu-1256692535.cos.ap-beijing.myqcloud.com/photos/20210715212605.png) 

 ![20210715213259](https://xiongshengyu-1256692535.cos.ap-beijing.myqcloud.com/photos/20210715213259.png)

- org.apache.rocketmq.remoting.netty.NettyRemotingAbstract#invokeOnewayImpl
- (1)semaphoreOneway tryAccquire,尝试抢占锁成功
- (2)将semaphoreOneway包装成SemaphoreOnlyOnce对象once,表示这个信号量只能被释放一次
- (3)将请求冲刷到channel中,并添加一个监听器
  - 传入ChannelFutureListner实例
  - 重写了io.netty.channel.ChannelFutureListener#operationComplete
    - 入参ChannelFuture f
    - once释放锁
##### ASYNC
- org.apache.rocketmq.client.impl.MQClientAPIImpl#sendMessageAsync

![20210715220732](https://xiongshengyu-1256692535.cos.ap-beijing.myqcloud.com/photos/20210715220732.png)

- (1)调用成员remotingClient的invokeAsync发送请求

- (2)入参传入一个InvokeCallback实例
  
- **#sendMessageAsync → org.apache.rocketmq.remoting.netty.NettyRemotingClient#invokeAsync**

![20210715221843](https://xiongshengyu-1256692535.cos.ap-beijing.myqcloud.com/photos/20210715221843.png)
- 类似的,先获得并创建channel,接着调用invokeAsyncImpl

- org.apache.rocketmq.remoting.netty.NettyRemotingAbstract#invokeAsyncImpl
  
在该方法中
- (1)同样使用semaphore来控制并发量,并对semaphore包装,使得锁只能被释放一次
- (2)<font color=red>**不同**:获得锁之后
  - (2.1)如果超时,才会释放锁
  - (2.2)创建了一个ResponseFuture实例</font>
- (3)将请求冲刷到channel中,并添加一个监听器
  - 传入ChannelFutureListner实例
  - 重写了io.netty.channel.ChannelFutureListener#operationComplete
    - 入参ChannelFuture f
    - 当f成功,才会设置responseFuture的发送请求成功标志为true

![20210715222731](https://xiongshengyu-1256692535.cos.ap-beijing.myqcloud.com/photos/20210715222731.png)


- **#sendMessageAsync → invokeCallback**
  - 获得responseFuture的响应,并处理发送结果

##### SYNC
- org.apache.rocketmq.client.impl.MQClientAPIImpl#sendMessageSync
- <font color=red>在这个方法里调用成员remotingClient的invokeSync方法,获得相应结果response</font>


![20210715225823](https://xiongshengyu-1256692535.cos.ap-beijing.myqcloud.com/photos/20210715225823.png)

org.apache.rocketmq.remoting.netty.NettyRemotingClient#invokeSync
![20210715230322](https://xiongshengyu-1256692535.cos.ap-beijing.myqcloud.com/photos/20210715230322.png)
- 类似的,先获得并创建channel,接着调用invokeSyncImpl

- org.apache.rocketmq.remoting.netty.NettyRemotingAbstract#invokeSyncImpl
<!-- 在这里补充一张截图 -->

```Java
 public RemotingCommand invokeSyncImpl(final Channel channel, final RemotingCommand request,
        final long timeoutMillis)
        throws InterruptedException, RemotingSendRequestException, RemotingTimeoutException {
        final int opaque = request.getOpaque();

        try {
            final ResponseFuture responseFuture = new ResponseFuture(channel, opaque, timeoutMillis, null, null);
            this.responseTable.put(opaque, responseFuture);
            final SocketAddress addr = channel.remoteAddress();
            channel.writeAndFlush(request).addListener(new ChannelFutureListener() {
                @Override
                public void operationComplete(ChannelFuture f) throws Exception {
                    if (f.isSuccess()) {
                        responseFuture.setSendRequestOK(true);
                        return;
                    } else {
                        responseFuture.setSendRequestOK(false);
                    }

                    responseTable.remove(opaque);
                    responseFuture.setCause(f.cause());
                    responseFuture.putResponse(null);
                    log.warn("send a request command to channel <" + addr + "> failed.");
                }
            });

            RemotingCommand responseCommand = responseFuture.waitResponse(timeoutMillis);
            if (null == responseCommand) {
                if (responseFuture.isSendRequestOK()) {
                    throw new RemotingTimeoutException(RemotingHelper.parseSocketAddressAddr(addr), timeoutMillis,
                        responseFuture.getCause());
                } else {
                    throw new RemotingSendRequestException(RemotingHelper.parseSocketAddressAddr(addr), responseFuture.getCause());
                }
            }

            return responseCommand;
        } finally {
            this.responseTable.remove(opaque);
        }
    }
```

在该方法中
- (1)<font color=red>没有使用semaphore来控制并发量</font>
- (2)~~<font color=red>**不同**:获得锁之后~~
  - ~~(2.1)如果超时,才会释放锁~~
  - ~~(2.2)创建了一个ResponseFuture实例</font>~~
- (2)<font color=red>创建了一ResponseFuture实例</font>
- (3)将请求冲刷到channel中,并添加一个监听器
  - 传入ChannelFutureListner实例
  - 重写了io.netty.channel.ChannelFutureListener#operationComplete
    - 入参ChannelFuture f
    - 当f成功,才会设置responseFuture的发送请求成功标志为true
      - <font color=red>否则,设置responseFuture发送请求标志为false</font>
- (4)<font color=red>获得responseFuture的响应(对比ASYNC,在回调函数中才会获得responseFuture的响应,接着进行后续操作)
  - 所以,同步方式发送,只要没有获得响应,就会一直阻塞</font>

**区别**:

ASYNC 和 SYNC

(1) ResponseFuture 对象处理:

- ASYNC方式: 
  - client.impl.MQClientAPIImpl#sendMessageAsync 中调用 emoting.netty.NettyRemotingClient#invokeAsync
    - 该方法的入参会传入**回调函数** InvokeCallback
    - 在**回调函数**中会获得responseFuture的响应(responseCommand),接着进行后续处理
    - 所以一次发送如果没有获得响应,在#invokeAsync方法中,并不会被阻塞
- SYNC方式: 在 remoting.netty.NettyRemotingAbstract#invokeSyncImpl 中获得responseFuture的响应(responseCommand)
  - 因为在发送消息的实现类中获得ResponseFuture的响应,所以在没有超时的情况下,如果一次发送 没有获取到响应,就会一直阻塞,直到获取到响应为止

![20210717095928](https://xiongshengyu-1256692535.cos.ap-beijing.myqcloud.com/photos/20210717095928.png)

![20210717100219](https://xiongshengyu-1256692535.cos.ap-beijing.myqcloud.com/photos/20210717100219.png)

(2) 并发: ASYNC需要控制并发,而SYNC不需要控制并发

- 在rocketmq中是通过信号量来控制并发
- 道理很简单, 由于获得responseFuture响应的时机不同, ASYNC如果一个线程一次发送没有获得响应不会被阻塞;而SYNC如果一个线程一次发送没有获得响应会被阻塞,所以SYNC无需控制并发
  
### step6:如果注册了消息发送钩子函数,执行after逻辑,如果消息发送过程中发生了能够被捕获的异常,该方法也会被执行



![20210713220601](https://xiongshengyu-1256692535.cos.ap-beijing.myqcloud.com/photos/20210713220601.png)

