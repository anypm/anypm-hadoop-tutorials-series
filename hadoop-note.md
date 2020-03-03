# Hadoop实战笔记
## 本地环境
### 环境搭建
1. 创建JDK和Hadoop的软件目录和运行时目录
$ cd /opt
$ mkdir module
$ mkdir software
$ chmod 777 module
$ chmod 777 software

2. 下载hadoop与jdk安装包并切换目录至软件所在目录，拷贝软件至已创建好的software目录
> $ cp hadoop-2.10.0.tar /opt/software/
> $ cp jdk-13.0.2_linux-x64_bin.tar /opt/software/

3. 解压jdk和hadoop至已创建好的module目录
> $ cd /opt/software/
> $ tar -xvf jdk-13.0.2_linux-x64_bin.tar -C /opt/module/
> $ tar -xvf hadoop-2.10.0.tar -C /opt/module/

4. 设置jdk和hadoop环境变量
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

### Grep官方案例
> $ mkdir input
> $ cp etc/hadoop/*.xml input

> $ bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.10.0.jar grep input output 'dfs[a-z.]+'
> $ hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.10.0.jar grep input output 'dfs[a-z.]+'

> $ cat output/*

### WordCount官方案例

> $ mkdir wcinput
> $ touch wc.input
> $ vim wc.input

> $ hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.10.0.jar wordcount wcinput/ wcoutput
> $ cd wcoutput
> $ cat part-r-00000


## 伪分布式

### 1. 启动HDFS并运行MR程序
1.1 配置`hadoop-env.sh`
Linux中获取JDK的安装路径：
> $ echo $JAVA_HOME
	
修改JAVA_HOME路径：
> $ vim etc/hadoop/hadoop-env.sh
> $ export JAVA_HOME=/opt/module/jdk-13.0.2

1.2 配置：core-site.xml
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

1.3 配置：hdfs-site.xml
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

### 2. 启动集群

2.1格式化NameNode(第一次启动时格式化，后续不要总格式化)
> $ bin/hdfs namenode -format
或
>$ hdfs namenode -format

2.2 启动NameNode
> $ sbin/hadoop-daemon.sh start namenode

2.3 启动DataNode
> $ sbin/hadoop-daemon.sh start datanode

2.4 查看
> http://117.51.150.179:50070/

创建hdfs路径
$ bin/hdfs dfs -mkdir -p /user/daniel/input

查看路径
$ bin/hdfs dfs -ls /
或
$ bin/hdfs dfs -lsr /

上传文件本地文件至hdfs路径
> $ bin/hdfs dfs -put wcinput/wc.input /user/daniel/input

### WordCount官方案例-HDFS文件系统
$ hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.10.0.jar wordcount /user/daniel/input /user/daniel/output
$ bin/hdfs dfs -cat /user/daniel/output/part-r-00000

### 查看集群
查看是否启动成功




