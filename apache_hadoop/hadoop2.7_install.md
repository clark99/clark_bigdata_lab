# 1 准备工作
## 1.1下载安装包
- hadoop
`wget http://mirrors.cnnic.cn/apache/hadoop/common/hadoop-2.7.3/hadoop-2.7.3.tar.gz`
- jdk1.8.0_121
- 下载mysql
`wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-community-server-5.7.17-1.el6.x86_64.rpm --no-check-certificate`
- 下载hive
`wget http://mirrors.cnnic.cn/apache/hive/hive-2.1.1/apache-hive-2.1.1-bin.tar.gz`
`wget http://mirrors.cnnic.cn/apache/hive/hive-2.1.1/apache-hive-2.1.1-src.tar.gz`
- 下载zookeeper
`wget https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/zookeeper-3.5.2-alpha/zookeeper-3.5.2-alpha.tar.gz   --no-check-certificate`
- 下载hbase
`wget https://mirrors.tuna.tsinghua.edu.cn/apache/hbase/1.3.0/hbase-1.3.0-bin.tar.gz --no-check-certificate`
- 下载storm
`wget https://mirrors.cnnic.cn/apache/storm/apache-storm-1.1.0/apache-storm-1.1.0.tar.gz --no-check-certificate`
- 下载sqoop
`wget https://mirrors.cnnic.cn/apache/sqoop/1.99.7/sqoop-1.99.7-bin-hadoop200.tar.gz --no-check-certificate`
- 下载spark
`wget https://mirrors.cnnic.cn/apache/spark/spark-2.1.0/spark-2.1.0-bin-hadoop2.7.tgz --no-check-certificate`
`wget https://mirrors.cnnic.cn/apache/spark/spark-2.1.0/spark-2.1.0-bin-without-hadoop.tgz --no-check-certificate`
- 下载kafka
`https://mirrors.cnnic.cn/apache/kafka/0.10.1.1/kafka_2.11-0.10.1.1.tgz --no-check-certificate`
- 下载flume
`wget https://mirrors.cnnic.cn/apache/flume/1.7.0/apache-flume-1.7.0-bin.tar.gz --no-check-certificate`

## 1.2 准备四个节点：
- node01,node02,node03,node04,而且已经配置了ssh互信，而且已经关闭了防火墙
- 应用解压安装目录为/app
- HDFS目录为：/app/dirhadoop/

## 1.3 配置环境变量
所有节点执行
```shell
echo 'export JAVA_HOME=/app/jdk1.8.0_121' >> /etc/profile
echo 'export CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar' >> /etc/profile
echo 'export HADOOP_HOME=/app/hadoop-2.7.3' >>/etc/profile
echo 'export PATH=$PATH:$HADOOP_HOME/bin:$JAVA_HOME/bin:$JAVA_HOME/jre/bin' >> /etc/profile
source /etc/profile
```
## 1.4 创建HDFS使用的目录
每个节点都要执行
```shell
#core-site.xml的hadoop.tmp.dir
mkdir -p /app/dirhadoop/tmp
#hdfs-site.xml的dfs.namenode.name.dir
mkdir -p /app/dirhadoop/dfs/name
#hdfs-site.xml的dfs.datanode.data.dir
mkdir -p /app/dirhadoop/dfs/data
```

# 2. hadoop-2.7的配置文件
**先配置一个节点的配置文件，然后scp到其他节点**
## 2.1 配置hadoop-env.sh
```shell
echo 'export JAVA_HOME=/app/jdk1.8.0_121'  >>/app/hadoop-2.7.3/etc/hadoop/hadoop-env.sh
```
## 2.2 配置core-site.xml
### 2.2.1 参数解释
- hadoop.tmp.dir定义了hdfs的临时文件存放目录
- fs.default.name指定了如何访问HDFS，例如hbase设置为：
```java
    <name>hbase.rootdir</name>
    <value>hdfs://node01:9000/hbase</value> 
```
### 2.2.2 配置文件
**打开/app/hadoop-2.7.3/etc/hadoop/core-site.xml**
```java
<configuration>  
  <property>  
    <name>fs.default.name</name>  
    <value>hdfs://node01:9000</value>  
  </property>  
  <property>  
    <name>hadoop.tmp.dir</name>  
    <value>/app/dirhadoop/tmp</value>  
  </property>  
</configuration> 
```
## 2.3 配置hdfs-site.xml
重要的参数:
- dfs.namenode.name.dir
- dfs.datanode.data.dir
```java
<configuration>  
  <property>  
    <name>dfs.replication</name>  
    <value>2</value>  
  </property>  
  <property>  
    <name>dfs.namenode.name.dir</name>  
    <value>file:/app/dirhadoop/dfs/name</value>  
  </property>  
  <property>  
    <name>dfs.datanode.data.dir</name>  
    <value>file:/app/dirhadoop/dfs/data</value>  
  </property>  
</configuration> 
```
## 2.4 配置mapred-site.xml
重要参数:
- mapreduce.jobhistory.webapp.address 定义了访问网址
- 部署完成之后，可以通过sbin/mr-jobhistory-daemon.sh start historyserver
  启动mapreduce.jobhistory.webapp
**打开/app/hadoop-2.7.3/etc/hadoop/mapred-site.xml**
```java
<configuration>  
  <property>  
    <name>mapreduce.framework.name</name>  
    <value>yarn</value>  
  </property>  
  <property>  
    <name>mapreduce.jobhistory.address</name>  
    <value>node01:10020</value>  
  </property>  
  <property>  
    <name>mapreduce.jobhistory.webapp.address</name>  
    <value>node01:19888</value>  
  </property>  
</configuration> 
```

## 2.5 配置yarn-site.xml
### 2.5.1 配置yarn的slaves
```shell
cat /app/hadoop-2.7.3/etc/hadoop/slaves
node02
node03
node04
```
### 2.5.2 配置文件的重要参数
- a.yarn.resourcemanager.webapp.address定义网页访问：
- /app/hadoop-2.7.3/sbin/start-yarn.sh启动之后，就可以类似
  http://IP.13.129:8088/cluster访问了
  
**打开/app/hadoop-2.7.3/etc/hadoop/yarn-site.xml**
```java
<configuration>   
<!-- Site specific YARN configuration properties -->  
  <property>  
    <name>yarn.nodemanager.aux-services</name>  
    <value>mapreduce_shuffle</value>  
  </property>  
  <property>  
    <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>  
    <value>org.apache.hadoop.mapred.ShuffleHandler</value>  
  </property>  
  <property>  
    <name>yarn.resourcemanager.address</name>  
    <value>node01:8032</value>  
  </property>  
  <property>  
    <name>yarn.resourcemanager.scheduler.address</name>  
    <value>node01:8030</value>  
  </property>  
  <property>  
    <name>yarn.resourcemanager.resource-tracker.address</name>  
    <value>node01:8031</value>  
  </property>  
  <property>  
    <name>yarn.resourcemanager.admin.address</name>  
    <value>node01:8033</value>  
  </property>  
  <property>  
    <name>yarn.resourcemanager.webapp.address</name>  
    <value>node01:8088</value>  
  </property>  
</configuration> 
```
## 2.6 配置同步到其他节点
节点配置完成之后,安装包同步到其他节点
```shell
scp -r /app/hadoop-2.7.3 node02:/app/
scp -r /app/hadoop-2.7.3 node03:/app/
scp -r /app/hadoop-2.7.3 node04:/app/
```

# 3. 启动hdfs,yarn
## 3.1 在master节点(node01)格式化hdfs
```shell
hdfs namenode -format
```
## 3.2 格式化完成之后,master节点启动HDFS
### 3.2.1 启动hdfs:
```shell
[root@node01 ~]# /app/hadoop-2.7.3/sbin/start-dfs.sh
Starting namenodes on [node01]
node01: Warning: Permanently added the RSA host key for IP address 'IP' to the list of known hosts.
node01: starting namenode, logging to /app/hadoop-2.7.3/logs/hadoop-root-namenode-node01.out
node02: starting datanode, logging to /app/hadoop-2.7.3/logs/hadoop-root-datanode-node02.out
node03: starting datanode, logging to /app/hadoop-2.7.3/logs/hadoop-root-datanode-node03.out
node04: starting datanode, logging to /app/hadoop-2.7.3/logs/hadoop-root-datanode-node04.out
Starting secondary namenodes [0.0.0.0]
0.0.0.0: starting secondarynamenode, logging to /app/hadoop-2.7.3/logs/hadoop-root-secondarynamenode-node01.out
```
网页查看：http://IP:50070/dfshealth.html#tab-overview

### 3.2.2 启动yarn
```shell
[root@node01 ~]# /app/hadoop-2.7.3/sbin/start-yarn.sh
starting yarn daemons
starting resourcemanager, logging to /app/hadoop-2.7.3/logs/yarn-root-resourcemanager-node01.out
node03: starting nodemanager, logging to /app/hadoop-2.7.3/logs/yarn-root-nodemanager-node03.out
node02: starting nodemanager, logging to /app/hadoop-2.7.3/logs/yarn-root-nodemanager-node02.out
node04: starting nodemanager, logging to /app/hadoop-2.7.3/logs/yarn-root-nodemanager-node04.out
```
网页查看：http://IP:8088/cluster/nodes

### 3.2.3 启动historyserver
```shell
sbin/mr-jobhistory-daemon.sh start historyserver
```
网页查看: http://IP:19888/jobhistory

# 4.安装Zookeeper
## 4.1 单节点创建文件myid
```shell
[root@node01 zookeeper-3.5.2-alpha]# cat myid
node01	1
node02	2
node03	3
node04	4
```

## 4.2 单个节点配置zoo.cfg
```shell
#[root@node01 conf]# cat zoo_sample.cfg
#vi zoo.cfg
tickTime=2000
dataDir=/app/zookeeper-3.5.2-alpha
clientPort=2181
initLimit=5
syncLimit=2
server.1=node01:2888:3888 
server.2=node02:2888:3888
server.3=node03:2888:3888
server.4=node04:2888:3888
#server.1,server.2,server.3,server.4和myid相匹配
```

注:新版的配置文件为:
```shell
[root@node01 conf]# vi zoo.cfg.dynamic.100000000 
[root@node01 conf]# cat zoo.cfg
initLimit=5
syncLimit=2
clientPort=2181
tickTime=2000
dataDir=/app/zookeeper-3.5.2-alpha
dynamicConfigFile=/app/zookeeper-3.5.2-alpha/conf/zoo.cfg.dynamic.100000000
[root@node01 conf]# cat zoo.cfg.dynamic.100000000
server.1=node01:2888:3888:participant
server.2=node02:2888:3888:participant
server.3=node03:2888:3888:participant
server.4=node04:2888:3888:participant
```


## 4.3 Zookeeper复制到其他所有节点：
```shell
scp -r /app/zookeeper-3.5.2-alpha node02：/app
scp -r /app/zookeeper-3.5.2-alpha node03：/app
scp -r /app/zookeeper-3.5.2-alpha node04：/app
```
## 4.4 启动zk
每个节点执行：
```shell
/app/zookeeper-3.5.2-alpha/bin/zkServer.sh start
ZooKeeper JMX enabled by default
Using config: /app/zookeeper-3.5.2-alpha/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
```
## 4.4 查看zk状态
```shell
/app/zookeeper-3.5.2-alpha/bin/zkServer.sh status
Mode: leader
Mode: follower
```
## 4.5 报错处理：
```java
2017-04-07 15:47:22,297 [myid:] - INFO  [main:QuorumPeerConfig@116] - Reading configuration from: /app/zookeeper-3.5.2-alpha/bin/../conf/zoo.cfg
2017-04-07 15:47:22,304 [myid:] - INFO  [main:QuorumPeerConfig@318] - clientPortAddress is 0.0.0.0/0.0.0.0:2181
2017-04-07 15:47:22,304 [myid:] - INFO  [main:QuorumPeerConfig@322] - secureClientPort is not set
2017-04-07 15:47:22,311 [myid:] - ERROR [main:QuorumPeerMain@86] - Invalid config, exiting abnormally
org.apache.zookeeper.server.quorum.QuorumPeerConfig$ConfigException: Address unresolved: node01:3888 
	at org.apache.zookeeper.server.quorum.QuorumPeer$QuorumServer.<init>(QuorumPeer.java:242)
	at org.apache.zookeeper.server.quorum.flexible.QuorumMaj.<init>(QuorumMaj.java:89)
	at org.apache.zookeeper.server.quorum.QuorumPeerConfig.createQuorumVerifier(QuorumPeerConfig.java:524)
	at org.apache.zookeeper.server.quorum.QuorumPeerConfig.parseDynamicConfig(QuorumPeerConfig.java:557)
	at org.apache.zookeeper.server.quorum.QuorumPeerConfig.setupQuorumPeerConfig(QuorumPeerConfig.java:530)
	at org.apache.zookeeper.server.quorum.QuorumPeerConfig.parseProperties(QuorumPeerConfig.java:353)
	at org.apache.zookeeper.server.quorum.QuorumPeerConfig.parse(QuorumPeerConfig.java:133)
	at org.apache.zookeeper.server.quorum.QuorumPeerMain.initializeAndRun(QuorumPeerMain.java:110)
	at org.apache.zookeeper.server.quorum.QuorumPeerMain.main(QuorumPeerMain.java:79)
```
分析：这个报错原因是zoo.cfg配置文件，server.1=node01:2888:3888后面有空格导致

# 5.安装HBASE

## 5.1 每个节点执行环境变量：
```shell
echo 'export HBASE_HOME=/app/hbase-1.3.0' >> /etc/profile
echo 'export PATH=$PATH:$HBASE_HOME/bin' >> /etc/profile
source /etc/profile
```
## 5.2 配置hbase-env.sh
```shell
mkdir logs #Hbase日志目录
echo 'export JAVA_HOME=/app/jdk1.8.0_121' >> conf/hbase-env.sh
echo 'export HADOOP_HOME=/app/hadoop-2.7.3'>> conf/hbase-env.sh
echo 'export HBASE_HOME=/app/hbase-1.3.0' >> conf/hbase-env.sh
echo 'export HBASE_MANAGES_ZK=false ' >> conf/hbase-env.sh
echo 'export HBASE_LOG_DIR=/app/hbase-1.3.0/logs' >> conf/hbase-env.sh
```
## 5.3 配置hbase-site.xml
### 5.3.1 重要参数
- hbase.master.info.port指定了hbase web客户端的访问网址
- hbase.rootdir指定了在HDFS中的位置，根目录参考core-sit.xml的fs.default.name
### 5.3.2 hbase.zookeeper相关属性参照
``` java
<configuration>  
  <property>  
    <name>hbase.rootdir</name>  
    <value>hdfs://node01:9000/hbase</value>  
  </property>  
  <property>  
     <name>hbase.cluster.distributed</name>  
     <value>true</value>  
  </property>  
  <property>  
      <name>hbase.master</name>  
      <value>node01:60000</value>  
  </property>  
   <property>  
    <name>hbase.zookeeper.property.dataDir</name>  
    <value>/app/zookeeper-3.5.2-alpha</value>  
  </property>  
  <property>  
    <name>hbase.zookeeper.quorum</name>  
    <value>node01,node02,node03,node04</value>  
  </property>  
  <property>  
    <name>hbase.zookeeper.property.clientPort</name>  
    <value>2181</value>  
  </property> 
  <property>
  <name>hbase.master.info.port</name>
  <value>60010</value>
</property> 
</configuration> 
```
### 5.3.3 配置regionservers
```shell
[root@node01 hbase-1.3.0]# cat conf/regionservers
node02
node03
node04
```
### 5.3.4 scp到各个节点:

## 5.4 启动hbase
  启动之前得保证ZK和hadoop已经启动
  `[master@master1 hbase]$ bin/start-hbase.sh`

## 5.5 报错处理：
```java
hbase(main):006:0> t1 = create 't1', 'f1'
ERROR: org.apache.hadoop.hbase.PleaseHoldException: Master is initializing
```
原因是四个节点时间不同步
### 5.5.1 处理：
配置ntp服务
```shell
[root@node01 ~]# cat /etc/ntp.conf
restrict default nomodify
restrict 127.0.0.1
server 127.127.1.0
fudge 127.127.1.0 stratum 10
driftfile /var/lib/ntp/drift
broadcastdelay 0.008
```
同步时间
```shell
[root@node02 ~]# ntpdate node01
 7 Apr 16:51:55 ntpdate[14684]: step time server IP.85.129 offset -38.280736 sec
[root@node03 ~]# ntpdate node01
 7 Apr 16:52:04 ntpdate[9019]: step time server IP.85.129 offset -40.609528 sec
[root@node04 ~]# ntpdate node01
 7 Apr 16:52:14 ntpdate[9042]: step time server IP.85.129 offset -38.668817 sec
```

# 6 安装HIVE
## 6.1 安装mysql
### 6.1.1 创建用户
```shell
yum install -y mysql-server mysql mysql-deve
groupadd mysql
useradd -g mysql mysql
chown -R mysql:mysql /var/lib/mysql
```
### 6.1.2 配置服务参数
```shell
[root@node01 ~]# cat /etc/my.cnf 
[mysqld]
datadir=/var/lib/mysql
#socket=/var/lib/mysql/mysql.sock
user=mysql
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
```
### 6.1.3 启动
```shell
[root@node01 ~]# service mysqld start
Starting mysqld:                                           [  OK  ]
```

### 6.1.4 配置mysql的访问用户
- 设置密码
    * mysqladmin -u root password 'root'
- 赋给权限
```sql
create user 'hive' identified by 'hive'
grant all privileges on *.* to 'hive' with grant option;
GRANT ALL PRIVILEGES ON *.* TO 'hive'@'node01' IDENTIFIED BY 'hive' WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON *.* TO 'hive'@'%' IDENTIFIED BY 'hive' WITH GRANT OPTION;
flush privileges;
```
- 测试
    * mysql -h node01 -u hive -p 

## 6.2 每个节点执行环境变量                 
```shell
echo 'export HIVE_HOME=/app/apache-hive-2.1.1-bin' >> /etc/profile
echo 'export PATH=$PATH:$HIVE_HOME/bin' >> /etc/profile
source /etc/profile
```
## 6.3 修改hive-env.sh
```
cp /app/apache-hive-2.1.1-bin/conf/hive-env.sh.template /app/apache-hive-2.1.1-bin/conf/hive-env.sh
echo 'export JAVA_HOME=/app/jdk1.8.0_121'  >>/app/apache-hive-2.1.1-bin/conf/hive-env.sh
ln -s $JAVA_HOME/lib/tools.jar $HIVE_HOME/lib/
```
### 6.3.1 放置mysql驱动
wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.41.tar.gz
cp mysql-connector-java-5.1.41-bin.jar /app/apache-hive-2.1.1-bin/lib/

## 6.4 修改hive-site.xml
### 6.4.1 重要参数
- (1)javax.jdo.option.ConnectionURL安装mysql的节点
- (2)javax.jdo.option.ConnectionUserName mysql的元数据的用户
- (3)javax.jdo.option.ConnectionPassword密码
### 6.4.2 修改配置文件
`cp hive-default.xml.template hive-site.xml`
```java
<configuration>
<property>  
  <name>javax.jdo.option.ConnectionURL</name>  
  <value>jdbc:mysql://node01:3306/hive?createDatabaseIfNotExist=true</value>  
  <description>JDBC connect string for a JDBC metastore</description>  
</property>  
  
<property>  
  <name>javax.jdo.option.ConnectionDriverName</name>  
  <value>com.mysql.jdbc.Driver</value>  
  <description>Driver class name for a JDBC metastore</description>  
</property>  
  
<property>  
  <name>javax.jdo.option.ConnectionUserName</name>  
  <value>hive</value>  
  <description>username to use against metastore database</description>  
</property>  
  
<property>  
  <name>javax.jdo.option.ConnectionPassword</name>  
  <value>hive</value>  
  <description>password to use against metastore database</description>  
</property>  
</configuration>
``` 
## 6.5 配置复制hive到其他节点
```shell
scp -r /app/apache-hive-2.1.1-bin/  node02:/app/
scp -r /app/apache-hive-2.1.1-bin/  node03:/app/
scp -r /app/apache-hive-2.1.1-bin/  node04:/app/
```
## 6.6 创建HDFS目录
```
hadoop fs -mkdir /hive
hadoop fs -mkdir /hive/warehouse
hadoop fs -chmod g+w /hive
hadoop fs -chmod g+w /hive/warehouse
```
## 6.7 初始化元数据模型
schematool -initSchema -dbType mysql

## 6.8 hive建表测试
```sql
create table test_ds  
(  
  id int comment '用户ID',  
  name string comment '用户名称'  
)  
comment '测试分区表'  
partitioned by(ds string comment '时间分区字段')  
clustered by(id) sorted by(name) into 32 buckets      
row format delimited   
fields terminated by '\t'  
stored as rcfile;
```

## 6.9 安装hive hwi
```shell
[root@node01 hwi]# pwd
/app/hive_src/apache-hive-2.1.1-src
jar cvfM0 hive-hwi-2.1.1.war -C web/ .
cp hive-hwi-2.1.1.war /app/apache-hive-2.1.1-bin/lib/
```
### 6.9.1 hive-site.xml中添加
```java
<property>
    <name>hive.hwi.war.file</name>
    <value>lib/hive-hwi-2.1.1.war</value>
    <description>This sets the path to the HWI war file, relative to ${HIVE_HOME}. </description>
</property>
<property>
    <name>hive.hwi.listen.host</name>
    <value>node01</value>
    <description>This is the host address the Hive Web Interface will listen on</description>
</property>
<property>
    <name>hive.hwi.listen.port</name>
    <value>9999</value>
    <description>This is the port the Hive Web Interface will listen on</description>
</property>
```

### 6.9.2 启动hwi
```shell
[root@node01 lib]# pwd
/app/apache-hive-2.1.1-bin/lib
[root@node01 lib]# nohup hive --service hwi &
http://IP:9999/hwi
```
http://IP:9999/hwi/index.jsp
