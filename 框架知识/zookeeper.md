##zookeeper
	分布式协调服务-监视集群节点状态
	基本架构
		文件系统
			多层级节点命名空间
			文件节点-关联数据-1M
			znode
				持久化 持久化有序 临时 临时有序
				临时节点不能创建子节点
				临时节点移除 -- SESSIONEXPIRED
		watch机制
			znode变化,通知client
			一次性触发,需重复注册
			数据版本
			注册watch
				数据watch  getData()和exists()
				子节点watch  getChildren()
			触发watch
				create、delete、setData
	核心	
		原子广播（zab协议）
			恢复模式
				选主 
					fast paxos算法
					全新
						server id大的胜出
					非全新
						version、serverid 和逻辑时钟
						1、逻辑时钟小的选举结果被忽略，重新投票
				　　　　2、统一逻辑时钟后，数据 version 大的胜出
				　　　　3、数据 version 相同的情况下，server id 大的胜出
				数据同步
					根据zxid(递增的事务id)确定同步点
				三种情况
					启动
					ping follower没超过一半
					leader 挂掉
			广播模式
				数据一致性 
				两阶段提交
				事务发起 -- 集群follower -- 投票超半数 -- commit
		server状态
			leading  --- 当前为leader
			following --- 同步状态
			looking --- 寻找leader
		有序性
			zxid
				低32位-递增计数器
				高32位-epoch 区分leader周期
			读数据-其中一台server
			写数据
				follower接受请求转发给leader
				leader接受请求广播所有server		
		集群机制
			超过半数的节点正常则正常服务
			follower
			1.PING消息： 心跳消息；
		　　2.PROPOSAL消息：Leader发起的提案，要求Follower投票；
		　　3.COMMIT消息：服务器端最新一次提案的信息；
		　　4.UPTODATE消息：表明同步完成；
		　　5.REVALIDATE消息：根据Leader的REVALIDATE结果，关闭待revalidate的session还是允许其接受消息；
		　　6.SYNC消息：返回SYNC结果到客户端，这个消息最初由客户端发起，用来强制得到最新的更新。
	应用
		注册中心 -- dubbo
			服务名对应节点，节点数据即为服务的ip和port
		集群管理 -- kafka
			监控服务器状态
			master选举
			实现：
				服务器加入，父节点下创建临时有序节点，设置watch
				服务器故障，节点自动删除
				节点id最小即为master
		动态配置
			创建节点绑定数据，注册数据watch
		分布式锁
			父节点下创建临时有序节点，注册watch，id最小的获得锁
	问题
		为什么需要leader ---- 一台机器执行，共享结果，避免重复计算
		zookeeper应用