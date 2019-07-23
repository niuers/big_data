
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
   * SystemML
   * SparkML (since 1.6.0)
   * Spark Streaming/Structured Streaming
   * GraphX
   * DeepLearning4j
   * H2O
1. High Performance for both batch and streaming data: DAG scheduler, a query optimizer, and a physical execution engine. Spark is scalable, massively parallel, and in-memory execution.
1. Ease of Use: Spark offers over 80 high-level operators that make it easy to build parallel apps and in various languages
   * Scala
   * Python
   * Java
   * R
   * SQL
1. Generality: SQL, Streaming, Graph and ML
1. Runs Everywhere: Spark runs on Hadoop, Apache Mesos, Kubernetes, standalone, or in the cloud. It can access diverse data sources.
1. Open Source

### General Description of Spark Job Execution
Spark is a distributed programming model in which the user specifies transformations. Multiple transformations build up a directed acyclic graph (DAG) of instructions. An action begins the process of executing that graph of instructions, as a single job, by breaking it down into stages and tasks to execute across the cluster. 

Spark is effectively a programming language of its own. 

Internally, Spark uses an engine called **Catalyst** that maintains its own type information through the planning and processing of work. 
Spark will convert an expression written in an input language (e.g. Python) to Spark's internal Catalyst representation of that same type 
information. It then will operate on that internal representation.











# References
1. [Spark Internals](https://github.com/JerryLead/SparkInternals)
