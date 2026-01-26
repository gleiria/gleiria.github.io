+++
title = 'Distributed Computing for Health Data - 8 - Spark Basic Usage'
date = 2026-01-22T07:42:49Z
draft = false
+++


These are my personal notes related to the course Distributed Computing for Health Data provided by Health Data Research UK.
I use this space to consolidate concepts and revisit material over time.

### Lesson 8 - Spark Basic Usage

In this lesson I follow the main core ideas of the course, but I deviate a bit from the way it is taught. The instructors use a pre-written PySpark Jupyter notebook and execute the workflow cell by cell. I prefer to work from VScode + terminal so created the following file strucutre:

```.
├── data
│   └── heart.csv
├── dist_venv
└── src
    └── analysis.py
```
* heart.csv is the Heart Disease Dataset downloaded from Kaggle
* dist_venv is just the python env
* analysis.py has the application code

to run:
```
../spark-4.1.1-bin-hadoop3/bin/spark-submit src/analysis.py
```
---

As discussed in the last lesson, SparkSession is the entry point to all Spark functionaly:

```python
# analysis.py

from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("test_med_app").getOrCreate()

# making spark less verbose. Only shows errors and a few prints that happen before this line
spark.sparkContext.setLogLevel("ERROR")

# for learning purposes, we load data from local. In real environment, this would be from say HDFS or S3
df = spark.read.csv("file:///Users/gnl202/Desktop/distributed_computing/data/heart.csv", header=True, inferSchema=True)

# from this point onwards we can interact with the DataFrame API
df.printSchema()
df.select("age", "chol").summary().show()
```
---

Things like **df.show()** are expensive and not used when dealing with big data and particular in production. With **df.show()** Spark has to schedule tasks, executors scan partitions, data processed and sent back to the driver, driver prints results. If we would go:

```python
df.show(5)
```
Spark would execute a full job just to print 5 rows.

That is why we use lazy distributed queries:

* count()
* groupBy().count()
* approxQuantile()
* sample()

This works for 300 rows, 300 million rows, or 300 billion rows

---

Important to have this in mind. When I run this locally with toy example on the laptop:

```
../spark-4.1.1-bin-hadoop3/bin/spark-submit src/analysis.py
```

We are still createting a Spark application with components that are logically separated:
| Spark component             | Where are they                                |
| ---------------------------- | ---------------------------------------------- |
| **Client**                   | We on the terminal (launching spark-submit)     |
| **Driver**                   | Python + Java VM process started by spark-submit |
| **Cluster Manager (Master)** | Local in-process scheduler                   |
| **Executors (Workers)**      | Java VM processes or threads on the laptop        |


Splitting the code up in analysis.py this is what is happening:

``` python
spark = SparkSession.builder.appName("test_med_app").getOrCreate()
```

1) A Driver is started
2) A local Cluster Manager is started 
3) Resources are requested (CPU, memory)
4) Launches Executors

--- 

Next, and importantly:

``` python
df = spark.read.csv("file:///.../heart.csv", header=True, inferSchema=True)
```
This is a lazy operation (as discussed on previous post). It does not read the file. It builds the logical plan.

* No data has moved around 
* No single worker has touched a file

---

Just a note on the .builder() as I worked with this design pattern in the [simulations]({{< relref "blog/microsimulations.md" >}}) I recently work on.

I aim of the builder() is to give the user a controlled step by step way to build complex objects. 

A SparkSession() is not a simple object. It, for examplem, includes:

* Connection to a cluster
* Configuration (memory, cores, SQL settings)
* Embedded SparkContext
* SQL engine
* Catalog of tables
* Session state

So the builder() basically exposes the cluster + execution subsystems

---

We also mentioned that Spark has two main libraries:

1) SparkSQl library: used for structured and semi-structured data manipulation. Allows to read and write data in different formats and use SQL-like interface to query the data

2) SparkDataFrames: Distrubuted in-memory tables with named columns and schemas. Each column has specific type


---

This mirrors real systems:

| Layer                  | Role                |
| ---------------------- | ------------------- |
| HDFS / S3 / ADLS       | Distributed storage |
| Spark                  | Distributed compute |
| Spark SQL / DataFrames | Data analytics      |
| BI / dashboards   | What stakeholders see|


In real world systems we would usually skip HDFS and use:

* AWS S3
* Azure Data Lake
* Google Cloud Storage

In fact, one of Spark's superpower is that it connects to many storage systems:

* HDFS
* S3
* Azure Data Lake
* GCS
* Local files
* Databases
* Kafka
* Parquet / ORC / CSV / JSON

