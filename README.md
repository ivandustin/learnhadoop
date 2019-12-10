# Learn Hadoop Ecosystem



[TOC]

## Apache Hadoop



### Components

1. NameNode
2. HDFS
3. MapReduce
4. YARN



### Configurations

1. core-site.xml - NameNode

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
   <configuration>
       <property>
           <name>fs.default.name</name>
           <value>hdfs://0.0.0.0:19000</value>
       </property>
   </configuration>
   ```

2. hdfs-site.xml - HDFS

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
   <configuration>
      <property>
         <name>dfs.replication</name>
         <value>1</value>
      </property>
      <property>
         <name>dfs.namenode.name.dir</name>
         <value>E:\hdfs\namenode</value>
      </property>
      <property>
         <name>dfs.datanode.data.dir</name>
         <value>E:\hdfs\datanode</value>
      </property>
   	<property>
   		<name>dfs.webhdfs.enabled</name>
   		<value>true</value>
   	</property>
   	<property>
   		<name>dfs.support.append</name>
   		<value>true</value>
   	</property>
   	<property>
   		<name>dfs.support.broken.append</name>
   		<value>true</value>
   	</property>
   </configuration>
   ```

3. mapred-site.xml - MapReduce

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
   <configuration>
      <property>
         <name>mapreduce.job.user.name</name>
         <value>%USERNAME%</value>
      </property>
      <property>
         <name>mapreduce.framework.name</name>
         <value>yarn</value>
      </property>
      <property>
         <name>yarn.apps.stagingDir</name>
         <value>/user/%USERNAME%/staging</value>
      </property>
      <property>
         <name>mapreduce.jobtracker.address</name>
         <value>local</value>
      </property>
   </configuration>
   ```

4. yarn-site.xml - YARN

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <configuration>
       <property>
           <name>yarn.nodemanager.aux-services</name>
           <value>mapreduce_shuffle</value>
       </property>
       <property>
           <name>yarn.nodemanager.auxservices.mapreduce.shuffle.class</name>
           <value>org.apache.hadoop.mapred.ShuffleHandler</value>
       </property>
       <property>
           <name>yarn.scheduler.maximum-allocation-mb</name>
           <value>2048</value>
       </property>
       <property>
           <name>yarn.nodemanager.disk-health-checker.max-disk-utilization-per-disk-percentage</name>
           <value>98.5</value>
       </property>
       <property>
           <name>yarn.log-aggregation-enable</name>
           <value>true</value>
       </property>
   </configuration>
   ```



### Pre-requisites

1. Java - JDK 8
2. Hadoop - v2.9.1



### Environment Variables

1. JAVA_HOME
2. HADOOP_HOME
3. PATH - `/bin` and `/sbin`



### Quick Setup

1. Install pre-requisites.

2. Setup environment variables.

3. Configure Hadoop.

4. Format HDFS.

   ```bash
   bin/hadoop namenode -format
   ```

5. Start all Hadoop daemons.

   ```bash
   sbin/start-dfs.sh
   sbin/start-yarn.sh
   ```
6. Verify the health of NameNode by accessing http://localhost:50070/dfshealth.html. Make sure there are _live_ nodes and **no** _dead_ nodes.

7. Verify the health of ResourceManager by accessing http://localhost:8088. Make sure there are **no** _unhealthy_ nodes and there are _active_ nodes.

   If there are unhealthy nodes due to disk utilization percentage exceeds YARN's default value of 90.0%, you can increase the threshold on `yarn-site.xml` with the following:
   
   ```xml
   <property>
       <name>yarn.nodemanager.disk-health-checker.max-disk-utilization-per-disk-percentage</name>
       <value>98.5</value>
   </property>
   ```
   
   Restart YARN by `stop-yarn` and `start-yarn`.



### Hadoop Daemons

1. HDFS - NameNode
2. HDFS - DataNode
3. YARN - ResourceManager
4. YARN - NodeManager



### Concepts

1. Node Master (HDFS NameNode and YARN ResourceManager)
2. Slave Nodes (HDFS DataNode and YARN NodeManager)



## Hadoop YARN



### Concepts

1. Container

   A container is a place in a node where the programs are run. It has a minimum and maximum memory allocation, the `yarn.scheduler.minimum-allocation-mb` and `yarn.scheduler.maximum-allocation-mb` respectively.

2. NodeManager

   This manages a single node in a cluster. You can specify the total amount of memory a particular node can be used using `yarn.nodemanager.resource.memory-mb` configuration.

3. ResourceManager

   This manages all the resources available in the cluster.

4. ApplicationMaster

   It negotiates resources with the ResourceManager for the NodeManager.



## Apache Spark

Spark is a general purpose cluster computing system. It can deploy and run parallel applications on clusters ranging from a single node to thousands of distributed nodes. Spark was originally designed to run Scala applications, but also supports Java, Python and R.

Spark can work standalone or using Hadoop's YARN for cluster computing. It is ideal to use YARN for Spark.



### Modes

1. Cluster Mode - runs everything inside the cluster.
2. Client Mode - runs the computation on the client, such as your laptop.



### Pre-requisites

1. Hadoop HDFS and YARN - v2.9.1
2. Spark - v2.4.3
3. Above 4GB RAM



###Environment Variables

1. HADOOP_CONF_DIR
2. SPARK_HOME
3. PATH - `/bin` and `/sbin`



### Configurations

1. spark-defaults.conf



### Integrating with YARN

1. Setup environment variables.

2. Edit `spark-defaults.conf` and add the following.

   ```txt
   spark.master    yarn
   ```



### Memory Allocation

Allocation of Spark containers to run in YARN containers may fail if memory allocation is not configured properly. For nodes with less than 4G RAM, the default configuration is not adequate and may trigger swapping and poor performance, or even the failure of application initialization due to lack of memory.

Be sure to understand how Hadoop YARN manages memory allocation before editing Spark memory settings so that your changes are compatible with your YARN cluster’s limits.



#### Give Your YARN Containers Maximum Allowed

If the memory requested is above the maximum allowed, YARN will reject creation of the container, and your Spark application won’t start.

1. Get the value of `yarn.scheduler.maximum-allocation-mb` in `$HADOOP_CONF_DIR/yarn-site.xml`. This is the maximum allowed value, in MB, for a single container.
2. Make sure that values for Spark memory allocation, configured in the following section, are below the maximum.

This guide will use a sample value of `1536` for `yarn.scheduler.maximum-allocation-mb`. If your settings are lower, adjust the samples with your configuration.



#### Configure the Spark Driver Memory Allocation in Cluster Mode

In **cluster mode**, the Spark Driver runs inside YARN Application Master. The amount of memory requested by Spark at initialization is configured either in `spark-defaults.conf`, or through the command line.

Set the default amount of memory allocated to Spark Driver in cluster mode via `spark.driver.memory` (this value defaults to `1G`). To set it to `512MB`, edit the file: `$SPARK_HOME/conf/spark-defaults.conf`

```txt
spark.driver.memory    512m
```



#### Configure the Spark Application Master Memory Allocation in Client Mode

In **client mode**, the Spark driver will not run on the cluster, so the above configuration will have no effect. A YARN Application Master still needs to be created to schedule the Spark executor, and you can set its memory requirements.

Set the amount of memory allocated to Application Master in client mode with `spark.yarn.am.memory` (default to `512M`) in `$SPARK_HOME/conf/spark-defaults.conf` file.

```txt
spark.yarn.am.memory    512m
```



### Concepts

![Spark Architecture](https://i.stack.imgur.com/cwrMN.png)

Spark uses a master/slave architecture. As you can see in the figure, it has one central coordinator (Driver) that communicates with many distributed workers (executors). The driver and each of the executors run in their own Java processes.

1. Drivers

   The driver is the process where the main method runs. First it converts the user program into tasks and after that it schedules the tasks on the executors.

   A Driver is your main program.

2. Tasks

   The Driver breaks up the program into small sub-programs called Tasks.

3. Executors

   Executors are worker nodes' processes in charge of running individual tasks in a given Spark job. They are launched at the beginning of a Spark application and typically run for the entire lifetime of an application. Once they have run the task they send the results to the driver. They also provide in-memory storage for RDDs that are cached by user programs through Block Manager.

   An Executor contains multiple Tasks that runs in a node.

4. Worker Node

   This is a single node in the cluster. Communicates to Cluster Manager about the availability of its resources.

5. Cluster Manager

   Manages all the nodes (Worker Nodes) in the cluster. Receives Executors/Tasks from the Driver and send them to the nodes.

6. RDD - Resilient Distributed Datasets

   This is the fundamental data structure of Spark that allows reliable distributed data processing.

7. Spark SQL
8. Spark Datasets
9. Spark DataFrames

#### Flow of Execution

With this in mind, when you submit an application to the cluster with spark-submit this is what happens internally:

1. A standalone application starts and instantiates a `SparkContext` instance (and it is only then when you can call the application a driver).
2. The driver program ask for resources to the cluster manager to launch executors.
3. The cluster manager launches executors.
4. The driver process runs through the user application. Depending on the actions and transformations over RDDs task are sent to executors.
5. Executors run the tasks and save the results.
6. If any worker crashes, its tasks will be sent to different executors to be processed again. In the book "Learning Spark: Lightning-Fast Big Data Analysis" they talk about Spark and Fault Tolerance:

> Spark automatically deals with failed or slow machines by re-executing failed or slow tasks. For example, if the node running a partition of a map() operation crashes, Spark will rerun it on another node; and even if the node does not crash but is simply much slower than other nodes, Spark can preemptively launch a “speculative” copy of the task on another node, and take its result if that finishes.

7. With `SparkContext.stop()` from the driver or if the main method exits/crashes all the executors will be terminated and the cluster resources will be released by the cluster manager.



#### Architecture

1. Spark Core - distributed data processing
2. Spark SQL - port of Apache Hive to Apache Spark, for SQL
3. Spark MLib - machine learning library
4. Spark Streaming API - real time processing with streaming data, like Apache Kafka
5. Spark GraphX - graph computation



#### Computation

Spark computes using RDD. The RDD supports two types of operations:

1. Transformations - are operations (such as map, filter, join, union, and so on) that are performed on an RDD and which yield a new RDD containing the result.
2. Actions - are operations (such as reduce, count, first, and so on) that return a value after running a computation on an RDD.



### Submitting a Spark Program to the YARN Cluster

Applications are submitted with the `spark-submit` command. The Spark installation package contains sample applications, like the parallel calculation of *Pi*, that you can run to practice starting Spark jobs.

To run the sample *Pi* calculation, use the following command:

```
spark-submit --deploy-mode client \
               --class org.apache.spark.examples.SparkPi \
               $SPARK_HOME/examples/jars/spark-examples_2.11-2.2.0.jar 10
```

The first parameter, `--deploy-mode`, specifies which mode to use, `client` or `cluster`.

To run the same application in cluster mode, replace `--deploy-mode client`with `--deploy-mode cluster`.



###Monitor Spark Applications

When you submit a job, Spark Driver automatically starts a web UI on port `4040` that displays information about the application. However, when execution is finished, the Web UI is dismissed with the application driver and can no longer be accessed.

Spark provides a History Server that collects application logs from HDFS and displays them in a persistent web UI. The following steps will enable log persistence in HDFS:

1. Edit `$SPARK_HOME/conf/spark-defaults.conf` and add the following lines to enable Spark jobs to log in HDFS:

   ```txt
   spark.eventLog.enabled  true
   spark.eventLog.dir hdfs://node-master:9000/spark-logs
   ```

2. Create the log directory in HDFS:

   ```
   hdfs dfs -mkdir /spark-logs
   ```

3. Configure History Server related properties in `$SPARK_HOME/conf/spark-defaults.conf`:

   ```txt
   spark.history.provider            org.apache.spark.deploy.history.FsHistoryProvider
   spark.history.fs.logDirectory     hdfs://node-master:9000/spark-logs
   spark.history.fs.update.interval  10s
   spark.history.ui.port             18080
   ```

   You may want to use a different update interval than the default `10s`. If you specify a bigger interval, you will have some delay between what you see in the History Server and the real time status of your application. If you use a shorter interval, you will increase I/O on the HDFS.

4. Run the History Server:

   ```
   $SPARK_HOME/sbin/start-history-server.sh
   ```

5. Repeat steps from previous section to start a job with `spark-submit` that will generate some logs in the HDFS.

6. Access the History Server by navigating to [http://node-master:18080](http://node-master:18080/) in a web browser:



### Run the Spark Shell

The Spark shell provides an interactive way to examine and work with your data.

1. Put some data into HDFS for analysis. This example uses the text of *Alice In Wonderland* from the Gutenberg project:

   ```
   cd /home/hadoop
   wget -O alice.txt https://www.gutenberg.org/files/11/11-0.txt
   hdfs dfs -mkdir inputs
   hdfs dfs -put alice.txt inputs
   ```

2. Start the Spark shell:

   ```
   spark-shell
   
   var input = spark.read.textFile("inputs/alice.txt")
   // Count the number of non blank lines
   input.filter(line => line.length()>0).count()
   ```



### Quick Setup

1. Make sure HDFS and YARN are running and properly configured.

2. Make sure the memory allocation of YARN containers are fine. Configure `yarn-site.xml` file.

   ```xml
   <property>
       <name>yarn.scheduler.maximum-allocation-mb</name>
       <value>2048</value>
   </property>
   ```

   The above gives 2GB maximum memory a YARN container can allocate.

3. Configure Spark by editing `spark/conf/spark-defaults.conf` file.

   ```txt
   spark.master    		yarn
   spark.driver.memory    	512m
   spark.executor.memory  	512m
   ```

   The `spark.master yarn` tells Spark that we will use YARN as the cluster manager. It also assigns maximum memory for the Spark Driver and Spark Executor.

   Make sure that the memory size does not exceed YARN container's maximum allocation memory.

4. Verify the setup by running a sample program.

   ```shell
   spark-submit --deploy-mode cluster --class org.apache.spark.examples.SparkPi   %SPARK_HOME%/examples/jars/spark-examples_2.11-2.4.3.jar 10
   ```

   Wait for the command to finish.

   To verify that it worked, go to YARN's ResourceManager Web UI at http://localhost:8088/ and check if the application status. If it failed, check the logs provided in the Web UI, or if your YARN's log aggregation is enabled, check the log using the following:

   ```shell
   yarn logs -applicationId <APP_ID> >log.txt
   ```

   You can obtain the App ID in the Web UI, for example: `application_1575031348949_0001`.



### Hello World

Apache Spark supports Scala, Java, and Python programming languages.

To run a program in a cluster, you use the `spark-submit` program.

To launch a Python shell, type `pyspark --master local`. This runs an interactive Python shell in your local machine. Running an interactive shell is not supported in cluster mode.

Here is an example program that reads a CSV file from HDFS in `pyspark`.

```python
df = spark.read.csv("hdfs:///user/ibilon/test.csv")
df.show()
```

The above program reads CSV from HDFS, returns a Spark DataFrame, and show the contents using the `.show()` method.

> **Note:** Pandas DataFrame is different from Spark DataFrame.

