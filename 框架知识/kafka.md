##kafka
	zookeeper作用
		broker动态加入和离开 
			topics
			partitions
		consumer动态加入和离开
			订阅的topic
			指定的group
		维护consumer与partition的负载均衡
		维护消费关系以及offset
		leader选举
	基本架构
		broker
			controller
				在ZooKeeper注册Watch
				与zookeeper交互
				leader 选举
				failover
		topic
			创建
			删除
		partition
			replica
				副本，保证分区高可用
				分配算法，不得大于broker数
			leader partition
				leader选举产生
				leader所在broker宕机后,zookeeper会重新选举
				与生产消费者交互
			follower partition
				leader接受消息后同步到follower	
		ISR
			leader记录的与其保持同步的replica列表
			移除条件
				数据落后太多
				超过一定时间未响应
			leader partition挂掉会从ISR中选举leader
		group
			被group的一个consumer消费,能被多个group消费
			group下 线程数=partition数量 效率最高
			group下的consumer并行消费不同的partition
		segment
			分区中消息存储单位
			当消息数量达到配置值或发布时间超过阈值，segment会flush到磁盘
			当segment达到一定数量后创建新的segment
			组成
				.index 
				.log
				文件名 - 前文件最后一条消息id+1
	生产者
		发消息
			1.Metadata 协议获取topic的partition列表，每个partition leader所在broker
			2.负载均衡获取partition
				hash 轮询
			3.给partition leader所在broker发消息
		消费可靠性
			at most once
			at least once 默认
			exactly once
		确认模式
			ack=0 发送即成功
			ack=1 leader接受即成功
			ack=-1 全部follower接受才成功
		批量发送
			减少网络请求和磁盘IO
	消费者
		group，topic，多线程，pull机制
		offset
			high-level --> zookeeper
			low-level --> 自己维护
		消费可靠性
			at most once
			at least once 默认
			exactly once
			先commit再处理消息
	message
		有序 持久化 有效期
		消息不会立即同步磁盘，暂存buffer，达到阈值，flush到磁盘
		以追加的方式顺序写入磁盘，消费时从磁盘读取
	特点
		分布式
		吞吐量
		持久化
		容错性
	高性能
		读消息-零拷贝
			省去两次复制
			读取文件：读到内核-复制到用户空间-复制到socket内核空间
			零拷贝：建立内存与磁盘的映射，数据直接从DMA内核到socket内核
		写消息-顺序写磁盘
			省去机械耗时
		批量压缩
	服务器配置
		参数
		优化
	问题
		重复消费问题
			幂等性处理
		消息丢失
			消费者commit后处理消息失败，人工补偿
		保证消息有序问题
			kafka只能保证单个partition的有序性
			可通过指定分区发送和消费保证整体有序
		对比其他消息中间件
			吞吐量
			事务性
			同步 异步
			push pull
			消息有效期