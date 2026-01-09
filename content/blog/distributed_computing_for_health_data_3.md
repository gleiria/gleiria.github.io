+++
title = 'Distributed Computing for Health Data - 3'
date = 2026-01-08T14:05:28Z
draft = false
+++


These are my personal notes related to the course Distributed Computing for Health Data provided by Health Data Research UK.
I use this space to consolidate concepts and revisit material over time.

### Lesson 3 - Distributed Computing 

---


**Distributed Computing**

* Use of autonomous networked machines (nodes) for solving a single problem
* Autonomous as each node only has access to its memory, processor and storage
* No shared memory across nodes so they communicate via messages and carrying data
* Data and processing tasks are distributed across nodes

---

**Data Parallelism**

* The model is replicated across multiple nodes
* And each node receives a batch of data to process
* Scales well when model fits in single machine memory 

Just a note here that 'model' is simply the computation or piece of code being applied to the data.

---

**Model Parallelism**

* The model is split across multiple nodes
* Challeging when there is dependency. When some part of the model depends on the output of other part
* Also useful when model too large to fit into a single machine memory
* In certain cases hybrid approach of data + model parallelism is used

Remember that data is easier to scale than models that depend on outputs of each other. This is because:

* Nodes must wait for outputs from other nodes
* Network latency matters (latency is time taken for data to travel from source to destination)
* Load balancing is hard
* Debugging is harder

(more on this later)

So, in data parallelism, you replicate the computation and split the data. In model parallelism, you split the computation itself, which introduces dependencies and makes scaling harder.

---

**Distributed Health Care Systems**

* Model for centralised care servives
* Reference architecture for big data analytics in healthcare

{{< figure src="/images/med_architec.png" title="" >}}

---

**Distributed Healthcare Systems - Requisites**

* Data format and interoperability
* Communication network
* “Transparent” data location and distribution (who needs to know what (more on this))
* Security and privacy (at all layers)
* Appropriate tools for each layer 
* Batch vs real-time workload management (ex: prioritise urgent services)
