# RocketMQ笔记总结

## 生产者

### 基础概念

- 概念：发布消息的角色。Producer通过 MQ 的负载均衡模块选择相应的 Broker 集群队列进行消息投递，投递的过程支持快速失败和重试。
- 队列

	- Topic分区，为了更高的水平扩展能力
	- 模型

		- 为了支持高并发和水平扩展，需要对 Topic 进行分区，在 RocketMQ 中这被称为队列，一个 Topic 可能有多个队列，并且可能分布在不同的 Broker 上。
		- 

	-  MinOffset起始位点
	- MaxOffset最大位点

- topic

	- 发送消息的主体

- body

	- 消息内容

- properties

	- 消息属性

- transactionId

	- 会在事务消息中使用。

- tag

	- Topic 与 Tag 都是业务上用来归类的标识，区别在于 Topic 是一级分类，而 Tag 可以理解为是二级分类。使用 Tag 可以实现对 Topic 中的消息进行过滤。
	- 例子：打成订单，订单号订单状态可以作为tag细粒度区分topic下面的消息
	- 模型

		- 

- DelayTimeLevel

	- 消息延时级别，0 表示不延时，大于 0 会延时特定的时间才会被消费

- WaitStoreMsgOK

	- 表示消息是否在服务器落盘后才返回应答。


- Keys

	- Apache RocketMQ 每个消息可以在业务层面的设置唯一标识码 keys 字段，方便将来定位消息丢失问题。 Broker 端会为每个消息创建索引（哈希索引），应用可以通过 topic、key 来查询这条消息内容，以及消息被谁消费。由于是哈希索引，请务必保证 key 尽可能唯一，这样可以避免潜在的哈希冲突。
	- 系统保留Key集合

		- RocketMQ系统保留的属性Key集合有如下，需要在使用过程中避免： TRACE_ON、MSG_REGION、KEYS、TAGS、DELAY、RETRY_TOPIC、REAL_TOPIC、REAL_QID、TRAN_MSG、PGROUP、MIN_OFFSET、MAX_OFFSET、BUYER_ID、ORIGIN_MESSAGE_ID、TRANSFER_FLAG、CORRECTION_FLAG、MQ2_FLAG、RECONSUME_TIME、UNIQ_KEY、MAX_RECONSUME_TIMES、CONSUME_START_TIME、POP_CK、POP_CK_OFFSET、1ST_POP_TIME、TRAN_PREPARED_QUEUE_OFFSET、DUP_INFO、EXTEND_UNIQ_INFO、INSTANCE_ID、CORRELATION_ID、REPLY_TO_CLIENT、TTL、ARRIVE_TIME、PUSH_REPLY_TIME、CLUSTER、MSG_TYPE、INNER_MULTI_QUEUE_OFFSET、_BORNHOST

### 发送功能

- 发送方式

	- 同步

		- 模型

			- 

		- 注意

			- 同步发送方式请务必捕获发送异常，并做业务侧失败兜底逻辑，如果忽略异常则可能会导致消息未成功发送的情况。

		- 定义

			- 同步发送是最常用的方式，是指消息发送方发出一条消息后，会在收到服务端同步响应之后才发下一条消息的通讯方式
	

		- 使用场景

			- 如重要的通知消息、短消息通知等

	- 异步

		- 模型

			- 

		- 定义

			- 异步发送是指发送方发出一条消息后，不等服务端返回响应，接着发送下一条消息的通讯方式。

		- 备注

			- 异步发送需要实现异步发送回调接口（SendCallback）

		- 使用场景

			- 消息发送方在发送了一条消息后，不需要等待服务端响应即可发送第二条消息，发送方通过回调接口接收服务端响应，并处理响应结果。异步发送一般用于链路耗时较长，对响应时间较为敏感的业务场景。例如，视频上传后通知启动转码服务，转码完成后通知推送转码结果等。

		- 代码案例

			- 

	- 单向传输

		- 定义

			- 发送方只负责发送消息，不等待服务端返回响应且没有回调函数触发，即只发送请求不等待应答。此方式发送消息的过程耗时非常短，一般在微秒级别。适用于某些耗时非常短，但对可靠性要求并不高的场景，例如日志收集。

		- 模型

			- 

		- 场景

			- 日志收集

		- 代码案例

			- 

		- 注意

			- 单向模式调用sendOneway，不会对返回结果有任何等待和处理。

- 发送模式

	- 普通消息发送
	- 顺序消息发送

		- 子主题 1
		- 子主题 2

	- 延迟消息发送
	- 批量消息发送
	- 事务消息发送

## 消费者

### 基础概念

- consumer

	- 消息消费的角色

- 分组

	- 为了消费能力的水平扩展，ConsumerGroup概念应运而生

### 负载均衡模式：

- 广播模式

	- 注意：

		- 同一个 ConsumerGroup 中的每个 Consumer 实例都处理全部的队列。需要注意的是，广播模式下因为每个 Consumer 实例都需要处理全部的消息，因此这种模式仅推荐在通知推送、配置同步类小流量场景使用

- 集群模式

	- 

- 推push
- 拉pull

## 名字服务器NameServer

### 基础概念

- NameServer是一个简单的 Topic 路由注册中心，支持 Topic、Broker 的动态注册与发现。

### 主要两个功能

- broker管理

	- NameServer接受Broker集群的注册信息并且保存下来作为路由信息的基本数据。然后提供心跳检测机制，检查Broker是否还存活

- 路由信息管理

	- 每个NameServer将保存关于 Broker 集群的整个路由信息和用于客户端查询的队列信息。Producer和Consumer通过NameServer就可以知道整个Broker集群的路由信息，从而进行消息的投递和消费。

### 都是独立的个体

- NameServer通常会有多个实例部署，各实例间相互不进行信息通讯。Broker是向每一台NameServer注册自己的路由信息，所以每一个NameServer实例上面都保存一份完整的路由信息。当某个NameServer因某种原因下线了，客户端仍然可以向其它NameServer获取路由信息。

## 代理服务器Broker

### 主要功能

- Broker主要负责消息的存储、投递和查询以及服务高可用保证。
- 子主题 2

## 模型图解

### 简单基础消费Pub/Sub模型

- 

### 扩展后的消息模型

- 模型
- 存储消息Topic的 代理服务器( Broker )，是实际部署过程对应的代理服务器。
- 在集群模式下，同一个 ConsumerGroup 中的 Consumer 实例是负载均衡消费，如图中 ConsumerGroupA 订阅 TopicA，TopicA 对应 3个队列，则 GroupA 中的 Consumer1 消费的是 MessageQueue 0和 MessageQueue 1的消息，Consumer2是消费的是MessageQueue2的消息。

### RocketMQ部署模型

- 提出问题：

	- roducer、Consumer又是如何找到Topic和Broker的地址呢？消息的具体发送和接收又是怎么进行的呢？

- 模型

	- 

## 业务背景

## 基础概念汇总：

### 队列

- 生产更多的消息，对消息写入的能力水平扩展,MQ进行了分区。这就是队列

### 分组

- 为了消费能力的水平扩展，ConsumerGroup概念应运而生

