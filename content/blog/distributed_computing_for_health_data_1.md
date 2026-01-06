+++
title = 'Distributed Computing for Health Data - 1'
date = 2026-01-05T13:31:45Z
draft = false
+++


These are my personal notes related to the course Distributed Computing for Health Data provided by Health Data Research UK.
I use this space to consolidate concepts, reflect on lessons, and revisit material over time.

### Lesson 1 - Key Concepts

---

**The 3 Vs of Big Data**

A classic way to understand big data is through the 3 Vs:

- **Volume** - how much data we have
- **Velocity** - how fast data is generated and needs to be processed
- **Variety** - how many different forms the data takes

In healthcare, all three Vs show up at once and that’s what makes the domain both challenging and powerful.

---

**How volume, velocity and variety work together**

This is challenging but creates unprecedented potential.

* Volume means we have more data than ever to uncover patterns

* Velocity allows us to act on insights in real time

* Variety ensures we capture the full clinical context, such as combining genetic data and EHR to support personalized medicine.

Distributed computing is the backbone that makes this possible. It allows us to store, process, and analyze data at scale, at speed, and across diverse formats.

---

**Volume: When Data No Longer Fits on One Machine**

We enter a big data scenario when the amount of data:

* Cannot fit into the memory or storage of a single machine

* Cannot be processed efficiently by a single CPU

* At that point, we need to aggregate memory, storage, and compute power across multiple machines.

To give a sense of scale:

* Electronic Health Records (EHRs) can reach TB to PB scale within a hospital or healthcare network

* Genomic data can be hundreds of GB per person

* Wearables can generate several GB per individual per year

* Patient-generated data, public health datasets, and surveillance systems can reach TB or PB scale

* Claims and billing data are massive and continuously growing

---

**Velocity: How Fast Data Moves and Matters**

Velocity refers to the speed at which data is:

* Generated

* Processed

* Analysed

This is particularly critical for real-time or near-real-time healthcare scenarios, such as:

* Intensive care units

* Accident & emergency departments

* Ambulance and emergency response services

* Telemedicine platforms

* Clinical decision support systems

In these contexts, delayed analysis can mean delayed care or missed opportunities for intervention.

---

**Variety: Many Forms of Health Data**

Healthcare data comes in many shapes and forms:

Data Types:

* Structured (tables, relational databases)

* Unstructured (clinical notes, audio, text)

* Time-series (vital signs, monitoring data)

* Sequence data (genomics)

Beyond Data Types:

Variety also includes:

* Storage formats: tables, databases, images, audio files, genomic formats, and more

* Clinical codes and semantics: different coding systems and meanings for the same concept

* Information exchange: interoperability between systems and institutions

* Managing this diversity is one of the hardest and most important problems in health data science.

---

**Analytics: Turning Data into Insight**

Big data is only useful if we can extract insight from it.

A typical analytics pipeline looks like:

1) Data acquisition

2) Exploratory data analysis

3) Data preparation

4) Analytical modelling

5) Interpretation of results

Depending on the use case, analytics can follow different processing paradigms.

---


**Batch vs Streaming Processing**

Batch Processing

Used when:

* Data can be aggregated

* There are no strict time constraints

* Results remain valuable even after minutes, hours, or days

* Examples include population-level analyses, retrospective studies, and large-scale simulations.

---

**Streaming Processing**

Used when:

* Data must be collected, processed, and analysed rapidly

* Results support time-critical decisions

This is essential for real-time monitoring, alerts, and adaptive clinical systems.

---

**Frameworks for Distributed Computing**

To handle big health datasets, we rely on frameworks-software and hardware ecosystems that support:

* Data management

* Distributed processing

* Scalable analysis

A key idea is that data is distributed across multiple machines, and the user defines how this distribution and computation should happen.

In this course, we focus on three core technologies:

* Hadoop – based on the MapReduce programming model

* Apache Spark – enables data parallelism with fault tolerance

* Apache Kafka – designed for low-latency, real-time data streaming

Together, these tools form the backbone of many modern big data health applications. (I'm not sure that Hadoop is still used in modern applications but I think it is good to have a mental model of how it works) (more on this later)













