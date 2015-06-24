# HadoopHiveTezInstall
Installation of Hadoop 2.6.0 + Hive 1.2.0 + Tez 0.7.0 + MySQL + BigFrame + Datahooks

## What are include

- Hive 1.2.0 (complied)
- Tez 0.7.0 (complied)

## System preparation
The tutorial is based on Ubuntu 14.04LTS 64-bit version.
- Add a new administrator user named hadoop.
- Install JAVA SDK and set up JAVA_HOME in bashrc file.

##Installation of Hadoop 2.6.0

In this part I will introduce how to set up a single node cluster in the pseudo-Distributed mode.
You can also refer the official instrucion [here](http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/SingleCluster.html)

###Download

Download Hadoop 2.6.0 from [the official website](http://apache.mirrors.pair.com/hadoop/common/hadoop-2.6.0/). 
If you use a 64-bit OS, please download the source code tarball. Follow the instruction [here](https://hadoop.apache.org/docs/r2.6.0/hadoop-project-dist/hadoop-common/NativeLibraries.html) to complie it. 
You can find your newly-built Hadoop here: `$ hadoop-dist/taget/hadoop-2.6.0`

Following instruction here to set up your Hadoop. 

Install ssh and rsync:

    $ sudo apt-get install ssh
    $ sudo apt-get install rsync

Copy or unzip your hadoop to the a proper place. For example, I put my hadoop under /usr/local.

	$ sudo cp -r ${my newly-built hadoop path} /usr/local/hadoop-2.6.0

### Configuration

Set up Hadoop environment variabes:

	$ sudo vim ~/.bashrc

Add the following line in the bashrc file:

	export HADOOP_HOME=/usr/local/hadoop-2.6.0
	export HADOOP_PREFIX=$HADOOP_HOME
	export PATH=$PATH:${HADOOP_HOME}/bin:${HADOOP_HOME}/sbin

Setup the JAVA_HOME in etc/hadoop/hadoop-env.sh. For example:

	export JAVA_HOME="/usr/lib/jvm/java-7-oracle"

Setup etc/hadoop/core-site.xml
```
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>file:/usr/local/hadoop-2.6.0/tmp</value>
        <description>Abase for other temporary directories.</description>
    </property>
</configuration>
```
Setup etc/hadoop/hdfs-site.xml:
```
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>1</value>
  </property>
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>file:/usr/local/hadoop-2.6.0/hdfs/namenode</value>
  </property>
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>file:/usr/local/hadoop-2.6.0/hdfs/datanode</value>
  </property>
</configuration>
```
Setup etc/hadoop/mapred-site.xml
```
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```
Setup etc/hadoop/yarn-site.xml:
```
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
</configuration>
```

###Setup passphraseless ssh
	$ ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa
    $ cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys
    $ export HADOOP\_PREFIX=/usr/local/hadoop





