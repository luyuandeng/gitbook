=======================================================keepalived========================================================
wget http://www.openssl.org/source/openssl-1.0.0a.tar.gz
yum install openssl-devel -y 
yum install popt-devel -y
wget http://www.keepalived.org/software/keepalived-1.2.20.tar.gz

将启动关闭脚本添加为服务
cp /usr/local/keepalived/etc/rc.d/init.d/keepalived /etc/init.d/ 将启动关闭脚本添加为服务 service keepalived start
cp /usr/local/keepalived/etc/sysconfig/keepalived /etc/sysconfig/  启动脚本默认使用的配置文件

创建/etc/keepalived/keepalived.conf默认配置文件
mkdir /etc/keepalived
cp /usr/local/keepalived/etc/keepalived/keepalived.conf etc/keepalived/
mkdir /etc/keepalived/scripts

创建运行日志文件
mkdir -p /usr/local/keepalived/var/log

======keepalived.conf====master
vrrp_script chk_http_port {
    script"/etc/keepalived/scripts/check_haproxy.sh"
    interval 2
    weight -2
}
vrrp_instance VI_1 {
   state MASTER              
   interface eth1            #对外提供服务的网络接口
   virtual_router_id 51      #VRRP组名，两个节点的设置必须一样，以指明各个节点属于同一VRRP组
   priority 101              #数值愈大，优先级越高
   advert_int 1              #同步通知间隔
   authentication {          #包含验证类型和验证密码。类型主要有PASS、AH两种，通常使用的类型为PASS，据说AH使用时有问题
      auth_type PASS
      auth_pass 1111
   }
  
   track_script {
      chk_http_port            #调用脚本check_haproxy.sh检查haproxy是否存活
   }
  
   virtual_ipaddress {      #vip地址，这个ip必须与我们在lvs客户端设定的vip相一致
      10.50.10.155 dev eth1 scope globa
   }
   notify_master /etc/keepalived/scripts/haproxy_master.sh
   notify_backup /etc/keepalived/scripts/haproxy_backup.sh
   notify_fault /etc/keepalived/scripts/haproxy_fault.sh
   notify_stop  /etc/keepalived/scripts/haproxy_stop.sh
}


=========keepalived.conf====slave
vrrp_script chk_http_port {
    script"/etc/keepalived/scripts/check_haproxy.sh"
    interval 2
    weight -2
}
vrrp_instance VI_1 {
   state BACKUP              
   interface eth0            #对外提供服务的网络接口
   virtual_router_id 51      #VRRP组名，两个节点的设置必须一样，以指明各个节点属于同一VRRP组
   priority 100              #数值愈大，优先级越高
   advert_int 1              #同步通知间隔
   authentication {          #包含验证类型和验证密码。类型主要有PASS、AH两种，通常使用的类型为PASS，据说AH使用时有问题
      auth_type PASS
      auth_pass 1111
   }
  
   track_script {
      chk_http_port            #调用脚本check_haproxy.sh检查haproxy是否存活
   }
  
   virtual_ipaddress {      #vip地址，这个ip必须与我们在lvs客户端设定的vip相一致
      10.50.10.155 dev eth0 scope globa
   }
   notify_master /etc/keepalived/scripts/haproxy_master.sh
   notify_backup /etc/keepalived/scripts/haproxy_backup.sh
   notify_fault /etc/keepalived/scripts/haproxy_fault.sh
   notify_stop  /etc/keepalived/scripts/haproxy_stop.sh
}

=====check_haproxy.sh===
#!/bin/bash
LOGFILE="/usr/local/keepalived/var/log/keepalived-haproxy-state.log"
a=$(ps aux | grep -v grep | grep -w haproxy | wc -l)
if [ $a -eq 0 ];then
echo "[ haproxy start]" >> $LOGFILE
date >> $LOGFILE
/usr/local/haproxy/sbin/haproxy -f /usr/local/haproxy/haproxy.cfg
sleep 5
fi
b=$(ps aux | grep -v grep | grep -w haproxy | wc -l)
if [ $b -eq 0 ];then
  echo "[ haproxy stoped]" >> $LOGFILE
  date >> $LOGFILE
  exit 1
else
  exit 0
fi
====haproxy_master.sh====
#!/bin/bash
LOGFILE="/usr/local/keepalived/var/log/keepalived-haproxy-state.log"
echo "[master][master][master][master][master][master]" >> $LOGFILE
date >> $LOGFILE


====haproxy_backup.sh====
#!/bin/bash
LOGFILE="/usr/local/keepalived/var/log/keepalived-haproxy-state.log"
echo "[backup][backup][backup][backup][backup][backup]" >> $LOGFILE
date >> $LOGFILE

====haproxy_fault.sh====
#!/bin/bash
LOGFILE=/usr/local/keepalived/var/log/keepalived-haproxy-state.log
echo "[fault][fault][fault][fault][fault][fault][fault]" >> $LOGFILE
date >> $LOGFILE

====haproxy_stop.sh====
LOGFILE=/usr/local/keepalived/var/log/keepalived-haproxy-state.log
echo "[stop][stop][stop][stop][stop][stop][stop][stop]" >> $LOGFILE
date >> $LOGFILE

=====赋予脚本权限====
chmod 777 /etc/keepalived/scripts/*


===================================================haproxy==================================================
wget --no-check-certificate https://fossies.org/linux/misc/haproxy-1.6.9.tar.gz
make TARGET=linux26 PREFIX=/usr/local/haproxy ARCH=x86_64
make install PREFIX=/usr/local/haproxy
useradd haproxy              
chown -R haproxy.haproxy *

cd /usr/local/haproxy
touch haproxy.cfg
vi haproxy.cfg
===haproxy.cfg===
global
log 127.0.0.1   local0  ##记日志的功能,对应/etc/rsyslog.d/haproxy.conf
    maxconn 4096
    chroot /usr/local/haproxy
    pidfile /var/run/haproxy.pid
    user haproxy
    group haproxy
    daemon
defaults
    log    global
    option    dontlognull
    retries    3
    option redispatch
    maxconn    2000
    contimeout    5000
    clitimeout    50000
    srvtimeout    50000
listen  admin_status 
bind 10.50.10.155:48800 
      stats uri /admin-status        
      stats auth  admin:admin
      mode    http
      option  httplog
listen  allmycat_service 
bind 10.50.10.155:8096 
      mode tcp
      option tcplog
      option httpchk OPTIONS * HTTP/1.1\r\nHost:\ www
      balance    roundrobin
        server    mycat_150 10.50.10.150:8066 check port 48700 inter 5s rise 2 fall 3
        server    mycat_151 10.50.10.209:8066 check port 48700 inter 5s rise 2 fall 3
      srvtimeout 20000
listen  allmycat_admin 
bind 10.50.10.155:8097 
      mode tcp
      option tcplog
      option httpchk OPTIONS * HTTP/1.1\r\nHost:\ www
      balance    roundrobin
        server    mycat_150 10.50.10.150:9066 check port 48700 inter 5s rise 2 fall 3
        server    mycat_151  10.50.10.209:9066 check port 48700 inter 5s rise 2 fall 3
      srvtimeout 20000

=====haproxy服务启动脚本====
#!/bin/bash
#
# descrition: haproxy loadbalancer
 
DAEMON=haproxy
PROG_DIR=/usr/local/haproxy
RETVAL=0
 
success() {                       
for ((i=0;i<=5;i++))
do
sleep 0.2
echo -n "."
done
}
 
start ()
{
    PROG_STAT=$(netstat -tlnp | grep ${DAEMON})
    if [ -z "$PROG_STAT" ]; then
  $PROG_DIR/sbin/$DAEMON -f $PROG_DIR/${DAEMON}.cfg
        echo -ne "Starting ${DAEMON}......\t\t\t"  && success
  echo -e "\e[32m[OK]\e[0m" 
    else
        echo "$DAEMON is already running"
RETVAL=65
    fi
}
 
stop ()
{
    PROG_STAT=$(netstat -tlnp | grep ${DAEMON})
    if [ -n "$PROG_STAT" ]; then
        echo -ne "stopping ${DAEMON}......\t\t\t"  && success
        ps -ef|grep -w haproxy|grep -v grep|awk '{print $2}'|xargs kill -9 
        echo -e "\e[32m[OK]\e[0m"
    else
        echo "$DAEMON is already stopped"
RETVAL=66
    fi
}
 
restart()
{
    echo -ne "restarting ${DAEMON}......\t\t\t"   && success
    PROG_PID=$(ps -ef|grep -w haproxy|grep -v grep|awk '{print $2}')
    $PROG_DIR/sbin/$DAEMON -f $PROG_DIR/${DAEMON}.cfg -st $PROG_PID
    echo -e "\e[32m[OK]\e[0m"
}
 
status ()
{
    PROG_STAT=$(netstat -tlnp | grep ${DAEMON})
    if [ -z "$PROG_STAT" ]; then
        echo "${DAEMON} stopped"
    else
        echo "${DAEMON} running"
    fi
}
 
case "$1" in
    start)
        start
        ;;
    stop)
        stop
        ;;
    restart)
        restart
        ;;
    status)
        status
        ;;
    *)
        echo "Usage /etc/init.d/$DAEMON {start | stop | restart | status}"
RETVAL=67
esac
exit $RETVAL
====================================================开启日志功能=======================================================
1.配置syslog模块,在Linux下是rsyslogd服务,yum –y install rsyslog
2.建立/etc/rsyslog.d/haproxy.conf文件  vi /etc/rsyslog.d/haproxy.conf
3.修改/etc/rsyslog.conf配置 vi /etc/rsyslog.conf
4.重启rsyslog服务 service rsyslog restart 查看/var/log/haproxy.log

##/etc/rsyslog.d/haproxy.conf
$ModLoad imudp
$UDPServerRun 514
local0.* /var/log/haproxy.log  #日志记录位置

##修改/etc/rsyslog.conf配置
1.在#### RULES ####上面加入
$IncludeConfig /etc/rsyslog.d/*.conf
2.在local7.* /var/log/boot.log的下面加入
local0.* /var/log/haproxy.log

==================================================配置监听mycat是否存活======================================
在Mycat上都需要添加检测端口48700的脚本，用到xinetd，xinetd为linux系统的基础服务
1.yum install xinetd -y
2.检查/etc/xinetd.conf的末尾是否有这一句：includedir /etc/xinetd.d 没有加上
3.检查/etc/xinetd.d文件夹是否存在，没有加上
4.增加/etc/xinetd.d/mycat_status文件 vi /etc/xinetd.d/mycat_status
5.创建/usr/local/bin/mycat_status脚本 vi /usr/local/bin/mycat_status 
6.修改脚本权限 chmod 777 /usr/local/bin/mycat_status chmod 777 /etc/xinetd.d/mycat_status
7.将启动脚本加入服务 vim /etc/services 在末尾加入mycat_status 48700/tcp # mycat_status
8.重启xinetd服务 service xinetd restart
9.验证mycat_status服务是否启动成功 netstat -antup|grep 48700
##/etc/xinetd.d/mycat_status
service mycat_status
{
flags = REUSE
socket_type = stream
port = 48700
wait = no
user = root
server =/usr/local/bin/mycat_status
log_on_failure += USERID
disable = no
}
##/usr/local/bin/mycat_status
#!/bin/bash
#/usr/local/bin/mycat_status.sh
#This script checks if a mycat server is healthy running on localhost. It will
#return:
#"HTTP/1.x 200 OK\r" (if mycat is running smoothly)
#"HTTP/1.x 503 Internal Server Error\r" (else)
mycat=`/usr/local/mycat/bin/mycat status |grep 'not running'| wc -l`
if [ "$mycat" = "0" ];
then
/bin/echo -e "HTTP/1.1 200 OK\r\n"
else
/bin/echo -e "HTTP/1.1 503 Service Unavailable\r\n"
fi
========================================================注意事项================================================================
#启动顺序
mycat haproxy keepalived
#对外提供服务的网络接口 根据ifconfig 获取到网卡信息填写
#将xinetd加入自启动服务 
chkconfig --add xinetd
chkconfig --level 2345 xinetd on
#将rsyslog加入自动启动服务
chkconfig --add rsyslog
chkconfig --level 2345 rsyslog on
#将keepalived加入自启动服务
chkconfig --add keepalived
chkconfig --level 2345 keepalived on
#将haproxy加入自启动服务 
编写haproxy启动脚本
chkconfig --add haproxy
chkconfig --level 2345 haproxy on
#haproxy日志 /var/log/haproxy.log
#keepalived日志 /usr/local/keepalived/var/log/keepalived-haproxy-state.log
#其他
net.ipv4.ip_forward = 1 #开启IP转发功能
net.ipv4.ip_nonlocal_bind = 1 #开启允许绑定非本机的IP
sudo sysctl -p  
/etc/sysctl.conf


========================================================mycat===============================================================
#mycat连接mysql需要物理机授权
cd /usr/local
wget http://dl.mycat.io/1.6-RELEASE/Mycat-server-1.6-RELEASE-20161028204710-linux.tar.gz
tar zxvf 
cd mycat
useradd mycat
chown -R mycat.mycat *
cd bin
chmod +x *
启动 ./mycat start
关闭 ./mycat stop
将mycat脚本复制到/usr/sbin/


=================================================================================工作原理======================================================================================================
1.主从切换需发送邮件告知
2.如何监控backup
master整台服务器down掉 直接切换到backup
master keepalived服务挂掉 backup keepalived接管vip 
master haproxy服务挂掉 会重启haproxy服务 若重启失败 降低权重 切到bakcup master会不断重启haproxy直到成功后恢复权重重新切回master
keepalived和haproxy必须装在同一台机器上（如172.17.210.210.83机器上，keepalived和haproxy都要安装），keepalived负责为该服务器抢占vip（虚拟ip），haproxy负载转发
haproxy负责将对vip的请求分发到mycat上。起到负载均衡的作用，同时haproxy也能检测到mycat是否存活，haproxy只会将请求转发到存活的mycat上。



=================================================================================haproxy原理===================================================================================================
#haproxy配置
http://www.cnblogs.com/dkblog/archive/2012/03/13/2393321.html
监控页面参数 
http://blog.csdn.net/youyudehexie/article/details/7588423
特性
1.可靠性和稳定性非常好，可以与硬件级的F5负载均衡设备相媲美；
2.最高可以同时维护40 000~50 000个并发连接，单位时间处理的最大请求数为20 000个，最大数据处理能力可达10Gbps
3.支持多于8种负载均衡算法，同时也支持session（会话）保持。
4.支持虚拟主机功能，这样实现web负载均衡更加灵活
5.支持（HAProxy1.3版本之后）连接拒绝、全透明代理等功能，这些是其他负载均衡器所不具备的
6.拥有一个功能强大的服务器状态监控页面，通过此页面可以实时了解系统的运行状况
7.拥有功能强大的ACL支持，能给使用带来很大方便

global 配置中的参数为进程级别的参数，且通常与其运行的操作系统有关
defaults：用于为所有其他配置段提供默认参数，这配置默认配置参数可由下一个"defaults"所重新设定
forntend：用于定义一系列监听的套接字，这些套接字可以接受客户端请求并与子建立连接
backend: 用于定义一系列“后端”服务器，代理将会将对应客户端的请求转发至这些服务器
listen: 用于定义通过关联“前段”和“后端”一个完整的代理，通常只对TCP流量有用

option httpclose ：HAProxy会针对客户端的第一条请求的返回添加cookie并返回给客户端，客户端发送后续请求时会发送
 此cookie到HAProxy，HAProxy会针对此cookie分发到上次处理此请求的服务器上，如果服务器不能忽略
 此cookie值会影响处理结果。如果避免这种情况配置此选项，防止产生多余的cookie信息。
option forwardfor ：如果服务器上的应用程序想记录发起请求的客户端的IP地址，需要在HAProxy上配置此选项，这样
 HAProxy会把客户端的IP信息发送给服务器，在HTTP请求中添加"X-Forwarded-For"字段。
option originalto ：如果服务器上的应用程序想记录发起请求的原目的IP地址，需要在HAProxy上配置此选项，这样HAProxy
会添加"X-Original-To"字段。
option dontlognull ：保证HAProxy不记录上级负载均衡发送过来的用于检测状态没有数据的心跳包。

BALANCE 选项：
balance source ：如果想让HAProxy按照客户端的IP地址进行负载均衡策略，即同一IP地址的所有请求都发送到同一服务
器时，需要配置此选项。
balance roundrobin ：HAProxy把请求轮流的转发到每一个服务器上，依据每台服务器的权重，此权重会动态调整。最常
见的默认配置。
===============================================================================keepalived原理==================================================================================================
#keepalived
http://freeloda.blog.51cto.com/2033581/1280962
http://blog.csdn.net/tantexian/article/details/50056229
Keepalived的作用是检测服务器的状态，如果有一台web服务器死机，或工作出现故障，Keepalived将检测到，并将有故障的服务器从系统中剔除，同时使用其他服务器代替该服务器的工作，
当服务器工作正常后Keepalived自动将服务器加入到服务器群中，这些工作全部自动完成，不需要人工干涉，需要人工做的只是修复故障的服务器。
1.openssl  
2.openssl-devel
3.keepalived
####特别注意
1、为避免同一个局域网中有多个keepalived组中的多播相互响应，采用单播通信
2、状态切换的过程触发邮件通知、短信通知、web通知、log记录，便于通过各种途径了解主备工作状态
3、nginx的检测脚本采用了轻量级的方式："killall -0 nginx"，还可以使用 pidof nginx的方式或者调用其他自定义检测脚本的方式
4、特别要注意优先级的大小及检测到异常时权重的变化
5、了解免费ARP的工作机制
6、了解VRRP协议的适用范围：局域网，第一跳网关冗余
7、单个vrrp实例工作在主备模式，为最大程度的利用2个节点的资源，可以做多个vrrp实例，实现高可用和负载均衡
8、防火墙影响  
iptables -A INPUT -i eth0 -p vrrp -s 10.0.37.4 -j ACCEPT
service iptables save
