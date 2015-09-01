# 一、环境介绍 #

**单机环境，VPS云主机**

	[root@Siffre ~]# cat  /etc/redhat-release 
	CentOS Linux release 7.1.1503 (Core) 
	[root@Siffre ~]# uname -r
	3.10.0-123.el7.x86_64
	
# 二、安装Docker #

## 2.1 yum安装即可（默认支持） ##

	[root@Siffre ~]# yum  install  docker -y
	查看版本
	[root@Siffre ~]# docker -v
	Docker version 1.7.1, build 3043001/1.7.1
	
## 2.2 启动Docker ##
	
	service  docker  start
 	或
 	/etc/init.d/docker  start
 	
## 2.3查看状态 ##

    [root@Siffre ~]# service docker status
    Redirecting to /bin/systemctl status  docker.service
    docker.service - Docker Application Container Engine
       Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled)
       Active: active (running) since Tue 2015-09-01 05:14:56 EDT; 30s ago
     Docs: http://docs.docker.com
     Main PID: 12685 (docker)
       CGroup: /system.slice/docker.service
       ?..12685 /usr/bin/docker -d --selinu...
    
    Sep 01 05:14:45 Siffre docker[12685]: time="20...
    Sep 01 05:14:55 Siffre docker[12685]: time="20...
    Sep 01 05:14:55 Siffre docker[12685]: time="20...
    Sep 01 05:14:55 Siffre docker[12685]: time="20...
    Sep 01 05:14:56 Siffre docker[12685]: time="20...
    Sep 01 05:14:56 Siffre docker[12685]: time="20...
    Sep 01 05:14:56 Siffre docker[12685]: time="20...
    Sep 01 05:14:56 Siffre docker[12685]: time="20...
    Sep 01 05:14:56 Siffre systemd[1]: Started Doc...
    Sep 01 05:15:17 Siffre docker[12685]: time="20...
    Hint: Some lines were ellipsized, use -l to show in full.
    
# 三、服务安装 #

数人科技源：

	curl -o /etc/yum.repos.d/dataman.repo http://get.dataman.io/repos/centos/7/0/dataman.repo
	
官方源：

	wget http://www.apache.org/dist/mesos/0.23.0/mesos-0.23.0.tar.gz

官方Git源：

	git clone https://git-wip-us.apache.org/repos/asf/mesos.git
	
mesosphere源：

	rpm -Uvh http://repos.mesosphere.com/el/7/noarch/RPMS/mesosphere-el-repo-7-1.noarch.rpm
	
**以下根据官方和mesosphere的源安装：**

## 3.1 zookeeper ##

导入源：

    [root@Siffre ~]#  rpm -Uvh http://archive.cloudera.com/cdh4/one-click-install/redhat/6/x86_64/cloudera-cdh-4-0.x86_64.rpm

yum安装

	[root@Siffre ~]# yum  install  zookeeper zookeeper-server  -y
	

## 3.2 mesos ##

导入源：

	[root@Siffre ~]# rpm -Uvh http://repos.mesosphere.com/el/7/noarch/RPMS/mesosphere-el-repo-7-1.noarch.rpm
	
安装：

	[root@Siffre ~]# yum  install  mesos -y
	
## 3.3 marathon ##

导入源：

	[root@Siffre ~]# rpm -Uvh http://repos.mesosphere.com/el/7/noarch/RPMS/mesosphere-el-repo-7-1.noarch.rpm

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
    
# 四、配置服务参数 #

# 4.1 Mesos #

自身配置：

	#配置mesos在zk的使用目录
	echo "zk://10.3.10.29:2181/mesos" > "/etc/mesos/zk"  #zookeeper2181默认端口

### 4.1.1 Mesos-Master ###

    #指定master配置目录
    MESOS_MASTER_CONF_DIR="/etc/mesos-master"
    #指定master的主机名
    echo "211.149.226.81" > $MESOS_MASTER_CONF_DIR/hostname  
    
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
    echo "211.149.226.81" > $MESOS_SLAVE_CONF_DIR/hostname 
    #用IP表示#指定slave支持的容器类型
    echo "docker,mesos" > $MESOS_SLAVE_CONF_DIR/containerizers
    #指定slave的ip
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
    echo "zk://211.149.226.81:2181/marathon" > $MARATHON_CONF_DIR/zk
    #事件订阅模式
    echo "http_callback" > $MARATHON_CONF_DIR/event_subscriber
    #指定marathon主机名
    echo "211.149.226.81" > $MARATHON_CONF_DIR/hostname  
    #用IP表示#指定mesos在zk目录路径
    echo "zk://211.149.226.81:2181/mesos" > $MARATHON_CONF_DIR/master   
    
## 4.3 bamboo ##

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
    log /dev/loglocal0
    log /dev/loglocal1 notice
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
    
### 4.3.2 修改bamboo配置 ###
    
    [root@Siffre application]# vim bamboo/config/production.json
    {  "Marathon":
    {"Endpoint": "http://211.149.226.81:8080"  },  "Bamboo": {"Endpoint": "http://211.149.226.81:8000",
    "Zookeeper": { 
    "Host": "211.149.226.81:2181", 
    "Path": "/marathon-haproxy/state",  
    "ReportingDelay": 5}  }, 
    "HAProxy":
    {"TemplatePath": "/application/bamboo/config/haproxy_template.cfg",
    "OutputPath": "/etc/haproxy/haproxy.cfg",
    "ReloadCommand": "PIDS=`pidof haproxy`; haproxy -f /etc/haproxy/haproxy.cfg -p /var/run/haproxy.pid -sf $PIDS && while ps -p $PIDS; do sleep 0.2; done"  },  
    "StatsD": 
    {"Enabled": false,
    "Host": "211.149.226.81:8125",
    "Prefix": "bamboo-server.development."  }}
    
## 4.4 zookeeper ##

    初始化
    service  zookeeper-server  init  --myid=1 #单机环境可以不初始化
    提示：若不能更改添加 --force
    service  zookeeper-server  init  --myid=1 --force 
    

# 五、服务启动 #
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
    wget  --no-cookies --no-check-certificate --header "Cookie: oraclelicense=accept-securebackup-cookie" 
    
## 5.2 Mesos-Master ##

    # 启动命令
    service  mesos-master  start
    #查看进程状态
    ps axuf | grep mesos-master | grep -v grep
    #查看日志
    tail -f /var/log/mesos/mesos-master.log   

## 5.3 Mesos-Slave ##

    # 启动命令
    service  mesos-slave  start
    #查看进程状态
    ps axuf | grep mesos-slave | grep -v grep
  
## 5.4 marathon ##

    # 启动命令
    service  marathon  start
    #查看进程状态
    ps axuf | grep marathon | grep -v grep
  
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
    

