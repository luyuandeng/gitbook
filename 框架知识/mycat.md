=======================================================================================知识点===============================================================================
http://3y.uu456.com/bp_368qf8k9pu1wxgv8jpt4_3.html
http://www.songwie.com/
#mycat版本功能预览
https://github.com/MyCATApache/Mycat-download/blob/master/Changelog.md
===============================================================================================mycat配置文件==================================================================
server.xml
系统配置信息，映射为SystemConfig
	system:全局系统配置
		####基础配置
		charest:保证mycat字符集与mysql一致
		defaultSQLparse:sql解析器，可选druidparse(默认),fdbparse
		bindIp：0.0.0.0
		serverPort:8066
		managerPort:9066

		useOffHeapForMerge:是否使用direct memory处理跨分片结果集 1使用 0不使用
			mycat内存分层：结果集处理（可选heap和direct），网络处理（direct memory），系统预留处理(heap)
			内存外存并放处理结果集：
			a.此时前端和后端空闲连接超时检测时间应该设置大些，避免空闲检测关闭 front 或者 backend 
			connection,造成 Mysqlclient 连接丢失时结果集无法正确。 
	   		b.设置-Xmn 值尽可能大些，新生代使用 UseParallelGC 垃圾回收器，-Xss 设置 512K 比较合适，物理内
			存足够时，MaxDirectMemorySize 尽可能设置大些，可以加快结果集处理时间， 
	   		例如:-Xmx1024m -Xmn512m -XX:MaxDirectMemorySize=2048m -Xss256k -XX:+UseParallelGC 

	   	handleDistributedTransactions：0为不过滤分布式事务，1为过滤分布式事务（如果分布式事务内只涉及全局表，则不过滤），2为不过滤分布式事务,但是记录分布式事务日志
		useGlobleTableCheck：1为开启全局表一致性检测、0为关闭 检测时间为默认为24小时
		sequnceHanderType:全局序列实现方式，可选0本地文件方式，1数据库方式，2时间戳方式，3分布式zk id生成器，4zk递增id
		useSqlStat：1为开启实时统计、0为关闭 默认开启
		useCompression：1为开启mysql压缩协议、0为关闭 默认关闭

		####线程相关
		processors:系统可用的线程数，默认为cpu核心线程数
		processorBufferChunk：每次从buffer池分配的buffer大小 ，默认4096
		processorBufferPool：buffer池的大小，所有线程共享，默认 bufferChunkSize(4096) * processors 属性 *  1000
		processorBufferLocalPercent：默认100，决定线程buffer百分比，线程buffer百分比=该值/线程数 ，
			buffer池分为线程共享的buffer pool和线程独有的ThreadLocalPool 
			ThreadLocalPool =线程buffer百分比/100 *processorBufferPool
	
		####心跳检测属性
		processorCheckPeriod : 清理 NIOProcessor 上前后端空闲、超时和关闭连接的间隔时间。默认是 1 秒，单位毫秒。。 
		dataNodeIdleCheckPeriod : 对后端连接进行空闲、超时检查的时间间隔，默认是 300 秒，单位毫秒。 
		dataNodeHeartbeatPeriod : 对后端所有读、写库发起心跳的间隔时间，默认是 10 秒，单位毫秒。 

		####mysql连接属性
		packetHeaderSize : 指定 Mysql 协议中的报文头长度。默认 4。 
		maxPacketSize : 指定 Mysql 协议可以携带的数据最大长度。默认 16M。 
		idleTimeout : 指定连接的空闲超时时间。某连接在发起空闲检查下，发现距离上次使用超过了空闲时间，那么这个连接会被回收，就是被直接的关闭掉。默认 30 分钟，单位毫秒。 
		charset : 连接的初始化字符集。默认为 utf8。 
		txIsolation : 前端连接的初始化事务隔离级别，只在初始化的时候使用
		sqlExecuteTimeout:SQL 执行超时的时间，Mycat 会检查连接上最后一次执行 SQL 的时间，若超过这个时间则会直接关闭这连接。默认时间为 300 秒，单位秒。 

		####tcp连接属性
		frontSocketSoRcvbuf 默认值： 1024 * 1024 
		frontSocketSoSndbuf 默认值： 4 * 1024 * 1024 
		frontSocketNoDelay 默认值： 1 
		backSocketSoRcvbuf 默认值： 4 * 1024 * 1024 
		backSocketSoSndbuf 默认值： 1024 * 1024 
		backSocketNoDelay 默认值： 1 

	user:登录信息和权限
		name：用户名
		password：登录密码
		schema：用户拥有的schema
		readonly：否只是读
		benchmark：服务降级，达到该数量的连接数后，开始拒绝请求，0表示不限制
		privileges：check=false表示不开启权限检查

	#防火墙配置


schema.xml
Schema,table,dataNode,dataHost,dataSource配置信息
	schema
		dataNode:非分片表的默认分片，需要分片的表在table标签上配置
		checkSQLschema：true，将带有schema的sql语句中去除schema
		sqlMaxLimit：设置该值后，若sql没有limit限制，mycat会自动加上sql的limit限制
	table
		name:逻辑表名
		dataNode:定义逻辑表的所在分片，多个分片逗号隔开,连续分片dn$1-32
		rule:分片规则名字，与rule.xml中的tableRule标签的name对应
		ruleRequired:是否绑定分片规则,true时若没有rule报错
		primaryKey:真实表的主键，若分片规则根据非主键分片，则主键查询时会请求所有分片，绑定该属性会缓存主键与分片的对应关系
		type:分为全局表和普通表，配置global则为全局表，否则为普通表
		autoIncrement:true,且真实表配置了主键自增，插入时取下表的全局序列号给分片，正常插入后，通过last_inserted_id()会返回分片键值，配合全局序列号使用
		needAddLimit:默认true,自动加上limit限制功能
		childTable:用于定义ER分片的子表
		subTables：库内分表属性
	dataNode
		name:数据节点名，建立表与分片的对应关系
		dateHost：该分片属于哪个数据库实例
		database：数据库实例上的具体物理库名

	dataHost
		name：dataHost唯一标示
		maxCon：读写实例连接池的最大连接，内嵌标签会根据该属性进行实例化连接池
		minCon:数据库实例连接池的最小连接，初始化的池大小
		dbType:数据库类型 例mysql
		dbDriver:native和jdbc，native二进制的mysql协议，适用mysql，jdbc适用其他数据库
		tempReadHostAvailable:1表示writeHost挂掉后对应的readHost依然可用，0为挂掉不可用，默认0
		balance:读策略 0不开启读写分离master请求读写 1全部readHost和空闲的masterHost参与读 2所有读操作随机发到所有writeHost和readHost 3所有读操作发到readHost
		wirteType:写策略 0所有写操作发到第一个writeHost,第一个挂了切到还生存的第二个writeHost,重新启动后以切换后的为准  1所有写操作负载到所有writeHost,1.5以后废弃不推荐
		switchType:
			1. switchType='-1' 表示不自动切换
			2. switchType='1' 默认值，表示自动切换
			3. switchType='2' 基于MySQL主从同步的状态决定是否切换,心跳语句为 show slave status
			4. switchType='3'基于MySQLgalary cluster的切换机制（适合集群）（1.4.1），心跳语句为 show status like 'wsrep%'。
			Mycat心跳机制通过检测 show slave status 中的 "Seconds_Behind_Master", "Slave_IO_Running","Slave_SQL_Running" 三个字段来确定当前主从同步的状态及Seconds_Behind_Master主从复制时延，当Seconds_Behind_Master>slaveThreshold时，读写分离筛选器会过滤掉此Slave机器，防止读到很久以前的旧数据，当主节点宕机后，切换逻辑会检查Slave上的Seconds_Behind_Master是否为0，为0时则表示主仅同步，可安全切换，否则不会切换。


rule.xml
	分片规则配置信息
	tableRule
		columns:指定要拆分的列名称
		algorithm:function标签中的name属性，具体的分片规则
	function
		name指定算法名称
	class 指定路由算法的具体类
	子标签property配置具体算法用到的属性	

	
####连续分片
<table name="t_user" dataNode="dn$1-32" rule="mod-long"/>
<dataNode name="dn$1-32" dataHost="localhost" database="test$1-32">


####chlieTable
<table name="t_user" dataNode="dn$1-32" rule="mod-long">
    <childTable name="t_user_detail" primaryKey="id" joinKey="user_id" parentKey="user_id" />
</table>

####subTables
<table name="t_user" dataNode="dn1" subTables="t_user$1-32" rule="mod-long"/>

####全局表
<table name="t_area" primaryKey="id" type="global" dataNode="dn1,dn2" />

=================================================================================分片=======================================================================================================
分片字段：拆分字段不可修改
主键分片：速度快，分布均匀
业务字段分片：mycat提供了“主键到分片”的内存缓存机制，主键查询时先查缓存找到分片再发送sql请求

分片规则
考虑:均匀分布 连续分布 后期扩容
1.固定分片的hash算法  （int long）
PartitionByLong
	partitionCount：4
	partitionLength：256
2.范围约定  （int long类型）
AutoPartitionByLong
	mapFile：
		0-500M=0
		500M-1000M=1
		1000M-1500M=2
所有的节点配置都是从0开始，及0代表节点1
3.取模法 （int long类型）
PartitionByMod
	count 
id进行十进制求模预算，此种在批量插入时需要切换数据源，id不连续
4.通配取模  （int long类型）
PartitionByPattern
	patternValue：256 求模基数
	defaultNode：2 默认节点，如果配置了默认，则不会按照求模运算
	mapFile：
		1-32=0
		33-64=1
		65-96=2
		97-128=3
		129-160=4
		161-192=5
		193-224=6
		225-256=7
		0-0=7
5.ASCII码通配求模 （int long string类型）
此方法类似通配求模，通配求模只适合分片字段为int long类型，该方法还适用于string类型
PartitionByPrefixPattern
	patternValue：256
	prefixLength：5
	mapFile：
		1-32=0
		33-64=1
		65-96=2
		97-128=3
		129-160=4
		161-192=5
		193-224=6
		225-256=7
		0-0=7

6.一致性hash （可很好的解决扩容问题）
PartitionByMurmurHash
	seed：默认0
	count：要分片的数据库节点数量，必须指定
	virtualBucketTimes：一个实际的数据库节点被映射为这么多虚拟节点，默认是160倍，也就是虚拟节点数是物理节点数的160倍
	weightMapFile：节点的权重，没有指定权重的节点默认是1。以properties文件的格式填写，以从0开始到count-1的整数值也就是节点索引为key，以节点权重值为值。所有权重值必须是正整数，否则以1代替
	bucketMapPath：用于测试时观察各物理节点与虚拟节点的分布情况，如果指定了这个属性，会把虚拟节点的murmur hash值与物理节点的映射按行输出到这个文件，没有默认值，如果不指定，就不会输出任何东西

4.日期分区法
PartitionByDate
	dateFormat：日期格式
	sBeginDate：开始日期
	sPartionDay：分区天数
PartitionByMonth
	dateFormat：日期格式
	sBeginDate：开始日期

关联查询
1.全局表
2.ER分片
3.catlet智能编程 ShareJoin
4.应用层sql拆分

####catlet详解
Catlet 编写完成并编译通过以后，必须放在Mycat_home/catlets目录下，系统会动态加载相关class（需要按照Java Class的目录结构存放，比如com\hp\catlet\XXXCatlet.class，目前还不支持Jar文件）
并每隔1分组扫描一次文件是否更新，若更新则自动重新加载，因此无需重启服务
在Mysql命令行连接Mycat Server后，执行带Catlet注解的SQL，则启动具体的Catlet完成SQL的解析，
如下面的例子，表明select a.*, b.title from travelrecord a ,hotnews b where a.id=b.id 这个SQL交给demo.catlets.MyHellowJoin来处理。
/*!mycat:catlet=demo.catlets.MyHellowJoin */select a.*, b.title from travelrecord a ,hotnews b where a.id=b.id

==================================================================================全局序列号================================================================================================
http://www.songwie.com/articlelist/68	
全局序列号值：next value for MYCATSEQ_XXX
查看：select next value for MYCATSEQ_XXX	select last_insert_id()							
##本地文件方式
1.配置server.xml   
 <property name="sequnceHandlerType">0</property>
2.在自增主键表上配置autoIncrement=”true”
3. 配置sequence_conf.properties
GLOBAL.HISIDS=
GLOBAL.MINID=10001
GLOBAL.MAXID=20000
GLOBAL.CURID=10000

COMPANY.HISIDS=
COMPANY.MINID=1001
COMPANY.MAXID=2000
COMPANY.CURID=1000

##数据库表方式
1.配置server.xml  <property name="sequnceHandlerType">1</property>
2.配置<table name="mycat_sequence" primaryKey="name" dataNode="dn1"/>
==========
3.在自增主键表上配置autoIncrement=”true”（表名为testauto）
4.在sequence_db_conf.properties中配置TESTAUTO=dn1(sequence所在dataNode)
5.在sequnce表中，插入相应的sequnce记录，并确定其初始值，以及增长步长，INSERT INTO MYCAT_SEQUENCE VALUES ('TESTAUTO', 0, 100);需要在数据库上而非Mycat上执行
提示：步长选择多大，取决与你数据插入的TPS，假如是每秒1000个，则步长为1000×60=6万，也不是很大，即60秒会重新从数据库读取下一批次的序列号值。 


##时间戳方式
1. 配置server.xml   
<property name="sequnceHandlerType">2</property>
2.在自增主键表上配置autoIncrement=”true”
3.在mycat下配置：sequence_time_conf.properties
WORKID=0-31 任意整数
DATAACENTERID=0-31 任意整数
mycat集群时的值不同

##分布式zk ID生成器

##zk 递增方式

注：单调插入可不显示插入，多条插入必须显示插入
注:自增主键配置（建议使用数据库全局序列号方式）
注：数据库全局序列号方式sql建表和方法


=============================================================================================数据迁移=====================================================================================
1.导出txt文件到服务器
select *from t_user into outfile '/usr/local/loaddata/t_user.txt' fields terminated by ',' optionally enclosed by '"' lines terminated by '\r\n';

2.将txt复制到客户端

3.在客户端load data将txt文件导入mysql
通过mycat导入
	####load 表 0 0
	####loaddata host port tableName
	例loaddata 127.0.0.1 8066 t_user 未成功将分表txt文件复制后直接导入mysql

直接导入mysql
	####load 表 起始节点 节点数
	####loaddata host port tableName
	例loaddata 10.50.10.151 3306 t_user

=============================================================================================性能调优=====================================================================================
#########JVM调优：
内存占用分两部分:
java堆内存+直接内存映射（DirectBuffer占用）建议堆内存适度大小，直接映射内存尽可能大，两种一起占据操作系统的1/2-2/3的内存。
服务器16G，Mycat堆内存4G，直接内存映射6G，JVM参数如下：
-server -Xms4G –Xmx4G XX:MaxPermSize=64M -XX:MaxDirectMemorySize=6G

conf\wrapper.conf：
wrapper.java.additional.5=-XX:MaxDirectMemorySize=6G
wrapper.java.initmemory=4096
wrapper.java.maxmemory=4096

#########操作系统调优：
最大文件句柄数量的修改，设置为5000-1万，在Mycat Server和Mysql数据库的机器上都设置。Linux操作系统对一个进程打开的文件句柄数量的限制
也包含打开的SOCKET数量,可影响MySQL的并发连接数目).这个值可用ulimit命令来修改，但ulimit命令修改的数值只对当前登录用户的目前使用环境有效,系统重启或者用户退出后就会失效。

#########Mysql调优：
最大连接数设置为2000
[mysqld]中有参数
max_connections = 2000
mysql> show global status like 'Max_used_connections';
理想的设置是：Max_used_connections / max_connections * 100% ≈ 85%
最大连接数占上限连接数的85%左右，如果发现比例在10%以下，MySQL服务器连接上线就设置得过高了。

#########Mycat调优：
conf/log4j.xml中，日志级别调整为至少info级别，默认是debug级别，用于排查错误，不能用于性能测试和正式生产中。
conf/server.xml中 有如下参数可以调整：
####processors：CPU核心数越多，可以越大，当发现系统CPU压力很小的情况下，可以适当调大此参数，如4核心的4CPU，可以设置为16，24核心的可以最大设置为128
####processorExecutor：这个参数为每个processor的线程池大小，建议可以是16-64，根据系统能力来测试和确定。
####processorBufferPool ：每个processor分配的Socket Direct Buffer，用于网络通信，每个processor上管理的所有连接共享，processorBufferPool参数的调整，需要观察show @@processor的结果来确定：
####BU_PERCENT为已使用的百分比、BU_WARNS为Socket Buffer Pool不够时，临时创新的新的BUFFER的次数，若百分比经常超过90%并且BU_WARNS>0，则表明BUFFER不够，需要增大processorBufferPool。基本上，连接数越多，并发越高，需要的POOL越大，建议BU_PERCENT最大在40-80%之间。

conf/schema.xml中有如下参数可以调整：
checkSQLschema属性建议设置为false，要求开发中，不能在sql中添加数据库的名称，如select * from TESTDB.company，这样可以优化SQL解析。
<dataHost name="localhost1" maxCon="500" minCon="10" balance="0"dbType="mysql" dbDriver="native" banlance="0">
最大连接池maxCon，可以改为1000至2000，同一个Mysql实例上的所有datanode节点的共享本dataHost 上的所有物理连接?
性能测试的时候，建议minCon=maxCon= mysql max_connections设为2000左右。

缓存优化调整：
Show @@cache命令展示了缓存的使用情况，经常观察其结果，需要时候进行调整：
一般来说：若CUR接近MAX，而PUT大于MAX很多，则表明MAX需要增大，
HIT/ACCESS为缓存命中率，这个值越高越好。重新调整缓存的最大值以后，观测指标都会跟随变化，调整是否有效，主要观察缓存命中率是否在提升，PUT则下降。
目前缓存服务的配置文件为：cacheservice.properties，主要使用的缓存为enhache，enhache.xml里面设定了enhance缓存的全局属性，下面定义了几个缓存：
factory.encache=org.opencloudb.cache.impl.EnchachePooFactory
pool.SQLRouteCache=
pool.ER_SQL2PARENTID=
layedpool.TableID2DataNodeCache= 50000,18000
SQLRouteCache为SQL 解析和路由选择的缓存，这个大小基本相对固定，就是所有SELECT语句的数量。
ER_SQL2PARENTID为ER分片时候，根据关联SQL查询父表的节点时候用到，没有用到ER分片的，这个缓存用不到
TableID2DataNodeCache，当某个表的分片字段不是主键时，缓存主键到分片ID的关系，这个是二层的缓存，每个表定义一个子缓存，如”TEST_ORDERS”，这里命名为
schema_tableName（tablename要大写），当有很多的根据主键查询SQL时，这个缓存往往需要设置比较大，才能更好的提升性能。

Mycat大数据量查询调优
1.返回结果比较多 建议调整 frontWriteQueueSize 在系统许可的情况下加大，默认值*3
==================================================================================监控========================================================================================================
mycat
jvm
mycat-web
haproxy 监控
==================================================================================其他========================================================================================================
#########################################catlet

#########################################mycat注解
MyCat 对自身不支持的 Sql 语句提供了一种解决方案——在要执行的 SQL 语句前添加额外的一段由注解
SQL 组织的代码，这样 Sql 就能正确执行，这段代码称之为“注解”。注解的使用相当于对 mycat 不支持的 sql
语句做了一层透明代理转发，直接交给目标的数据节点进行 sql 语句执行，其中注解 SQL 用于确定最终执行 SQL
的数据节点。

注解的使用方式是： 
/*!mycat: sql=注解 Sql 语句*/真正执行 Sql 

原理： 
注解是告诉 MyCat 按照注解内的 SQL（称之为注解 SQL）去进行解析处理，解析出分片信息后，将注解后真正要执行的 SQL 语句（称之为原
始 SQL）发送到该分片对应的物理库上去执行。 若无法解析出分片信息会将sql发送到所有分片

解决问题： 
1. MySql 不支持的语法结构，如 insert …select… 
2. 同一个实例内的跨库关联查询，如用户库和平台库内的表关联 
3. 存储过程调用。 
4. 表，存储过程创建。 

原则
1.  注解 SQL 使用 select 语句，不允许使用 delete/update/insert 等语句；虽然 delete/update/insert 等
语句也能用在注解中，但这些语句在 Sql 处理中有额外的逻辑判断，从性能考虑，请使用 select 语句 
2.  注解 SQL 禁用表关联语句 
3.  注解 SQL 尽量用最简单的 SQL 语句，如 select id from tab_a where id=’10000’ 
4.  无论是原始 SQL 还是注解 SQL，禁止 DDL 语句 
5.  能不用注解的尽量不用

补充说明: 
使用注解并不额外增加 MyCat 的执行时间；从解析复杂度以及性能考虑，注解 SQL 应尽量简单。至于一个
SQL 使用注解和不使用注解的性能对比，不存在参考意义，因为前提是 MyCat 不支持的 SQL 才使用注解。 
5.2   注解使用示例 
注解支持的'!'不被 mysql 单库兼容， 
注解支持的'#'不被 mybatis 兼容 
新增加 mycat 字符前缀标志 Hintsql:"/** mycat: */" 
从 1.6 开始支持三种注解方式： 
1. Mycat 端执行存储创建表或存储过程为： 
存储过程： 
/*!mycat: sql=select 1 from test */  CREATE  PROCEDURE `test_proc`() BEGIN END  ; 
表： 
/*!mycat: sql=select 1 from test */create table test2(id int); 
注意注解中语句是节点的表请替换成自己表如 select 1 from 表 ，注解内语句查出来的数据在哪个分片，数
据在那个节点往哪个节点建. 
2. 特殊语句自定义分片： 
/*!mycat: sql=select 1 from test */insert into t_user(id,name) select id,name from t_user2; 
3. 读写分离   
配置了 Mycat 读写分离后，默认查询都会从读节点获取数据，但是有些场景需要获取实时数据，如果从读节
点获取数据可能因延时而无法实现实时，Mycat 支持通过注解/*balance*/来强制从写节点查询数据： 
a. 事务内的 SQL，默认走写节点，以注解/*balance*/开头，则会根据 schema.xml 的 dataHost 标签属性的
balance=“1”或“2”去获取节点 
b. 非事务内的 SQL，开启读写分离默认根据 balance=“1”或“2”去获取，以注解/*balance*/开头则会走写节
点解决部分已经开启读写分离，但是需要强一致性数据实时获取的场景走写节点 
/*balance*/ select a.* from customer a where a.company_id=1; 
4. 多表 ShareJoin 
/*!mycat:catlet=demo.catlets.ShareJoin */ select a.*,b.id, b.name as tit from customer a,company b on 
a.company_id=b.id; 
5.读写分离数据源选择 
/*!mycat:db_type=master*/ select * from travelrecord 
/*!mycat:db_type=slave*/ select * from travelrecord 	 
/*#mycat:db_type=master*/ select * from travelrecord 
/*#mycat:db_type=slave*/ select * from travelrecord 

#########################################存储过程
完美支持以下三种情况： 
1.无返回值 
/*#mycat: sql=SELECT * FROM test  */call p_test(1,@pout) 
2.返回普通 out 参数 
/*#mycat: sql=SELECT * FROM test   */ set @pin=111;call p_test(@pin,@pout);select @pout 
3.返回结果中有结果集时，则必须加注解，且注解中必须在 list_fields 中包括所有结果集参数名称，以逗号隔开 
结果集参数必须在最后 
/*#mycat: sql=SELECT * FROM test  where id=1 ,list_fields='@p_CURSOR,@p_CURSOR1' */ 

#########################################事务处理
http://blog.csdn.net/d6619309/article/details/52330334
http://sanwen8.cn/p/19cZhak.html
http://3y.uu456.com/bp_2t6jv8avh635m4z30uul_1.html 事务

#########################################缓存
目前的系统大部分都会使用缓存，本地缓存 oscache,ehcache,分布式缓存 memcached,redis 等
Mycat 缓存的数据分别是：
SQLRouteCache(SQL 语句路由缓存)
TableID2DataNodeCache(表主键节点缓存)
ER_SQL2PARENTID(ER 关系缓存)。 
Mycat 缓存支持 ehcache,mapdb,leveldb,通过配置文件 cacheservice.properties，决定使用种缓存。

#########################################数据扩容


#########################################联合冗余字段的分片使用 
在拆分过程中碰到一个场景，无法满足拆分原则，通过引入联合冗余字段，达到了拆分目的，场景如下： 
某几个表业务上都与经营区域相关，但是所有经营区域只有 10 多个，按照数据量预估这个表会有 10 亿的
量，按照经营区域拆分，单表能达到 1 亿，如果考虑高峰区域和冷门区域问题，这个峰值会更大，可能 2 亿都有
可能。但是又没有其他好的拆分维度可以用，后来想到这个表中还有一个日期字段，查询时都可以加上时间区域
的限制，但是如果按照自然月拆分会如何呢？单表也会超过 800 万，最后确定如果联合这两个字段，多大的数据
量都能拆开了，弄出了一个联合字段 zone_yyyymm，表示区域+自然月，1 年 12 个月，10 多个区域，能够拆分
成 100 多个分片，这下来再大的数据量也能拆分开了。 

#########################################编码
##日志中与字符集有关的主要有三部分：
1.初始化MyCAT连接池
2.心跳检测
3.在执行SQL语句时的连接同步。		
1,2与服务端字符集有关，3与客户端字符集有关
##三个地方的编码指定
#mysql字符集编码
http://blog.knowsky.com/254652.htm
#客户端字符集
命令行：--default-character-set=utf8
jdbc:characterEnconding=utf8
#mycat
server.xml指定
##编码查看
mysql:show variables like 'character_set_%';
9066:show @@connection 
9066:show @@backend

#########################################压缩协议
在查询返回大量结果集和load data大量数据时性能提示比较明显，节省网络流量，消耗少量cpu资源
若要启动协议压缩，必须在客户端，mycat，mysql都启动才行
##启动
客户端启动：命令行加入-C参数启动，jdbc在url上加上参数useCompression=true启动
mycat在server.xml配置1启动
mysql服务端一般默认开启支持压缩协议
##条件
压缩协议属于通信协议的一部分
客户端和服务器都支持zlib算法

#########################################命令行
登录9066端口，show @@help查看所有命令
##reload @@config 更新配置文件
##show @@heartbeat 心跳报告，1正常 -1连接出错 -2连接超时 0初始化状态，节点故障，连续5个周期的心跳检查，失败后变为-1.
##show @@version mycat版本
##show @@connection 应用与mycat的连接状态
##kill @@connection id,id,id 杀掉前端连接
##show @@backend 查看mycat与数据库的连接状态
##show @@cache 查看mycat缓存，包括sql路由缓存，主键与分片对应缓存，ER分片主表与子表关系缓存
##show @@datasource 查看数据源状态，若配置了多从或多主可以切换
##swith @@datasource name:index 切换数据源，mycat不可用
##show @@syslog limit 查看系统日志，用于远程查看mycat运行日志
##show @@sql 显示通过8066端口向mycat请求的sql
##show @@sql.slow 显示mycat执行超过慢时间阈值的sql，设置时间阈值 reload @@sqlslow=0
##show @@sql.sum  显示mycat执行的sql统计信息
##reload @@user_stat 清空刚刚show @@sql，show @@sql.slow等的缓存
##Show @@threadpool 
当前线程池的执行情况，是否有积压(active_count)以及task_queue_size，后者为积压的待处理的SQL，若积压数目一直保值，则说明后端物理连接可能不够或者SQL执行比较缓慢。
##Show @@processor
显示当前processors的处理情况，包括每个processor的IO吞吐量(NET_IN/NET_OUT)、IO队列的积压情况(R_QUEY/W_QUEUE)，Socket Buffer Pool的使用情况

通过 show @@help; 可以查看所有的命令，如下： 
mysql> show @@help; 
+--------------------------------------+-----------------------------------+ 
| STATEMENT                            | DESCRIPTION                       | 
+--------------------------------------+-----------------------------------+ 
| clear @@slow where datanode = ?      | Clear slow sql by datanode        | 
| clear @@slow where schema = ?        | Clear slow sql by schema          | 
| kill @@connection id1,id2,...        | Kill the specified connections    | 
| offline                              | Change MyCat status to OFF        | 
| online                               | Change MyCat status to ON         | 
| reload @@config                      | Reload all config from file       | 
| reload @@route                       | Reload route config from file     | 
| reload @@user                        | Reload user config from file      | 
| rollback @@config                    | Rollback all config from memory   | 
| rollback @@route                     | Rollback route config from memory | 
| rollback @@user                      | Rollback user config from memory  | 
| show @@backend                       | Report backend connection status  | 
| show @@cache                         | Report system cache usage         | 
| show @@command                       | Report commands status            | 
| show @@connection                    | Report connection status          | 
| show @@connection.sql                | Report connection sql             | 
| show @@database                      | Report databases                  | 
| show @@datanode                      | Report dataNodes                  | 
| show @@datanode where schema = ?     | Report dataNodes                  | 
| show @@datasource                    | Report dataSources                | 
| show @@datasource where dataNode = ? | Report dataSources                | 
| show @@heartbeat                     | Report heartbeat status           | 
| show @@parser                        | Report parser status              | 
| show @@processor                     | Report processor status           |  
 
| show @@router                        | Report router status              | 
| show @@server                        | Report server status              | 
| show @@session                       | Report front session details      | 
| show @@slow where datanode = ?       | Report datanode slow sql          | 
| show @@slow where schema = ?         | Report schema slow sql            | 
| show @@sql where id = ?              | Report specify SQL                | 
| show @@sql.detail where id = ?       | Report execute detail status      | 
| show @@sql.execute                   | Report execute status             | 
| show @@sql.slow                      | Report slow SQL                   | 
| show @@threadpool                    | Report threadPool status          | 
| show @@time.current                  | Report current timestamp          | 
| show @@time.startup                  | Report startup timestamp          | 
| show @@version                       | Report Mycat Server version       | 
| stop @@heartbeat name:time           | Pause dataNode heartbeat          | 
| switch @@datasource name:index       | Switch dataSource              

#########################################mycat-web
1.zookeeper环境支持
mycat-eye需要zookeeper作配置中心
下载后将conf下的zoo_simple.cfg改为zoo.cfg
启动：bin\zkServer.bat
2.下载mycat-web	
windos:http://dl.mycat.io/mycat-web-1.0/Mycat-web-1.0-SNAPSHOT-20170102153329-win.zip
linux:http://dl.mycat.io/mycat-web-1.0/Mycat-web-1.0-SNAPSHOT-20170102153329-linux.tar.gz
下载解压后修改mycat.properties zookeeper=127.0.0.1:2181
windos启动：start.bat
linux启动:./start.sh
3.访问http://localhost:8082/mycat/

#########################################批量插入：
如果没有用mycat 的全局序列号，是普通的批量插入 ：
insert（a,b,c） values(x,x,x),(x,x,x); 
如果用了全局序列号必须加注解：
/*!mycat:catlet=demo.catlets.BatchInsertSequence */insert（a,b,c） values(x,x,x),(x,x,x); 
是sharding key 必须包含在列枚举中，
特别是主键是自增的时候必须显示调用：
/*!mycat:catlet=demo.catlets.BatchInsertSequence */insert（id,a,b,c） values(,next value for  MYCATSEQ_ID,x,x,x),(next value for  MYCATSEQ_ID,x,x,x); 

#########################################不支持的sql
MyCat的一些自带函数 sum,min,max等可以正确使用，但多分片执行的avg有bug，执行的结果是错误的
谨慎使用子查询，外层查询没有分片查询条件， 则会在所有分片上执行
group by，order by务必使用标准语法 select count(1),type from tab_a group by type; 错误示例：select a.type as type from tab_a group by a.type 别名与group by字段不一致
批量插入 insert into tab_a(c1,c2) values(v1,v2),(v11,v21)
复制插入 insert into x select from y 若x,y分片规则不一致错误
Call procedure() MyCat未支持存储过程定义, 因而不允许调用存储过程，但可通过注解来调用各个分片上的存储过程
Select func(); 不支持这种方式直接调用自定义函数， 但支持 select id, func() from employee 只需employee所在的所有分片上存在这个函数。
跨分片事务，目前为弱XA事务	
分片的table不能执行lock table，会导致死锁，经常出现在sqldump导入导出中

#########################################mysql高可用
mysql高可用方案 http://jingyan.baidu.com/article/1612d500a8f396e20f1eee4e.html
1.mysql双主+keepalived http://blog.sina.com.cn/s/blog_4f9fc6e10102w6xy.html
2.MMM/MHA 
3.Heartbeat/SAN
4.Heartbeat/DRBD
5.mysql双主+mycat

#########################################mysql主从复制
1.mySQL replication方案
##原理
从服务器到主服务器拉取二进制日志文件，然后再将日志文件解析成相应的SQL在从服务器上重新执行一遍主服务器的操作，通过这种方式保证数据的一致性
##配置
http://369369.blog.51cto.com/319630/790921/
http://www.2cto.com/database/201503/386265.html
将第二个mysql/data/auto.cf中的uuid修改否则报错
可配置互为主从
2.Galera cluter方案
http://www.thinksaas.cn/topics/0/492/492331.html

#########################################mycat高可用
haproxy+keepalived+mycat
http://blog.csdn.net/wdw1206/article/details/44201331



=======================================mycat 源码
processors--Runtime.getRuntime().availableProcessors()-操作系统核心线程数
	executor线程池 businessExecutor---threadPoolSize----system.getProcessorExecutor---prossors*2
	bufferPool 缓冲池 bufferPool bufferPoolPageSize bufferPoolChunkSize bufferPoolPageNumber
	前后端连接集合
	netin netout

totalNetWorkBufferSize
myCatMemory


NIoReactorPool--processor数：开启4个nioReact线程 用于监听读写事件 （前后端的读写）

nioConnection --一个线程--用于监听后端连接事件--连接后丢到nioReact中

NIOAcceptor ---manager--一个线程 监听前端管理端口连接事件 连接后丢到nioReact中
NIOAcceptoe --server --一个线程 监控前端服务端口连接事件 连接后丢到nioReact中

scheduler---1
heartbeatScheduler ---1
timerExecutor----systemConfig---2