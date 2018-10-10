======================资源==========================
#学习
http://man.linuxde.net/
http://c.biancheng.net/cpp/html/2726.html
http://www.runoob.com/linux/linux-command-manual.html
#linux服务 
http://blog.163.com/koumm@126/blog/static/9540383720092992634315/
#为什么linux更适合做服务器
http://wenda.so.com/q/1377727018062634
#linux 安装redis 
http://blog.csdn.net/lgh1117/article/details/48270085


============================linux命令分类
=======基础
grep
find http://www.cnblogs.com/wanqieddy/archive/2011/06/09/2076785.html
top
whereis http://man.linuxde.net/whereis
which http://man.linuxde.net/which
find http://man.linuxde.net/find
wget
yum
apt-get
scp 

awk http://www.cnblogs.com/ggjucheng/archive/2013/01/13/2858470.html
wc
cut 
sort
xargs 
debugfs
man
host
sed
管道
    
=======文件内容
cat 显示全文，内容太大时只能看到最后面的内容
more 分屏显示，往下翻屏
less 分页显示，可上下滚动（一行 或 一页）
tail -f  实时显示文件输入
配合通道符查看
    cat test | grep -C 5 'aaaa' | less
    less -p 'aaa'

=======进程端口
netstat –anp | grep 8080 查看端口占用
ps -aux | grep java 查看进程
pidof java 查看进程pid
ps ef|grep -v grep|grep java|awk '{print $2}' 查看进程pid
kill -9 pid  pid终止进程
pkill -9 java  进程名终止pid进程
killall -9 java 终止所有进程
pgrep -l java 根据进程名查看制定进程

=======文件目录操作
mv 移动文件或目录
cp 复制文件或目录
ls  http://man.linuxde.net/ls
touch 创建文件
mkdir 创建目录
rm -rf 删除文件或目录
zip 压缩
unzip 解压缩
ln 建立文件或目录的链接 有-s为软链接，没有为硬链接
    1.ln命令会保持每一处链接文件的同步性，也就是说，不论你改动了哪一处，其它的文件都会发生相同的变化； 
    2.软连接：它只会在你选定的位置上生成一个文件的镜像，不会占用磁盘空间，硬链接：它会在你选定的位置上生成一个和源文件大小相同的文件，无论是软链接还是硬链接，文件都保持同步变化。软链接是可以跨分区的，但是硬链接只能在同一分区内
    3.目录不能建立硬链接，但可以建立软链接。
cd - 返回前一次所在目录
rz 从远程上传 需要安装 lrzsz
sz 下载到远程 需要安装 lrzsz
tar -xvzf 解压tar文件
stat 查看文件修改时间
du 查看文件或目录大小

============系统网络
reset 清屏
logout 退出当前用户
shutdown -h now 立即关机
sudo passwd root 设置root密码
su 切换到root
su - liuheng 切换到用户liuheng
reboot 重启
ctrl+z 退出当前命令

ifconfig 
tracepath
tcpdump
ip addr
lsof -i:port

============================基本常识
linux系统分类： 
1.RedHat系列：Redhat、Centos、Fedora等
2.Debian系列：Debian、Ubuntu等
redhat系列支持rpm包和tar 高级rpm包管理工具yum
debian系列支持deb包和tar 高级deb包管理工具apt-get


uname -a 查看系统信息
cat /proc/version 运行的内核版本
cat /etc/issue  发行版本信息


============================文件编辑
vim详解 http://blog.csdn.net/xiaolong2w/article/details/8224839
#vi vim区别
#删除全部文件内容
    G跳到文件尾部 
    :1,.d  1代表第一行 .代表当前行 d删除
#跳到指定行
    :行号


============================系统服务
http://blog.csdn.net/taiyang1987912/article/details/41698817
添加系统服务
    1.将启动脚本添加到/etc/init.d 
    脚本包含进程的启动关闭重启状态查看 有的软件自带该脚本 否则需要自己编写
    2.chkconfig --add 服务名 
    在chkconfig工具服务列表中增加此服务，此时服务会被在/etc/rc.d/rcN.d中赋予K/S入口了；

设置服务自启动
    1.chkconfig 服务名 on  (7个级别 默认2345) chkconfig --level 2345 服务名 on
    2.设置软连接 ln -s /etc/init.d/服务  /etc/rc.d/rc3.d/软连接名

关闭服务自启动
    chkconfig 服务名 off

查看全部服务
    chkconfig --list

查看指定服务
    chkconfig --list 服务名

删除服务
    chkconfig --del 服务名

============================权限用户组相关
chmod
    使用权限 : 所有使用者 
    语法为：chmod abc file   chmod 777 file
    其中a,b,c各为一个数字，分别表示User、Group、及Other的权限。 
    r=4 可读，w=2 可写，x=1 可执行 
    若要rwx属性则4+2+1=7； 
    若要rw属性则4+2=6； 
    若要rx属性则4+1=5。

chown
    使用权限 : root
    chown将指定文件的拥有者改为指定的用户或组
    # chown [-R] [用户名称] [文件或目录]
    # chown[-R] [用户名称:组名称] [文件或目录]
    chown -R -v root:mail test6 改变指定目录以及其子目录下的所有文件的拥有者和群组 
    chown也提供了-R参数，这个参数对目录改变属主和属组极为有用，可以通过加 -R参数来改变某个目录下的所有文件到新的属主或属组。

用户与组
    groups 查看当前登录用户的组内成员
    groups name 查看用户所在的组,以及组内成员
    /etc/group文件包含所有组
    /etc/shadow和/etc/passwd系统存在的所有用户名

============================软件下载安装卸载
###包安装
1.wget
wget是一个从网络上自动下载文件的自由工具，支持通过HTTP、HTTPS、FTP三个最常见的TCP/IP协议下载，并可以使用HTTP代理
2.yum
rpm高级包管理工具，能够从指定的服务器自动下载RPM包并且安装，可以自动处理依赖性关系，并且一次安装所有依赖的软件包，无须繁琐地一次次下载、安装
3.apt-get 
deb高级包管理工具，主要用于自动从互联网的软件仓库中搜索、安装、升级、卸载软件或操作系统


###源码安装
wget  下载
tar zxvf  tar.gz 或 tar jxvf tar.bz2
    解压产生的一个名为configure的可执行脚本程序
./configure 
    执行configure脚本，用于检查系统是否有编译时所需的库，以及库的版本是否满足编译的需要等安装所需要的系统信息，检查通过后，将生成用于编译的MakeFile文件
make 编译
make install 安装

###卸载软件
包括二进制文件，库，配置文件
makefile 
make uninstall
make clean 
make distclean



============================文件目录介绍
/etc：配置文件

/bin：重要执行档 可以被root与一般帐号所使用

/sbin：Linux有非常多指令是用来设定系统环境的,重要的系统执行文件

/dev：所需要的装置设备文件

/lib：执行档所需的函式库与核心所需的模块

/boot：Linux核心档案以及开机选单与开机所需设定档

/home

/root

/opt:第三方协力软体放置的目录

/srv:srv可以视为service的缩写，是一些网路服务启动之后，这些服务所需要取用的资料目录

/usr:Unix Software Resource,也就是Unix操作系统软件资源所放置的目录

/var:软件运作所产生的文件， 包括程序文件(lock file, run file)

绝对路径：以/开头
相对路径：不以/开头
.  ：代表当前的目录，也可以使用 ./ 来表示；
.. ：代表上一层目录，也可以 ../ 来代表。

============================常用系统配置
文件系统 
    /etc/fstab 开机时挂载的文件系统 
    /etc/mtab 当前挂载的文件系统 
用户系统 
    /etc/passwd 用户信息 
    /etc/shadow 用户密码 
    /etc/group 群组信息 
    /etc/gshadow 群组密码 
    /etc/sudoers Sudoer列表（请使用“visudo”命令修改此文件，而不要直接编辑） 
Shell 
    /etc/shell 可用Shell列表 
    /etc/inputrc ReadLine控件设定 
    /etc/profile 用户首选项 
    /etc/bash.bashrc bash配置文件 
系统环境 
    /etc/environment 环境变量 
    /etc/updatedb.conf 文件检索数据库配置信息 
    /etc/issue 发行信息 
    /etc/issue.net 
    /etc/screenrc 屏幕设定 
网络 
    /etc/iftab 网卡MAC地址绑定 
    /etc/hosts 主机列表 
    /etc/hostname 主机名 
    /etc/resolv.conf 域名解析服务器地址 
    /etc/network/interfaces 网卡配置文件 

============================磁盘分区
http://blog.csdn.net/pzxwhc/article/details/45845255
硬盘--分区--选择文件系统--挂载到目录
df -a 查看硬盘分区和已挂在的文件系统的磁盘空间
mount 是挂载文件系统，不加参数是显示已挂载的文件系统和目录
fdisk -l 显示当前设备的分区表


============================环境变量
http://www.cnblogs.com/hust-chenming/p/4943268.html
查看环境变量
    echo $PATH
修改方法一：
    export PATH=/usr/local/mongodb/bin:$PATH
    配置完后可以通过echo $PATH查看配置结果。
    生效方法：立即生效
    有效期限：临时改变，只能在当前的终端窗口中有效，当前窗口关闭后就会恢复原有的path配置
    用户局限：仅对当前用户

修改方法二：
    通过修改.bashrc文件:vim ~/.bashrc 
    在最后一行添上：
    export PATH=/usr/local/mongodb/bin:$PATH
    生效方法：（有以下两种）
    1、关闭当前终端窗口，重新打开一个新终端窗口就能生效
    2、输入“source ~/.bashrc”命令，立即生效
    有效期限：永久有效
    用户局限：仅对当前用户

修改方法三:
    通过修改profile文件:vim /etc/profile
    找到设置PATH的行，添加
    export PATH=/usr/local/mongodb/bin:$PATH
    生效方法：系统重启
    有效期限：永久有效
    用户局限：对所有用户

修改方法四:
    通过修改environment文件:vim /etc/environment
    在PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games"中加入“:/usr/local/mongodb/bin”
    生效方法：系统重启
    有效期限：永久有效
    用户局限：对所有用户

============================防火墙
http://blog.chinaunix.net/uid-9950859-id-98279.html
    
/etc/sysconfig/iptables 默认文件

service iptables save 保存防火墙配置

service iptables stop 停止防火墙

iptables -F 清除防火墙原来规则

#开放ssh防火墙
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
#开放数据库防火墙规则
iptables -A INPUT -s 192.168.3.0/24 -p tcp --dport 3306 -j ACCEPT
#开放回环地址访问规则
iptables -A INPUT -i lo -j ACCEPT
#开放ping防火墙规则
iptables -A INPUT -p icmp -j ACCEPT 
#开放vrrp防火墙规则
iptables -A INPUT -p vrrp -j ACCEPT 
#开放防火墙规则
iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
#防火墙进来默认规则为drop
iptables -P INPUT DROP
#防火墙出去默认规则为drop
iptables -P OUTPUT ACCEPT


#-j accept/drop
*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
-A INPUT -d 192.168.10.200 -j ACCEPT
-A INPUT -d 192.168.10.201 -j ACCEPT
-A INPUT -p vrrp -j ACCEPT
-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -j REJECT --reject-with icmp-host-prohibited
COMMIT

==============================其他
#/etc/ld.so.conf 
此文件记录了编译时使用的动态库的路径，也就是加载so库的路径。
默认情况下，编译器只会使用/lib和/usr/lib这两个目录下的库文件，而通常通过源码包进行安装时，如果不
指定--prefix会将库安装在/usr/local目录下，而又没有在文件/etc/ld.so.conf中添加/usr/local/lib这个目录>。这样虽然安装了源码包，但是使用时仍然找不到相关的.so库，就会报错。也就是说系统不知道安装了源码包。
对于此种情况有2种解决办法：
(1)在用源码安装时，用--prefix指定安装路径为/usr/lib。这样的话也就不用配置PKG_CONFIG_PATH
(2)直接将路径/usr/local/lib路径加入到文件/etc/ld.so.conf文件的中。在文件/etc/ld.so.conf中末尾直接添加：/usr/local/lib（这个方法给力！）

crontab 定时任务
    服务状态
     /etc/init.d/cron stop
     /etc/init.d/cron start
     /etc/init.d/cron status

    命令
    1.more /etc/crontab
    2.编写用户定时任务 存放于/var/spool/cron/
    3.crontab -l 查看当前用户任务
    4.crontab -r 删除当前用户任务
    5.crontab -e 编辑当前用户任务 select-editor修改编辑模式

    相关文件
    cron.daily是每天执行一次的job
    cron.weekly是每个星期执行一次的job
    cron.monthly是每月执行一次的job
    cron.hourly是每个小时执行一次的job
    cron.d是系统自动定期需要做的任务
    crontab是设定定时任务执行文件
    cron.deny文件就是用于控制不让哪些用户使用Crontab的功能

    cron表达式
    * * * * * /home/backup/bin/xyj_order.sh 
    40 18 16 5 * /home/banner.sh
    minute： 表示分钟，可以是从0到59之间的任何整数。
    hour：表示小时，可以是从0到23之间的任何整数。
    day：表示日期，可以是从1到31之间的任何整数。
    month：表示月份，可以是从1到12之间的任何整数。
    week：表示星期几，可以是从0到7之间的任何整数，这里的0或7代表星期日。
    command：要执行的命令，可以是系统命令，也可以是自己编写的脚本文件。

    星号（*）：代表所有可能的值，例如month字段如果是星号，则表示在满足其它字段的制约条件后每月都执行该命令操作。
    逗号（,）：可以用逗号隔开的值指定一个列表范围，例如，“1,2,5,7,8,9”。
    中杠（-）：可以用整数之间的中杠表示一个整数范围，例如“2-6”表示“2,3,4,5,6”。
    正斜线（/）：可以用正斜线指定时间的间隔频率，例如“0-23/2”表示每两小时执行一次。同时正斜线可以和星号一起使用，例如*/10，如果用在minute字段，表示每十分钟执行一次。


开机自启动
    1.在/etc/rc.local文件中加入启动命令
    2.在/etc/rc[0-6].d/目录建立软链接，软链接指向/etc/init.d/目录下的控制脚本


wc
    wc -l
    wc -w
    wc -c


grep
    grep -A
    grep -B
    grep -C
    grep -n 
    grep -c


sort
    -n:比较数字
    -r:分割后的第几列
    -k:指定分隔符


vi
    Ctrl+u：向文件首翻半屏；
    Ctrl+d：向文件尾翻半屏；
    Ctrl+f：向文件尾翻一屏；
    Ctrl+b：向文件首翻一屏；
    Esc：从编辑模式切换到命令模式；
    ZZ：命令模式下保存当前文件所做的修改后退出vi；
    :行号：光标跳转到指定行的行首；
    :$：光标跳转到最后一行的行首；
    x或X：删除一个字符，x删除光标后的，而X删除光标前的；
    D：删除从当前光标到光标所在行尾的全部字符；
    dd：删除光标行正行内容；
    ndd：删除当前行及其后n-1行；
    /字符串：文本查找操作，用于从当前光标所在位置开始向文件尾部查找指定字符串的内容，查找的字符串会被加亮显示；
    ？name：文本查找操作，用于从当前光标所在位置开始向文件头部查找指定字符串的内容，查找的字符串会被加亮显示；
    :set number：在命令模式下，用于在最左端显示行号；
    :set nonumber：在命令模式下，用于在最左端不显示行号；



find
    -name
    -path
    -type
    -regex
    ! -name
    
    find . -type f -atime -7 搜索最近七天内被访问过的所有文件
    
    find . -type f -atime 7 搜索恰好在七天前被访问过的所有文件
    
    find . -type f -atime +7 搜索超过七天内被访问过的所有文件
    
    find . -type f -amin +10 搜索访问时间超过10分钟的所有文件


文件查找和比较
    diff
    cmp
    locate
    which
    find
    whereis