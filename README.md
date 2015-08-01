# Using BigFrame to compare the performance of Hive-MR and Hive-Tez

Tutorial of Hadoop 2.6.0 + Hive 1.2.0 + Tez 0.7.0 + Tomact 8.0 + Tez UI 0.7.0 + MySQL + BigFrame + Datahooks

- [About this repository](#1)
- [System Prequisite](#2)
- [Installation of Hadoop 2.6.0](#3)
- [Installation of MySQL](#4)
- [Installation of Hive 1.2.0](#5)
- [Installation of Tez 0.7.0](#6)
- [Installation of Tez UI 0.7.0](#10)
- [Using BigFrame](#7)
- [An example of Comparing the Performance of Hive-MR and Hive-Tez](#8)
- [An example of bashrc file](#9)

## <a name="1"/>What are include in this repository

- Hive 1.2.0 (built)
- Tez 0.7.0 (built)

## <a name="2"/>System Prequisite
The tutorial is based on Ubuntu 14.04LTS 64-bit version.
- Add a new administrator user named hadoop.
```
$ sudo useradd hadoop
$ sudo passwd hadoop
$ sudo mkdir /home/hadoop
$ sudo chown hadoop /home/hadoop
$ sudo adduser hadoop sudo
```
- Install JAVA SDK 1.7 or later and set up JAVA_HOME in bashrc file. Instruction is [here](https://www.digitalocean.com/community/tutorials/how-to-install-java-on-ubuntu-with-apt-get)
- Install Maven 3 or later
- Install Protocol Buffers 2.5.0 or later

##<a name="3"/>Installation of Hadoop 2.6.0

In this part I will introduce how to set up a single node cluster in the pseudo-Distributed mode.
You can also refer the official instrucion [here](http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/SingleCluster.html).

###Download

Download Hadoop 2.6.0 from [the official website](http://apache.mirrors.pair.com/hadoop/common/hadoop-2.6.0/). 
If you use a 64-bit OS, please download the source code tarball. Follow the instruction [here](https://hadoop.apache.org/docs/r2.6.0/hadoop-project-dist/hadoop-common/NativeLibraries.html) to complie it. 

You need to install protobuf 2.5.0, CMake and etc.

You can find your newly-built Hadoop here:
 ```$ hadoop-dist/taget/hadoop-2.6.0```

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

##<a name="4"/>Installation of MySQL

use the following command:

    apt-get install mysql-server

During the installation, you will be asked to setup the root password of mysql.

use the following command to check whether mysql is running:
```
netstat -tap|grep mysql
```

Modify /etc/mysql/my.cnf like this:
```
#bind-address        = 127.0.0.1
```
Login:
```
mysql -u root -p
```
Create Hive user:
```
CREATE USER 'hive'@'%' IDENTIFIED BY 'hive';
GRANT ALL PRIVILEGES ON *.* TO 'hive'@'%' WITH GRANT OPTION;
flush privileges;
```
Restart MySQL:
    $ /etc/init.d/mysql restart

##<a name="5"/>Installation of Hive

You can follow the official instruction [here](https://cwiki.apache.org/confluence/display/Hive/GettingStarted).

### Download
You can download the newly-built version from this repository or [the official website](http://mirror.nexcess.net/apache/hive/hive-1.2.0/). If you want to build it by yourself, you need to install Maven. The instuction is [here](http://maven.apache.org/download.cgi).

Copy your Hive to the proper directory. For example: 

	$ sudo cp -r ${my newly-built hive path} /usr/local/hive-1.2.0
	$ sudo chown -R hadoop:hadoop /usr/local/hive-1.2.0



### Configuration

Add the following lines to the bashrc file:

	export HIVE_HOME=/usr/local/hive-1.2.0
	export HIVE_CONF_DIR=$HIVE_HOME/conf
	export PATH=$PATH:$HIVE_HOME/bin 

Hive by default gets its configuration from `$HIVE_HOME/conf/hive-default.xml`. Copy `hive-default.xml` and rename it as `hive-site.xml`. Modify `hive-site.xml` for using MySQL as metastore:
```
<property>
	<name>javax.jdo.option.ConnectionURL</name>
	<value>jdbc:mysql://localhost:3306/hive?createDatabaseIfNotExist=true</value>
	<description>JDBC connect string  a JDBC metastore</description>
</property>

<property>
	<name>javax.jdo.option.ConnectionDriverName</name>
	<value>com.mysql.jdbc.Driver</value>
	<description>Driver class name  a JDBC metastore</description>
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
```
Also you need to setup localscratch dir, downloaded dir etc, which are the values that conatain "${system:java.io.tmpdir}".

[Download](http://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.35.tar.gz) MySQL JDBC Drvier and copy it to the `$HIVE_HOME/lib`. 

Remove `jline-0.9.94.jar` under `$HADOOP_HOME` directory , or replace with jline-2.12.jar under `$HIVE_HOME/lib` directory.

###Running Hive

you must create `/tmp` and `/user/hive/warehouse` (aka hive.metastore.warehouse.dir) and set them chmod g+w in HDFS before you can create a table in Hive. Commands to perform this setup:

    $ $HADOOP_HOME/bin/hadoop fs -mkdir       /tmp
    $ $HADOOP_HOME/bin/hadoop fs -mkdir       /user/hive/warehouse
    $ $HADOOP_HOME/bin/hadoop fs -chmod g+w   /tmp
    $ $HADOOP_HOME/bin/hadoop fs -chmod g+w   /user/hive/warehouse

To use the [Hive command line interface (CLI)](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Cli) from the shell:

    $HIVE_HOME/bin/hive

To use the [Hiveserver2](https://cwiki.apache.org/confluence/display/Hive/Setting+Up+HiveServer2) on background:

    $HIVE_HOME/bin/hive --service hiveserver2 2>&1 &

##<a name="6"/>Tez installation
You can follow the official instruction [here](http://tez.apache.org/install.html).
Tez need Protocol Buffers 2.5.0 and Maven 3 or later.

###Installation of Protocol Buffers 2.5.0

Download protocol buffer. Protocol buffer libs can be downloaded [here](https//protobuf.googlecode.com/files/protobuf-2.5.0.tar.gz).

Check if g++ compiler is installed on box. Protocol buffer needs g++ compiler to be present on your box before it can be builded. This is a crisp post on how to install g++ compiler on your box. 

Extract the protocol buffer archive and switch to the extracted directory.

Inside the extracted directory hit the below commands to install protocol buffer. These may take a while, kindly be patient.

$ sudo ./configure
$ sudo make
$ sudo make check
$ sudo make install
$ protoc --version

Add the following line to the bashrc file:
```
export PATH=$PATH:/usr/local/protobuf-2.5.0/bin
```
Protocol buffer is installed

###Download and Build Tez
You can download the newly-built Tez from this repository. Also you can download from [the official website](http://psg.mtu.edu/pub/apache/tez/0.7.0/). Explode the tarball wherever you want.

Build tez using:
```
mvn clean package -DskipTests=true -Dmaven.javadoc.skip=true
```
If you meet the problem with com.github.eirslett, change the pom.xml as following:
```
<groupId>com.github.eirslett</groupId>
<artifactId>frontend-maven-plugin</artifactId>
<version>0.0.23</version>
```

After built, copy the tez-dist/target/tez-0.7.0 to the proper directory. For example:
```
$ sudo cp -r ${my newly-built hadoop path} /usr/local/tez-0.7.0
```
###Configuration
Add the following lines to the bashrc file:
``` 
export TEZ_INSTALL_DIR='/usr/local/tez-0.7.0'
export TEZ_CONF_DIR=${TEZ_INSTALL_DIR}/conf
export TEZ_JARS=$(echo "$TEZ_INSTALL_DIR"/*.jar | tr ' ' ':'):$(echo "$TEZ_INSTALL_DIR"/lib/*.jar | tr ' ' ':')
export HIVE_AUX_JARS_PATH="${TEZ_JARS}"

export HADOOP_CLASSPATH="${TEZ_CONF_DIR}:${TEZ_JARS}:${HADOOP_CLASSPATH}"
```
Create tez-site.xml and Add the following contents:
```
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
<property>
  <name>tez.lib.uris</name>
  <value>${fs.defaultFS}/apps/tez-0.7.0/,${fs.defaultFS}/apps/tez-0.7.0/lib/</value> 
</property>
</configuration>
```
###Copy the library to HDFS
```
$ bin/hdfs dfs ­mkdir ­p /apps/tez­-0.7.0
$ bin/hdfs dfs ­chmod g+w /apps/tez­-0.7.0
$ bin/hdfs dfs ­mkdir ­p /apps/tez­-0.7.0/lib
$ bin/hdfs dfs ­chmod g+w /apps/tez­-0.7.0/lib
$ bin/hdfs dfs ­copyFromLocal $TEZ_INSTALL_DIR/* /apps/tez­-0.7.0
$ bin/hdfs dfs ­copyFromLocal $TEZ_INSTALL_DIR/lib/* /apps/tez­-0.7.0/lib
```
Tez has been successfully installed.

##<a name = "10"/>Installation of Tez UI
I deploy Tez UI web application by Tomcat. There are also alternative ways to deploy it.
The official setup instruction of Tomcat is [here](http://tomcat.apache.org/tomcat-8.0-doc/setup.html#Unix_daemon).
The official setup instruction of Tez UI is [here](http://tez.apache.org/tez-ui.html).

## Download Tomcat 8.0.23
You can download the Tomcat on [its official website](http://tomcat.apache.org/download-80.cgi).


## Configuration fo Tomcat
Explode the tarball in the proper directory. I put it under `/usr/local/apache-tomcat-8.0.23`.

Add the following line to bashrc file:
```
# Tomact
export CATALINA_HOME=/usr/local/apache-tomcat-8.0.23
```

Add the following line to `$CATALINA_HOME/bin/startup.sh`:
```
export JAVA_HOME=/usr/lib/jvm/java-7-oracle
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export CATALINA_HOME=/usr/local/apache-tomcat-8.0.23
```
Add user of Tomcat to `$CATALINA_HOME/conf/tomcat-user.xml`:
```
<role rolename="tomcat"/>
<user username="tomcat" password="tomcat" roles="manager-gui,manager-script,manager-jmx,manager-status"/>
```
Add the following lines to `$TEZ_HOME/conf/tez-site.xml`:
```
<property>
  <description>Log history using the Timeline Server</description>
  <name>tez.history.logging.service.class</name>
  <value>org.apache.tez.dag.history.logging.ats.ATSHistoryLoggingService</value>
</property>
<property>
  <description>Publish configuration information to Timeline server.</description>
  <name>tez.runtime.convert.user-payload.to.history-text</name>
  <value>true</value>
</property>
```
Add the following configuration to `$HADOOP_HOME/etc/hadoop/yarn-site.xml`:
```
<property>
  <description>
  The hostname of the Timeline service web application.
  </description>
  <name>yarn.timeline-service.hostname</name>
  <value>localhost</value>
</property>

<property>
  <description>
    The setting that controls whether yarn system metrics is
  published on the timeline server or not by RM.
  </description>
  <name>yarn.resourcemanager.system-metrics-publisher.enabled</name>
  <value>true</value>
</property>

<property>
  <description>
  Indicate to clients whether to query generic application
  data from timeline history-service or not. If not enabled then application
  data is queried only from Resource Manager.
  </description>
  <name>yarn.timeline-service.generic-application-history.enabled</name>
  <value>true</value>
</property>

<property>
  <description>
  Enables cross-origin support (CORS) for web services where
  cross-origin web response headers are needed. For example, javascript making
  a web services request to the timeline server.</description>
  <name>yarn.timeline-service.http-cross-origin.enabled</name>
  <value>true</value>
</property>
<property>
  <description>
  Store file name for leveldb timeline store. Defaults to ${hadoop.tmp.dir}/yarn/timeline. 
  </description>
  <name>yarn.timeline-service.leveldb-timeline-store.path</name>
  <value>/usr/local/hadoop-2.6.0/tmp/yarn/timeline</value>
</property>
```

###Depoly Tez UI

The build of Tez generates a war file of the distribution in the tez-ui/target. Copy the war to `$CATALINA_HOME/webapps`.

Start yarn timeline server:
```
$ $HADOOP_PREFIX/sbin/yarn-daemon.sh start timelineserver
```
Start Tomact:
```
$ sudo $CATALINA_HOME/bin/startup.sh
```
### View the web application
You can browse the web page with the address:
```
http://localhost:8080/tez-ui-0.7.0/
```
You can manage Tomact from:
```
http://localhost:8080/
```
![Image of tezUI running](https://github.com/willzxd/HadoopHiveTezInstall/blob/master/tezUI.png)
![Image of tezUIGraphView running](https://github.com/willzxd/HadoopHiveTezInstall/blob/master/tezUIGraphView.png)
##<a name="9"/> An example of bashrc file
```
# Java
export JAVA_HOME=/usr/lib/jvm/java-7-oracle

# MAVEN
export PATH=$PATH:/usr/local/apache-maven-3.3.3/bin
export MAVEN_POTS="-Xms256m -Xmx512"

# Protobuf
export PATH=$PATH:/usr/local/protobuf-2.5.0/bin

# Hadoop
export HADOOP_HOME=/usr/local/hadoop-2.6.0
export HADOOP_PREFIX=$HADOOP_HOME
export PATH=$PATH:${HADOOP_HOME}/bin:${HADOOP_HOME}/sbin

# Hive
export HIVE_HOME=/usr/local/hive-1.2.0
export HIVE_CONF_DIR=$HIVE_HOME/conf
export PATH=$PATH:$HIVE_HOME/bin


# Tez
export TEZ_INSTALL_DIR='/usr/local/tez-0.7.0'
export TEZ_CONF_DIR=${TEZ_INSTALL_DIR}/conf
export TEZ_JARS=$(echo "$TEZ_INSTALL_DIR"/*.jar | tr ' ' ':'):$(echo "$TEZ_INSTALL_DIR"/lib/*.jar | tr ' ' ':')
export HIVE_AUX_JARS_PATH="${TEZ_JARS}"

export HADOOP_CLASSPATH="${TEZ_CONF_DIR}:${TEZ_JARS}:${HADOOP_CLASSPATH}"

# Tomact
export CATALINA_HOME=/usr/local/apache-tomcat-8.0.23
```

##<a name="7"/>Using BigFrame

[BigFrame 'thoth'](https://github.com/bigframeteam/BigFrame/tree/thoth) BigFrame is a benchmark generator for big data analytics, which means that it can instantiate different kinds of benchmarks. Unlike the exsiting benchmarks, e.g TPCDS, HiveBench, etc. which are either micro-benchmarks or benchmarks in very specific domains, making them not fit into the big data enviroment taday, BigFrame can instead generate the bechmark tailored to a specific set of data and workload requirements.

For example, some enterprises may be grappling with increasing data volumes; with the variety and velocity of their data not being pressing concerns. Some other enterprises may be interested in benchmarking systems that can handle larger volumes and variety of data; but with the volume of unstructured data dominating that of the structured data by orders of magnitude. A third category of enterprises may be interested in understanding reference architectures for data analytics applications that need to deal with large rates of streaming data (i.e., large velocity of data).

BigFrame captures your requirement by the 3V's, namely, Volume, Variety and Velocity emphasized in the Big Data environment. Given the benchmark specification you provided, it will generate

* The set of data for initial load (with data loading utility)
* The refresh pattern for each data set (with refresh driver)
* The query stream (with query implementation and driver to run on different systems)
* The benchmark metrics

### Prequisite

Since BigFrame relies on hadoop to do the parallel data generation, it is a must to install hadoop beforehand. 

BigFrame requires:
* JDK 1.7 or later
* Hadoop 2.6.0 (other versions are not tested)
* Hive 1.2.0
* Tez 0.7.0
* MySQL 14.14 Distrib 5.5.43

To use the latest version of datahooks ([https://github.com/dukedbgroup/data-hooks](https://github.com/dukedbgroup/data-hooks))

To build BigFrame from source, execute the the command,
    
    sbt/sbt assembly

### Benchmark specification
You can tailor the specification to meet you special need by modifying the file

    conf/bigframe-core.xml

For example, to select an application domain to benchmark on, you can specify the corresponding domain name

	<property>
		<name>bigframe.application.domain</name>
		<value>BI</value>
		<description>
			Choose the application domain you want to benchmark on.
			Currently, supported applications are: BI
		</description>
	</property>

To specify the data volume, you can enter an number presented your data size,
	
	<property>
		<name>bigframe.datavolume</name>
		<value>10</value>
		<description>
			tiny: around 10GB
			small: around 100GB
			medium: around 1TB
			large: around 10TB
			extra large: around 100TB 
		</description>
	</property>

Besides the 3V's, you can also specify which engine the query will run on. Further more, if a query involves several data types, you can even tell BigFrame which engine will do the job for each specific data type. Now, it support hive, hive-on-Tez (can't hybrid with hive), hadoop and spark. For example,

	<property>
		<name>bigframe.queryengine.relational</name>
		<value>hive-tez</value>
	</property>
	
	<property>
		<name>bigframe.queryengine.graph</name>
		<value>hive-tez</value>
	</property>
	
	<property>
		<name>bigframe.queryengine.nested</name>
		<value>hive-tez</value>
	</property>

	<property>
		<name>bigframe.queryengine.text</name>
		<value>hive-tez</value>
	</property>

Of course, you need to install and setup the corresponding systems before actually run the queries, BigFrame will not do this job for you. 

### BigFrame Configuration

Before running BigFrame, you need to edit the `conf/config.sh` to set the following variables:

    HADOOP_HOME: By default, it tries to get it from the environment variables.
    HADOOP_CONF_DIR: By default, it is `$HADOOP_HOME/etc/hadoop`.
    HADOOP_SLAVES=By default, it is `$HADOOP_HOME/etc/hadoop/slaves`.
    TPCDS_LOCAL: A temp directory to store the imtermediate data for tpcds generator.

There are other variable related to the drivers. For example, you need to tell where BigFrame can find Spark if you want to run the benchmark on Spark. This can be done by specified the SPARK_HOME parameter as follow

    SPARK_HOME=/path/to/spark_home

You need to edit the `conf/hadoop-env.sh` to set the following variables:

    HDFS_ROOT_DIR: The HDFS Root Directory to store the generated data
    HADOOP_USERNAME: The username can access the HDFS_ROOT_DIR
    HIVE_HOME: The Hive HOME Directory
    ORC_SETTING: The Hive ORC File Setting
    HIVE_JDBC_SERVER: The Hive JDBC Server Address
    HIVE_USERNAME: The Hive UserName

You need to edit the `conf/thoth-config.sh` to set the following variables:

    METADATA_DB_NAME: The database name you want to store the profiling result
    METADATA_DB_CONNECTION: The connection address to connect to MySQL server
    METADATA_DB_USERNAME: The username that can write to the database you use.
    METADATA_DB_PASSWORD: The password of the username

You need to create the database with the same name in MySQL.

You may need to remove hsqldb-2.0.0.jar from your hadoop installation directory.


### Run BigFrame

After finish all the setup above, you can now run the BigFrame program. The first program you need to run is the data generator. To start the data generator, you can type the following command in BigFrame's root directory. 

    /bin/datagen -mode datagen

Then, it will try to generate the set of data you specified before. Be sure that you have started the HDFS and MapReduce Engine. 

You may need to copy the nested_data and graph_data to HDFS manually:
```
bin/hdfs dfs ­copyFromLocal $BIGFRAME_HOME/nested_data /user/hadoop/nested_data
bin/hdfs dfs ­copyFromLocal $BIGFRAME_HOME/graph_data /user/hadoop/graph_data
```

After the data generation finish, you can then run the benchmark queries by this command

	$HIVE_HOME/bin/hive --service hiveserver2 2>&1 &
    /bin/qgen -mode runqueries

It will prepare a set of queries based on your benchmark specification, and then run the queries on the system you specified.
The following image shows the Hive engine driver is running.
![Image of hive-mr running](https://github.com/willzxd/HadoopHiveTezInstall/blob/master/hiveMrRun.png)
You will see the INFO indicates that queries have finished.

##<a name="8"/> Compare pure hive-mr and pure hive-tez using BigFrame: A simple example

If you follow the previous instruction, you can use the default configuration of BigFrame to do the experiment. The only thing you need to do is to change the engine twice and run it twice.

![Image of result records in MySQL](https://github.com/willzxd/HadoopHiveTezInstall/blob/master/mysqlResult.png)

My computer has 8GB memory with Intel Core i7-2600 3.40GHz CPU. I generated 10GB data combined with the realtional data, graph data and nested data. I only use a single node cluster in the pseudo-Distributed mode. For Hive on Tez, I ran for three times. The first three results are 5.53072 min, 5.70933 min and 5.82627 min. It costs less than 6 minutes on average to finish all the queries. 

Modify hive-site.xml in `$HIVE_HOME/conf` to enable the vectorization :
```
<property>
    <name>hive.vectorized.execution.enabled</name>
    <value>true</value>
    <description>
      This flag should be set to true to enable vectorized mode of query execution.
      The default value is false.
    </description>
 </property>
 ```
 This results are 5.3 min and 5.5 min, which are a bit better. If we use a truely distributed cluster, the results should be more obvious.

 When use MapReduce framework, it will take a much longer time. The 6th record in the picture shows the result. It tooks 65.2135min to run all the queries under hive-mr mode. 

Before the workflows runs, BigFrame starts the Yarn listener that can record all the yarn metrics and store the records in MySql. The Yarn listener will automatically query the Yarn server by ResourceManager REST API and Nodemanager REST API. It consists by four listeners running seprately. They constantly (every 250ms or so) query for cluster metrics, scheduler metrics, application metrics and applications and containers metrics of every node. When certain metrics is changing, BigFrame will record the corresponding information with a timestamp to MySql. For example, the state of a application changes from RUNNING to FINISHED. The address of Yarn server can be changed in `resource/conf.properties`.

The tables of Yarn metrics are applications_metrics, cluster_metrics, scheduler_metrics, nodes_apps_metrics and nodes_containers_metrics.

In the applications_metrics table, it will record a whole pic of applications that are running or has finished. In every row, it will store the application ID, the metrics name, recording time and the value. 

![Image of result records in MySQL](https://github.com/willzxd/HadoopHiveTezInstall/blob/master/imgs/applications_metrics.png)
application metrics includes:
```
finishedTime
amContainerLogs
trackingUI
state
user
id
clusterId
finalStatus
amHostHttpAddress
progress
name
startedTime
elapsedTime
diagnostics
trackingURL
queue
allocatedMB
allocatedVCores
runningContainers
```
It is easy to use the data. For example, we can query like this:
```
mysql> select * from applications_metrics where AppId = 'application_1438376023026_0001' and MetricsName = 'runningContainers';
```
Then we get a whole picture of the number of running containers of application_1438376023026_0001.
![Image of result records in MySQL](https://github.com/willzxd/HadoopHiveTezInstall/blob/master/imgs/applicationsRunningContainers.png)

In the cluster_metrics table, you can find all the changes about cluster.
![Image of result records in MySQL](https://github.com/willzxd/HadoopHiveTezInstall/blob/master/imgs/cluster_metrics.png)
Metrics of cluster includes:
```
appsSubmitted
appsCompleted
appsPending
appsRunning
appsFailed
appsKilled
reservedMB
availableMB
allocatedMB
totalMB
reservedVirtualCores
availableVirtualCores
allocatedVirtualCores
totalVirtualCores
containersAllocated
containersReserved
containersPending
totalNodes
activeNodes
lostNodes
unhealthyNodes 
decommissionedNodes
rebootedNodes
```
In the scheduler_metrics table, you can find all the changes about cluster.
![Image of result records in MySQL](https://github.com/willzxd/HadoopHiveTezInstall/blob/master/imgs/scheduler_metrics.png)
Metrics of scheduler includes:
```
capacity
usedCapacity
maxCapacity
absoluteCapacity
absoluteMaxCapacity
absoluteUsedCapacity
numApplications
usedResources
queueName
state
queues
resourcesUsed
type
numActiveApplications
numPendingApplications
numContainers
maxApplications
maxApplicationsPerUser
maxActiveApplications
maxActiveApplicationsPerUser
userLimit
userLimitFactor
users
```

In the nodes_apps_metrics table, you can find which app is running on which nodes and which containers used by certain app.
Since we only have one node, the talbe looks like following.
![Image of result records in MySQL](https://github.com/willzxd/HadoopHiveTezInstall/blob/master/imgs/nodes_apps_metrics.png)
Metrics of applications in nodes includes:
```
id
user
state
containerids
```
In the nodes_apps_metrics table, you can get changes of all containers in every node.
Since we only have one node, the talbe looks like following.
![Image of result records in MySQL](https://github.com/willzxd/HadoopHiveTezInstall/blob/master/imgs/nodes_containers_metrics.png)
Metrics of containers in nodes includes:
```
id
state
nodeId
containerLogsLink
exitCode
diagnostics
totalMemoryNeededMB
totalVCoresNeeded
```