mybastis
http://www.mybatis.org/mybatis-3/zh/index.html
架构
	接口层:增删改查
	数据处理层：参数映射，类型解析， sql解析，sql执行，结果映射，类型转换
	基础支持层：连接管理，事务管理，缓存处理，配置加载


流程
	sqlSessionFactoryBuilder 根据配置创建工厂
	sqlSessionFactory sqlSession工厂，全局唯一
	sqlSession 完成一次数据库的访问和结果的映射，主要配置Configuration和Executor
	Mapper对象 
		应用程序访问getMapper时，Mybatis会根据传入的接口类型和对应的XML配置文件生成一个代理对象
		通过这个Mapper对象来访问Mybatis的SqlSession对象
	Executor 调用StatementHandler访问数据库，并将查询结果存入缓存中
	StatementHandler 执行增删改查，真正访问数据库
	ResultSetHandler 处理结果集


优点
	1.解耦，sql语句不用写在程序中
	2.映射，数据库字段与对象映射
	3.动态sql，在sql中使用逻辑判断，基于OGNL表达式



缓存
	一级缓存
		在 sql 语句映射文件中加入 <cache /> 语句 , 并且相应的 model 类要实现 java Serializable 接口，因为缓存说白了就是序列化与反序列化的过程，所以需要实现这个接口. 单纯的 <cache /> 表示如下意思:
		1.所有在映射文件里的 select 语句都将被缓存。
		2.所有在映射文件里 insert,update 和 delete 语句会清空缓存。
		3.缓存使用“最近很少使用”算法来回收
		4.缓存不会被设定的时间所清空。
		5.每个缓存可以存储 1024 个列表或对象的引用（不管查询出来的结果是什么） 。
		6.缓存将作为“读/写”缓存，意味着获取的对象不是共享的且对调用者是安全的。不会有其它的调用者或线程潜在修改。
	二级缓存 
		ehcache实现


整合spring
	1.mapperFactoryBean  多dao多个配置分别注入xml
	2.MapperScannerConfigurer 扫描包路径 多个dao一个配置注入xml（优先采用）
	3.sqlSession的实现类sqlSessionTemplate 配置dao注入sqlSessionTemplate，需要熟悉sqlSession方法
	4.dao继承抽象类SqlSessionDaoSupport 无需配置dao，直接通过sqlSessionDaoSupport获取sqlSession操作数据库

关联查询结果集映射
分页插件
动态sql
代码自动生成mybatis-generator
日志
二级缓存
注解
映射配置，全局配置