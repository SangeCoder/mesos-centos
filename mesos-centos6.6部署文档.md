# 一、环境介绍 #
单机环境、跳板机（根据自己的需要即可）

    [root@Siffre ~]# cat   /etc/redhat-release 
    CentOS release 6.6 (Final)
    [root@Siffre ~]# uname -r
    2.6.32-504.el6.x86_64
    
# 二、安装Docker #
## 2.1 下载官网rpm包 

    [root@Siffre ~]# wget   https://get.docker.com/rpm/1.7.1/centos-6/RPMS/x86_64/docker-engine-1.7.1-1.el6.x86_64.rpm
    
##  2.2安装rpm包 ##
安装前我们需要安装一个依赖包：
    
    [root@Siffre ~]# yum  install  libcgroup  -y
安装rpm包：

    [root@Siffre ~]# rpm -ivh   docker-engine-1.7.1-1.el6.x86_64.rpm 
    
## 2.3检查并启动Docker ##

检查docker版本

    [root@Siffre ~]# docker  version
    Client version: 1.7.1
    Client API version: 1.19
    Go version (client): go1.4.2
    Git commit (client): 786b29d
    OS/Arch (client): linux/amd64
    Server version: 1.7.1
    Server API version: 1.19
    Go version (server): go1.4.2
    Git commit (server): 786b29d
    OS/Arch (server): linux/amd64
启动docker

     service  docker  start
     或
     /etc/init.d/docker  start
     
#     三、服务安装 #
数人科技源：

    curl -o /etc/yum.repos.d/dataman.repo http://get.dataman.io/repos/centos/6/6/dataman.repo
官方源：

    wget http://www.apache.org/dist/mesos/0.23.0/mesos-0.23.0.tar.gz
官方Git源：

    git clone https://git-wip-us.apache.org/repos/asf/mesos.git


**以下根据数人科技的源安装，后期会编译安装**

## 3.1 zookeeper ##
导入源：

    [root@Siffre ~]#  rpm -Uvh http://archive.cloudera.com/cdh4/one-click-install/redhat/6/x86_64/cloudera-cdh-4-0.x86_64.rpm
yum安装

    [root@Siffre ~]# yum  install  zookeeper zookeeper-server  -y
    
##  3.2 mesos ##
导入源：

    [root@Siffre ~]# curl -o /etc/yum.repos.d/dataman.repo http://get.dataman.io/repos/centos/6/6/dataman.repo
yum安装：

    [root@Siffre ~]# yum  install  mesos -y
## 3.3 marathon ##
导入源：

    [root@Siffre ~]# curl -o /etc/yum.repos.d/dataman.repo http://get.dataman.io/repos/centos/6/6/dataman.repo
yum安装：

    [root@Siffre ~]# yum install marathon -y
## 3.4 haproxy ##
yum安装即可：

    [root@Siffre ~]# yum  install  haproxy  -y
## 3.5 bamboo ##

导入源：

    [root@Siffre ~]# mkdir  /application
    [root@Siffre ~]# cd /application/
    [root@Siffre application]# wget http://datamanpub.ufile.ucloud.com.cn/download/dataman-bamboo-0.9.0.tar.gz
    [root@Siffre application]# ls
    dataman-bamboo-0.9.0.tar.gz
    [root@Siffre application]# tar -zxf  dataman-bamboo-0.9.0.tar.gz
    [root@Siffre application]# ls
    bamboo  dataman-bamboo-0.9.0.tar.gz
 
#  四、配置服务参数 #
## 4.1 Mesos ##
自身配置：


    #配置mesos在zk的使用目录
    echo "zk://10.3.10.29:2181/mesos" > "/etc/mesos/zk"  #zookeeper2181默认端口
### 4.1.1 Mesos-Master ###

    #指定master配置目录
    MESOS_MASTER_CONF_DIR="/etc/mesos-master"
    #指定master的主机名
    echo "10.3.10.29" > $MESOS_MASTER_CONF_DIR/hostname  
    
    #用本机IP表示#指定master的ip
    echo "0.0.0.0" > $MESOS_MASTER_CONF_DIR/ip
    #副本的仲裁数量的大小（集群配置很重要，本次试验只有1台所以写1）
    echo "1" > $MESOS_MASTER_CONF_DIR/quorum
    #注册表中存储持久性信息的地址
    echo "/var/lib/mesos" > $MESOS_MASTER_CONF_DIR/work_dir
    
### 4.1.2 Master-Slave ###

    #指定slave配置目录
    MESOS_SLAVE_CONF_DIR="/etc/mesos-slave"
    #指定slave的主机名(这里不能用localhost)
    echo "10.3.10.29 " > $MESOS_SLAVE_CONF_DIR/hostname 
    #用IP表示#指定slave支持的容器类型
    echo "docker,mesos" > $MESOS_SLAVE_CONF_DIR/containerizer
    s#指定slave的ip
    echo "0.0.0.0" > $MESOS_SLAVE_CONF_DIR/ip
    #执行器注册超时时间
    echo "5mins" > $MESOS_SLAVE_CONF_DIR/executor_registration_timeout
    #指定mesos资源控制的内容(这里只有打开对CPU和内存的控制)
    echo "cgroups/cpu,cgroups/mem" > $MESOS_SLAVE_CONF_DIR/isolation
## 4.2 marathon ##
    ＃创建marathon目录
    mkdir /etc/marathon/conf  -p
    #指定marathon配置目录
    MARATHON_CONF_DIR="/etc/marathon/conf"
    #指定marathon在zk目录路径
    echo "zk://10.3.10.29:2181/marathon" > $MARATHON_CONF_DIR/zk
    #事件订阅模式
    echo "http_callback" > $MARATHON_CONF_DIR/event_subscriber
    #指定marathon主机名
    echo "10.3.10.29" > $MARATHON_CONF_DIR/hostname  
    #用IP表示#指定mesos在zk目录路径
    echo "zk://10.3.10.29:2181/mesos" > $MARATHON_CONF_DIR/master
    
##  4.3 bamboo ##
### 4.3.1 注释模版的8080部分，否则该8080端口和marathon自带默认端口冲突 ###

    [root@Siffre application]# vim  /application/bamboo/config/haproxy_template.cfg 

    #注释以下部分
    frontend websocket-in 
    #注意是websocket-in而不是http-in
    bind *:8080  
    {{ $services := .Services }}
    {{ range $index, $app := .Apps }} {{ if
    $app.Env.BAMBOO_WEBSOCKET_OPEN }} {{ if hasKey $services $app.Id }} {{ $service := getService $services $app.Id }}
    acl {{ $app.EscapedId }}-websocket-aclrule {{ $service.Acl}}:8080
    use_backend {{ $app.EscapedId }}-websocket-cluster if {{ $app.EscapedId }}-websocket-aclrule
    
    {{ end }} {{ end }} {{ end }}
    
    stats enable
    # CHANGE: Your stats credentials
    
    stats auth admin:admin
    stats uri /haproxy_stats
    
    {{ range $index, $app := .Apps }} {{ if 
    
    $app.Env.BAMBOO_WEBSOCKET_OPEN }}backend {{ $app.EscapedId }}-websocket-cluster{{ if $app.HealthCheckPath }}
    
    option httpchk GET {{ $app.HealthCheckPath }}{{ end }}
    balance leastconn
    option httpclose
    option forwardfor
    {{ range $page, $task := .Tasks }}
    server {{ $app.EscapedId }}-{{ $task.Host }}-{{ index $task.Ports 1 }} 
    {{ $task.Host }}:{{ index $task.Ports 1 }} {{ end }}
    {{ end }}
    {{ end }}
    
    #提示：由于centos6.6安装的haproxy版本问题，下面两句也需要注释掉
    global
        log /dev/log    local0
        log /dev/log    local1 notice
        chroot /var/lib/haproxy
        #stats socket /run/haproxy/admin.sock mode 660 level admin   #注释此句
        stats timeout 30s
        user haproxy
        group haproxy
        daemon

        # Default SSL material locations
        ca-base /etc/ssl/certs
        crt-base /etc/ssl/private
        #ssl-default-bind-options no-sslv3  #注释此句

        # Default ciphers to use on SSL-enabled listening sockets.
        # For more information, see ciphers(1SSL).
        # ssl-default-bind-ciphers kEECDH+aRSA+AES:kRSA+AES:+AES256:RC4-SHA:!kEDH:!LOW:!EXP:!MD5:!aNULL:!eNULL
    
    
###   4.3.2 修改bamboo配置 ###
    [root@Siffre application]# vim bamboo/config/production.json
    {  "Marathon":
    {"Endpoint": "http://10.3.10.29:8080"  },  "Bamboo": {"Endpoint": "http://10.3.10.29:8000",
    "Zookeeper": { 
    "Host": "10.3.10.29:2181", 
    "Path": "/marathon-haproxy/state",  
    "ReportingDelay": 5}  }, 
    "HAProxy":
    {"TemplatePath": "/application/bamboo/config/haproxy_template.cfg",
    "OutputPath": "/etc/haproxy/haproxy.cfg",
    "ReloadCommand": "PIDS=`pidof haproxy`; haproxy -f /etc/haproxy/haproxy.cfg -p /var/run/haproxy.pid -sf $PIDS && while ps -p $PIDS; do sleep 0.2; done"  },  
    "StatsD": 
    {"Enabled": false,
    "Host": "10.3.10.29:8125",
    "Prefix": "bamboo-server.development."  }}
    
    
## 4.4 zookeeper  ##

 
    初始化    
    service  zookeeper-server  init  --myid=1 #单机环境可以不初始化    
    提示：若不能更改添加 --force
    service  zookeeper-server  init  --myid=1 --force 
    
#  五、服务启动 #
## 5.1 zookeeper ##
    #启动命令
    service  zookeeper-server  start
    #查看进程
    ps -ef|grep  zookeeper|grep  -v  grep
    #查看日志
    tail -f /var/log/zookeeper/zookeeper.log
    
    提示：
    若zookeeper出现无法启动问题，可能是由于跳板机、sudo、ssh远程登录导致相关变量时效，无法启动java相关程序
    解决方法：安装jdk7的rpm包，不建议去手动去改，可能会出错
    安装方法：
    wget  --no-cookies --no-check-certificate --header "Cookie: oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/7u55-b13/jdk-7u55-linux-x64.rpm" -O jdk-7-linux-x64.rpm
    
## 5.2 Mesos-Master ##

    # 启动命令
    nohup mesos-master --zk=zk://10.3.10.29:2181/mesos  --ip=0.0.0.0 --work_dir=/var/lib/mesos --quorum=1 --log_dir=/var/log/mesos &>> /var/log/mesos/mesos-master.log &
    #查看进程状态
    ps axuf | grep mesos-master | grep -v grep
    #查看日志
    tail -f /var/log/mesos/mesos-master.log
    
##  5.3 Mesos-Slave ##

    # 启动命令
    nohup  mesos-slave --master=10.3.10.29:5050 --hostname=10.3.10.29 &>> /var/log/mesos/mesos-slave.log &
    #查看进程状态
    ps axuf | grep mesos-slave | grep -v grep
    #查看日志
    tail  -f  /var/log/mesos/mesos-slave.log
    
## 5.4 marathon ##
    #添加目录和日志文件
    mkdir  /var/log/marathon 
    touch  /var/log/marathon/marathon.log
    # 启动命令
    nohup marathon  &>> /var/log/marathon/marathon.log &#查看进程状态ps axuf | grep marathon | grep -v grep
    #查看日志
    tail -f /var/log/marathon/marathon.log

## 5.5 haproxy ##

    # 启动命令
    service haproxy start
    #进程状态
    ps axuf | grep haproxy | grep -v grep
    
## 5.6 bamboo ##
        # 启动命令
    nohup /application/bamboo/bamboo -config /application/bamboo/config/production.json -log /var/log/bamboo-server.log &>>/var/log/bamboo.log &
    #查看日志
    tail -f  /var/log/bamboo.log 
    提示：
    1、默认安装haproxy时，相关文件的路径可能会有些不同。这时在启动bamboo时，会发现bamboo启动失败，查看日志：找不到错误类型的文件
    解决方法：
    [root@Siffre haproxy]# cd  /usr/share/haproxy/
    [root@Siffre haproxy]# ls
    400.http  403.http  408.http  500.http  502.http  503.http  504.http  README  
    #将这些文件复制或移到以下目录
    [root@Siffre haproxy]# mkdir  /etc/haproxy/errors
    [root@Siffre haproxy]# mv  *   /etc/haproxy/errors
    [root@Siffre haproxy]# cd   /etc/haproxy/errors/
    [root@Siffre errors]# ls
    400.http  403.http  408.http  500.http  502.http  503.http  504.http  README
    
    2、有时你会在日志文件里面看到这样一个错误提示：STATSD_ENABLED没有设置
    解决方法： export  STATSD_ENABLED=false/true 
    此处是看源码得来的，具体详见bamboo源码！
    
#  六、镜像制作 #
制作镜像的方法很多哦！（至少4种）这里我们不用dockerfile！

## 6.1 基础镜像制作 ##
### 6.1.1 设置docker镜像源 ###

    [root@Siffre ~]# yum install -y yum-priorities && rpm -ivh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm && rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6 
### 6.1.2 安装febootstrap，用来制作centos镜像，到时候会生成个centos的镜像 ###
    # 如果没有安装docker，则需要先安装docker  
    [root@Siffre ~]# service docker start  
    # 启动docker  
    [root@Siffre ~]# yum -y install febootstrap  # 制作docker镜像工具
    
### 6.1.3 作CentOS镜像文件centos6-image目录 ###
    [root@Siffre ~]# febootstrap -i bash -i wget -i yum -i iputils -i iproute -i man -i vim -i openssh-server -i openssh-clients -i tar -i gzip centos6 centos6-image http://mirrors.aliyun.com/centos/6/os/x86_64/  
    
### 6.1.4 复制home目录基础文件到root目录 ###

这时root目录下没有任何文件，也没有隐藏的点文件，如：.bash_logout .bash_profile .bashrc如果这时制作出来的镜像使用ssh登录，会直接进入根目录下，而一般镜像都是进入root目录下的，所以可以在centos6-image目录的root目录把.bash_logout .bash_profile .bashrc这三个文件设置一下。

    [root@Siffre ~]# cd centos6-image && cp etc/skel/.bash* root/

### 6.1.5生成基础镜像 ###
`[root@Siffre centos6-image]# tar -c .|docker import - centos6-base ` 
### 6.1.6 查看镜像，也可以直接进入centos6-base查看 ###

    #这个是查看所有生成的镜像  
    [root@Siffre centos6-image]# docker images 
    #进终端（没有ssh服务），-i 分配终端，-t表示在前台执行，-d表示在后台运行
    [root@Siffre centos6-image]# docker run -i -t centos6-base:latest /bin/bash
    
##  6.2制作中间件镜像 ##
        根据基础镜像制作jdk7的docker镜像
    [root@Siffre centos6-image]# mkdir  -p  /application/data/jdk7
    [root@Siffre centos6-image]#  cd   /application/data/jdk7
    [root@Siffre jdk7]# vim  Dockerfile  #注意换行
    
    FROM centos6-base
    MAINTAINER Siffre
    # install epel
    RUN yum install -y yum-priorities && rpm -ivh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm && rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6
    # install jdk7
    RUN yum update -y
    RUN yum install -y java-1.7.0-openjdk
    #构建镜像
    [root@Siffre jdk7]# docker build -t jdk7  .
    
    
## 6.3 制作应用镜像  ##
  
  
    根据jdk7的镜像生成tomcat7镜像
    [root@Siffre jdk7]# mkdir  tomcat7
    [root@Siffre jdk7]# cd tomcat7/
    [root@Siffre tomcat7]# vim  Dockerfile
    FROM jdk7
    MAINTAINER Siffre
    ENV CATALINA_HOME /usr/local/tomcat
    ENV PATH $CATALINA_HOME/bin:$PATH
    RUN mkdir -p "$CATALINA_HOME"
    WORKDIR $CATALINA_HOME
    # see https://www.apache.org/dist/tomcat/tomcat-8/KEYS
    RUN gpg --keyserver pool.sks-keyservers.net --recv-keys \
    05AB33110949707C93A279E3D3EFE6B686867BA6 \
    07E48665A34DCAFAE522E5E6266191C37C037D42 \
    47309207D818FFD8DCD3F83F1931D684307A10A5 \
    541FBE7D8F78B25E055DDEE13C370389288584E7 \
    61B832AC2F1C5A90F0F9B00A1C506407564C17A3 \
    713DA88BE50911535FE716F5208B0AB1D63011C7 \
    79F7026C690BAA50B92CD8B66A3AD3F4F22C4FED \
    9BA44C2621385CB966EBA586F72C284D731FABEE \
    A27677289986DB50844682F8ACB77FC2E86E29AC \
    A9C5DF4D22E99998D9875A5110C01C5A2F6059E7 \
    DCFD35E0BF8CA7344752DE8B6FB21E8933C60243 \
    F3A04C595DB5B6A5F1ECA43E3B7BBB100D811BBE \
    F7DA48BB64BCB84ECBA7EE6935CD23C10D498E23
    ENV TOMCAT_MAJOR 7
    ENV TOMCAT_VERSION 7.0.63
    ENV TOMCAT_TGZ_URL https://www.apache.org/dist/tomcat/tomcat-$TOMCAT_MAJOR/v$TOMCAT_VERSION/bin/apache-tomcat-$TOMCAT_VERSION.tar.gz
    RUN set -x \
    && curl -fSL "$TOMCAT_TGZ_URL" -o tomcat.tar.gz \
    && curl -fSL "$TOMCAT_TGZ_URL.asc" -o tomcat.tar.gz.asc \
    && gpg --verify tomcat.tar.gz.asc \
    && tar -xvf tomcat.tar.gz --strip-components=1 \
    && rm bin/*.bat \&& rm tomcat.tar.gz*
    EXPOSE 8080
    CMD ["catalina.sh", "run"]   #自带的脚本
    #构建镜像    
    [root@Siffre tomcat7]# docker build -t  tomcat7 .
    
# 七、marathon添加app应用 #

## 7.1 编写添加APP脚本 ##
    [root@localhost marathon-app]# vim dataman-tomcat-test.sh
    
    curl -v -X POST http://10.3.10.29:8080/v2/apps -H Content-Type:application/json -d 
    '{  "id": "dataman-tomcat-test",    #ID自定义即可  
    "cpus": 0.1,  
    "mem": 128.0, 
    "instances": 3,  
    "container": 
    { "type": "DOCKER", 
    "docker": 
    {"image": "tomcat7:latest",#镜像ID 
    "network": "BRIDGE", "portMappings": [   
    { "containerPort": 8080, "hostPort": 0, 
    "servicePort": 10000, 
    "protocol": "tcp" } ]}   
    }, 
    "healthChecks": 
    [{ "protocol": "HTTP",  
    "portIndex": 0,  
    "path": "/",  
    "gracePeriodSeconds": 5,  
    "intervalSeconds": 20,  
    "maxConsecutiveFailures": 3 
    }  ]}'

## 7.2 执行dataman-tomcat-test.sh ##
    [root@localhost marathon-app]# sh  dataman-tomcat-test.sh    
    #显示如下信息
    * About to connect() to 10.3.10.29 port 8080 (#0)
    *   Trying 10.3.10.29... connected
    * Connected to 10.3.10.29 (10.3.10.29) port 8080 (#0)
    > POST /v2/apps HTTP/1.1
    > User-Agent: curl/7.19.7 (x86_64-redhat-linux-gnu) libcurl/7.19.7 NSS/3.16.2.3 Basic ECC zlib/1.2.3 libidn/1.18 libssh2/1.4.2
    > Host: 10.3.10.29:8080
    > Accept: */*
    > Content-Type:application/json
    > Content-Length: 839
    > 
    < HTTP/1.1 201 Created
    < X-Marathon-Leader: http://10.3.10.29:8080
    < Cache-Control: no-cache, no-store, must-revalidate
    < Pragma: no-cache
    < Expires: 0
    < Location: http://10.3.10.29:8080/v2/apps/dataman-tomcat-test
    < Content-Type: application/json
    < Transfer-Encoding: chunked
    < Server: Jetty(8.y.z-SNAPSHOT)
    < 
    {"id":"/dataman-tomcat-test",
    "cmd":null,
    "args":null,
    "user":null,
    "env":{},"instances":3,
    "cpus":0.1,"mem":128.0,"disk":0.0,"executor":""
    ,"constraints":[],"uris":[],"storeUrls":[],"ports":[0],"requirePorts":false,"backoffFactor":1.15,"container":{"type":"DOCKER","volumes":[],"docker":
    {"image":"tomcat7:latest",
    "network":"BRIDGE","portMappings":[{"containerPort":8080,"hostPort":0,
    "servicePort":10000,"protocol":"tcp"}],
    "privileged":false,"parameters":[],"forcePullImage":false}},"healthChecks":[{"path":"/","protocol":"HTTP","portIndex":0,"command":null, 
    "gracePeriodSeconds":5,"intervalSeconds":20,"timeoutSeconds":20,
    "maxConsecutiveFailures":3,"ignoreHttp1xx":false}],"dependencies":[],"upgradeStrategy":
    {"minimumHealthCapacity":1.0,"maximumOverCapacity":1.0},"labels":{},"acceptedResourceRoles":null,
    "version":"2015-08-17T08:25:44.586Z",
    "deployments":[{"id":"045f70ba-4114-4263-af41-73f23f2b2634"}],"tasks":[],"tasksStaged":0,"tasksRunning":0,"tasksHealthy":0,"tasksUnhealthy":0,
    "backoffSeconds":1,"maxLaunchDelaySeconds":36* Connection #0 to host 10.3.10.29 left intact
    * Closing connection #0
    00}
    
##  7.3查看marathon mesos状态 ##

**隧道建立**
*******************************************************************************
在查看状态之前我们需要做一些小的工作：建立隧道，此时要根据你所用系统决定！

**（一）Windows系统**

所需软件：secCRT 、Xshell等，我使用的是xshell4，如下：

![](http://i.imgur.com/OCGDsHk.png)

![](http://i.imgur.com/YPoyO28.png)

打开谷歌浏览器---->扩展程序，推荐一款代理软件：switchysharp

![](http://i.imgur.com/w2ROKlW.png)

安装即可：

![](http://i.imgur.com/fw9nZyb.png)


然后，保存即可！

**（二）MAC系统也是同样的道理**
****************************************************************************

通过Mesos默认调度框架Marathon创建简单使用样例，创建Marathon任务有2种方法:通过web－ui和使用json脚本的方法，下面就通过简单实例进行说明。
### 7.3.1 用webui创建一个top任务(非docker) ###
#### 7.3.1.1 打开marathon－web-ui界⾯面 ####
    http://你安装mesos系统的机器ip地址:8080

![](http://i.imgur.com/0FBreiX.png)

#### 7.3.1.2 创建新任务(点击+NewApp) ####

	填写本次任务的数据信息
	#任务名
	ID dataman-top-test
	#任务需要使用cpu最小大小
	CPUs 0.1
	#任务需要使用内存最小容量
	Memory 16
	#任务需要使用磁盘大小
	Disk 0
	#需要同时跑几个任务
	Instances 1
	#执行器执行命令
	Command top -b
	#执行器
	Executor 空
	#容器调度Slave端口方法
	Ports
	#通过wget模式将容器外部资源动态的获取到容器内部的work_dir中
	URIs 空
	#约束
	Constraints 空

#### 7.3.1.3 执行创建任务(点击+Create) ####
#### 7.3.1.4 marathon创建过程 ####
可以看到marathon的任务表中显示的任务状态

![](http://i.imgur.com/YIypUxc.png)

	任务id(/dataman-top-test)
	内存信息(16)
	cpu信息(0.1)
	运行实例信息(0/1)
	健康心跳状态(空)
	状态(Deploying)
	
#### 7.3.1.5 创建结束查看结果 ####
和创建过程信息一样，但是已经可以看到运行实例信息(1/1),状态是(Running)

![](http://i.imgur.com/YIypUxc.png)

#### 7.3.1.6 进入单一任务详细状态 ####
![](http://i.imgur.com/evVzBfg.png)

	点击任务名进入单一任务详细操作界面
	Suspend 将任务设置为空
	Scale 动态设置任务数量
	Refresh 刷新
	Restart App 重启任务
	Destroy App 删除任务
	version 任务已创建时间
	Updated 最新任务操作时间
	
#### 7.3.1.7 查看任务数据信息 ####
点击Configuration任务创建的数据信息
![](http://i.imgur.com/Eb6ThfC.png)

#### 7.3.1.8 打开mesos-master ####
    http://你安装mesos系统的机器ip地址:5050
    
#### 7.3.1.9 mesos-master总览 ####
![](http://i.imgur.com/IQpYhkX.png)

#### 7.3.1.10 mesos-master信息 ####
左上角可以看到mesos-master信息，包括集群名、masterip、创建时间、集群启动时间等
![](http://i.imgur.com/F2f42AW.png)

#### 7.3.1.11 LOG ####
LOG可以查看mesos-master的实时运行日志
![](http://i.imgur.com/x5W8Sdj.png)

#### 7.3.1.12 Slaves ####
	Activated #集群中存活的从机数量
	Deactivated #集群中死亡的从机数量
![](http://i.imgur.com/p25FeGN.png)

#### 7.3.1.13 Task ####

这里说明这个统计是从Mesos启动后的累加值，并不是当前状态，仅供参考
![](http://i.imgur.com/p9gqMXd.png)

	Staged #创建过任务的数量
	Started #正在开始任务的数量
	Finished #任务正常完成的数量
	Killed #任务手动取消的数量
	Failed #任务执行失败的数量
	Lost #任务丢失的数量
	
#### 7.3.1.14 Resources ####
统计集群cpu和内存资源情况

![](http://i.imgur.com/sKve5nf.png)

	Total 总资源
	Used 使用资源
	Offered 申请资源
	Idle 空闲资源
	
#### 7.3.1.15 Active Tasks ####
正在运行的任务统计
![](http://i.imgur.com/0I7Toyt.png)

这里可以看到刚才创建的任务

#### 7.3.1.16 Sandbox ####
这里可以查看任务运行内部的动态日志，包括正确和错误的

![](http://i.imgur.com/oxuHG8O.png)

	stderr
	#日志样例
	I0701 14:35:19.243409 5881 exec.cpp:132] Version: 0.22.1
	I0701 14:35:19.246486 5883 exec.cpp:206] Executor registered on slave
	20150701-140046-33620746-5050-5032-S0
	stdout
	#日志样例
	5508 root 20 0 4440 636 536 S 0.0 0.0 0:00.00 sh
	5509 root 20 0 125208 9348 4932 S 0.0 0.2 0:00.06 docker
	5564 root 20 0 141600 10904 4408 S 0.0 0.3 0:00.06 docker
	5585 root 20 0 4440 652 548 S 0.0 0.0 0:00.01 sh
	5591 root 20 0 85876 4056 2952 S 0.0 0.1 0:00.00 nginx
	5592 www-data 20 0 86216 2016 604 S 0.0 0.0 0:00.09 nginx
	5593 www-data 20 0 86216 1760 460 S 0.0 0.0 0:00.20 nginx
	5594 www-data 20 0 86216 1760 460 S 0.0 0.0 0:00.17 nginx
	5595 www-data 20 0 86216 1760 460 S 0.0 0.0 0:00.17 nginx
	5596 root 20 0 4440 648 544 S 0.0 0.0 0:00.00 sh
	5597 root 20 0 4440 644 544 S 0.0 0.0 0:00.00 sh
	5598 root 20 0 736344 11648 10076 S 0.0 0.3 0:05.34 mesos-exec+
	........
	
#### 7.3.1.17 Completed Tasks ####
集群启动后完成的任务(不一定是成功，也有失败等状态)
![](http://i.imgur.com/51KI9Nw.png)

### 7.3.2 用json脚本开放一个nginx网络服务 ###
#### 7.3.2.1 首先创建一个Nginx的dockerfile ####

	vim Dockerfile
	FROM centos6-base
	MAINTAINER Siffre
	#install nginx
	RUN yum install -y nginx
	# forward request and error logs to docker log collector
	RUN ln -sf /dev/stdout /var/log/nginx/access.log
	RUN ln -sf /dev/stderr /var/log/nginx/error.log
	#off nginx daemon
	RUN echo "daemon off;" >> /etc/nginx/nginx.conf
	
#### 7.3.2.2 使用dockerfile生成docker镜像 ####
	生成docker
	docker build -t nginx-base  .	
	查看镜像
	docker images
	REPOSITORY TAG IMAGE ID CREATED VIRTUAL SIZE
	nginx-base latest c5dc79088bb8 41 hours ago 227.5 MB
	
#### 7.3.2.3 Json启动脚本 ####
	vi dataman-nginx-test.sh
	
	curl -v -X POST http://127.0.0.1:8080/v2/apps -H Content-Type:application/json -d \
	'{
	"id": "dataman-nginx-test",
	"cmd": "nginx",
	"cpus": 0.1,
	"mem": 128.0,
	"instances": 5,
	"container": {
	"type": "DOCKER",
	"docker": {
	"image": "nginx-base",
	"network": "BRIDGE",
	"portMappings": [
	{ "containerPort": 80, "hostPort": 0, "servicePort": 10000,
	"protocol": "tcp" }
	]
	}
	},
	"healthChecks": [
	{ "protocol": "HTTP",
	"portIndex": 0,
	"path": "/",
	"gracePeriodSeconds": 5,
	"intervalSeconds": 20,
	"maxConsecutiveFailures": 3 }
	]
	}'
	
参数说明:
	
	http://127.0.0.1:8080/v2/apps #Marathon地址
	id #任务名
	cmd #启动命令
	cpus #划分cpu资源
	mem #划分内存资源
	instances #实际运行任务总数量
	container #容器数据
	type #容器类型
	image #容器镜像名
	network #容器网络模式
	protMappings #容器端口设置
	containerPort #容器内部服务端口
	hostPort #容器映射到主机端口
	servicePort #一个辅助端口，用来做服务发现
	protocol #容器网络支持协议
	healthChecks #心跳检查设置
	protocol #检查协议
	portIndex #检查公共端口对应的服务，比如haproxy转发服务端口为80和443，第⼀一个80对应的索引
	就是0，第二个443对应的索引就是1
	path #检查地址
	gracePeriodSeconds #一次健康检查以后,marathon认为服务健康不检查的时间段
	intervalSeconds #检查间隔时间
	maxConsecutiveFailures #失败检查重试次数，过次数后认为不可用
	
#### 7.3.2.4 运行脚本生成任务 ####
	sh dataman-nginx-test.sh
	
	执行结果
	* Hostname was NOT found in DNS cache
	* Trying 127.0.0.1...
	* Connected to 127.0.0.1 (127.0.0.1) port 8080 (#0)
	> POST /v2/apps HTTP/1.1
	> User-Agent: curl/7.35.0
	> Host: 127.0.0.1:8080
	> Accept: */*
	> Content-Type:application/json
	> Content-Length: 1041
	> Expect: 100-continue
	>
	< HTTP/1.1 100 Continue
	< HTTP/1.1 201 Created
	< Cache-Control: no-cache, no-store, must-revalidate
	< Pragma: no-cache
	< Expires: 0
	< Location: http://127.0.0.1:8080/v2/apps/dataman-nginx-test
	< Content-Type: application/json
	< Transfer-Encoding: chunked
	* Server Jetty(8.y.z-SNAPSHOT) is not blacklisted
	< Server: Jetty(8.y.z-SNAPSHOT)
	<
	* Connection #0 to host 127.0.0.1 left intact
	{"id":"/dataman-nginx-test","cmd":"nginx","args":null,"user":null,"env":{},"instances":5,"cpus":
	0.1,"mem":128.0,"disk":0.0,"executor":"","constraints":[],"uris":[],"storeUrls":[],"ports":
	[0],"requirePorts":false,"backoffFactor":1.15,"container":{"type":"DOCKER","volumes":[],"docker":
	{"image":"ubuntu-nginx-base","network":"BRIDGE","portMappings":[{"containerPort":80,"hostPort":
	0,"servicePort":10000,"protocol":"tcp"}],"privileged":false,"parameters":
	[],"forcePullImage":false}},"healthChecks":[{"path":"/","protocol":"HTTP","portIndex":
	0,"command":null,"gracePeriodSeconds":5,"intervalSeconds":20,"timeoutSeconds":
	20,"maxConsecutiveFailures":3,"ignoreHttp1xx":false}],"dependencies":[],"upgradeStrategy":
	{"minimumHealthCapacity":1.0,"maximumOverCapacity":1.0},"labels":
	{},"version":"2015-07-01T10:37:19.979Z","deployments":[{"id":"ec0ccd2e-c5d9-4b07-87c9-
	e61cd411cdcd"}],"tasks":[],"tasksStaged":0,"tasksRunning":0,"tasksHealthy":0,"tasksUnhealthy":
	0,"backoffSeconds":1,"maxLaunchDelaySeconds":3600}
#### 7.3.2.5 检查 ####

**nginx-1**

这里需要注意的是因为配置了心跳监控，所以心跳监控的变成绿色了
#### 7.3.2.6 检查容器nginx网络服务 ####
**nginx-2**

点击这里会可以直接跳到nginx服务界面，说明服务正常

#### 7.3.2.7 bamboo设置 ####
**（1） bamboo主界面**

    进入bambooweb界面
    http://测试主机ip:8000/
	bamboo1

**（2）添加bamboo规则转发nginx**

	转发规则默认2种:目录转发和域名转发，本次测试使用目录格式，需要将nginx的web服务端口转发到haproxy 80端口的根目录。
	bamboo2

**（3）直接访问主机80端口**

    浏览器访问http://测试主机ip
	bamboo3
	
# 八、编译安装 #
# 九、故障处理 #
# 十、Docker化部署 #
# 十一、自动化部署 #