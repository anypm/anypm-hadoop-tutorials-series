# Hadoop实战笔记

## Hadoop介绍
略


## 环境搭建
创建JDK和Hadoop的软件目录和运行时目录
$ cd /opt
$ mkdir module
$ mkdir software
$ chmod 777 module
$ chmod 777 software

下载hadoop与jdk安装包并切换目录至软件所在目录，拷贝软件至已创建好的software目录

> $ cp hadoop-2.10.0.tar /opt/software/

> $ cp jdk-13.0.2_linux-x64_bin.tar /opt/software/

解压jdk和hadoop至已创建好的module目录
> $ cd /opt/software/

> $ tar -xvf jdk-13.0.2_linux-x64_bin.tar -C /opt/module/

> $ tar -xvf hadoop-2.10.0.tar -C /opt/module/

设置jdk和hadoop环境变量

>$ sudo vim /etc/profile

```
##JAVA_HOME
export JAVA_HOME=/opt/module/jdk-13.0.2
export PATH=$PATH:$JAVA_HOME/bin

##HADOOP_HOME
export HADOOP_HOME=/opt/module/hadoop-2.10.0
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
```
更新配置文件
> $ source /etc/profile

检查jdk与hadoop是否安装正常

>$ java -version

>$ hadoop


## Hadoop运行模式
### 本地模式
#### Grep官方案例

> $ mkdir input

> $ cp etc/hadoop/*.xml input

> $ bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.10.0.jar grep input output 'dfs[a-z.]+'

或

> $ hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.10.0.jar grep input output 'dfs[a-z.]+'

查看结果

> $ cat output/*

#### WordCount官方案例

> $ mkdir wcinput

> $ touch wc.input

> $ vim wc.input

> $ hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.10.0.jar wordcount wcinput/ wcoutput

> $ cd wcoutput

> $ cat part-r-00000


### 伪分布式

### 启动HDFS并运行MR程序
#### 配置`hadoop-env.sh`

Linux中获取JDK的安装路径：

> $ echo $JAVA_HOME
	
修改JAVA_HOME路径：

> $ vim etc/hadoop/hadoop-env.sh

```
export JAVA_HOME=/opt/module/jdk-13.0.2
```

#### 配置：core-site.xml

> $ vim etc/hadoop/core-site.xml

```
<configuration>

    <!-- 指定HDFS中NameNode的地址-->
    <property>
                    <name>fs.defaultFS</name>
                    <value>hdfs://localhost:9000</value>
    </property>
    
    <!-- 指定Hadoop运行时产生文件的存储目录 -->
    <property>
                    <name>hadoop.tmp.dir</name>
                    <value>/opt/module/hadoop-2.10.0/data/tmp</value>
    </property>
</configuration>
```

#### 配置：hdfs-site.xml

> $ vim etc/hadoop/hdfs-site.xml

```
<configuration>
	    <!-- 指定HDFS中副本的数量 -->
            <property>
                            <name>dfs.replication</name>
                            <value>1</value>
            </property>
</configuration>
```

#### 启动集群

##### 格式化NameNode(第一次启动时格式化，后续不要总格式化)

> $ bin/hdfs namenode -format

或

>$ hdfs namenode -format

##### 启动NameNode

> $ sbin/hadoop-daemon.sh start namenode

##### 启动DataNode

> $ sbin/hadoop-daemon.sh start datanode


#### 查看集群

##### 查看是否启动成功

> $ jps


##### Web端查看HDFS文件系统

> http://IP:50070/

#### 操作集群

##### 创建hdfs路径

> $ bin/hdfs dfs -mkdir -p /user/daniel/input

##### 查看路径

> $ bin/hdfs dfs -ls /

或

> $ bin/hdfs dfs -lsr /

##### 上传文件本地文件至hdfs路径

> $ bin/hdfs dfs -put wcinput/wc.input /user/daniel/input

##### WordCount官方案例-HDFS文件系统

> $ hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.10.0.jar wordcount /user/daniel/input /user/daniel/output

> $ bin/hdfs dfs -cat /user/daniel/output/part-r-00000


#### NN格式化前的注意事项：集群挂掉重新格式化步骤
##### 查看NameNode和DataNode进程是否运行

> $ jps

如果运行则关掉进程

##### 删除hadoop下data和log文件夹

> $ sudo rm -rf data

> $ sudo rm -rf log

##### 格式化DataNode

> $ bin/hdfs namenode -format

##### NameNode格式化注意事项

**格式化NameNode会产生新的集群ID，导致NameNode和DataNode集群ID不一致，集群找不到以往的数据。所以格式化NameNode时一定要先删除data数据和log日志，然后再格式化NameNode。**


#### log日志查看

进入log日志文件夹

> $ cd /opt/module/hadoop-2.10.0/logs

查看日志

> $ cat hadoop-root-namenode-10-255-20-11.log

> $ cat hadoop-root-datanode-10-255-20-11.log

#### 启动YARN并运行MR程序

- 配置集群在YARN上运行MR
- 启动、测试集群增、删、查
- 在YARN上执行WordCount案例

##### 配置集群

1.配置：yarn-env.sh

Linux中获取JDK的安装路径：

> $ echo $JAVA_HOME
	
修改JAVA_HOME路径：

> $ vim etc/hadoop/yarn-env.sh

```
export JAVA_HOME=/opt/module/jdk-13.0.2
```

2.配置：yarn-site.xml


etc/hadoop/yarn-site.xml:

```
<configuration>
    <!-- Reducer获取数据的方式 -->
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <!-- 指定YARN的ResourceManager的地址 -->
       <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>localhost</value>
    </property>
    
    <property>
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
    </property>
</configuration>
```

3.配置：mapred-env.sh

Linux中获取JDK的安装路径：

> $ echo $JAVA_HOME
	
修改JAVA_HOME路径：

> $ vim etc/hadoop/mapred-env.sh

```
export JAVA_HOME=/opt/module/jdk-13.0.2
```
对mapred-site.template重命名为mapred-site.xml

> $ mv etc/hadoop/mapred-site.xml.template etc/hadoop/mapred-site.xml

4.配置：mapred-site.xml

> $ vim etc/hadoop/mapred-site.xml

```
<!-- 指定MR运行在YARN上 -->
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
```

##### 启动集群

1.启动前确保NameNode和DataNode已经启动
2.启动ResourceManager

> $ sbin/yarn-daemon.sh start resourcemanager

3.启动NodeManager

> $ sbin/yarn-daemon.sh start nodemanager



##### 集群操作

Web端查看

> http://IP:8088







### 完全分布式模式





## Hadoop源码编译

