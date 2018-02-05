## 1 jdk1.8安装
1.    **下载jdk1.8**
2.    **解压缩到相应目录 centos 默认安装在/user 目录下 如果没有自行创建文件夹，版本号对应**

```
# 解压缩安装文件
tar -zxvf jdk-8-linux-x64.tar.gz

# 把文件目录拷贝到/usr下
mv jdk1.8.0_161 /usr/java
```
3.   **引入PATH以及JAVA_HOME环境变量**


```
# 编辑/etc/profile文件
vi /etc/profile

# 在末尾添加下面两行
export JAVA_HOME=/usr/java/jdk1.8.0_161
export PATH=$PATH:$JAVA_HOME/bin

# 当前配置手动触发生效
source /etc/profile
```

编辑完后，你就可以看到JAVA_HOME的变量了


```
echo $JAVA_HOME
```

4.   **替换OpenJDK**
  
   上面的步骤完成，如果你执行java -version，就会发现版本还是不对。（如果没有问题，可以忽略这个步骤）
因为/usr/bin下的java其实是默认链接到openjdk的。因此:

```
# 以root身份，进入/usr/bin目录
cd /usr/bin

# 备份原有的链接
mv java java.bak

# 创建新的链接
ln -s /usr/java/jdk1.8.0_161/bin/java java

# 此时执行java -version就可以看到效果了
[root@localhost bin]# ll java
lrwxrwxrwx. 1 root root 27 Aug 16 10:15 java -> /usr/java/jdk1.8.0/bin/java
[root@localhost bin]# java -version
java version "1.8.0_161"
Java(TM) SE Runtime Environment (build 1.8.0-b132)
Java HotSpot(TM) 64-Bit Server VM (build 25.0-b70, mixed mode)
```
## 1 mongo3.0.5集群搭建


1. 首先分别在三台机器上下载好mongodb安装包mongodb-linux-x86_64-rhel70-3.0.5.tgz

2. 使用tar命令解压安装包然后修改解压后的目录名

```
   tar zxvf  mongodb-linux-x86_64-rhel70-3.4.2.tgz

   mv mongodb-linux-x86_64-rhel70-3.4.2 mongodb
```
3.进入 mongodb目录中新建三个目录conf、logs 、db 
conf存储配置文件目录，logs用来存储日志目录，db用来存储数据目录


```
  cd conf
  touch mongodb.conf
```
4.编写配置文件mongodb.conf,内容如下 
其中dbpath是数据库文件目录，logpath是日志目录，port是mongodb所占用的端口，fork是true的时候表示在后台启动


```
dbpath=/data/mongodb/db
logpath=/data/mongodb/log/mongodb.log
port=27017
fork=true
```
注意：此处的路径可以是mongo安装包的文件夹，或者另起文件夹建立相应的文件夹

#### mongodb.conf 详细配置


```
--logpath 日志文件路径
--master 指定为主机器
--slave 指定为从机器
--source 指定主机器的IP地址
--pologSize 指定日志文件大小不超过64M.因为resync是非常操作量大且耗时，最好通过设置一个足够大的oplogSize来避免resync(默认的 oplog大小是空闲磁盘大小的5%)。
--logappend 日志文件末尾添加
--port 启用端口号
--fork 在后台运行
--only 指定只复制哪一个数据库
--slavedelay 指从复制检测的时间间隔
--auth 是否需要验证权限登录(用户名和密码)
 
 
-h [ --help ]             show this usage information
--version                 show version information
-f [ --config ] arg       configuration file specifying additional options
--port arg                specify port number
--bind_ip arg             local ip address to bind listener - all local ips
                           bound by default
-v [ --verbose ]          be more verbose (include multiple times for more
                           verbosity e.g. -vvvvv)
--dbpath arg (=/data/db/) directory for datafiles    指定数据存放目录
--quiet                   quieter output   静默模式
--logpath arg             file to send all output to instead of stdout   指定日志存放目录
--logappend               appnd to logpath instead of over-writing 指定日志是以追加还是以覆盖的方式写入日志文件
--fork                    fork server process   以创建子进程的方式运行
--cpu                     periodically show cpu and iowait utilization 周期性的显示cpu和io的使用情况
--noauth                  run without security 无认证模式运行
--auth                    run with security 认证模式运行
--objcheck                inspect client data for validity on receipt 检查客户端输入数据的有效性检查
--quota                   enable db quota management   开始数据库配额的管理
--quotaFiles arg          number of files allower per db, requires --quota 规定每个数据库允许的文件数
--appsrvpath arg          root directory for the babble app server 
--nocursors               diagnostic/debugging option 调试诊断选项
--nohints                 ignore query hints 忽略查询命中率
--nohttpinterface         disable http interface 关闭http接口，默认是28017
--noscripting             disable scripting engine 关闭脚本引擎
--noprealloc              disable data file preallocation 关闭数据库文件大小预分配
--smallfiles              use a smaller default file size 使用较小的默认文件大小
--nssize arg (=16)        .ns file size (in MB) for new databases 新数据库ns文件的默认大小
--diaglog arg             0=off 1=W 2=R 3=both 7=W+some reads 提供的方式，是只读，只写，还是读写都行，还是主要写+部分的读模式
--sysinfo                 print some diagnostic system information 打印系统诊断信息
--upgrade                 upgrade db if needed 如果需要就更新数据库
--repair                  run repair on all dbs 修复所有的数据库
--notablescan             do not allow table scans 不运行表扫描
--syncdelay arg (=60)     seconds between disk syncs (0 for never) 系统同步刷新磁盘的时间，默认是60s
 
Replication options:
--master              master mode 主复制模式
--slave               slave mode 从复制模式
--source arg          when slave: specify master as <server:port> 当为从时，指定主的地址和端口
--only arg            when slave: specify a single database to replicate 当为从时，指定需要从主复制的单一库
--pairwith arg        address of server to pair with
--arbiter arg         address of arbiter server 仲裁服务器，在主主中和pair中用到
--autoresync          automatically resync if slave data is stale 自动同步从的数据
--oplogSize arg       size limit (in MB) for op log 指定操作日志的大小
--opIdMem arg         size limit (in bytes) for in memory storage of op ids指定存储操作日志的内存大小
 
Sharding options:
--configsvr           declare this is a config db of a cluster 指定shard中的配置服务器
--shardsvr            declare this is a shard db of a cluster 指定shard服务器
```


5.分别在三台机器上启动mongodb 
其中–replSet表示副本集群参数 ，mongoTest是副本集名称，这里的名字可以任意取,另外两台机也要和这个一样


```
/data/mongodb/bin/mongod –f /data/mongodb/conf/mongodb.conf  --replSet mongoTest
```
注意：如果没有放在环境变量则用 ./mongod 启动

如果启动成功会看到类似下面的提示

```
about to fork child process, waiting until server is ready for connections.
forked process: 15398
child process started successfully, parent exiting
```

### 6. 配置mongodb副本集

1. 进入其中一台机器的mongo shell操作

```
/data/mongodb/bin/mongo -port 27017
```
2. 切换到admin

```
use admin
```
3. mongo副本配置

```
config={_id:"mongoTest",members:[{_id:0,host:"xxx.xxx.xxx.11:27017"},{_id:1,host:"xxx.xxx.xxx.12:27017"},{_id:2,host:"xxx.xxx.xxx.13:27017"}]}

rs.initiate(config)
```

4. 查看状态

```
rs.status()
```
==部署中可能遇到的问题==

5. 设置mongo开机自启动

将mongodb启动项目追加入rc.local保证mongodb在服务器开机时启动（此处不需要进行密码验证）
```
echo "/opt/mongo/mongodb/bin/mongod --dbpath=/data/mongodb/db –logpath=/data/mongodb/log –logappend  –port=27017" >> /etc/rc.local
```

6.生产环境权限控制

1.创建第一个用户

在上面部署成功的集群上执行以下步骤，在数据库admin中创建第一个具有最高root权限的用户root:

```
use admin
db.createUser({user : "root", pwd : "q,.wemr213oiz923*(*LNY", roles : [{role : "root", db : "admin"}]})
```
2.关闭所有上面部署的节点，可以用

```
db.shutdownServer()
```
3.产生keyFile，并复制到每个运行集群节点的服务器上。

```
openssl rand -base64 741 > mongodb-keyfile
chmod 600 mongodb-keyfile
```
4.在每个节点的配置文件中加上选项

```
keyFile = <key_file_path>
```

5.在出mongos外的所有节点的配置文件中加上选项

```
auth = true
```
6.重启所有节点，到此权限认证已经搞完了，现在就可以插入数据库，并按需求添加用户，赋予相应的权限。进行认证授权的函数为db.auth(), 例如：

```
db.auth("root", "<password>")
```


#### 1.防火墙问题

错误信息如下

```
{  
    "ok" : 0,  
    "errmsg" : "replSetInitiate quorum check failed because not all proposed set members responded affirmatively: 192.168.3.221:27017 failed with Failed attempt to connect to 192.168.3.221:27017; couldn't connect to server 192.168.3.221:27017 (192.168.3.221), connection attempt failed",  
    "code" : 74  
}  
```
解决方法：
分别关闭3台集群的防火墙

1. iptables -nvL 查看防火墙
2. iptables -F
3. service iptables save
4. service iptables stop
5. 重启mongo服务，重新按照上面副本启动，看到如下信息则表示集群创建成功

```
{ "ok" : 1 }
```
## 3 solr 5.3.0集群搭建

1 服务器环境搭建
 配置主机名和IP映射，在3台服务器的/etc/hosts文件中添加以下几行：
 
```
xxx.xxx.xxx..11 solr-cloud-master
xxx.xxx.xxx..12 solr-cloud-slave1
xxx.xxx.xxx..13 solr-cloud-slave2
```
2.将文件同步到其它机器上

```
scp /etc/hosts  solr-cloud-slave1:/etc/
scp /etc/hosts  solr-cloud-slave2:/etc/
```

3.**安装配置Zookeeper**
1. 解压 zookeeper-3.4.11.tar.gz  到 /opt/SolrCloud/Zookeeper/ 目录，并修改目录所有者
2. 配置zookeper

```
cd /opt/SolrCloud/Zookeeper/zookeeper-3.4.11/conf
cp zoo_sample.cfg zoo.cfg
```
3.修改zoo.cfg文件

```
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
dataDir=/data/zookeeper/data
dataLogDir=/data/zookeeper/logs
# the port at which the clients will connect
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the 
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1
server.1=xxx.xxx.xxx..11:2888:3888
server.2=xxx.xxx.xxx..12:2888:3888
server.3=xxx.xxx.xxx..13:2888:3888
```

4.同步zoo.cfg文件到其他从服务器上，注意其他从服务需要先创建对应的数据和日志文件夹

```
scp -r /opt/SolrCloud/Zookeeper/zookeeper-3.4.11/  solr-cloud-slave1:/opt/SolrCloud/Zookeeper/
scp -r /data/zookeeper/  solr-cloud-slave1:/data/

scp -r /opt/SolrCloud/Zookeeper/zookeeper-3.4.11/  solr-cloud-slave2:/opt/SolrCloud/Zookeeper/
scp -r /data/zookeeper/  solr-cloud-slave2:/data/
```
5.创建myid文件，并写入zoo.cfg中对于的Server ID：

```
master

echo "1" > /data/zookeeper/data/myid
cat /data/zookeeper/data/myid

slaver1

echo "2" > /data/zookeeper/data/myid
cat /data/zookeeper/data/myid

slaver2

echo "3" > /data/zookeeper/data/myid
cat /data/zookeeper/data/myid
```
6.分别启动三台Zookeeper服务器：

```
cd /opt/SolrCloud/Zookeeper/zookeeper-3.4.11/bin/
./zkServer.sh  start 
```
7.查看Zookeeper状态

./zkServer.sh  status


4.**SolrCloud分布式集群搭建**(内置jetty版本)

1.解压solr 到/opt/SolrCloud

2.修改bin下solr.in.sh文件


```
ZK_HOST="xxx.xxx.xxx.11:2181,xxx.xxx.xxx.12:2181,xxx.xxx.xxx.13:2181"

 　　　　去掉 ZK_CLIENT_TIMEOUT 的注释
```

3.复制Solr 到2个从节点

```
scp -r /opt/SolrCloud/Solr/  solr-cloud-slave1:/opt/SolrCloud/Solr/
scp -r /opt/SolrCloud/Solr/  solr-cloud-slave2:/opt/SolrCloud/Solr/
```
4. 创建solrhome以及配置文件

```
mkdir /opt/solrhome
mkdir /opt/solrhome/appclient_core
cp -r /data/solr-5.5.3/example/example-DIH/solr/solr/conf  /data/solrhome/appclient_core
```
cp 步骤可以复制完好的配置文件

5. 修改solrconfig.xml 和solrcheml.xml 文件

6.上传配置文件到zookeeper(在/data/solr-.5.3/server/scripts/cloud-scripts/zkcli.sh  同样可以上传)

```
./zkcli.sh -zkhost  xxx.xxx.xxx.11:2181,xxx.xxx.xxx.12:2181,xxx.xxx.xxx.13:2181 -cmd upconfig -confdir /opt/solrhome/appclient_core/ -confname appclient_core
```
7.zookeeper集群操作上传的文件


```
进入zookeeper的bin下

　　　　　　./zkCli.sh  连接zookeeper集群

　　　　　　ls /configs/appclient_core    查看上传的配置文件    

　　　　　　delete /configs/appclient_core/solrconfig.xml   删除文件

　　　　　　delete /configs/appclient_core    删除空文件夹

　　　　　　get  /configs/appclient_core/solrconfig.xml   查看文件内容

　　　　　　rmr /configs/appclient_core  递归删除（慎重使用）
```
8.启动solr集群

```
./bin/solr restart 
```
9.创建collection 

```
http://xxx.xxx.xxx.11:8983/solr/admin/collections?action=CREATE&name=appclient_core&numShards=3&replicationFactor=3&maxShardsPerNode=3&collection.configName=appclient_core&collection=appclient
```
==注意点==

#### 1. 修改solr数据存放的路径

建立指定文件夹例如solrhome,修改solr.in.sh文件， 放开SOLR_HOME
```
SOLR_HOME="/opt/solrhome"
```
由于通过zookeper集群管理，上传配置文件未上传solr.xml文件，这个文件夹要放solr.xml文件，位置在solr/server/solr 文件下

solr.in.sh文件可以配置虚拟机参数，jetty启动的日志文件位置存放路径，根据需求配置


==zookeper 管理solr集群原理 :zookeper 的数据会统一放在特定文件夹,任意节点数据刷新会同步到别的节点,solr的配置文件可配置solr_home文件夹，在web.xml 中配置读取solr配置文件，将solr_home文件夹定义问zookeper的数据存放文件夹，当在任意solr集群中创建文档，配置文件会通过zookeper同步到其他节点，solr集群会同步到配置文件，自动创建相应的分片和副本。==

## 4 redis 搭建

1.安装gcc 环境  否则解压后无法进行编译

```
yum  install  gcc
```
2.解压缩

3.进入redis文件夹 进行编译

```
cd redis-3.0.2
make
```
4.进入src文件夹 进行安装


```
make install
```
5.在redis-3.0.2 文件夹建立bin 和etc 文件夹，方便管理redis


```
cd redis-3.0.2
mkdir bin etc
mv redis.conf /opt/redis-3.0.0/etc
cd src
mv mkreleasdhdr.sh redis-benchmark redis-check-aof redis-check-dump redis-cli redis-server /opt/redis-3.0.2/bin
```

6.修改redis.conf 文件，让以守护进程启动

daemonize 改问yes
bind 127.0.0.1 改为0.0.0.0否则外部无法连接redis只能本机访问
可以单独设置redis防火墙，这里不做累述
```
vi redis.conf

```
7.启动redis服务
进入bin文件夹

```
redis-server /opt/redis-3.0.2/etc/redis.conf
```
8.redis 安装涉及的一些命令


```
redis-server /usr..../redis.conf 启动redis服务，并指定配置文件
redis-cli 启动redis 客户端
pkill redis-server 关闭redis服务
redis-cli shutdown 关闭redis客户端
netstat -tunpl|grep 6379 查看redis 默认端口号6379占用情况


```
## 5 tomcat 日志切割cronolog

1.下载1.6.2版本的cronolog解压到文件夹


```
# tar zxvf cronolog-1.6.2.tar.gz 
```


2.进入下载文件夹(如果眉头gcc 库请先用yum安装，百度)


```
# ./configure 
# make 
# make install 
```
3.查看cronolog安装后所在目录（验证安装是否成功）


```
which cronolog 
一般情况下显示为：/usr/local/sbin/cronolog
```
进入此文件夹

```
cp cronolog /user/bin
```

4.配置catalina.sh 文件，384行

```
  shift
    "$_RUNJAVA" "$LOGGING_CONFIG" $LOGGING_MANAGER $JAVA_OPTS $CATALINA_OPTS \
      -Djava.endorsed.dirs="$JAVA_ENDORSED_DIRS" -classpath "$CLASSPATH" \
      -Djava.security.manager \
      -Djava.security.policy=="$CATALINA_BASE"/conf/catalina.policy \
      -Dcatalina.base="$CATALINA_BASE" \
      -Dcatalina.home="$CATALINA_HOME" \
      -Djava.io.tmpdir="$CATALINA_TMPDIR" \
      org.apache.catalina.startup.Bootstrap "$@" start 2>&1 \
      | /usr/local/sbin/cronolog "$CATALINA_OUT" >> /dev/null &

  else
    "$_RUNJAVA" "$LOGGING_CONFIG" $LOGGING_MANAGER $JAVA_OPTS $CATALINA_OPTS \
      -Djava.endorsed.dirs="$JAVA_ENDORSED_DIRS" -classpath "$CLASSPATH" \
      -Dcatalina.base="$CATALINA_BASE" \
      -Dcatalina.home="$CATALINA_HOME" \
      -Djava.io.tmpdir="$CATALINA_TMPDIR" \
      org.apache.catalina.startup.Bootstrap "$@" start 2>&1 \
      | /usr/bin/cronolog "$CATALINA_OUT" >> /dev/null &
```

## 6 ngnix负载均衡安装和配置

1.进去opt文件夹 安装PCRE库


```
cd /opt
wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.39.tar.gz 
tar -zxvf pcre-8.37.tar.gz
cd pcre-8.34
./configure
make
make install
```
如make出错，安装g++开发库


```
yum install gcc gcc-c++
```

2. 安装zlib库

```
cd /opt
wget http://zlib.net/zlib-1.2.11.tar.gz
tar -zxvf zlib-1.2.11.tar.gz
cd zlib-1.2.11
./configure
make
make install
```
3. 安装openssl（某些vps默认没装ssl)


```
cd /opt
wget https://www.openssl.org/source/openssl-1.0.1t.tar.gz
tar -zxvf openssl-1.0.1t.tar.gz
```
4. 安装nginx



```
cd /opt
wget http://nginx.org/download/nginx-1.1.10.tar.gz
tar -zxvf nginx-1.1.10.tar.gz
cd nginx-1.1.10
./configure
make
make install
```
 
5.此时如果没有可执行文件，可进入/user/local/sbin/nginx 复制一个过来


```
cp nginx /opt/nginx-1.1.10/
```

==6.如果运行还出租错==

```

1
# ldd $(which /usr/local/nginx/sbin/nginx)
2
    libpthread.so.0 => /lib64/libpthread.so.0 (0x00000030e8400000)
3
    libcrypt.so.1 => /lib64/libcrypt.so.1 (0x00000030f9a00000)
4
    libpcre.so.1 => not found
5
    libcrypto.so.6 => /lib64/libcrypto.so.6 (0x00000030f2a00000)
6
    libz.so.1 => /lib64/libz.so.1 (0x00000030e8800000)
7
    libc.so.6 => /lib64/libc.so.6 (0x00000030e7800000)
8
    /lib64/ld-linux-x86-64.so.2 (0x00000030e7400000)
9
    libdl.so.2 => /lib64/libdl.so.2 (0x00000030e8000000)

```
应该是报库为找到
查看结果显示 ： libpcre.so.1 => not found ，同时注意lib库的路径，有/lib/* 和 /lib64/* 之分。

比如上面的是 /lib64/*,这个和下面解决问题时创建的软连接有关系

解决方案

1、首先确认已经安装好pcre 软件（nginx 依赖该软件）

2、创建软连接

对于/lib/* 32位系统来说：

1查看lib库
```
ls /lib/ |grep pcre
libpcre.so.0
libpcre.so.0.0.1
```
2添加软连接

```
# ln -s /lib/libpcre.so.0.0.1 /lib/libpcre.so.1
```
ps: 也有可能 pcre lib文件在目录：/usr/local/lib/

对于/lib64/* 64位系统来说：
1 查看lib库
```
ls /lib64/ |grep pcre
libpcre.so.0
libpcre.so.0.0.1
```
2 添加软连接

```
# ln -s /lib64/libpcre.so.0.0.1 /lib64/libpcre.so.1
```
ps: 也有可能 pcre lib文件在目录：/usr/local/lib64/。


7.nginx 一些命令

1、验证nginx配置文件是否正确

进入nginx安装目录sbin下，输入命令./nginx -t

看到如下显示nginx.conf syntax is ok

nginx.conf test is successful

说明配置文件正确！

2.重启nginx

进入nginx可执行目录sbin下，输入命令./nginx -s reload 即可


#### 8.nginx负载均衡配置
部分配置解决session共享，偷懒方式用ip_哈希算法

每个请求按访问ip的hash结果分配，这样每个访客固定访问一个应用服务器，可以解决session共享的问题。

或者随机分配，用redis中间件解决。


```
		
	upstream  solr {   
    server    xxx.xxx.xxx.11:8983;
    server    xxx.xxx.xxx.12:8983;
	server    xxx.xxx.xxx..13:8983;
	}    
	
	
	upstream  api-service {   
    server    xxx.xxx.xxx..15:8002;  
    server    xxx.xxx.xxx..16:8002;
	}

	
	upstream  sdkweb {   
    server   xxx.xxx.xxx..15:8001;  
    server   xxx.xxx.xxx..16:8001;
    ip_hash;  
	}
	
	
	

		
		location /solr {
            proxy_pass http://solr;
			proxy_redirect default;
        }
		
		location /api-service {
            proxy_pass http://api-service;
			proxy_redirect default;
        }
		
		location /sdk-web {
            proxy_pass http://sdkweb/sdk-web;
			proxy_redirect default;
        }
```

##### 服务器介绍


```
xxx.xxx.xxx.11  solr / mongo 集群A
xxx.xxx.xxx.12  solr / mongo 集群B
xxx.xxx.xxx.13  solr / mongo 集群C
xxx.xxx.xxx.14  redis 缓存
xxx.xxx.xxx.15  api服务A
xxx.xxx.xxx.16  api服务B
xxx.xxx.xxx.17  前置网关，负载均衡
```

搭建中的一些linux 问题，如缺少相应的库，自行百度解决.