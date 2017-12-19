---
title: Mac下的Hadoop2.7.3环境配置
date: 2017-12-19 15:05:24
tags:
---

## 前提条件：Java环境配置没问题
## 配置过程
1. 配置Hadoop环境变量
  - 官网上下载Hadoop 2.7.3 二进制压缩包，并进行解压
  - 配置Hadoop环境变量，在~/.bash_profile文件末尾添加
  ```这里引用自己真实的安装目录
  export HADOOP_HOME=/Users/zhuyali/prjs/hadoop
  export PATH="$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$JAVA_HOME/bin" 
  ```
  添加完成后，编译该文件使其立即生效。`source ~/.bash_profile`
  - 然后在命令行输入`hadoop version`可以查看到Hadoop的版本信息，那么说明至此步骤已经全部正确完成

2. 创建目录，后续配置文件中会用到
  ```在Hadoop目录下
  $ mkdir tmp
  $ mkdir hdfs
  $ mkdir hdfs/data
  $ mkdir hdfs/name
  ```


3. 修改Hadoop目录/etc/hadoop/hadoop-env.sh文件，在该文件末尾添加
  ```
  export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_144.jdk/Contents/Home
  ```

4. 修改Hadoop目录/etc/hadoop/yarn-env.sh文件，在该文件末尾添加
  ```
  export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_144.jdk/Contents/Home
  ```

5. 修改Hadoop目录/etc/hadoop/core-site.xml文件，设置临时目录和文件系统
  ```
  <configuration>
    <property>  
      <name>fs.default.name</name>  
        <value>hdfs://localhost:9000</value>  
        <description>HDFS的URL(文件系统:namenode标识:端口号)</description>  
      </property>  
      <property>  
        <name>hadoop.tmp.dir</name>  
        <value>/Users/zhuyali/prjs/hadoop/tmp</value>  
        <description>本地hadoop临时文件夹</description>  
      </property>  
  </configuration>
  ```

6. 修改Hadoop目录/etc/hadoop/hdfs-site.xml文件
  ```
  <configuration>
    <property>  
      <name>dfs.name.dir</name>  
      <value>/Users/zhuyali/prjs/hadoop/hdfs/name</value>  
      <description>用于确定将HDFS文件系统的元信息保存在什么目录下</description>  
    </property>  
    <property>  
      <name>dfs.data.dir</name>  
      <value>/Users/zhuyali/prjs/hadoop/hdfs/data</value>  
      <description>用于确定将HDFS文件系统的数据保存在什么目录下</description>  
    </property>  
    <property>  
      <name>dfs.replication</name>  
      <value>1</value>  
      <description>副本个数，应该小于datanode机器数量，默认情况为3</description>  
    </property>  
  </configuration>
  ```

7. 修改Hadoop目录/etc/hadoop/yarn-site.xml文件
  ```
  <configuration>
  <!-- Site specific YARN configuration properties -->
    <property>  
      <name>yarn.nodemanager.aux-services</name>  
      <value>mapreduce_shuffle</value>  
    </property>  
    <property>  
      <name>yarn.resourcemanager.webapp.address</name>  
      <value>localhost:8099</value>  
    </property>  
  </configuration>
  ```

8. 修改Hadoop目录/etc/hadoop/mapred-site.xml
  ```
  <configuration>
    <property>
      <name>mapreduce.framework.name</name>
      <value>yarn</value>
    </property>
  </configuration>
  ```

9. 配置SSH密码
  运行该命令，看是否需要输入密码
  ```
  $ ssh localhost
  ```
  如果需要输入密码，则执行下面的命令
  ```
  $ ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
  $ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
  $ chmod 0600 ~/.ssh/authorized_keys
  ```

## 运行过程
1. 格式化NameNode，仅在第一次启动Hadoop时运行该命令
  ```
  $ bin/hdfs namenode -format
  ```

2. 启动HDFS
  ```
  $ sbin/start-dfs.sh  
  ```

3. 启动yarn
  ```
  $ sbin/start-yarn.sh 
  ```

4. 查看进程
  ```
  $ jps
  ```
  然后出现类似下图的效果，则启动正确

  ![](https://wx4.sinaimg.cn/mw690/79b5b053gy1fmm27hzfknj204p02raaf.jpg)

  此时可以通过 http://localhost:50070 查看HDFS的web页面

## 示例代码Wordcount运行
1. 创建输入目录
  ```
  $ hadoop fs -mkdir /input
  ```

2. 将Hadoop目录下的README.txt文件上传到HDFS输入目录下
  ```
  $ hadoop fs -put README.txt /input
  ```

3. 运行Wordcount程序
  ```
  $ hadoop jar hadoop-mapreduce-examples.jar wordcount /input /output
  ```

4. 查看结果
  ```
  $ hadoop fs -cat /output/par*
  ```

## 遇到的问题
有时候会遇到执行 `jps` 后，发现 datanode 没有启动的问题。一种可能的解决方案为：
```进入到存放HDFS元信息的文件夹中
$ cd /Users/zhuyali/prjs/hadoop/hdfs/name/current
```
然后将该目录下 VERSION 文件中的 clusterId 复制到
```
$ cd /Users/zhuyali/prjs/hadoop/hdfs/data/current
```
该文件夹下的 VERSION 文件中的 clusterId 处

出现该问题的原因是：在第一次格式化 dfs 后，启动并使用了 hadoop，后来又重新执行了格式化命令(hdfs namenode -format)，这时 namenode 的 clusterID 会重新生成，而 datanode 的 clusterID 保持不变。