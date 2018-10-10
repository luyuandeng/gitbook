=======================================================
安装(ubuntu)
    https://docs.docker.com/engine/installation/linux/ubuntu/#install-using-the-repository
    apt-get install docker.io
    docker -v

更新
    http://blog.csdn.net/delphiwcdj/article/details/42836423

配置加速器
    国外docker hub(Docker Registry)太慢，需要国内代理
    配置docker hub代理(DaoCloud) 
    curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://df1e2ed4.m.daocloud.io
    该脚本可以将 --registry-mirror 加入到你的 Docker 配置文件中。
    当用户的Docker设定了--registry-mirror参数后，用户的Docker拉取镜像时，首先去Docker加速器中查找镜像，若命中则说明该镜像已经在Docker加速器中缓存，用户直接从Docker加速器中下载。若没有命中，则说该镜像还没有被缓存，那么Docker加速器首先会被驱使去DockerHub中下载该镜像，并进行缓存，最终让用户从Docker加速器中下载该镜像
    DaoCloud简介 http://www.oschina.net/news/57894/daocloud

安装Docker Compose
     pip install docker-compose

kubernate
    kubernate集群 
    http://www.csdn.net/article/2014-12-24/2823292-Docker-Kubernetes
    kubernate单机
    http://www.tuicool.com/articles/URF7nqI

docker架构原理
    为了便于理解，大家可以把Docker容器，理解为一个或多个运行进程，而这些运行进程将占有相应的内存，相应的CPU计算资源，相应的虚拟网络设备以及相应的文件系统资源。而Docker容器所占用的文件系统资源，则通过Docker镜像的镜像层文件来提供
    转化的依据是什么？
        转化的依据是每个镜像的json文件，Docker可以通过解析Docker镜像的json的文件，获知应该在这个镜像之上运行什么样的进程，应该为进程配置怎么样的环境变量，此时也就实现了静态向动态的转变
    由谁来执行这个转化操作？
        Docker守护进程
        Docker容器实质上就是一个或者多个进程，而容器的父进程就是Docker守护进程。这样的，转化工作的执行就不难理解了：Docker守护进程 手握Docker镜像的json文件，为容器配置相应的环境，并真正运行Docker镜像所指定的进程，完成Docker容器的真正创建

docker镜像
    docker镜像为容器提供文件系统内容
    镜像的层级：例底层为ubuntu镜像，在加一个mysql层，变成mysql镜像
    镜像内容：镜像层文件和镜像json文件
    镜像与容器：
        容器是一个动态的环境，每一层镜像中的文件属于静态内 容，然而 Dockerfile 中的 ENV、VOLUME、CMD 等内容最终都需要落实到容器的运行环境中，而这些内容均不可能直接坐落到每一层镜像所包含的文件系统内容中，那此时每一个Docker镜像还会包含 json文件记录与容器之间的关系

Dockerfile,镜像与容器
    Dockerfile 是软件的原材料，Docker 镜像是软件的交付品，而 Docker 容器则可以认为是软件的运行态。从应用软件的角度来看，Dockerfile、Docker 镜像与 Docker 容器分别代表软件的三个不同阶段，Dockerfile 面向开发，Docker 镜像成为交付标准，Docker 容器则涉及部署与运维，三者缺一不可，合力充当 Docker 体系的基石



=========================================================================================================
阿里云镜像
https://dev.aliyun.com/list.html
资料汇总
https://github.com/hangyan/docker-resources/blob/master/README_zh.md#%E6%97%A5%E5%BF%97
中文社区
http://www.docker.org.cn/
教程
http://blog.opskumu.com/docker.html
http://www.open-open.com/lib/view/open1423703640748.html#articleHeader7
http://blog.oneapm.com/tags-Docker.html

阿里云镜像代理
docker compose应用
  http://www.cnblogs.com/xirongliu/p/6055257.html
docker run 参数
  网络模式
  文件挂载，修改配置后重启，修改容器后重启 http://blog.csdn.net/shanyongxu/article/details/51460930
docker exec
docker 修改容器后build
dockerfile 镜像生成
容器监控
docker文件目录
  http://dockone.io/question/70
  docker位置：/var/lib/docker/
  镜像层文件位置： /var/lib/docker/aufs/diff/
  镜像json文件位置：对于每一个镜像层，Docker 都会保存一份相应的 json 文件，json 文件的存储路径为 /var/lib/docker/graph
基本概念
  json文件
  镜像层
  dockerfile
  镜像
  容器
  守护进程，容器进程，容器资源（网络，内存，文件系统）
docker应用场景
    https://www.zhihu.com/question/22969309
    http://dockone.io/article/126
docker web应用
    http://www.open-open.com/lib/view/open1451870095745.html
    http://www.open-open.com/lib/view/open1437363835818.html
容器集群管理
容器可视化
docker隔离
  http://cloud.51cto.com/art/201603/507783.htm
  http://www.tuicool.com/articles/Qrq2Ynz
  http://cloud.idcquan.com/yzx/111190.shtml
  http://www.cnblogs.com/276815076/p/4673607.html
基于docker的软件供应链
  http://blog.oneapm.com/apm-tech/696.html
docker 容器内存限制
  http://blog.csdn.net/u010472499/article/details/52994454

==============================================================================================================================docker run参数
-a, --attach=[]            Attach to stdin, stdout or stderr.
-c, --cpu-shares=0         CPU shares (relative weight)                       # 设置 cpu 使用权重
--cap-add=[]               Add Linux capabilities
--cap-drop=[]              Drop Linux capabilities
--cidfile=""               Write the container ID to the file                 # 把容器 id 写入到指定文件
--cpuset=""                CPUs in which to allow execution (0-3, 0,1)        # cpu 绑定
-d, --detach=false         Detached mode: Run container in the background, print new container id # 后台运行容器
--device=[]                Add a host device to the container (e.g. --device=/dev/sdc:/dev/xvdc)
--dns=[]                   Set custom dns servers                             # 设置 dns
--dns-search=[]            Set custom dns search domains                      # 设置 dns 域搜索
-e, --env=[]               Set environment variables                          # 定义环境变量
--entrypoint=""            Overwrite the default entrypoint of the image      # ？
--env-file=[]              Read in a line delimited file of ENV variables     # 从指定文件读取变量值
--expose=[]                Expose a port from the container without publishing it to your host    # 指定对外提供服务端口
-h, --hostname=""          Container host name                                # 设置容器主机名
-i, --interactive=false    Keep stdin open even if not attached               # 保持标准输出开启即使没有 attached
--link=[]                  Add link to another container (name:alias)         # 添加链接到另外一个容器
--lxc-conf=[]              (lxc exec-driver only) Add custom lxc options --lxc-conf="lxc.cgroup.cpuset.cpus = 0,1"
-m, --memory=""            Memory limit (format: <number><optional unit>, where unit = b, k, m or g) # 内存限制
--name=""                  Assign a name to the container                     # 设置容器名
--net="bridge"             Set the Network mode for the container             # 设置容器网络模式
                             'bridge': creates a new network stack for the container on the docker bridge
                             'none': no networking for this container
                             'container:<name|id>': reuses another container network stack
                             'host': use the host network stack inside the container.  Note: the host mode gives the container full access to local system services such as D-bus and is therefore considered insecure.
-P, --publish-all=false    Publish all exposed ports to the host interfaces   # 自动映射容器对外提供服务的端口
-p, --publish=[]           Publish a container's port to the host             # 指定端口映射
                             format: ip:hostPort:containerPort | ip::containerPort | hostPort:containerPort
                             (use 'docker port' to see the actual mapping)
--privileged=false         Give extended privileges to this container         # 提供更多的权限给容器
--restart=""               Restart policy to apply when a container exits (no, on-failure[:max-retry], always)
--rm=false                 Automatically remove the container when it exits (incompatible with -d) # 如果容器退出自动移除和 -d 选项冲突
--security-opt=[]          Security Options
--sig-proxy=true           Proxify received signals to the process (even in non-tty mode). SIGCHLD is not proxied.
-t, --tty=false            Allocate a pseudo-tty                              # 分配伪终端
-u, --user=""              Username or UID                                    # 指定运行容器的用户 uid 或者用户名
-v, --volume=[]            Bind mount a volume (e.g., from the host: -v /host:/container, from docker: -v /container)
                           # 挂载卷
--volumes-from=[]          Mount volumes from the specified container(s)      # 从指定容器挂载卷
-w, --workdir=""           Working directory inside the container             # 指定容器工作目录




=======================================================================================================================================docker命令
http://blog.csdn.net/wsscy2004/article/details/25878363
docker pull sameersbn/redis:latest 下载镜像
docker run --name= -tid sameersbn/redis:latest 运行容器
docker logs -f name 查看容器日志
docker ps 查看正在运行的容器
docker rm $(docker ps -a -q) 删除所有容器
docker rm name 删除指定name的容器
docker start 启动容器
docker stop 停止容器
docker images 查看所有镜像
docker rmi  $(docker images | grep none | awk '{print $3}' | sort -r) 
docker save busybox-1 > /home/save.tar 保存镜像
docker load < /home/save.tar 在另一台机器加载镜像
docker build -t <镜像名> <Dockerfile路径> 构建自己的镜像
docker commit 
docker push
docker search
docker exec
docker attach
docker diff
docker inspect
docker save 
docker load
docker cp 
docker tag


1.docker build 使用dockerfile构建镜像
2.docker commit 保存容器为镜像
3.容器可视化监控
4.容器镜像迁移
5.容器数据卷
6.docker 网络模式
