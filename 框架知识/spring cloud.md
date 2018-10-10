##spring cloud
	Spring Boot专注个体服务
	Spring Cloud关注全局的服务治理
	zuui 网关
	config 配置中心
		server
			存储配置：git|svn|本地
			多文件环境隔离
			通过接口提供配置
			@EnableConfigServer
		client
			spring.cloud.config.name=neo-config
			spring.cloud.config.profile=dev
			spring.cloud.config.uri=http://localhost:8001/
			spring.cloud.config.label=master
			服务化：
				spring.cloud.config.discovery.enabled=true
				spring.cloud.config.discovery.serviceId=spring-cloud-config-server
			refresh机制
				1.添加actuator依赖 关闭安全验证management.security.enabled=false
				2.@RefreshScope添加在需要refresh的变量所在类
				3.访问http://XXX/refresh (webhook自动触发)
	eureka 注册中心
		c-s架构
		心跳连接
		服务提供方、消费方
		互相注册实现高可用
		@EnableEurekaServer@EnableEurekaClient
		eureka.client.serviceUrl.defaultZone
	bus 消息总线
		AMQP
		广播状态的变化（例如配置变化）
		应用场景：配置中心客户端刷新
	feign 服务调用
		@EnableFeignClients
		@FeignClient
		ribbon 负载均衡
		hystrix 熔断器
			熔断：失败数达到一定数量，切换开关，一定时间后判断服务正常后，切换开关
			降级：实现fallback方法，服务异常时调用
			feign.hystrix.enabled=true
			监控面板
				Hystrix Dashboard
				Turbine：分布式中聚合多个节点展示
	sleuth 分布式服务跟踪