
# Concepts
### What is Apache Spark?
> Apache Spark is a unified analytics/computing engine and a set of libraries for large-scale parallel data processing on computer clusters.

### General Description of Spark Job Execution
Spark is a distributed programming model in which the user specifies transformations. Multiple transformations build up a directed acyclic graph (DAG) of instructions. An action begins the process of executing that graph of instructions, as a single job, by breaking it down into stages and tasks to execute across the cluster. 

Spark is effectively a programming language of its own. 

Internally, Spark uses an engine called **Catalyst** that maintains its own type information through the planning and processing of work. 
Spark will convert an expression written in an input language (e.g. Python) to Spark's internal Catalyst representation of that same type 
information. It then will operate on that internal representation.












# References
1. [Spark Internals](https://github.com/JerryLead/SparkInternals)
