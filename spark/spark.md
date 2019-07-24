
# Concepts
### What is Apache Spark?
> Apache Spark is an in-memory, cluster-based, unified analytics/computing engine and a set of libraries for large-scale parallel data processing.

### Characteristics of Apache Spark
1. *A Unified Data Analytics System*: supports a wide range of data analytics tasks over the same computing engine and with a consistent set of APIs.
   * Data loading
   * SQL queries
   * Machine learning
   * Streaming computation
1. *An Analytics/Computing Engine not a Persistence Store*: Compare with Hadoop Architecture, which has HDFS as storage and MapReduce as computing engine.
1. *Libraries*:
   * Spark Core
   * Spark SQL
   * MLlib
   * SparkML (since 1.6.0)
   * Spark Streaming/Structured Streaming
   * GraphX
   * Third Party Libraries not shipped with Spark
     * Apache SystemML
     * DeepLearning4j
     * H2O
1. High Performance for both batch and streaming data: DAG scheduler, a query optimizer, and a physical execution engine. Spark is scalable, massively parallel, and in-memory execution.
1. Ease of Use: Spark offers over 80 high-level operators that make it easy to build parallel apps and in various languages
   * Scala
   * Python
   * Java
   * R
   * SQL
1. Runs Everywhere: Spark runs on Hadoop, Apache Mesos, Kubernetes, standalone, or in the cloud. It can access diverse data sources.
1. Open Source

### General Description of Spark Job Execution
Spark is a distributed programming model in which the user specifies transformations. Multiple transformations build up a directed acyclic graph (DAG) of instructions. An action begins the process of executing that graph of instructions, as a single job, by breaking it down into stages and tasks to execute across the cluster. 

Spark is effectively a programming language of its own. 

Internally, Spark uses an engine called **Catalyst** that maintains its own type information through the planning and processing of work. 
Spark will convert an expression written in an input language (e.g. Python) to Spark's internal Catalyst representation of that same type 
information. It then will operate on that internal representation.

### The Spark Context
> The Spark context, can be defined via a Spark configuration object and Spark URL. The Spark context connects to the Spark cluster manager.

### SparkSession
> SparkSession is created from the Spark context and provides the means to load and save data files of different types using DataFrames and Datasets and manipulate columnar data with SQL, among other things. It can be used for the following functions:

* Executing SQL via the sql method
* Registering user-defined functions via the udf method
* Caching
* Creating DataFrames
* Creating Datasets

### Apache Parquet
> It is another columnar-based data format used by many tools in the Hadoop ecosystem, such as Hive, Pig, and Impala. It increases performance using efficient compression, columnar layout, and encoding routines.

### Projection
> It means to use the DataFrame's select method to filter columns from the data. In SQL or relational algebra, this is called projection.

### Task
> Stages in Spark consist of tasks. Each task corresponds to a combination of blocks of data and a set of transformations that will run on a single executor. 

* For a Spark application, a task is the smallest unit of work that Spark sends to an executor

* If there is one big partition in our dataset, we will have one task. If there are 1,000 little partitions, we will have 1,000 tasks that can be executed in parallel. A task is just a unit of computation applied to a unit of data (the partition). Partitioning your data into a greater number of partitions means that more can be executed in parallel. This is not a panacea, but it is a simple place to begin with optimization

* A task's execution time can be broken up as Scheduler Delay + Deserialization Time + Shuffle Read Time (optional) + Executor Runtime + Shuffle Write Time (optional) + Result Serialization Time + Getting Result Time.




### Partitions
http://stackoverflow.com/questions/10666488/what-are-success-and-part-r-00000-files-in-hadoop
https://www.ibm.com/support/knowledgecenter/en/SSZU2E_2.3.0/performance_tuning/application_spark_parameters.html

### JVM
Java Virtual Machine (JVM) is a general-purpose byte code execution engine.

* Note that the Executor and Driver have bidirectional communication all the time, so network-wise, they should also be sitting close together

# Cluster Management
## Edge Nodes
> Edge nodes in the cluster will be client-facing, on which reside the client-facing components such as *Hadoop NameNode* or perhaps the *Spark master*. Majority of the big data cluster might be behind a firewall. The edge nodes would then reduce the complexity caused by the firewall as they would be the only points of contact accessible from outside.

Generally, the edge nodes will need greater resources than the cluster processing nodes within the firewall. When many Hadoop ecosystem components are deployed on the cluster, all of them will need extra memory on the master server. You should monitor edge nodes for resource usage and adjust in terms of resources and/or application location as necessary. YARN, for instance, is taking care of this.

## Cluster Manager
> The Spark cluster manager allocates resources (e.g. executors) across the worker nodes for the application. It copies the application JAR file to the workers and finally allocates tasks.

## Cluster Manager Options
### Local Mode
You can run spark application locally on a single machine.

### Standalone Mode 
This simple cluster manager currently supports only **FIFO (first-in first-out)** scheduling.

### Apache YARN

### Apache Mesos
It allows multiple frameworks to share a cluster by managing and scheduling resources. 

# Cloud Deployment
## Three abstraction levels of cloud systems
### Infrastructure as a Service (IaaS)
The new way to do IaaS is Docker and Kubernetes, basically providing a way to automatically set up an Apache Spark cluster within minutes.

### Platform as a Service (PaaS)
PaaS takes away from you the burden of installing and operating an Apache Spark cluster because this is provided as a service.

### Software as a Service (SaaS)




1. Can be one of Standalone cluster manager, YARN, or Mesos etc.
1. The manager starts a master and many worker nodes.
1. When it comes time to actually run a Spark Application, we request resources from the cluster manager to run it. 
Depending on how our application is configured, this can include a place to run the Spark driver or might be just resources for the executors for our Spark Application. 
Over the course of Spark Application execution, the cluster manager will be responsible for managing the underlying machines that our application is running on.



# Data APIs
### RDD
### DataFrame
> DataFrames are columnar data storage structures roughly equivalent to relational database tables. 

### Dataset
> It unifies the RDD and DataFrame APIs. Datasets are statically typed and avoid runtime type errors. Therefore, Datasets can be used only with Java and Scala. 

# Performance

### The Cluster Structure
* The size and structure of your big data cluster is going to affect performance. 
* Shared vs. Unshared cluster: if you have a cloud-based cluster, your IO and latency will suffer in comparison to an unshared hardware cluster. 
* The positioning of cluster components on servers may cause resource contention. 

For instance, think carefully about locating Hadoop NameNodes, Spark servers, Zookeeper, Flume, and Kafka servers in large clusters. With high workloads, you might consider segregating servers to individual systems. You might also consider using an Apache system such as Mesos that provides better distributions and assignment of resources to the individual processes.

* Potential parallelism. 

The greater the number of workers in your Spark cluster for large Datasets, the greater the opportunity for parallelism. One rule of thumb is one worker per hyper-thread or virtual core respectively.

### Alternatives to Hadoop Distributed File System (HDFS)
* HDFS is designed as a write once, read many filesystem. It runs in a Java Virtual Machine (JVM) that in turn runs as an operating system process. IBM's GPFS (General Purpose File System) have improved performance on these aspects.
* **Ceph** is an open source alternative to a distributed, fault-tolerant, and self-healing filesystem for commodity hard drives like HDFS.

* **Cassandra** is not a filesystem but a NoSQL key value store and is tightly integrated with Apache Spark and is a valid and powerful alternative to HDFS--or even to any other distributed filesystem--especially as it supports predicate push-down using ApacheSparkSQL and the Catalyst optimizer.

### Data Locality
* The key for good data processing performance is avoidance of network transfers. This is less relevant for tasks with high demands on CPU and low I/O, but for low demand on CPU and high I/O demand data processing algorithms, this still holds.
* HDFS is one of the best ways to achieve data locality as chunks of files are distributed on the cluster nodes, in most of the cases, using hard drives directly attached to the server systems. This means that those chunks can be processed in parallel using the CPUs on the machines where individual data chunks are located in order to avoid network transfer.

* Another way to achieve data locality is using ApacheSparkSQL. Depending on the connector implementation, SparkSQL can make use of data processing capabilities of the source engine. So for example when using MongoDB in conjunction with SparkSQL parts of the SQL statement are preprocessed by MongoDB before data is sent upstream to Apache Spark.

### Memory
* Consider the level of physical memory available on your Spark worker nodes. Can it be increased? Check on the memory consumption of operating system processes during high workloads in order to get an idea of free memory. Make sure that the workers have enough memory.
* Consider data partitioning. Can you increase the number of partitions? As a rule of thumb, you should have at least as many partitions as you have available CPU cores on the cluster. Use the repartition function on the RDD API.
* Can you modify the storage fraction and the memory used by the JVM for storage and caching of RDDs? Workers are competing for memory against data storage. Use the Storage page on the Apache Spark user interface to see if this fraction is set to an optimal value. Then update the following properties:
  * spark.memory.fraction
  * spark.memory.storageFraction
  * spark.memory.offHeap.enabled=true
  * spark.memory.offHeap.size
  
* Consider using Parquet as a storage format, which is much more storage effective than CSV or JSON
* Consider using the DataFrame/Dataset API instead of the RDD API as it might resolve in more effective executions

### Coding
* Filter your application-based data early in your ETL cycle. 

### [Spark Documentation](http://spark.apache.org/docs/latest/tuning.html)

# Project Tungsten


# The Catalyst Optimizer
### Idea
> Catalyst creates a **Logical Execution Plan (LEP)** from a SQL query and optimizes this LEP to create multiple **Physical Execution Plans (PEPs)**. Based on statistics, Catalyst chooses the best PEP to execute. This is very similar to **cost-based optimizers** in **Relational Data Base Management Systems (RDBMs)**.







# References
1. [Spark Internals](https://github.com/JerryLead/SparkInternals)
2. Mastering Apache Spark 2.x - 2nd Edition
