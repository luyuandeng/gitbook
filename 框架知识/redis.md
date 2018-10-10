##redis
	核心
		数据类型
			string 
				int - shared常量池
				raw
				包含任何数据，图片，序列化对象，字符串
			hash 
				数组-小数据量-默认64
				hashmap-大数据量
				存储对象
					id->{
						field:value
					}
					id+field操作属性数据,无需操作整个对象
			list 双向链表
			set 去重，value=null的hashmap 
			zset 
				score hashMap 跳跃表 排序
		内存模型
			key-value
				实现同hashmap，都是数组+链表方式，entry对象（key value next）
			数据结构抽象
				redisObject
					type encoding lru ptr指针
			虚拟内存
				会把不常用键的值放到磁盘
				内存与磁盘swap，影响性能
			淘汰策略
				内存使用完毕，再次申请内存
			 	1.拒绝处理，报错(默认)
				2.主键或有过期时间主键，移除key(LRU最近未使用,随机,失效时间近)
			过期策略
				1.给每一个key设置timer
				2.定期删除
					定期扫描一定数量设置了过期时间的key，然后清除
				3.惰性过期
					get时判断过期删除
		网络模型
			io多路复用
			单线程
			epoll
		持久化
			快照
				定时任务-是否达到触发条件-次数/时间
				fork子进程-共享内存空间-遍历写到RDB文件
				copy-on-write
				缺点：
					丢失上次快照与重启之间所有的数据
					Buffer IO问题：快照文件读写时操作系统会有cache，重复存储
			aof
				数据改变追加到log文件，重启读取文件执行一遍
				缺点：
					log过大，恢复时间长
					每次改变都要写log
		事务
			一次执行多个命令
			1.原子性
				全部成功或全部失败
			2.隔离性
				序列化 顺序执行 不可打断
			MULTI 、 EXEC 、 DISCARD 和 WATCH
			事务提交前，不会执行任何指令，只会把它们存到一个队列
			CAS
		Pipelining
			客户端一次发送多个命令 顺序执行
	集群
		三种技术
			replication
				避免单机故障
				原理
					slave启动-发送sync到master-master生成rdb和命令缓存同步给slave-后面命令逐个同步的(异步)
					slave断开重连-增量复制
				2.8后可选无硬盘复制
			sentinel
				哨兵机制-监控+故障转移
				1.心跳监测
				2.选举master，并重新设置slave的master
				3.等待老master恢复并设为slave
			sharding
		两种方案
			主从+哨兵
				一主多从+故障转移
				写主读随机
				节点存全量数据，node压力大
			分片+主从
				数据多节点分布存储
				hash读写node
				主从保证单node高可用，支持故障重新选举
				节点存部分数据，node压力小，多服务器
				分片实现
					客户端分片
						优点：服务器独立，实现简单
						缺点: 无法动态扩容，额外维护HA以及监控节点状态
					代理分片
						Twemproxy
						路由透明，节点自动删除
					服务器分片
						Redis Cluster
						优点：支持HA 动态扩容 节点状态同步
						原理
							hash槽
								key的空间被分到16384个hash slot里
								计算key属于哪个slot
								每个节点负责一定数量的槽
							扩容
								迁移槽和数据
	优化
		设置最大内存 - 避免使用虚拟内存 - 避免频繁swap
		关闭vm
		适当选择或关闭持久化方式
		选择合适数据类型
			设置常量池
			设置hash存储结构
		不要超过3/5
		Master-slave配置 只在slave上配置数据持久化
		redis与DB同步写的问题，先写DB，后写redis，因为写内存基本上没有问题
	高性能
		即使单线程也很快的原因 
			10w+QPS
			1.减少线程切换，不用考虑锁
			2.纯内存操作
			3.epoll 多路复用
			4.数据结构设计
			5.纯c语言编写
		单线程缺点：
			耗时请求会阻塞
			不能有效利用多核
	问题
		先更新缓存还是数据库
		Memcache与Redis的区别都有哪些
		持久化方式为啥不rdb和aof结合使用
		分布式锁实现
			SETNX 当key不存在，set成功返回1 单线程顺序执行
		keys会阻塞，代替keys方法
			scan 无阻塞 有重复
		异步队列
			list结构作为队列，rpush生产消息，lpop消费消息 blpop消费时会阻塞直到消息到来
			pub/sub实现1-n 消费者下线消息会丢失
			延时队列 使用sortedset 时间戳排序
		大量key设置相同过期时间
			在时间上加一个随机值 防止卡顿