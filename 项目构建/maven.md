maven
http://blog.jobbole.com/107576/
http://mvnrepository.com/

maven创建web项目
http://blog.csdn.net/chuyuqing/article/details/28879477

仓库详解
http://blog.csdn.net/wanghantong/article/details/36427433


####仓库
简介
    Maven 中, 任何一个依赖、插件或项目构建的输出, 都可称为构件, 而Maven仓库就是集中存储这些构件的地方.
分类
    本地仓库与远程仓库. 当Maven根据坐标寻找构件时, 它会首先检索本地仓库, 如果本地存在则直接使用, 否则去远程仓库下载.


####pom.xml详解
groupId:一个组织或项目的唯一id，一般使用项目包名，例如com.test中,group ID为com.test中/com/test中
artifactId:正在构建的项目名称，例如test1
version:项目的具体版本号,例如1.0.0
构建后生成结果位于MAVEN_REPO/com/test/test1/1.0.0/test1-1.0.0.jar

所有的pom都会继承base pom，从而默认从中央仓库下载构件
01.  <repositories>  
02.    <repository>  
03.      <id>central</id>  
04.      <name>Central Repository</name>  
05.      <url>http://repo.maven.apache.org/maven2</url>  
06.      <layout>default</layout>  
07.      <snapshots>  
08.        <enabled>false</enabled>  
09.      </snapshots>  
10.    </repository>  
11.  </repositories>  


####依赖
maven依赖
    groupId、artifactId和version
    type:jar,war
    scope:compile、provided、runtime、test、system和import 
    optional:依赖是否可选，默认false，即子项目默认都继承，否则需显示的引入
    exclusions：排除传递性依赖
外部依赖
    如果依赖的jar不在本地和中央仓库中，有三种解决办法
    1.本地安装这个插件install plugin
    例如：mvn install:intall-file -Dfile=non-maven-proj.jar -DgroupId=som.group -DartifactId=non-maven-proj -Dversion=1
    2.创建自己的repositories并且部署这个包，使用类似上面的deploy:deploy-file命令，
    3.设置scope为system,并且指定系统路径
快照依赖
    在版本号后追加-SNAPSHOT，则告诉Maven你的项目是一个快照版本
    <version>1.0-SNAPSHOT</version>


####mvn命令详解
mvn dependency:tree 查看当前项目的依赖树


####maven目录结构
- src
  - main
    - java
    - resources
    - webapp
  - test
    - java
    - resources
- target
src目录是源代码和测试代码的根目录。main目录是应用的源代码目录。test目录是测试代码的目录。main和test下的java目录，分别表示应用的java源代码和测试代码。

resources目录包含项目的资源文件，比如应用的国际化配置的属性文件等。
如果是一个web项目，则webapp目录为web项目的根目录，其中包含如WEB-INF等子目录。

target目录是由Maven创建的，其中包含编译后的类文件、jar文件等。当执行maven的clean目标后，target目录会被清空。

####聚合与继承

####集成myeclipse

####nexus搭建私服
http://www.oschina.net/question/698806_159372
nexus:maven仓库管理软件
简介
    当 Maven 需要下载构件时，直接请求私服，私服上存在则下载到本地仓库；否则，私服请求外部的远程仓库，将构件下载到私服，再提供给本地仓库下载
下载
    专业版付费，下载开源版Nexus OSS
访问
    http://localhost:8081/nexus/ 登录：admin,admin123
配置maven：
    <mirrors> 
        <mirror>
            <id>central</id> 
            <mirrorOf>*</mirrorOf> 
            <name>central-mirror</name> 
            <url>http://localhost:8081/nexus/content/groups/public/</url> 
        </mirror> 
    </mirrors>
特性
    1.节省自己的外网带宽：减少重复请求造成的外网带宽消耗
    2.加速Maven构件：如果项目配置了很多外部远程仓库的时候，构建速度就会大大降低
    3.部署第三方构件：有些构件无法从外部仓库获得的时候，我们可以把这些构件部署到内部仓库(私服)中，供内部maven项目使用
    4.提高稳定性，增强控制：Internet不稳定的时候，maven构建也会变的不稳定，一些私服软还提供了其他的功能
    5.降低中央仓库的负荷：maven中央仓库被请求的数量是巨大的，配置私服也可以大大降低中央仓库的压力
配置访问认证
    setting.xml中配置server
更换私服的数据来源：稳定的maven镜像

####maven构建
构建生命周期、阶段和目标，一个构建周期由一系列的构建阶段组成，每一个构建阶段由一系列的目标组成
生命周期: clean、default 和 site
构建阶段：阶段是有顺序的, 并且后面的阶段依赖于前面的阶段

####maven插件
生命周期的阶段phase与插件的目标goal相互绑定, 用以完成实际的构建任务
为了能让用户几乎不用任何配置就能使用Maven构建项目,Maven默认为一些核心的生命周期绑定了插件目标，当用户通过命令调用生命周期阶段时对应的插件目标就会执行相应的逻辑.


####setting.xml详解


####pom.xml build
http://www.cnblogs.com/Binhua-Liu/p/5604841.html

http://www.cnblogs.com/qq78292959/p/3711501.html
http://www.cnblogs.com/adolfmc/archive/2012/07/31/2616908.html
http://blog.jobbole.com/107576/
http://codecloud.net/13263.html
http://blog.csdn.net/liujiahan629629/article/details/39272321
