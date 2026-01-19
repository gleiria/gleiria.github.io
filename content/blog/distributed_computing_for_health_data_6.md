+++
title = 'Distributed Computing for Health Data - 6'
date = 2026-01-18T10:08:56Z
draft = false
+++




These are my personal notes related to the course Distributed Computing for Health Data provided by Health Data Research UK.
I use this space to consolidate concepts and revisit material over time.

### Lesson 6 - Spark and Spark Components

**What is Spark**

* Software platform for data engineering, data science, and machine learninng
* Designed to be fast and general purpose
* Multi-language: Python, R, SQL, Java, Scala.

---

**Spark vs Hadoop**

[See paragraph relevant section on lesson 5]({{< ref "distributed_computing_for_health_data_5.md" >}}#dist-comp-rationale)


---

**Spark Ecosystem**

1) **Spark Core**

* Foundation execution engine for all Spark functionalities
* Task schedulling, memory managment, fault recovery, and storage access
* RDDs (resilient distributed datasets): collection of items distributed across nodes

2) **Spark SQL**

* Querying over structured data, based in DataFrames and Datasets APis

3) **Spark Streaming**

* Processing of live and continuous streams of data

4) **Graph Processing (Graph)**

* Graph-parallel computation and algorithms based on RDDs

5) **Machine Learning(MLlib)**

* RDD and DataFrames APIs for machine leanrning models, feature engineering, and pipelines

---

**Spark Core**

Contains:
* Driver program: application code + distributed data
* SparkContext: API to access Spark functionality
* Executor: task coordinator on local nodes

---

**Driver Program**

This is the application code (for example Python script)

It:

* Defines the logic of the computation
* Creates distributed datasets
* Coordinates execution across the cluster
* There is one driver per Spark application.
---

**SparkContext**

The main API/entry point for accessing Spark functionality. Represents a connection to the Spark cluster (local or remote).

* Connects your code to the Spark cluster
* Manages resources (CPU, memory)
* Lets you create RDDs, access storage, run jobs

Through it, the driver:

* Creates and manipulates RDDs
* Submits jobs to the cluster
* Communicates with the cluster manager


---

**Executors**

Processes that run on worker nodes.

Responsible for:

* Executing tasks
* Performing computations
* Storing data in memory or disk
* Each Spark application has its own set of executors.

---

**Driver + SparkContext + Executors**

* A Spark application starts when the driver program is launched
* The driver creates a SparkContext, which connects to the master node
* The SparkContext requests resources from the cluster
* The cluster launches executor processes on worker nodes
* Executors run tasks on partitions of the data
* Results are sent back to the driver if needed


(instructor says that each node has a container process that hosts executers. Need to check this better as I think containers can by different depending on Kubernetes, Hadoop and YARN)

{{< figure src="/images/spark_diagram.png" title="" >}}

---

**Configuration**

When creating a SparkContext, configuration options can be specified using a SparkConf object, such as:

* Application name
* Master node (local or remote cluster)
* Resource-related settings

``` python
    from pyspark import SparkConf, SparkContext

    conf = SparkConf().setMaster("local").setAppName("My_app")
    sc = SparkContext(conf = conf)

```

Above, "local" is used to refer to a Spark application running on a local machine but a URL could be used to connect to a remote cluster.

---

**SparkContext vs modern Spark**

In the course they use SparkContext as the main entry point. In Spark 2+, this role is mostly taken by SparkSession.

``` python
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("My_app").getOrCreate()
```
SparkSession is a higher level entry point which internally contains SparkContext.


---

**RDDs (Resilient Distributed Datasets)**

* Represent the underlaying data structure of Spark 
* All work is expressed by either creating new RDDs, transforming existing RDDs or calling operations on RDDs
* RDDs are immutable which means that any operation on a RDD results in a new RDD
* They are split into different partitions that are allocated to different nodes.

Just a note here as I think the slides are a bit misleading:

* It's not the RDD object that is split but the data represented by the RDD
* The RDD keeps track of the partitions
* The RDD is a single logical object; the data it represents is split into partitions that are processed in parallel across nodes

RDDs can be created in different ways:

1) Loading an external file (e.g. text, json, csv file)

``` python
distFile = sc.textFile("README.md", 4) #4 is the number of data partitions
```

2) We can also use the parallelize() method to ask Spark to distribute a collection of data items over a number of partitions

``` python
data = [1, 2, 3, 4, 5]

rDD = sc.parallelize(data, 4)
```

* There is no standard way of defining the number of partitions
* Ideally, equal to the number of nodes (or processors) for max degree of parallelism (need to come back to this)

Related functions:
``` python
rdd.getNumPartitions() # retrieve n of partitions being used for a specific RDD

# other methods can be used to increase or decrease number of partitions
repartition()
coalasce()

# need to check docs to check how this methods work
```
---

*Users can perform 2 types of RDD operations:*

* transformations -> operations that create a new RDD, such as map() and filter()
* actions -> instruct Spark to apply computation and pass the result back to the Spark driver
---

**Transformations**

Example of transformation:

``` python
    # load text file
    inputRDD = sc.textFile("log.txt")
    # filter lines containing the word 'error'
    errorsRDD = inputRDD.filter(lambda x: "error" in x)
```
An important aspect of RDD transformations is that they are lazy operations (details down bellow)

---

**Actions**

* Operations that return a result to the driver program or write it to storage, such as count() and take()
* They 'force' the execution of the transformations required for an RDD
* Everytime an action is called Spark will check all the transformations that were defined over a RDD and evaluate the best way of executing them
---
**Lazy evaluation**

* All transformation operations applied to create a new RDD are “lazy,” which is to say that the data is not loaded or computed immediately
* Instead, transformations are tracked in a directed acyclic graph (DAG) and run only when there is a specific call to action for a driver program
* The driver program directs the primary function and operations for cluster computing on Spark jobs, such as aggregation, collecting, counting or saving output to a file system
* Dozens of possible actions and transformations include aggregateByKey, countByKey, flatMap, groupByKey, reduceByKey and sortbyKey.
* Lazy evaluation helps optimize data processing pipelines by eliminating unnecessary processing and clipping of unnecessary computations.

Example:

``` python
    # load data as RDD
    patients = sc.textFile("patients.csv")
    # filter high-risk patients (age > 65)
    elderly = patients.filter(lambda x: int(x.split(",")[2]) > 65)
    # extract names of high risk patients
    names = elderly.map(lambda x: x.split(",")[1])
```
At this point:
* No file has been read
* No filtering has happened
* No mapping has happened
* Spark has only built a logical plan. The lineage graph

**Now an action is trigered:**

``` python 
    names.count()
```
Only now:
* The file is read
* Data is partitioned
* Tasks are scheduled
* Transformations are applied

If you run transformations and never call an action Spark does nothing which is zero cost.

--- 

**Why lazy evaluation matters for distributed systems**

In a distributed system, early execution would cause:

* Unnecessary disk reads
* Unnecessary network communication
* Poor scheduling decisions

Lazy evaluation allows Spark to:

* Combine operations
* Minimize data movement
* Schedule tasks efficiently
* Recover from failures using lineage


---

I am kind of repeating myself a lot but really need to solidify this idea:

* You can define chains of transformations on RDDs, which are not executed immediately
* When an action is called, Spark executes only the transformations needed for that action
* The results are not saved by default, but can be cached if you explicitly request it (rdd.cache() or rdd.persist())

---

**Common Transformations in Spark**

* Some operations can be applied to all RDDs regardless of their data structure
* These are called basic RDDs
* map(), filter() are the two most common transformations
* flatMap() transformation allows splitting up an input element into multiple output elements
* flatMap() is similar to map(). It applies a function to all elements of a RDD and then flattens the results

{{< figure src="/images/transformations.png" title="" >}}

* it is worth mentioning the sample() function which returns a sample subset of the RDD
* sample() is very useful when we need to control the distribution of a target population
* RDDs also support operations applied to sets such as union() and intersection()
* The only restriction here is that the RDDs we are operating on must be of the same type
* We can also perform operations on RDDs containing key,value pairs
* These transformations are very useful in many applications as we can act on each key in parallel or group data across nodes
* For example, we can aply reduceByKey() to combine values with the same key
* Or groupByKey() to group values with the same key
* There are many transformations that can be applied to key,vaklue pairs (check this)

---

**Common actions on RDDs:**

* collect() can take all elements of a RDD into the memory of the driver program
* collect() needs be used consciously specially with large datasets as all elements form all partitions will be returned
* As alternative to collect() we can use take(num) and top(num) to show a small number of elements from a RDD
* reduce(func()) which operates on two elements the RDD and returns a new element of the same type
* count()
* countByValue()
* On top of all actions available for basic RDDs we can apply aditional actions to take advantage of the key,value nature of paired RDDs

---

**Impact of narrow and wide transformations**

One last observation about RDD operations is related to how transformations influence the number of partitions and data movement inside the network.

Transformations can have dependencies classified as:

* Narrow: a single output partition is computed from a single input partition
* Wide: data from multiple partitions is read in, combined (via shuffle), and written into the output partition

Wide operations are very resource intensive and time consuming and can change num of partitions at the end.

---

**This was a very long lesson. In summary, these are the key points to grasp:**

1) Spark prioritises fast (in-memory) computations
2) Spark Core's main abstractions:
    * SparkContext and RDDs
3) RDDs are immutable objects
4) Transformations and actions
    * applied to basic RDDs and key/value pair RDDs
5) Impact of narrow and wide transformations











