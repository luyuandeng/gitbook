本地
工作区-暂存区-版本库

git status
对比暂存区与工作区
对比暂存区和版本库

工作区命令
git add  将工作区修改推送到暂存区
git checkout --file/git checkout . 将暂存区覆盖工作区，撤销本地修改


暂存区命令
git commit 将暂存区修改推送到版本库

版本库
git push 将版本库推送到远程库


git reset
	mixed 默认 回退head到commit，commit同步本地仓库，暂存区-暂存区变化回退工作区
	soft 回退head到commit，commit同步本地仓库，本地仓库变化回退到暂存区，工作区不变
	hard 回退head到commit，commit同步本地仓库，暂存区，工作区不变

	HEAD：当前commit
	head^ 或 head~1 上一个commit版本
	head~n 上n个commit版本

冲突解决
1.本地
	1.1git commit
	1.2git merge master(master最新或origin master)
	1.3提示冲突，修改冲突文件
	1.4再次git commit
	1.5git push
	1.6merge request

	1.1git pull 
	1.2git merge master
2.远程
	1.1git push&merge request
	1.2远程解决冲突

	1.1git pull
	1.2git merge

gitlab权限
	1.设置用户权限未developer
	2.设置master分支未受保护分支，只允许master提交,合并

git-bash免密码操作
1.SSH 
2.HTTP

git tag

git network
=========

搭建gitlab环境
    使用docker镜像安装gitlab
    http://www.open-open.com/lib/view/open1438009785316.html
    gitlab高可用
    https://about.gitlab.com/high-availability/
    docker-gitlab源码
    https://github.com/sameersbn/docker-gitlab

gitlab架构
    http://img0.tuicool.com/F3AjA3V.png!web
    •前端：Nginx，用于页面及Git tool走http或https协议
    •后端：Gitlab服务，采用Ruby on Rails框架，通过unicorn实现后台服务及多进程
    •SSHD：开启sshd服务用于用户上传sshkey进行版本克隆及上传。注：用户上传的ssh key是保存到git账户中
    •数据库：目前仅支持MySQL和PostgreSQL
    •Redis：用于存储用户session和任务，任务包括新建仓库、发送邮件等等
    •Sidekiq：Rails框架自带的，订阅redis中的任务并执行

docker-gitlab架构
    redis镜像：link到gitlab
    postgreSql镜像：link到gitlab
    gitlab镜像：nginx,sshd,ruby onRails,Sidekiq

docker gitlab配置
    https://github.com/sameersbn/docker-gitlab/blob/master/README.md
    1.启动容器带上环境变量参数即可实现GitLab参数的配置
    2.docker exec -it 进入容器修改配置文件

gitlab特性
    issues
    code review
    CI
    CD 

    

客户端访问gitlab
git可视化工具
git 命令 http://www.ruanyifeng.com/blog/2015/12/git-workflow.html
=============================================


每次maven:3-jdk-8去执行build和test都会重新拉取镜像，下载依赖的jar包，比较耗时耗资源。这是因为docker image每次构建都是在独立的container里， maven的 .m2文件并不会被多次构建公用，这里我们可以通过修改gitlab-runner的配置，将maven .m2目录加到volumes中，并增加镜像拉取规则（默认是从远程拉取镜像，这里修改为优先获取本地镜像，不存在时才去远程拉取镜像）。


http://www.jianshu.com/p/2b43151fb92e
我把Pipelines理解为流水线，流水线包含有多个阶段（stages），每个阶段包含有一个或多个工序（jobs）

===============
gitlab+gitlab-ci+gitlab-runner
1.gitlab
代码仓库服务器
管理项目，用户，组，处理git事件

2.gitlab-ci
持续集成组件，已集成到gitlab服务器中，接收gitlab监听的push或meger master事件后传递过来的runner
gitlab-ci根据传递过来的runner调用gitlab-runner服务器中的runner环境执行脚本

3.gitlab-runner 
运行自动化脚本组件，注册runner到gitlab-ci中，以供gitlab-ci调用，此组件相当于jeknies

##安装gitlab
1.安装postgresql
docker run -m 256m --name gitlab-postgresql -d \
--env 'DB_NAME=gitlabhq_production' \
--env 'DB_USER=gitlab' --env 'DB_PASS=password' \
--env 'DB_EXTENSION=pg_trgm' \
--volume /srv/gitlab/postgresql:/var/lib/postgresql \
registry.cn-hangzhou.aliyuncs.com/acs-sample/postgresql-sameersbn
注意：DB_EXTENSION=pg_trgm必填

2.安装redis
docker run -m 256m --name gitlab-redis -d \
--volume /srv/gitlab/redis:/var/lib/redis \
registry.cn-hangzhou.aliyuncs.com/acs-sample/redis-sameersbn

3.安装gitlab
docker run --name gitlab -d \
--link gitlab-postgresql:postgresql --link gitlab-redis:redisio \
--publish 10022:22 --publish 10080:80 \
--env 'GITLAB_PORT=10080' --env 'GITLAB_SSH_PORT=10022' \
--env 'GITLAB_HOST=gitlab.bitemen.cn' \
--env 'GITLAB_SSH_HOST=gitlab.bitemen.cn' \
--env 'GITLAB_TIMEZONE=Beijing' \
--env 'GITLAB_BACKUP_SCHEDULE=weekly' \
--env 'GITLAB_EMAIL=m15926599530_1@163.com' \
--env 'SMTP_ENABLED=true' \
--env 'SMTP_DOMAIN=163.com' \
--env 'SMTP_HOST=smtp.163.com' \
--env 'SMTP_USER=m15926599530_1@163.com' \
--env 'SMTP_PASS=liuheng159357' \
--env 'SMTP_AUTHENTICATION=login' \
--env 'GITLAB_SECRETS_DB_KEY_BASE=long-and-random-alphanumeric-string' \
--env 'GITLAB_SECRETS_SECRET_KEY_BASE=long-and-random-alphanumeric-string' \
--env 'GITLAB_SECRETS_OTP_KEY_BASE=long-and-random-alphanumeric-string' \
--volume /srv/gitlab/gitlab:/home/git/data \
registry.cn-hangzhou.aliyuncs.com/acs-sample/gitlab-sameersbn
注意：不能直接进入容器修改gitlab配置然后重启容器，必须以gitlab run指定环境变量

##安装gitlab-runner
1.安装gitlab-runner
docker run -d --name gitlab-runner \
  -v /srv/gitlab-runner/config:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
registry.cn-hangzhou.aliyuncs.com/nichozuo/gitlab-runner:latest

2.注册runner到gitlab-ci
gitlab-ci-multi-runner register
----gitlab服务器url
----gitlab服务器runner token（在gitlab管理页runner查看）
----执行器executor
----runner标示
执行器可选
	shell 需要自己安装build环境，比如maven，在gitlab-runner中执行脚本
	docker 直接拉取镜像，在容器中执行脚本（gitlab-runner容器中的容器）
删除注册runner
	gitlab-runner unregister --url  --token

3.安装maven，ant环境,jdk8环境，scp,zip
apt-get install maven
apt-get install ant  || 添加jsch.jar到/usr/share/ant/lib
java环境变量
export JAVA_HOME=/home/jdk1.8.0_144  
export JRE_HOME=${JAVA_HOME}/jre  
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib  
export PATH=${JAVA_HOME}/bin:$PATH
设置默认jdk环境
update-alternatives --install /usr/bin/java java /home/jdk1.8.0_144/bin/java 300  
update-alternatives --install /usr/bin/javac javac /home/jdk1.8.0_144/bin/javac 300  
update-alternatives --config java
update-alternatives --config javac


##gitlab使用
管理员
1.登录&重置密码
http://gitlab.bitemen.cn:10080/
root
5iveL!fe
2.新建group
3.添加users&拉入group
4.添加project

用户
1.登录&重置密码
2.添加ssh key
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
将公钥复制到gitlab
3.初始化项目，推送master分支
4.测试持续集成 编写.gitlab-ci.yml 和 ant.xml
=================================================================================================================================

##gitlab-runner
runner分类
	Shared Runner 多个工程共享
	Specific Runner 单个工程可用
注册完成后
	写配置到/etc/gitlab-runner/config.toml
	GitLab-CI会为这个Runner生成一个唯一的token，以后Runner就通过这个token与GitLab-CI进行通信
	自动运行runner

runner信息位置
	/etc/gitlab-runner/config.toml root用户
	~/.gitlab-runner/config.toml 非root用户
	配置文件里面有Runner运行时所需要的信息

运行runner
	单个运行
		gitlab-ci-multi-runner run-single
	批量运行
		gitlab-ci-multi-runner run
		指定
			服务名（默认gitlab-runner）
			工作目录
			配置文件
			用户（使用该用户权限运行runner）
		运行指定配置文件下的所有runner
		多个配置文件，分批运行
服务管理runner
	gitlab-ci-multi-runner install 
	指定
		服务名
		工作目录
		配置文件
		用户（使用该用户权限运行runner）
	该服务包含指定配置文件下的runner
	gitlab-ci-multi-runner start|restart|stop|status service-name 
	ps -aux | grep gitlab-runner

=================================================================================================================================

gitlab环境变量配置
https://github.com/sameersbn/docker-gitlab/blob/master/README.md


gitlab登录
http://gitlab.bitemen.cn:10080/
root
5iveL!fe


gitlab-runner
http://www.jianshu.com/p/2b43151fb92e
https://segmentfault.com/a/1190000006120164#articleHeader2


.gitlab-ci.yml
http://blog.csdn.net/wmq880204/article/details/70141771
https://docs.gitlab.com/ee/ci/yaml/README.html


gitlab-ci-multi-runner run，install参数
USAGE:
   command run [command options] [arguments...]

OPTIONS:
   -c, --config "/etc/gitlab-runner/config.toml" Config file [$CONFIG_FILE]
   -n, --service "gitlab-runner"   Use different names for different services
   -d, --working-directory     Specify custom working directory
   -u, --user       Use specific user to execute shell scripts
   --syslog      Log to syslog

-c, --config选项
这个选项是用来指定配置文件路径的。如果你想同时运行多个Runner，你必须得知道你要运行哪些Runner以及这些Runner运行时所需要的信息。而前面我们说过，配置文件里面就存放着Runner运行时所需要的信息。而且一个配置文件是可以存放多个Runner的信息的。如果不指定这个选项，就会使用默认的配置文件。
-n, --service选项
这个选项是用来指定服务的别名的。为什么要有这个选项呢？指定别名有什么意义呢？我们从上一个选项可以看出来，一次只能运行一批Runner，因为一次只能指定一个配置文件。那如果我有多个配置文件，我要运行多批Runner，那是不是给每一次批量运行服务取不同的别名来区分更好一点呢。
-d, --working-directory选项
这个选项是用来指定此次批量运行服务的工作目录的。如果自己没有指定builds_dir的话，此次运行起来的Runner会把builds_dir放到这个目录里面。
-u, --user选项
这个选项很重要，它指定了该以什么用户权限来运行Runner。为了安全，我认为不应该给运行Runner的用户过高的权限，更不应该以root用户来运行Runner。
--syslog选项
如果指定了这个选项，则把日志记录到系统日志。


gitlab未完成
	邮件
	Reply by email
	LDAP
	Container Registry
	gitlab issue
	https配置
	gitlab-ci-multi-runner 和 gitlab-runner区别