---
layout: post
title: 安装Hadoop、Spark单机版
date: 2017-01-03 17:30:33 +08:00
author: "izumo"
tags: 
    - Hadoop
    - Spark
---

## 默认环境
+ Debian/Linux
+ Java 8u101
+ Scala 2.11.8

## hadoop安装方法

### 编辑/etc/profile

    #java
    export JAVA_HOME=/usr/local/jdk1.8.0_101
    export JRE_HOME=$JAVA_HOME/jre
    export CLASSPATH=.:$JAVA_HOME/lib:$JRE_HOME/lib:CLASSPATH
    export PATH=$JAVA_HOME/bin:$PATH
    
    #scala
    export SCALA_HOME=/usr/local/scala-2.11.8
    export PATH=$SCALA_HOME/bin:$PATH
    
    #hadoop
    export HADOOP_HOME=/usr/local/hadoop-2.7.3
    export PATH=$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH
    export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
    export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib"
    
    另一个#hadoop版本：
    export HADOOP_HOME=/usr/local/hadoop-2.7.3
    export HADOOP_INSTALL=$HADOOP_HOME
    export HADOOP_MAPRED_HOME=$HADOOP_HOME
    export HADOOP_COMMON_HOME=$HADOOP_HOME
    export HADOOP_HDFS_HOME=$HADOOP_HOME
    export YARN_HOME=$HADOOP_HOME
    export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
    export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin

### 将hadoop文件夹权限给予用户
    
    sudo chown -R <username> hadoop-2.7.3
    sudo chgrp -R <username> hadoop-2.7.3

### 在etc/hadoop/hadoop-env.sh中添加JAVA_HOME

    export JAVA_HOME=/usr/local/jdk1.8.0_101

### 配置etc/hadoop/core-site.xml

    <configuration>
            <property>
                    <name>hadoop.tmp.dir</name>
                    <value>file:/usr/local/hadoop-2.7.3/tmp</value>
                    <description>Abase for other temporary directories.</description>
            </property>
            <property>
                    <name>fs.defaultFS</name>
                    <value>hdfs://localhost:9000</value>
            </property>
    </configuration>

### 配置etc/hadoop/hdfs-site.xml

    <configuration>
            <property>
                    <name>dfs.replication</name>
                    <value>1</value>
            </property>
            <property>
                    <name>dfs.namenode.name.dir</name>
                    <value>file:/usr/local/hadoop-2.7.3/tmp/dfs/name</value>
            </property>
            <property>
                    <name>dfs.datanode.data.dir</name>
                    <value>file:/usr/local/hadoop-2.7.3/tmp/dfs/data</value>
            </property>
    </configuration>

### ssh无密码访问

    ssh-keygen -t rsa -P ""
    cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys

### 初始化/启动hdfs

    hdfs namenode -format
    sbin/start-dfs.sh

> Hdfs Web地址：`http://localhost:50070/`

### 配置yarn

    ========etc/hadoop/mapred-site.xml========
    <configuration>
        <property>
            <name>mapreduce.framework.name</name>
            <value>yarn</value>
        </property>
    </configuration>
    
    ========etc/hadoop/yarn-site.xml========
    <configuration>
        <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
        </property>
    </configuration>

### 启动yarn

    sbin/start-yarn.sh

> Yarn Web地址：`http://localhost:8088/`

## Spark安装方法

### 编辑/etc/profile
    #spark
    export SPARK_HOME=/usr/local/spark-2.0.2-bin-hadoop2.7
    export PATH=$SPARK_HOME/bin:$PATH

另一个版本：
    export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
    export SPARK_DIST_CLASSPATH=$(hadoop classpath) #????

### 将spark文件夹权限给予用户
略

### 配置conf/spark-env.sh
    export JAVA_HOME=/usr/local/jdk1.8.0_101
    export SCALA_HOME=/usr/local/scala-2.11.8
    export SPARK_MASTER_HOST=<host_ip>
    export SPARK_MASTER_PORT=7077
    export SPARK_WORKER_MEMORY=1G

### 启动spark
    sbin/start-all.sh

### 检测spark是否安装成功
    bin/run-example SparkPi

> Spark Web地址：http://localhost :8080
> Spark Master端口：spark://<host_ip>:7077

