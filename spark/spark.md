
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

### Smart Data Sources

> Smart data sources are those that support data processing directly in their own engine--where the data resides--by preventing unnecessary data to be sent to Apache Spark. 

#### Apache Spark MongoDB

* One example is a relational SQL database with a smart data source. Data locality is made use of by letting the SQL database do the filtering of rows based on conditions. This is implemented in a Apache Spark MongoDB connector. The connector class needs implement the **PrunedFilteredScan** trait adding the **buildScan** method in order to support filtering on the data source itself. The code can remove columns and rows directly using the MongoDB API.

* This means that if in a PEP data has to be filtered, then this filter will be executed in the SQL statement on the RDBMS when the underlying data is read. This way, reading unnecessary data is avoided.

#### Apache Parquet
> It is another columnar-based data format used by many tools in the Hadoop ecosystem, such as Hive, Pig, and Impala. It increases performance using efficient compression, columnar layout, and encoding routines.

* The Parquet files are **smart data sources**, since projection and filtering can be performed at the storage level, eliminating disk reads of unnecessary data. 

* This can be seen in the **PushedFilters** and **ReadSchema** sections in the explained plan, where the **IsNotNull** operation and the **ReadSchema** projection on id, clientId, and familyName is directly performed at the read level of the Parquet files.

### Projection
> It means to use the DataFrame's select method to filter **columns** from the data. In SQL or relational algebra, this is called projection.

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
### SQL
* Since Apache Spark 2.0, the catalog API is used to create and remove temporary views from an internal meta store. This is necessary if you want to use SQL, because it basically provides the mapping between a virtual table name and a DataFrame or Dataset.

* Temporary views are stored in the SparkSession object, as persistent tables are stored in an external metastore.

* 

### RDD

* RDD is still the central data processing API where everything else (DataFrame, DataSet) builds on top. 

* RDD is discouraged to use unless there is a strong reason to do so for the following reasons:
  * RDDs, on an abstraction level, are equivalent to assembler or machine code when it comes to system programming
  * RDDs express how to do something and not what is to be achieved, leaving no room for optimizers
  * RDDs have proprietary syntax; SQL is more widely known

### DataFrame
> DataFrames are columnar data storage structures roughly equivalent to relational database tables. It allows for structured data APIs.

### Dataset
> It unifies the RDD and DataFrame APIs. It is basically a strongly typed version of DataFrames. Datasets are **statically typed** and avoid runtime type errors. 

* Static types allow for a lot of further performance optimization and also adds compile type safety to your applications.

* A DataFrame since Apache Spark 2.0 is nothing else but a **Dataset where the type is set to Row**. A DataFrame-equivalent Dataset would contain only elements of the Row type. This means that you actually lose the strongly static typing and fall back to a dynamic typing. Note that the difference between Datasets and DataFrame is that the Row objects are not static types as the schema can be created during runtime by passing it to the constructor using StructType objects.

* Datasets can be used only with Java and Scala. Dynamically typed languages such as Python or R are not capable of using Datasets because there isn't a concept of strong, static types in the language. 

* Both APIs (DataFrame and DataSet) are usable with SQL or a relational API. 

### Summary
* The DataFrame and DataSet APIs are much faster than RDDs because Apache SparkSQL provides such dramatic performance improvements over the RDD API.
* Whenever possible, use Datasets because their static typing makes them faster. As long as you are using statically typed languages such as Java or Scala, you are fine. Otherwise, you have to stick with DataFrames.

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

# The Catalyst Optimizer

### Idea
> Catalyst creates a **Logical Execution Plan (LEP)** from a SQL query and optimizes this LEP to create multiple **Physical Execution Plans (PEPs)**. Based on statistics, Catalyst chooses the best PEP to execute. This is very similar to **cost-based optimizers** in **Relational Data Base Management Systems (RDBMs)**.

* Catalyst takes your high-level program and transforms it into efficient calls on top of the RDD API.

### Catalyst Optimization Procedure

* First of all, it has to be understood that it doesn't matter if a DataFrame, the Dataset API, or SQL is used. The Apache Spark SQL parser returns an **abstract syntax tree (AST)**. They all result in the same tree-based structure called **Unresolved Logical Execution Plan (ULEP)**. The ULEP basically reflects the structure of an AST.

> A QueryPlan is unresolved if the column names haven't been verified and the column types haven't been looked up in the catalog. 

* ULEP is mainly composed of sub-types of the **LeafExpression** objects, which are bound together by the **Expression** objects, therefore forming a tree of the **TreeNode** objects since all these objects are sub-types of **TreeNode**. Overall, this data structure is a **LogicalPlan**, which is therefore reflected as a **LogicalPlan** object. Note that **LogicalPlan** extends **QueryPlan**, and **QueryPlan** itself is **TreeNode** again. In other words, **LogicalPlan** is nothing else than a set of **TreeNode** objects.

* From ULEP to RLEP
  * The first thing that is checked is if the referred relations exist in the catalog. This means all table names and fields expressed in the SQL statement or relational API have to exist. 
  * If the table (or relation) exists, the column names are verified. 
  * In addition, the column names that are referred to multiple times are given an alias in order to read them only once. This is already a first stage optimization taking place here. 
  * Finally, the data types of the columns are determined in order to check if the operations expressed on top of the columns are valid. So for example taking the sum of a string doesn't work and this error is already caught at this stage. 
  * The result of this operation is a Resolved Logical Execution Plan (LEP).

* A **Resolved Logical Execution Plan (RLEP)** is then transformed multiple times, until it results in an Optimized Logical Execution Plan. LEPs don't contain a description of how something is computed, but only what has to be computed. 

* How to optimized the RLEP?
  * This is done using a set of transformation rules. Since a LEP is just a tree, the rules transform from one tree to another. This is done iteratively until no rule fires anymore and keeps the LEP stable and the process is finished. The result of this step is an **Optimized Logical Execution Plan**.

* The **optimized LEP** is transformed into multiple **Physical Execution Plans (PEP)** using so-called strategies.
  * PEPs are execution plans that have been completely resolved. This means that a PEP contains detailed instructions to generate the desired result. 
  * Strategies are used to optimize selection of join algorithms based on statistics. In addition, rules are executed for example to pipeline multiple operations on an RDD into a single, more complex operation.
  * The multiple PEPs all will return the exact same result.

* An optimal PEP is selected to be executed using a cost model (to minimize execution time) by taking statistics and heuristics about the Dataset to be queried into account. 
  * In case the data source supports it, operations are pushed down to the source, namely for filtering (predicate) or selection of attributes (projection).
  * The main idea of predicate push-down is that parts of the AST are not executed by Apache Spark but by the data source itself. So for example filtering rows on column names can be done much more efficient by a relational or NoSQL database since it sits closer to the data and therefore can avoid data transfers between the database and Apache Spark. Also, the removal of unnecessary columns is a job done more effectively by the database.

* Code Generation
  * Since Apache Spark runs on the **Java Virtual Machine (JVM)**, this allows byte code to be created and modified during runtime and optimized more.
  * Normally, an expression (e.g. 'a+b') had to be **interpreted** by the JVM for each row of the Dataset. It would be nice if we could generate the JVM ByteCode for this expression on the fly such that less code has to be interpreted, which speeds things up. 
  * This is possible using a Scala feature called **Quasiquotes**, which basically allows an arbitrary string containing Scala code to be compiled into ByteCode on the fly, if it starts with `q`.
  
* Example of Join
  * BroadCastHashJoin, but in reality, is a join spans partitions over - potentially - multiple physical nodes. Therefore, the two tree branches are executed in parallel and the results are shuffled over the cluster using hash bucketing based on the join predicate.


* Note that the final execution takes place on RDD objects.

### Put the picture of execution plan transformation here


# Project Tungsten
> Project Tungsten is the core of the Apache Spark execution engine, aims at improving performance at the CPU and main memory level. 


## JVM and Garbage Collection (GC)
* The JVM Garbage Collector is in support of the whole object's life cycle management the JVM provides. Whenever you see the word new in Java code, memory is allocated in a JVM memory segment called heap.

* Java completely takes care of memory management and it is impossible to overwrite memory segments that do not belong to you (or your object). So if you write something to an object's memory segment on the heap (for example by updating a class property value of type Integer, you are changing 32 bit on the heap) you don't use the actual heap memory address for doing so but you use the reference to the object and either access the object's property or use a setter method.

* How to free up memory in Java? Java Garbage collector continuously monitors how many active references to a certain object on the heap exist and once no more references exist those objects are destroyed and the allocated memory is freed.

* References to an object can either exist within objects on the heap itself or in the stack. The stack is a memory segment where variables in method calls and variables defined within methods reside. They are usually short lived whereas data on the heap might live for longer.

* The Garbage Collector is a highly complex component and has been optimized to support **Online Transaction Processing (OLTP)** workloads whereas Apache Spark concentrates on **Online Analytical Processing (OLAP)**. There is a clash between optimizing a garbage collector for both disciplines at the same time since the object life cycle is very different between those two disciplines.

## Tungsten
* The main idea behind Tungsten is to get rid of the JVM Garbage Collector. Tungsten bypasses the managed (and safe) memory management system which the JVM provides and uses the classes from the sun.misc.Unsafe package, which allows Tungsten to manage memory layout on its behalf.

### UnsafeRow Object
* Tungsten uses org.apache.spark.sql.catalyst.expressions.UnsafeRow, which is a binary representation of a row object. An UnsafeRow object consists of three regions.
  * Null bit set
  * Values (fixed length)
  * Values (variable length)
  
* All regions, and also the contained fields within the regions, are 8-byte aligned. Therefore, individual chunks of data perfectly fit into 64 bit CPU registers. This way, a compare operation can be done in a single machine instruction only. In addition, 8-byte stride memory access patterns are very cache-friendly.
  
#### The Null Bit Set Region
* In this region, for each field contained in the row, a bit is reserved to indicate whether it contains a null value or not. This is very useful for filtering, because only the bit set has to be read from memory, omitting the actual values. The number of fields and the size of this region are reflected as individual properties in the org.apache.spark.sql.catalyst.expressions.UnsafeRow object. Therefore, this region can be of variable size as well, which is an implicit requirement since the number of fields varies as well.







# References
1. [Spark Internals](https://github.com/JerryLead/SparkInternals)
2. Mastering Apache Spark 2.x - 2nd Edition
