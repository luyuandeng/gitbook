##mysql
	索引
		基础
			加速查询-避免扫描全表
			顺序查找较慢，合适的数据结构能快速定位数据
		索引文件
			索引较大，以文件的形式存储，检索文件需要磁盘io
			设计的数据结构要尽可能减少磁盘io
		数据结构
			存储器原理
				内存随机读写，数据的距离不影响读取速度
				磁盘
					寻道
					磁盘旋转
					局部性原理
						当一个数据被用到时，其附近的数据也通常会马上被使用
					磁盘预读
						顺序读取 很少的旋转时间
						以页为单位预读
					缺页异常
						主存和磁盘以页为单位交换数据
						数据不在主存触发缺页异常
			二叉搜索树
				节点度为2
				左节点小于父节点，右节点大于父节点
			b-tree
				节点度>=2
				节点由key和指针相互间隔，key的值从左往右递增，指针数量比key多1个
				所有叶节点都在同一层
				指针指向子节点所有key要大于指针左节点，小于指针又节点
				查找算法：
					节点key采用二分法查找，命中返回，否则向下递归
			b+tree
				与b-tree区别
					1.节点指针数=节点key
					2.内节点只有指针和key，叶节点只有数据和key
				带有顺序指针的b+tree
					节点之间顺序指针连接
					查询连续记录时，效率更高
			二叉树&b-tree
				h为树的高度
				检索一次最多访问h个节点
				根据磁盘预读原理将节点大小设为一个页，故磁盘io次数为h
				二叉树的度为2，h一般较大
				b-tree的度一般较大，h较小
			b-tree&b+tree
				h越小性能越好，而d越大h越小
				d=page/(指针+key+data)
				b+tree内节点没有data，使得节点的度变大，性能更好
			hash&b-tree
				hash搜索单条记录快
				b-tree搜索多条记录快
		索引实现
			MyISAM
				主键索引与辅助索引没啥区别，都是非聚集索引
			InnoDB
				主键索引
					聚集索引
					默认生成，主键为索引的key（innodb表一定要有主键的原因）
				辅助索引
					data保存的是主键值
					辅助索引查找数据，要检索本身和主键索引
			聚集与非聚集
				聚集索引-data保存完整的数据记录
				非聚集索引-data存放的地址
		索引分类
			普通索引：加速查找
			主键索引：加速查找+非空唯一约束
			唯一索引：加速查找+唯一约束
			组合索引：多列组合，key为组合最左列，data含有组合的全部列	
			全文索引：搜索大数据字段
		索引优化
			最左原则
				组合索引从最左边字段向右匹配直到遇到范围查询(>、<、between、like)就停止匹配
				若where字段没有组合的最左字段，无法使用索引搜索，只能扫码索引文件
				字段的查询顺序不影响，查询优化器会优化成组合索引顺序
			前缀索引
				用列的前缀代替整个列，使data变小
			覆盖索引
				直接从索引数据中返回结果，不访问表文件
				需要指定查询字段，多条件查询必须覆盖组合索引全部字段
			主键连续
				插入时顺序添加到节点的后续位置，否则会随机插入页的任意位置，打乱索引结构，导致频繁移动
			索引排序
			组合代替多个单列索引
		索引使用
			缺点
				占用存储空间，插入修改删除维护
			不适用
				1.表记录少
				2.唯一性差
				3.频繁更新的字段
			索引失效
				1.组合索引最左原则
				2函数，表达式
				3.左模糊 
				4.普通索引的!=
				5.or - 除非or的列都有索引
				6.字符串 - 使用引号
				7.is null

	事务
		ACID
			原子性：要么成功要么失败
			一致性：一种状态转化到另一种状态
			隔离性：事务之间不会相互影响
			持久性：一旦提交，数据永久保存
			实现
				锁实现隔离性
				日志实现原子性，一致性，持久性
		隔离&锁
			并发问题
				脏读 - 读到事务未提交的数据
				不可重复读 - 两次读取的数据不同（修改）
				幻读 - 两次读取的数据记录不同（添加，删除）
			隔离级别
				读未提交 - 啥问题都会出现
				读已提交 - 解决脏读
				可重复读 - 解决不可重复读（默认），特殊情况下会出现幻读
				串行 - 全部解决
			锁
				mvcc 
					多版本并发控制器,读非阻塞
					原理
						隐藏列-创建时间,删除时间
						select:create<当前事务id&delete>当前事务id
						insert:create为当前事务id
						delete:delete为当前事务id
						update
							新建一条记录create为当前事务id，原来记录delete为当前事务id
							定时清理无效记录-purge
				悲观锁
					粒度
						表锁 - 开销小，并发性能低
						行锁 - 开销大，并发性能高
					类型
						共享锁（读锁）
						排它锁（写锁）
						意向锁
				间隙锁
			实现
				快照读
					普通查询使用
					快照
						时间点
							RC-事务中每个select都会生成快照
							RR-事务的第一个select生成快照
						内容
							read_view对象
								活跃的事务id集合
								最小事务id
								即将分配的事务id
							根据算法导出逻辑内容为生成点之前所有提交了的数据
					解决脏读，重复读，幻读：mvcc
				当前读
					增删改以及特殊查询使用
					读到最新数据
					解决幻读：行锁+gap锁=next key锁
				读未提交
					不加锁
				RC
					mvcc+行锁
				RR
					mvcc+next-key
				串行
					基于悲观锁
	日志
		错误日志
			启动运行关闭过程中的错误信息
		查询日志
			普通select sql
		慢日志
			执行超过一定时间的sql
		二进制日志
			记录增删改，备份和还原
		中继日志
			从主服务器binlog复制的事件，保存为新的binlog
		事务日志
			保证事务原子性，持久性
			数据与日志
				data_buffer data_file
				log_buffer log_file
			原子性
				undo log - 修改前的数据备份 - 逻辑日志 - 对行进行记录
				回滚原理：
					可以认为当delete一条记录时，undo log中会记录一条对应的insert记录
					反之亦然，当update一条记录时，它记录一条对应相反的update记录
			持久性
				背景：
					为减少磁盘io，数据存在内存缓冲区，先操作缓冲区再同步磁盘
					当缓冲区操作成功返回成功后，系统崩溃重启，磁盘数据无法恢复到最新
				redo log - 修改后的数据备份 - 物理日志 - 数据页的记录
			事务过程
				1.写undo log 到log_buffer
				2.修改data buffer
				3.写redo log 到log_buffer
				4.redo log刷盘到log_file
				5.commit
			日志刷盘
				red buffer同步到log_file,又引入了新的磁盘io
				规则
					1.innodb_flush_log_at_trx_commit 控制
						1.每次提交前写os buffer且同步磁盘
						2.每隔1秒将log buffer写os buffer且同步磁盘
						3.提交写os buffer，每隔1秒同步磁盘
					2.log buffer内存过半
					3.触发checkpoint
				优化
					redo log存储在连续的空间，系统启动完全分配，日志顺序追加
					批量写
					并发事务共享redo log 空间
			数据刷盘
				checkpoint : 将脏数据和脏日志同步到磁盘,减少数据恢复时间,保证buffer可用
				1.sharp checkpoint
					一次性刷盘
				2.fuzzy checkpoint
					部分刷盘
					触发时间
						1.线程控制，秒级定时触发
						2.LRU列表，可用页少于最小空闲页时
						3.redo log不可用时
						4.脏页达到一定数量时
					LSN
						创建阶段：事务创建一条日志 LSN1
						日志刷盘：日志写入到磁盘上的日志文件 LSN2
						数据刷盘：日志对应的脏页数据写入到磁盘上的数据文件 LSN3
						写CKP：日志被当作Checkpoint写入日志文件 LSN4
			启动恢复
	存储引擎
		InnoDB
			支持事务，支持mvcc
			支持行级锁
			聚集索引
			支持外键
			不支持全文索引
		MyISAM
			非事务
			表级锁
			非聚集索引
			支持全文索引
		场景
			MyISAM
				(1)做很多count 的计算；
				(2)插入不频繁，查询非常频繁；
				(3)没有事务。
			InnoDB
				(1)可靠性要求比较高，或者要求事务；
				(2)表更新和查询都相当的频繁，并且行锁定的机会比较大的情况。
	主从复制
		用途
			读写分离
			冗余备份
		原理
			1.io线程
				监听master二进制文件变化
				保存为中继日志
			2.sql线程
				执行中继日志中的事件 
		问题
			1.数据丢失 - 半同步复制 - 写完binlog后需要从库ack应答
			2.复制延时 - 并行复制 - 库级别多线程监听binlog
		实现
			MySQLProxy中间件
			客户端-服务器中间，建立连接池，读写自动切换且负载均衡
	集群
		高可用
			vip+keepalived+heartbeat+haproxy
			MHA MMM
			MySQL Galera
			MySQL cluster
		分表分库
			mycat
	性能分析&故障排查&优化
		优化
			sql优化
				避免select *,多用limit top
				order by 使用索引,避免表达式
				where替换having,exists替代distinct
				exists替代in,not exists替代not in
				where条件从右到左，能过滤大部分数据的条件放在最右
				连接查询,小数据集驱动大数据集
				使用临时表
				建立索引,避免索引失效
			表优化
				1.索引
				2.数据类型
					更小 更简单 避免null
					1.整数
						UNSIGNED存储非负
						不必指定长度
						tinyint代替enum
					2.实数 
						BIGINT代替DECIMAL
					3.字符串
						指定长度
						char代替varchar
					4.时间
						date - 只存储日志
						datetime - 存储时分秒，范围大，8字节
						timestamp - 存储时分秒，范围小，4字节
						使用时间戳代替
				3.表结构
					三范式
					反范式
			服务器优化
		性能分析
			慢日志
				默认不开启
				分析工具mysqldumpslow
			explain
				select_type
					simple primary subquery derived
				type
					system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL
				possible_keys
				key
				key_len
				rows
				Extra
					Using index Using where Using temporary
	未完成
		命令 工具
		函数
		系统变量
		启动参数
		系统数据库
		备份 权限 授权 用户
		表设计
		性能优化 性能分析 监控 故障排查
		存储引擎
		缓存
		集群 主从 分片


mysql备份
	mysqldump -uroot -proot xyj_order | gzip > /home/backup/xyj_order_$(date +%Y%m%d).sql.gz
	if [ -f "/home/backup/xyj_order_$(date +%Y%m%d -d '-5 day').sql.gz" ];then
		rm -rf /home/backup/xyj_order_$(date +%Y%m%d -d '-5 day').sql.gz
	fi
	备注：mysqldump -uroot -proot -d xyj_order (参数-d只导出表结构不包含数据)


	mysqldump -uroot -pPwliceQ5ZlTdY4rI xyj_order > xyj_order.sql

	mysqldump -uroot -pPwliceQ5ZlTdY4rI xyj_basic > xyj_basic.sql

	mysqldump -uroot -pPwliceQ5ZlTdY4rI xyj_data > xyj_data.sql

	mysqldump -uroot -pPwliceQ5ZlTdY4rI xyj_wx > xyj_wx.sql

	mysqldump -uroot -pPwliceQ5ZlTdY4rI xyj_product > xyj_product.sql

	mysqldump -uroot -pPwliceQ5ZlTdY4rI xyj_user > xyj_user.sql


	
mysql权限
	1.查看用户
	select Host,User from mysql.user
	2.创建用户
	create user application identified by '1le8unf6523dwgwt';
	3.修改指定用户密码
	update mysql.user set password=password('新密码') where User="test" and Host="localhost";
	4.删除用户
	delete from user where User='test' and Host='localhost';
	5.赋予权限
 	grant select,insert,update,delete on *.* to application@'120.78.224.41' identified by '1le8unf4768dwgwt';
 	6.查看权限
 	show grants for develop@'%';


#mysql服务注册表
"D:\soft\mysql5.5\mysql3306\bin\mysqld" --defaults-file="D:\soft\mysql5.5\mysql3306-d	ata\my.ini" MySQL55_1
"D:\soft\mysql5.5\mysql3307\bin\mysqld" --defaults-file="D:\soft\mysql5.5\mysql3307-data\my.ini" MySQL55_2
#生成，删除mysql服务
mysqld install mysql --defaults-file="E:\MySQL\mysql_base\ini\my.ini"
mysqld remove mysql 
#启动停止服务
net mysql start
net mysql top
net mysql restart


=====================================================搭建两台mysql服务器====================================================
wget http://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.16-linux-glibc2.5-x86_64.tar.gz
tar zxvf

#添加用户和组
groupadd mysql
useradd -g -r mysql mysql
chown -R mysql:mysql /usr/local/mysql

#初始化数据库
bin/mysql_install_db --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data3306 
bin/mysql_install_db --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data3307

#data3306,data3307中新建log和pid文件 
touch mysqld.log
touch mysqld.pid
chown -R mysql:mysql /usr/local/mysql/data

#复制/tmp/mysql.sock
cd /tmp/mysql.sock
mv mysql.sock mysql3306.sock
ln mysql.sock mysql3307.sock

#配置/etc/my.cnf
[mysqld_multi]
mysqld = /usr/local/mysql/bin/mysqld_safe 
mysqladmin = /usr/local/mysql/bin/mysqladmin
[mysqld1]
user=mysql
port = 3306
socket = /tmp/mysql3306.sock
basedir=/usr/local/mysql
datadir=/usr/local/mysql/data3306
log-error=/usr/local/mysql/data3306/mysqld.log
pid-file=/usr/local/mysql/data3306/mysqld.pid
log-bin=mysql-bin
server-id=1
character-set-server=utf8

[mysqld2]
user=mysql
port = 3307
socket = /tmp/mysql3307.sock
basedir=/usr/local/mysql
datadir=/usr/local/mysql/data3307
log-error=/usr/local/mysql/data3307/mysqld.log
pid-file=/usr/local/mysql/data3307/mysqld.pid
log-bin=mysql-bin
server-id=2
character-set-server=utf8

[client]
default-character-set=utf8

#将mysql/bin加入环境变量


#启动
mysqld_multi --defaults-file=/etc/my.cnf start 1
mysqld_multi --defaults-file=/etc/my.cnf start 2 
mysqld_multi --defaults-file=/etc/my.cnf start 1-2  

#关闭
ps -ef | grep mysqld* | grep -v grep |awk '{print $2}'|xargs kill -s 9

#连接
mysql -uroot -p -S /tmp/mysql3306.sock

#开启主从
##master
mysql>GRANT REPLICATION SLAVE ON *.* to 'slave'@'127.0.0.1' identified by 'slave'
show master status
##slave 
change master to master_host='127.0.0.1',master_user='slave',master_password='slave',master_log_file='mysql-bin.000003',master_log_pos=981;
start slave 
show slave status

#首次登录
1.在mysqld1,mysqld2最后一行添加skip-grant-tables，重启服务
2.mysql -u root mysql -S /tmp/mysql3306.sock直接进入
3.use mysql; update user set authentication_string=PASSWORD("root") where user='root';
4.退出后去掉mysqld1,mysqld2最后一行添加skip-grant-tables，重启服务
5.mysql -uroot -p -S /tmp/mysql3306.sock登录
6.SET PASSWORD = PASSWORD('root');



#mysql三种启动方式
mysqld_safe
mysql.server
mysql_multi

=======================================内置函数
http://www.mamicode.com/info-detail-250393.html
数学函数
	abs（x）
	rand()
	sign(x)
	round(x)
	pow(x,y)
	mod(x,y)
字符串函数
	char_length(s)
	length(s)
	left(s,n)
	right(s,n)
日期时间函数
	curDate()
	curTime()
	now()
	unix_timestamp()
	month(d)
	monthname(d)
	dayname(d)
	dayofweek(d)
	weekday()
	week()
	year()
	extract()
	time_format
流程函数
	if(expr,x1,x2)
系统信息函数	
	VERSION()
	CONNECTION_ID() 返回服务器的连接数，也就是到现在为止mysql服务的连接次数
	DATABASE(),SCHEMA() 返回当前数据库名
	USER() 返回当前用户的名称 
	CHARSET(str) 返回字符串str的字符集
	COLLATION(str) 返回字符串str的字符排列方式
	LAST_INSERT_ID() 返回最后生成的auto_increment值
 

加密函数
	PASSWORD(str)
	MD5(str)
	
其他函数

=======================================系统数据库
information_schema：维护用户新建的数据库信息
	TABLES 表信息
	COLUMNS 表的列信息
	KEY_COLUMN_USAGE 表键值信息
	TABLE_CONSTRAINTS  check约束信息
	STATISTICS 表索引信息
mysql：账户信息，权限信息，存储过程，event,时区等信息

=======================================表设计
基础表 中间表 临时表
	中间表是存放统计数据的表，它是为数据仓库、输出报表或查询结果而设计的，有时它没有主键与外键（数据仓库除外）。临时表是程序员个人设计的，存放临时记录，为个人所用。基表和中间表由DBA维护，临时表由程序员自己用程序自动维护。
三范式
	第一范式：无重复的列，原子性，不可再分解
	第二范式：非主属性依赖主键
	第三范式：属性不依赖其他非主属性，冗余性，即任何字段不能由其他字段派生出来
不符合第三范式的例子:
	学号, 姓名, 年龄, 所在学院, 学院联系电话，关键字为单一关键字"学号"; 
	存在依赖传递: (学号) → (所在学院) → (学院地点, 学院电话)
数据库设计的实用原则是：在数据冗余和处理速度之间找到合适的平衡点
数据类型选择：
	1.存储空间越小越好
	2.数据类型越简单越好
	3.尽量不要null
	整型
	实数
	字符串
	时间日期

=======================================权限 组 用户 授权
http://www.cnblogs.com/fslnet/p/3143344.html
用户管理
	mysql>use mysql;
查看
	mysql> select host,user,password from user ;
创建
	mysql> create user  zx_root   IDENTIFIED by 'xxxxx';   //identified by 会将纯文本密码加密作为散列值存储
修改
	mysql>rename   user  feng  to   newuser；//mysql 5之后可以使用，之前需要使用update 更新user表
删除
	mysql>drop user newuser;   //mysql5之前删除用户时必须先使用revoke 删除用户权限，然后删除用户，mysql5之后drop 命令可以删除用户的同时删除用户的相关权限
更改密码
	mysql> set password for zx_root =password('xxxxxx');
 	mysql> update  mysql.user  set  password=password('xxxx')  where user='otheruser'
查看用户权限
	mysql> show grants for zx_root;
赋予权限
	mysql> grant select on dmc_db.*  to zx_root;
回收权限
	mysql> revoke  select on dmc_db.*  from  zx_root;  //如果权限不存在会报错



常用拓扑结构
	master-slaves：一个master对应一个或多个slave
	master-master：两台服务器既是master，也是slave
	master-slaves-slaves:级联复制
	Master-Master with Slaves:带从服务器的Master-Master结构s
	mysql不支持多主复制（MySQL Cluster可解决）
mysql架构
	MMM/MHA
	Heartbeat/SAN
	Heartbeat/DRBD
	mysql cluster/Galera cluster
	mysql router/mysql proxy

=======================================性能优化
http://www.cnblogs.com/luxiaoxun/p/4694144.html
引入缓存或搜索引擎
表设计
	范式优化：消除冗余
	反范式优化：适当增加冗余，减少join
	表拆分：水平与垂直拆分
	数据类型优化
系统配置
	http://blog.csdn.net/xuanjiewu/article/details/50801697
	网络连接
	查询缓存
	临时表
	会话内存
	慢查询日志
sql优化
	1）小结果集连接查询
	select *from (select *from a where id=1) a left join b on a.id=b.idx`x`
	2）应尽量避免在 where 子句中使用!=或<>
	3）应尽量避免在 where 子句中对字段进行 null 值判断
	4）exists 代替 in 
	5）Where子句替换HAVING子句
	6）保证Join语句中被驱动表上Join条件字段已经被索引
	1. 优化更需要优化的Query；
	2. 定位优化对象的性能瓶颈；
	3. 明确的优化目标；
	4. 从Explain入手；
	5. 多使用profile
	6. 永远用小结果集驱动大的结果集；
	7. 尽可能在索引中完成排序；
	8. 只取出自己需要的Columns；
	9. 仅仅使用最有效的过滤条件；
	10.尽可能避免复杂的Join和子查询；

性能分析
	1、慢查询 （分析出现出问题的sql）
		开启
			log-slow-queries=/var/lib/mysql/slowquery.log 
			long_query_time=2 (记录超过的时间，默认为10s)
			log-queries-not-using-indexes (记录下来没有使用索引的query)
		查看
			使用mysql自带命令mysqldumpslow查看
	2、Explain （显示了mysql如何使用索引来处理select语句以及连接表。可以帮助选择更好的索引和写出更优化的查询语句）
	3、Profile（查询到 SQL 会执行多少时间, 并看出 CPU/Memory 使用量, 执行过程中 Systemlock, Table lock 花多少时间等等.）
show
	show tables;
	show databases;
	show variables;
	show status;
	show processlist;
	show engines;
	show grants for user
profile
	select @@profiling 查看是否开启
	set profiling=1 开启
	show profiles 查看sql语句执行信息列表
	show profile for query n 查看sql执行详情