+++
title = 'Distributed Computing for Health Data - 4'
date = 2026-01-12T06:41:44Z
draft = false
+++

These are my personal notes related to the course Distributed Computing for Health Data provided by Health Data Research UK.
I use this space to consolidate concepts and revisit material over time.

### Lesson 4 - Big Data Processing Tools: Hadoop

---

**Hadoop**

* Framework for distributed processing of large datasets
* Allows an application to scale up from a single computer to thousands of machines while implementing data distribution and fault tolerance
* Primarily intended for batch processing
* Hadoop is an 'ecosystem' comprised of core and specific modules, each providing a different service
* At its core, Hadoop has a distributed file system and a distributed processing layer, HDFS and MapReduce

Fault tolerance is the capability to continue operating smoothly despite failures or errors in one or more components of a distributed system. This resilience is crucial for maintaining system reliability, availability, and consistency.

---

**Hadoop Distributed Filesystem (HDFS)**

* Hadoop has an abstract notion of file system to allow access to local and remote (cloud-based) file systems for reading, writing and managing files

"Abstract notion of a file system" in the way that Hadoop does not assume one specific file system implementation. It can work with many different file systems via a common interface.

* HDFS is one implementation to store very large files with a “streaming data access” pattern (continuous read operations).

* HDFS has 3 main abstractions:
    * Blocks (block refers to the minimum amount of data that can be read or written on a disk)
    * Name node (master node) 
    * Datanode(worker nodes) 

Again, block refers to the minimum amount of data that can be read or written on disk.
HDFS uses large blocks. Blocks are divided into 128MB blocks and stored as independent units across data nodes. Each data block is replicated in 3 nodes.

--- 

**Note:**
Hadoop is a big-data architecture that tightly couples distributed storage and distributed processing. HDFS is the original storage system Hadoop was designed around, so its concepts (blocks, NameNode, DataNodes, streaming access) define how Hadoop-style processing works. Even though Hadoop can now process data from many storage systems via an abstract file system interface, HDFS is emphasized because it shaped the entire model of large-scale data processing.

---

**HDFS API**

* Set of interfaces and methods that allow applications to interact with and manage data stored in HDFS

* Main access through **hadoop fs**, which interacts with HDFS or any other filesystem supported by Hadoop

Key methods:

```python
create(): # Creates a new file in HDFS.
open(): # Opens an existing file for reading.
exists(): # Checks if a file or directory exists.
isFile() / isDirectory(): # Determines if a path points to a file or directory.
rename(): # Renames a file or directory.
copy(): # Copies files or directories between paths.
delete(): # Removes files or directories. 

```

---

**Map Reduce**

* A programming model that breaks the processing into two phases (map and reduce), with corresponding functions.

Just some notes I have from previously learning about functional programming which link to this:

**Functions as first-class citizens:**

* In functional programming, a function is a first-class object like any other object - not only can you compose/chain functions together, but functions can be used as inputs to, passed around or returned as results from other functions (remember, in functional programming code is data)

**Higher Order Functions:**

* A function that has another function as an argument

or

* A function that returns another function

For example:

* map() and reduce()

Often we want to apply some sort of transformation to each datapoint of a dataset and perform some sort of aggregation (reduction) at the end. A technique used here can be MapReduce. MapReduced is behind big data handling tools such as Hadoop and Spark.

The term Mapping comes from applying the same operation to each entry in a structure (aka mapping).

Then performing a reduction operation which collects/aggregates all the individual results together to produce a single result.

This technique relies heavily on composability and parallelisability of functional programming.

---

**Mapping**

map(f, C) is a function map that takes another function f() and a collection C of data items as inputs.

Calling map(f, C) applies the function f(x) to every data item x in a collection C and returns the resulting values as a new collection of the same size. 

Although I use use Python for the examples bellow, Map and Reduce concepts apply to distributed systems such as Hadoop and Spark.

```python

list_a = ["goncalo","beatriz","helena", "maria]

x = map(len, list_a)

```

out:

```python

x = [7, 7, 6, 5]

```

This is a very simple example but its possible to see that:

* Each operation is independent
* There are no dependencies between elements
* Work can be split across multiple CPUs or multiple machines
* This makes map operations easy to parallelize and highly scalable

Another example with lambda functions:

```python
squares = map(lambda x: x*x, [2,3,4,5,6,7,8,9])
```

**Note:** Lambda functions are small anonymous functions that you define inline. They are used for simple operations you need to perform once. They stay in memory like any other functions but usually you don't reference them so they are only use where they exist.

---

**Reducing**

Reducing is a programming pattern used to aggregate a collection of values into a single result.

```python
from functools import reduce

sequence = [1,2,3,4,5]

def product(a,b):
return a * b

result = reduce(product, sequence)
print(result)
```

out:
```python
120
```
---

**Why Reduce deserves special attention as it often introduces scalability challenges:**

* Partial results may need to be collected on a single machine
* This can lead to:

    * Memory constraints

    * High communication costs

    * Poorly designed reduce steps can become performance bottlenecks

* In distributed systems:

    * The number of map tasks is usually driven by input data size (one per data chunk)

    * The number of reduce tasks is typically driven by cluster size and can often be configured by the user

---

**YARN (Yet Another Resource Negotiator)**

YARN is responsible for resource management and job scheduling/monitoring, with dedicated services for each function.

* There is one resource manager per cluster 
* There is one node manager per node in the cluster to launch and monitor containers which execute the application
* YARN provides functions for requesting and working with cluster resources

Typical workflow:

1) Client asks ResourceManager (RM) to run an application
2) RM finds a NodeManager (NM) and launches a container to host the application
3) NM can request additional resources
4) New NM/cointainer is allocated to run a distributed computation

YARN was introduced to improve the execution of MapReduce applications as well as support other distributed computing paradigms in Hadoop.

