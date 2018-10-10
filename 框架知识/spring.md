##spring
	spring core
		BeanFactory
		创建，装配，销毁bean
		ioc
			获得依赖对象的方式反转了
			一个对象依赖的其它对象会通过被动的方式传递进来
			1.读取配置 2.创建组装 3.从容器获取
			接口，set，构造器
			scope: 单例、多例、request、session、applciation
			scope,lazy,depends-on,init-method,destroy-method
			@Component@Repository@Service@Constroller
			@Qualifier@autowired@resource
			反射
				运行时创建对象
				获取Class的三种方式
				根据class获取类的类名，属性，方法，构造函数，注解，父类，子类
				根据class创建对象-无参构造函数
				增强程序灵活性，避免写死
				ASM字节码
	spring context
		ApplicationContext 实现BeanFactory所有功能，并添加了事件处理，国际化，资源文件处理等企业服务
	spring aop
		静态代理
			编译时生成代理类，一个被代理类对应一个代理类
		动态代理
			运行时动态生成，一个代理类可以代理所有
			继承InvocationHandler接口，实现invoke方法
			调用Proxy.newProxyInstance，传入被代理类接口和InvocationHandler
			通过反射创建代理类
				1.获取代理类的所有方法
				2.覆盖代理类接口中的方法，调用InvocationHandler的invoke
		jdk与cglib
			接口
			抽象类
		spring
			切面
			增强
				前置、后置、环绕、返回、异常
			切点
			连接点
	spring mvc