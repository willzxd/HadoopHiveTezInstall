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
You can find your newly-built Hadoop here:
 `$ hadoop-dist/taget/hadoop-2.6.0`

Following instruction here to set up your Hadoop. 

Install ssh and rsync:

    $ sudo apt-get install ssh
    $ sudo apt-get install rsync

Copy or unzip your hadoop to the a proper directory. For example, I put my hadoop under /usr/local.

	$ sudo cp -r ${my newly-built hadoop path} /usr/local/hadoop-2.6.0
	$ sudo chown -R hadoop:hadoop /usr/local/hadoop-2.6.0

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

If you cannot ssh to localhost without a passphrase, execute the following commands:

	$ ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa
    $ cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys
    $ export HADOOP\_PREFIX=/usr/local/hadoop

Now check that you can ssh to the localhost without a passphrase:

	$ ssh localhost

### Execution


Format the filesystem:

    $ bin/hdfs namenode -format

The hadoop daemon log output is written to the $HADOOP_LOG_DIR directory (defaults to $HADOOP_HOME/logs).

Browse the web interface for the NameNode; by default it is available at:

    NameNode - http://localhost:50070/

Make the HDFS directories required to execute MapReduce jobs:

  $ bin/hdfs dfs -mkdir /user
  $ bin/hdfs dfs -mkdir /user/<username>

Start NameNode daemon, DataNode daemon, ResourceManager daemon and NodeManager daemon:

    $ sbin/start-dfs.sh
    $ sbin/start-yarn.sh

Stop NameNode daemon, DataNode daemon, ResourceManager daemon and NodeManager daemon:

	$ sbin/stop-dfs.sh
    $ sbin/stop-yarn.sh

use jps command to check everything is running. You should have a list like this:

	5231 SecondaryNameNode
	4840 NameNode
	5641 NodeManager
	14188 Jps
	5499 ResourceManager
	5015 DataNode

### Test: Wordcount exmaple

    $ bin/hdfs dfs -put etc/hadoop/*.xml input
    $ bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.6.0.jar grep input output 'dfs[a-z.]+'
    $ bin/hdfs dfs -get output output
    $ cat output/*

##Installation of Hive

You can follow the official instruction [here](https://cwiki.apache.org/confluence/display/Hive/GettingStarted).

### Download
You can download the newly-built version from this repository or [the official website](http://mirror.nexcess.net/apache/hive/hive-1.2.0/). If you want to build it by yourself, you need to install Maven. The instuction is [here](http://maven.apache.org/download.cgi).

Copy your Hive to the proper directory. For example: 

	$ sudo cp -r ${my newly-built hadoop path} /usr/local/hive-1.2.0
	$ sudo chown -R hadoop:hadoop /usr/local/hive-1.2.0


### Configuration

Add the following lines to the bashrc file:

	export HIVE_HOME=/usr/local/hive-1.2.0
	export HIVE_CONF_DIR=$HIVE_HOME/conf
	export PATH=$PATH:$HIVE_HOME/bin 


###Running Hive

you must create /tmp and /user/hive/warehouse (aka hive.metastore.warehouse.dir) and set them chmod g+w in HDFS before you can create a table in Hive. Commands to perform this setup:

    $ $HADOOP_HOME/bin/hadoop fs -mkdir       /tmp
    $ $HADOOP_HOME/bin/hadoop fs -mkdir       /user/hive/warehouse
    $ $HADOOP_HOME/bin/hadoop fs -chmod g+w   /tmp
    $ $HADOOP_HOME/bin/hadoop fs -chmod g+w   /user/hive/warehouse

To use the [Hive command line interface (CLI)](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Cli) from the shell:

  $ $HIVE_HOME/bin/hive

To use the [Hiveserver2](https://cwiki.apache.org/confluence/display/Hive/Setting+Up+HiveServer2) on background:

  $ $HIVE_HOME/bin/hive --service hiveserver2 2>&1 &


