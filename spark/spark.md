
## Table of Contents
* [Concepts](#concepts)
  * [What is Apache Spark?](#what-is-apache-spark)
* [Cluster Management](#cluster-management)
* [Cloud Deployment](#cloud-deployment)
* [Data APIs](#data-apis)
* [The Catalyst Optimizer](#the-catalyst-optimizer)
* [Project Tungsten](#project-tungsten)
* [Deployment](#deployment)
* [References](#references)


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
Everything on PaaS is a service, ready to be consumed, and the operation of components is done by the cloud provider. 

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
> DataFrames are columnar data storage structures roughly equivalent to relational database tables, which is backed by a RDD[Row] object. It allows for structured data APIs.

* Unfortunately, while the wrapping of values into the Row objects has advantages for memory consumption (less memory footprint than RDD), it has disadvantages for execution performance (Probably longer execution time than RDD). Fortunately, when using Datasets, this performance loss is mitigated and we can take advantage of both the fast execution time and the efficient usage of memory.

* Why are we using DataFrames and Datasets at all if RDDs are faster? We could also just put additional memory to the cluster. Note that although this particular execution (cache a list of integers and count) on an RDD runs faster, Apache Spark jobs are very rarely composed only out of a single operation on an RDD. In theory you could write very efficient Apache Spark jobs on RDDs only, but actually re-writing your Apache Spark application for performance tuning will take a lot of time. So the best way is to use DataFrames and Datasets, to make use of the Catalyst optimizer, in order to get efficient calls to the RDD operations generated.

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
### [HDFS Data Locality](http://www.hadoopinrealworld.com/data-locality-in-hadoop/)

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

#### The Fixed Length Values Region

* This region stores two things. 
  * Values fitting into the 8 bytes - called fixed length values like long, double, or int
  * In the event the value of a particular field doesn't fit into that chunk only a pointer or a reference is stored. This reference points to a byte offset in the variable length values region. In addition to the pointer, the length (in bytes) of that field is also stored. Both integer values are combined into a long value. This way, two 4-byte integer values are stored in one 8-byte field. Again, since the number of fields is known, this region is of variable length and reserves 8 bytes for each field.

#### The Variable Length Values Region
* It contains variable-sized fields like strings.

### BytesToBytesMap
#### Drawbacks of java.util.HashMap
* Memory overhead due to usage of objects as keys and values
* Very cache-unfriendly memory layout
* Impossible to directly address fields by calculating offsets

This means that simple but important things, such as sequential scans, result in very random and cache-unfriendly memory access patterns. Therefore, Tungsten has introduced a new data structure called BytesToBytesMap, which has improved the memory locality and has led to less space overhead by avoiding the usage of heavyweight Java objects, which has improved performance. Sequential scans are very cache friendly, since they access the memory in sequence.

### Cache-Friendly Memory Layout of Data
* Memory on a modern computer system is addressed using 64 bit addresses pointing to 64 bit memory blocks. Remember, Tungsten tries to always use 8-byte Datasets which perfectly fit into these 64-bit memory blocks.

* So between your CPU cores and main memory, there is a hierarchical list of L1, L2, and L3 caches-with increasing size. Usually, L3 is shared among all the cores. If your CPU core requests a certain main memory address to be loaded into the CPU core's register (a register is a memory area in your CPU core) - this happens by an explicit machine code (assembler) instruction - then first the L1-3 cache hierarchy is checked to see if it contains the requested memory address.

* We call data associated with such an address a memory page. If this is the case, then main memory access is omitted and the page is directly loaded from the L1, L2, or L3 cache. Otherwise, the page is loaded from main memory, resulting in higher latency. The latency is so high that the CPU core is waiting (or executing other work) for multiple CPU clock cycles, until the main memory page is transferred into the CPU core's register. In addition, the page is also put into all caches, and in case they are full, less frequently accessed memory pages are deleted from the caches.

* This brings us to the following two conclusions:

  * Caching only makes sense if memory pages are accessed multiple times during a computation.
  * Since caches are far smaller than main memory, they only contain subsets of the main memory pages. Therefore, a temporally close access pattern is required in order to benefit from the caches, because if the same page is accessed at a very late stage of the computation, it might have already gotten evicted from the cache.

#### Cache Eviction Strategies and Pre-fetching

* Modern computer systems not only use **least recently used (LRU)** memory page eviction strategies to delete cached memory pages from the caches, they also try to predict the access pattern in order to keep the memory pages that are old but have a high probability of being requested again. 
* Modern CPUs also try to predict future memory page requests and try to pre-fetch them. 
* Nevertheless, random memory access patterns should always be avoided and the more sequential memory access patterns are, usually the faster they are executed.

#### So how can we avoid random memory access patterns? 
* A side-effect of hashing is that even close by key values, such as subsequent integer numbers, result in different hash codes and therefore end up in different buckets. Each bucket can be seen as a pointer pointing to a linked list of key-value pairs stored in the map. These pointers point to random memory regions (Java Heap). Therefore, sequential scans are impossible.

* In order to improve sequential scans, Tungsten does the following trick: the pointer not only stores the target memory address of the value, but also the key. This way, the keys are stored sequentially together with pointers.

* We have learned about this concept already, where an 8-byte memory region is used to store two integer values, for example, in this case the key and the pointer to the value. This way, one can run a sorting algorithm with sequential memory access patterns (for example, quick-sort). 

* This way, when sorting, the key and pointer combination memory region must be moved around but the memory region where the values are stored can be kept constant. While the values can still be randomly spread around the memory, the key and pointer combination are sequentially laid out.

### Code Generation

#### JVM
* Every class executed on the JVM is byte-code. This is an intermediate abstraction layer to the actual machine code specific for each different micro-processor architecture. This is the major selling point for Java.

* The basic workflow is:
  * Java source code gets compiled into Java byte-code.
  * Java byte-code gets interpreted by the JVM.
  * The JVM translates this byte-code and issues platform specific-machine code instructions to the target CPU.

* Tungsten actually transforms an expression into byte-code and have it shipped to the executor thread, instead of make the Java Virtual Machine to execute (interpret) this expression one billion times, which is a huge overhead.

* These days nobody ever thinks of creating byte-code on the fly, but this is what's happening in code generation. Apache Spark Tungsten analyzes the task to be executed and instead of relying on chaining pre-compiled components it generates specific, high-performing byte code as written by a human to be executed by the JVM.

* Another thing Tungsten does is to accelerate the serialization and deserialization of objects, because the native framework that the JVM provides tends to be very slow. Since the main bottleneck on every distributed data processing system is the shuffle phase (used for sorting and grouping similar data together), where data gets sent over the network in Apache Spark, object serialization and deserialization are the main contributor to the bottleneck (and not I/O bandwidth), also adding to the CPU bottleneck. Therefore increasing performance here reduces the bottleneck.

### Columnar Storage
* Many on-disk technologies, such as parquet, or relational databases, such as IBM DB2 BLU or dashDB, support it.
* In contrast to row-based layouts (i.e. normal table layout, each line is a row), where fields of individual records are memory-aligned close together, in columnar storage (transpose of row-based table, each line is a column) values from similar columns of different records are residing close together in memory. This changes performance significantly. Not only can columnar data such as parquet be read faster by an order of magnitude, columnar storage also benefits when it comes to indexing individual columns or projection operations.

#### Whole stage code generation

* To understand whole stage code generation, we have to understand the Catalyst Optimizer as well, because whole stage code generation is nothing but a set of rules during the optimization of a Logical Execution Plan (LEP). The object called **CollapseCodegenStages**, extending Rule[SparkPlan], is the object used by Apache Spark for transforming a LEP, by fusing supported operators together. This is done by creating byte code of a new custom operator with the same functionality of the original operators which has been fused together.

* Operator fusing: Other data processing systems call this technique operator fusing. In Apache Spark whole stage code generation is actually the only means for doing so.

* An asterisk symbol, in the explained output, indicates that these operations are executed as a single thread with whole stage generated code. Without asterisk symbol means that each operator is executed as a single thread or at least as different code fragments, passing data from one to another.


#### The Volcano Iterator Model
* The volcano iterator model is an internal data processing model. Nearly every relational database system makes use of it. 
* It basically says that each atomic operation is only capable to processing one row at a time. 

* By expressing a database query as a directed acyclic graph (DAG), which connects individual operators together, data flows in the opposite direction of the edge direction of the graph. Again, one row at a time, which means that multiple operators run in parallel, but every operator processes a different, and only one, row at a time. 

* When fusing operators together (this is what whole stage code generation does), the volcano iterator model is violated. It is interesting to see that such a violation was done after decades of database research and actually leads to better performance.

# Deployment
## Bare Metal, virtual machine deployment
### Bare Metal Deployment
* There are approximately three layers. On the bare metal hardware layer, an operating system is installed which accesses different hardware components through drivers and makes them available to the user application through a common API. 

### Virtual Machine Deployment
* There is an additional component present here, called hypervisor, on the top of operating system layer. This component basically runs as a user application on the bare metal operating system and emulates a complete hardware stack, so that it can host another operating system running user applications.

* The operating system running on top (or within) of the hypervisor is called **guest** while the operating system running on top of the real bare metal hardware is called **host**. The idea is that the guest operating system is not aware that it is running inside a virtualized hardware, which has two main advantages:
  * Security: The guest operating system and guest user application can't access resources on the host operating system without being controlled by the hypervisor.
  * Scalability and elasticity: multiple guest systems can be run on a (powerful) host system allowing for scaling (imagine there are many host systems in the data center). Also, starting and stopping guest systems is relatively fast (in minutes), so a system can react to different load situations (especially in cloud deployments where virtually infinite resources are present across multiple host systems).

* Finally, the user applications are installed on top of the **guest** operation system, i.e. virtual machine operating system.
* As you can see, there is a huge overhead. You need run many virtual machine OS on top of the OS running on bare metal.
* Some **hypervisor** can run directly on bare metal, thus save the cost of running **host** operating system.

### Containerization

* We get rid of the guest operating systems and replaced them with containers. 
* A container does not run on top of a virtualized hardware stack. Instead it runs directly on the **host** operating system. In fact, all user applications are run directly on the host operating system. 
* The only difference is that the individual **operating system processes** (the runtime components making up a user application) are fenced against each other and also against the host operating system. In fact, a user application has the feeling that it is alone on the host operating system since it doesn't see what's going on, on the host operating system, and therefore also doesn't see the contents of other containers.
  * **Operating system processes** are the central units of work that an operating system runs. So each user application, which basically is nothing other than a set of machine code on permanent storage, is transformed into a running process by reading it from disk, mapping it to the main memory, and starting to execute the contained set of instructions on the processor.
  * An application can contain one or more processes, and on Linux, each application thread is executed as a separate process sharing the same memory area. Otherwise, a process can't access memory that the other processes are using; this is an important concept for security and stability. But still, all processes can see and use the same set of resources that the operating system provides.

* So how is this achieved? This concept was born on the Linux kernel and is the de facto standard for container-based virtualization. There are two major Linux kernel extensions, which make this possible: cgroups and namespaces.

#### namespaces
* Linux needed some way to separate resource views of different operating system processes from each other. The answer was namespaces. There are currently six namespaces implemented:
  * mnt: controls access to the filesystems
  * pid: controls access to different processes
  * net: controls access to networking resources
  * ipc: controls inter-process communication
  * uts: returns a different hostname per namespace
  * user: enables separate user management per process
* In order to use namespaces, only one single system call has been implemented: setns().
* The root name spaces are mapped to encapsulated and controlled namespaces with a specific ID. Since every process (or group of processes) is assigned to a sub-namespace of all the six groups mentioned earlier, access to filesystems, other processes, network resources, and user IDs can be restricted. This is an important feature for security.
* Try list the namespaces using command: `ls -al /proc/1000/ns`

#### cgroups
* Control groups is a mechanism to prevent a single process from eating up all the CPU power of the host machine. This subsystem is used to control resources such as:
  * Main memory quota
  * CPU slices
  * Filesystem quota
  * Network priority
* In addition to that, cgroups provide an additional transient filesystem type. This means that, all data written to that filesystem is destroyed after reboot of the host system. This filesystem is ideal for being assigned to a container. This is because, as we'll learn later, access to real (persistent) filesystems is not possible within a container. Access to persistent filesystems must be specified through mounts during the start of the containers.

#### LXC (Linux Containers)
* LXC has been part of the vanilla Linux kernel since February 20, 2014, and therefore can be used out-of-the-box on every Linux server.

#### Docker
* Docker basically makes use of LXC but adds support for building, shipping, and running operation system images. So there exists a layered image format, which makes it possible to pack the filesystem components necessary for running a specific application into a Docker images file.
* The advantage is that this format is layered, so downstream changes to the image result in the addition of a layer. Therefore, a Docker image can be easily synchronized and kept up to date over a network and the internet, since only the changed layers have to be transferred. In order to create Docker images, Docker is shipped with a little build tool which supports building Docker images from so-called Dockerfiles.

#### Kubernetes 
* Kubernetes (K8s) is an orchestrator of containers. 

##### Master node manages the whole of the cluster.
* API server: The API server provides a means of communication between the Kubernetes cluster and external system administrators. It provides a REST API used by the kubectl command line tool. This API can also be used by other consumers, making it possible to plug-in Kubernetes in existing infrastructures and automated processes.
* Controller manager: The controller manager is responsible for managing the core controller processes within a Kubernetes cluster.
* Scheduler: The scheduler is responsible for matching the supply of resources provided by the aggregated set of Kubernetes nodes to the demand of resources of the currently undeployed pods.

  In addition, the scheduler also needs to keep track of the user-defined policies and constraints, such as node affinity or data locality, in order to take the correct decisions as to where to place the container.
  
Data locality is only considered by Kubernetes during deployment.
* Etcd: Etcd is a reliable key-value store, storing the overall state of the cluster and all deployed applications at any given point in time.

##### Kubernetes Nodes

* PODs are the basic scheduling units of Kubernetes. A POD consists of at least one or more containers, which are co-located on the same node, so that they can share resources. Such resources include IP addresses (in fact, one physical IP is assigned to each POD so no port conflicts arise between them) and local disk resources which are assigned to all containers of the POD.
* The kublet is the Kubernetes manager for each Kubernetes node. It is responsible for starting and stopping containers as directed by the controller. It is also responsible for maintaining a PODs state (for example, the number of active containers) and monitoring. Through regular heartbeats, the master is informed about the states of all PODs managed by the kublet.
* Kube-proxy: The kube-proxy serves as a load balancer among the incoming requests and PODs.

* What **outside** means. Generally, Kubernetes runs in cloud infrastructures and expects that a load balancer is already there for it to use. The load balancer is dynamically and transparently updated when creating a service in Kubernetes, so that the load balancer forwards the specified ports to the correct PODs.

#### Benefits of Using K8s
* The only disadvantage is the effort you invest in installing and maintaining Kubernetes. 
* What you gain are the following:
  * Easy installation and updates of Apache Spark and other additional software packages (such as Apache Flink, Jupyter, or Zeppelin)
  * Easy switching between different versions
  * Parallel deployment of multiple clusters for different users or user groups
  * Fair resource assignment to users and user groups
  * Straightforward hybrid cloud integration, since the very same setup can be run on any cloud provider supporting Kubernetes as a service
  
# Spark MLlib
### History
* MLlib is the original machine learning library that is provided with Apache Spark. This library is still based on the RDD API. 

# SparkML 
* SparkML library speeds up machine learning by supporting DataFrames and the underlying Catalyst and Tungsten optimizations.

## Apache SparkML pipeline
* DataFrame: This is the central data store where all the original data and intermediate results are stored in.
* Transformer: As the name suggests, a transformer transforms one DataFrame into another by adding additional (feature) columns in most of the cases. Transformers are stateless, which means that they don't have any internal memory and behave exactly the same each time they are used; this is a concept you might be familiar with when using the map function of RDDs.
  * Example: **OneHotEncoder**
  * Example: **org.apache.spark.ml.feature.VectorAssembler**
* Estimator: In most of the cases, an estimator is some sort of machine learning model. In contrast to a transformer, an estimator contains an internal state representation and is highly dependent on the history of the data that it has already seen.
  * Another easy way to distinguish between an estimator and a transformer is the additional method called **fit** on the estimators. Fit actually populates the internal data management structure of the estimators based on a given dataset, which, in the case of StringIndexer, is the mapping table between label strings and label indexes.
  * Example: **org.apache.spark.ml.feature.StringIndexer** is also an estimator.
* Pipeline: This is the glue which is joining the preceding components, DataFrame, Transformer and Estimator, together.
* Parameter: Machine learning algorithms have many knobs to tweak. These are called hyperparameters and the values learned by a machine learning algorithm to fit data are called parameters. By standardizing how hyperparameters are expressed, Apache SparkML opens doors to task automation.

# Apache SystemML
* Apache Spark can also serve as runtime for third-party components, making it as some sort of operating system for big data applications.
* So, SystemML is a declarative markup language that can transparently distribute work on Apache Spark. It supports Scale-up using multithreading and SIMD instructions on CPUs as well as GPUs and also Scale-out using a cluster, and of course, both together.
* Plus there is a cost-based optimizer in place to generate low-level execution plans taking statistics about the Dataset sizes into account. In other words, Apache SystemML is for machine learning, what Catalyst and Tungsten are for DataFrames.








# References
1. [Spark Internals](https://github.com/JerryLead/SparkInternals)
2. Mastering Apache Spark 2.x - 2nd Edition
3. [HDFS Design](http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html)
