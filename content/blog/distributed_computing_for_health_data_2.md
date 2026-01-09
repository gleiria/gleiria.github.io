+++
title = 'Distributed Computing for Health Data - 2'
date = 2026-01-06T15:03:10Z
draft = false
+++


These are my personal notes related to the course Distributed Computing for Health Data provided by Health Data Research UK. I use this space to consolidate concepts and revisit material over time.

### Lesson 2 - Big Data Sources

---

**Data Types and Data Sources**

There are 5 data types:

* Structured (e.g. Electronic Health Records) 
* Semi-structured (EHR and vital-signs for a patien but not vital-signs for a second patient)
* Unstructured (CT scan)
* Time series (ECG)
* Sequence data (RNA or DNA sequencing for example)

---

**Structured Data:**

* Tabular Format
* Mostly quantitative (continuous or categorical)
* Example: electronic health records
* Stored in relational or transactional data bases
* Processed with SQL
* Human readable
* Can be exported to CSV, XML, etc
* Can be processed by machine programs


Just a note on relational and transactional data bases as I never worked with the second.

 In relational databases you store data in tables (rows and columns) with predefined schemas (e.g., patient records, lab results). You use SQL for queries and enforce relationships between tables (e.g., foreign keys).

Example for relational:

A table for Patients (ID, name, DOB) linked to a Visits table (visit_ID, patient_ID, date, diagnosis).
Query: "Find all patients over 65 with hypertension diagnoses in 2025."

Transactional databases are a subset of relational databases optimised for high-speed transactions (e.g., real-time updates, concurrent access). Focus on short, frequent operations (e.g., appointment scheduling, prescription updates).

Example for transactional:

A system for real-time bed assignment in an intensive-care unit, where multiple nurses update patient statuses concurrently.
Transaction: "Assign Patient X to Bed 5 and mark bed as 'occupied'."

Again, Transactional databases are optimised for:

* Many short, fast operations

* Concurrent users

* Consistency (ACID properties)

A transactional database can be tiny or huge. Size is not the defining feature.

---

**Semi-Structured Data**

* Organized in terms of tags and metadata to separate semantic elements and establish hierarchies or records and fields.
* Common formats: XML, JSON, HTML, FHIR
* Main advantage is the flexibility in representing data items with slightly different structures
* For example, all record vital-signs for a patient but not for a second patient. The record for the second patient will not have the entire hierarchy of vital signs
* Semi-structured data is ideal for health systems because it balances organisation (for easy access) and flexibility (to handle real-world variability)
* It’s widely used in distributed computing to integrate data from diverse sources like wearables, EHRs without requiring a rigid schema. 

This is usually saved in structures like Python dictionaries with key-value pairs:

```python
patient_record = {
    "patient_id": 12345,
    "name": "Alan Turing",
    "vital_signs": {  # Nested dictionary (hierarchy)
        "blood_pressure": {"systolic": 120, "diastolic": 80},
        "heart_rate": 72,
    },
    "medications": ["aspirin", "lisinopril"],  # List of values
    "allergies": None  # Missing field (flexibility)
}
```
* Tags/Metadata: Keys like "patient_id" or "vital_signs" define the structure.
* Hierarchy: Nested dictionaries (e.g., "vital_signs") create levels.
* Flexibility: Not all records need the same fields (another patient might lack "allergies").

---

**Unstructured Data**

* Does not follow a pre-defined data model
* Quantitative and qualitative data
* Diverse formats: text, image, audio, video
* Although stored in specific formats such as jpeg, mp3, mp4, etc… this type of data requires specific tools and models for processing, analysis and interpretation

Harder to search, analyze, or integrate into systems without advanced tools like natural language processing, computer vision, or machine learning.

---

**Time_Series Data**

* Semi or Unstructured data, measured at equal time intervals
* Mostly quantitative, with temporal patterns
* Stored in csv files or using specific compressed formats



---

**Sequence Data**

* Usually DNA sequences or other biological data under the broad term “Multi-Omics Data”
* Stored using specific and compressed formats (fastq files etc)


---

**Key Aspects Management of Big Datasets**

* Scalability
* Data Distribution
* Programming model
* Batch vs streaming
* (real-time) processing
* Data sampling (sample vs entire datasets)

(more details on each of the above as course progresses)

---

**Scalability**

* Adjust resources based on demand
* Vertical: add more processors or memory cards to a single machine
* Horizontal: keep single machine as is and aggregate more machines into a cluster

In big data we explore both approaches although horizontal scalability is usually first choice due to its cheaper implementation and theoretical endless capacity to add more resources as data sets grow.

--- 

**Data Distribution**

* An Important feature of tools is to support for data distribution and to what extend it is transparent to the user
* When scaling applications from a single machine to a cluster, data is split into small chunks and distributed across all available resources


Data distribution is essential for scaling big data applications. Tools like Hadoop and Spark make it transparent, so you focus on analysis, not infrastructure.

---

**Programming Model**

* Providing a set of machines with the same application called but different pieces of data, it is a relatively simple strategy to implement big data applications.

* Later we will explore MapReduce: A programming model that uses a “divide-and-conquer” approach
* Map: processing data stored in each machine
* Reduce: merge partial results to generate the final output

Data is distributed across nodes of a cluster and processed in parallel (map phase) and results produced and collected back to the user (reduce phase)

(more details with code down the line)

---

**Batch vs Streaming (real-time) processing**

Batch:

* Organising chunks of data into batches and pass each batch to a machine for processing

* Typically used to processing large amounts of data

* nIdeal for tasks that do not require real-time processing

Event Streams:

* Sensor or data from real-time sources
* Data processed as it arrives in (near) real time
* Ideal for tasks that require real-time insights or immediate action
* Processed via a streaming pipeline
* Generate alerts or trigger specific actions
* Velocity more important than volume 

Time-series and genome sequencing data are special cases. They can be processed in batches but following an ordered element for example nucleotide sequencing in DNA.

---
**Data Sampling (sample vs entire dataset)**

* When designing a big data solution we don’t need to start big 
* Use small but representative samples of our data data
* There are dif techniques for data sub-sampling.
* Smal samples: prototyping and testing
* Larger samples: modelling and analysis
* Only move to fully distributed approach when all initial questions and checks are addressed.

Again, time series and sequence data provide extra complexity due to their ordered nature and other intrinsic characteriscs like outliers, seosonality, etc…


















