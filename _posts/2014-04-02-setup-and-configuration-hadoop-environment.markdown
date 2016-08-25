---
layout: post
title: Hadoop集群环境搭建及配置
date: 2014-04-02 09:06
tags: 
---

![](/assets/images/2014/04/hadoop-elephant_logo.png)

### 1.环境准备

* 三台服务器：CentOS6.5系统，IP地址如下，分别配置hosts：

```
vi /etc/hosts
10.2.15.12  master   #NameNode JobTracker
10.2.15.13  slave1   #DataNode TaskTracker
10.2.15.14  slave2   #DataNode TaskTracker
```

* JDK安装及Java环境配置

下载&nbsp;[jdk1.7.0_51](http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html)

```
tar zxvf  -C /apps/ # 解压到/apps/目录
chown -R root.root jdk1.7.0_51   # 授权
vi /etc/profile   # 配置环境变量
export JAVA_HOME=/apps/jdk1.7.0_51
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin
source /etc/profile  # 使修改生效
```

更新 alternatives，选择jdk版本

```
update-alternatives --install /usr/bin/java java /apps/jdk1.7.0_51/bin/java 60
update-alternatives --config java   # 选择3回车
```


* 创建hadoop用户

```
groupadd hadoop   # 添加组
useradd -g hadoop -d /home/hadoop hadoop   # 添加用户
passwd hadoop   # 修改密码
```


* SSH无密码验证配置

```
mkdir ~/.ssh
su - hadoop
cd ~/.ssh
ssh-keygen -t  rsa       # 一直回车生成密钥
cat id_rsa.pub >> authorized_keys   # 将公钥id_rsa.pub追加到授权的key里面去
svervice sshd restart   # 重启SSH服务
ssh slave1   # ssh到slave1
mkdir ~/.ssh   # 创建.ssh目录
ssh slave2   # ssh到slave2
mkdir ~/.ssh   # 创建.ssh目录
exit
exit    # 返回到master
scp authorized_keys slave1:~/.ssh/   # copy授权key到slave1
scp authorized_keys slave2:~/.ssh/   # copy授权key到slave2
vi /etc/ssh/sshd_config   #开启RSA认证
RSAAuthentication yes
PubkeyAuthentication yes
AuthorizedKeysFile      .ssh/authorized_keys
ssh slave1   # ssh到slave1
chmod 700 ~/.ssh #目录权限必须设置700
ssh slave2   # ssh到slave2
chmod 700 ~/.ssh #目录权限必须设置700
exit
exit    # 返回到master
ssh salve1   # 验证
exit
ssh slave2   # 验证
```


### 2.Hadoop安装与配置


* 下载&nbsp;[hadoop-2.2.0](http://www.apache.org/dyn/closer.cgi/hadoop/common/)

```
mkdir /apps/hadoop  # 创建hadoop目录作为安装hadoop的home目录
tar zxvf hadoop-2.2.0.tar.gz -C /apps/hadoop   # 解压到hadoop目录
chown hadoop.hadoop -R /home/hadoop/hadoop-2.3.0/   # 授权
vi /etc/profile   #添加hadoop变量，方便使用，不必每次操作cd到bin目录
HADOOP_HOME=/home/hadoop/hadoop-2.2.0/
PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
export HADOOP_HOME PATH
source /etc/profile   # 生效
```

* 修改hadoop-env.sh、yarn-env.sh Hadoop运行的环境变量

```
vi hadoop-env.sh
export JAVA_HOME=/apps/jdk1.7.0_51
vi yarn-env.sh 
export JAVA_HOME=/apps/jdk1.7.0_51
```

* 修改core-site.xml配置文件,主要对NameNode地址端口及系统级别的参数配置，如HDFS URL、Hadoop临时目录以及用于rack-aware集群中的配置文件的配置等。

```
<configuration>
	<property>
    	<name>fs.default.name</name>
     	<value>hdfs://master:9000/</value>
     	<description>The name of the default file system. A URI whose scheme and authority determine the FileSystem implementation. The uri's scheme determines the config property (fs.SCHEME.impl) naming the FileSystem implementation class. The uri's authority is used to determine the host, port, etc. for a filesystem.</description>
	</property>
    <property>
    	<name>io.file.buffer.size</name>
        <value>131072</value>
	</property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/apps/hadoop/tmp</value>
        <description>A base for other temporary directories.</description>
    </property>
    <property>
        <name>hadoop.proxyuser.hduser.hosts</name>
        <value>*</value>
    </property>
    <property>
        <name>hadoop.proxyuser.hduser.groups</name>
        <value>*</value>
    </property>
</configuration>
```

* 修改hdfs-site.xml配置文件，主要对HDFS相关属性的配置，如文件副本的个数、块大小及是否使用强制权限等。

``` 
<configuration>
	<property>
    	<name>dfs.namenode.name.dir</name>
        <value>file://${hadoop.tmp.dir}/dfs/name</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file://${hadoop.tmp.dir}/dfs/data</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/apps/hadoop/hdfs/tmp/</value>
    </property>
    <property>
        <name>dfs.replication</name>   #数据副本数量，默认3，我们是两台设置2
        <value>2</value>    
    </property>
    <property>
        <name>dfs.webhdfs.enabled</name>
        <value>true</value>
     </property>
</configuration>
```

* 修改mapred-site.xml MapReduce配置文件，主要对JobTracker地址及端口的配置。

```
<configuration>
	<property>
    	<name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <property>
    	<name>mapreduce.jobhistory.address</name>
        <value>master:10020</value>
    </property>
    <property>
        <name>mapreduce.jobhistory.webapp.address</name>
        <value>master:19888</value>
    </property>
<configuration>
```

* 修改slaves配置文件，slaves主机列表，可以是alias或IP地址。

```
slave1
slave2
```

* 修改yarn-site.xml配置文件，yarn地址及端口信息配置。

```
<configuration>
	<property>
		<name>yarn.resourcemanager.address</name>
		<value>master:8032</value>
	</property>
	<property>
		<name>yarn.resourcemanager.scheduler.address</name>
		<value>master:8030</value>
	</property>
	<property>
		<name>yarn.resourcemanager.resource-tracker.address</name>
		<value>master:8031</value>
	</property>
	<property>
		<name>yarn.resourcemanager.admin.address</name>
		<value>master:8033</value>
	</property>
	<property>
		<name>yarn.resourcemanager.webapp.address</name>
 		<value>master:8088</value>
	</property>
	<property>
		<name>yarn.nodemanager.aux-services</name>
		<value>mapreduce_shuffle</value>
	</property>
 	<property>
 		<name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
 		<value>org.apache.hadoop.mapred.ShuffleHandler</value>
 	</property>
</configuration>
```

```
scp hadoop slave1:~/apps/   # 复制到其他节点slave1、slave2
scp hadoop slave2:~/apps/
```

<font color=red size=1> 注意：只需要原样复制即可，不必改动上面的xml配置文件。 </font>


### 3.启动关闭与验证

* 启动与关闭

```
hdfs namenode -format    # 格式化HDFS
start-dfs.sh    # 启动HDFS文件系统
jps   # 检查守护进程是否启动
start-yarn.sh   # 启动新mapreduce架构
jps   # 检查守护进程是否启动
start-all.sh    # 也可通过star-all.sh启动所有进程
stop-all.sh     # 关闭所有进程
hdfs dfsadmin -report   # 查看集群状态
```

* 浏览器中查看

[http://master:8088](http://master:8088/)   &nbsp;&nbsp;&nbsp;&nbsp; #通过web查看资源

[http://master:50070](http://master:50070/)  &nbsp;&nbsp; # 查看HDFS状态

* 验证集群计算，执行Hadoop自带的examples，执行如下命令：

```
bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.2.0.jar randomwriter out
```

至此，Hadoop的集群已经搭建完成，以上内容主要参考[`Apache Hadoop`](http://hadoop.apache.org)官网！

详细配置请参考：[`Hadoop Cluster Setup`](http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/ClusterSetup.html)


