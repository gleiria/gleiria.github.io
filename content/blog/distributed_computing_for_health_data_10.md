+++
title = 'Distributed Computing for Health Data - 10 - Kafka Basic Usage'
date = 2026-01-30T06:29:24Z
draft = false
+++

These are my personal notes related to the course Distributed Computing for Health Data provided by Health Data Research UK.
I use this space to consolidate concepts and revisit material over time.

### Lesson 10 - Kafka Basic Usage

In the first half of the lesson, they follow Kafkas's [quickstart tutorial](https://kafka.apache.org/quickstart/) so I won't be copying and pasting it here. Instead, since I had the need to simplify core concepts and build a mental model of how Kafka actually works I slightly deviated from the the first half of the lesson. 

--- 

**Thinking in Events vs State**

For a long time, most software systems have been built around databases. Databases encourage us to think of the world in terms of **things**:

* users
* devices
* patients
* trains

These **things** have a **state**, and we store that **state** in tables. This model worked for many years but more and more people are now thinking of the world in terms of **events**. 

---

**From State to Events**

An **event** represents something that happened at a specific point in time:

* a user logged in
* a sensor sent a reading
* a patient’s heart rate changed
* a train arrived at a station

**Events** also have state, but a different kind of state: what happened, and when it happened.So instead of storing only the latest state of a thing, we store the full history of what happened to it.

---

**Logs as the Core Abstraction**

Logs are timestamped records that document events, actions, and messages occurring within an application, operating system, or server.

Storing large volumes of immutable events in traditional databases is not practical.
Event-driven systems therefore rely on a simpler abstraction: **logs**. Kafka is a system for managing logs.

So again, a log is:

* an ordered, append-only sequence of events
* When an event happens, we append it to the log. Never updated or deleted

Also:
* They are simple to reason about
* They are easy to replicate
* Easy to escalate horizontaly

Apache Kafka is essentialy a distributed system for managing logs.

In Kafka, logs are called **topics**. So **topics** are (again):

* an ordered collection of events
* stored durably on disk
* replicated across multiple machines

This means Kafka is resilient to hardware failures. Data does not disappear if a single machine goes down. **Topics** can:

* store data for minutes, hours, days, or years
* be small or extremely large
* be replayed at any point in time
* Each event in a topic represents something that happened in the business

Kafka encourages us to think in **events first**, **things second**.

---

**From Monoliths to Microservices and Event-Driven Systems**

When databases dominated system design, most architectures were monoliths:

* one large application
* one central database
* tightly coupled components

As systems grew, these architectures became difficult to maintain, reason about, and evolve. Kafka fits naturally with microservices architectures.

In an event-driven system:

* services consume events from topics
* perform some computation
* publish new events to other topics

for example:

```bash
Topic A → Service 1 → Topic B → Service 2 → Topic C
```
Each service is independent, loosely coupled, and communicates only through events.

---

**Kafka as an Integration Layer**

Kafka is not just a runtime system. It is also an integration platform.

**Kafka Connect**

Kafka Connect allows external systems to connect to Kafka:

* databases
* cloud storage
* APIs
* SaaS platforms

It provides a large ecosystem of connectors:

* some open source
* some commercial

This makes Kafka a central hub for data in motion.

---

**Stream Processing with Kafka**

Kafka-based services typically:

* read from topics
* transform data
* write to new topics

Common operations include:

* filter()
* groupBy()
* count()
* aggregate()
* enrichment() which is the equivalent to join() in databases

Kafka Streams is a Java API that supports this model directly, allowing developers to build stream-processing applications embedded inside services.

---

**Why Kafka Matters**

In addition to being a messaging system, Kafka represents a deeper architectural shift:

* **from** modelling systems around current state
* **to** modelling systems around immutable event histories

This shift enables:

* real-time analytics
* system replay and debugging
* better fault tolerance
* loosely coupled architectures
* scalable data pipelines

In many modern systems, Kafka often becomes the backbone that connects storage, computation, and real-time services into a single event-driven platform.

---

**Kafka Producer()**

Producers make the connection between data source and Kafka. We can build a producer for any data source (for example a real-time API). Its job is to:

1) Set up connections, instantiate KafkaProducer()
2) Load data
3) Transform data (convert to message)
4) Encode and send to Kafka topic
5) Clean up and close


``` python
# needs pip install kafka-python
from kafka import KafkaProducer
import csv
import time

##### ---------- Set Up and Conection-------- #####
KAFKA_TOPIC = "patient-monitoring"
KAFKA_BROKER = "localhost:9092" # change if kafka is somewhere else

# Create Kafka producer
producer = KafkaProducer(bootstrap_servers = KAFKA_BROKER)

##### ---------- Data Transformation and Sending -------- #####

# Read and send csv data
with open("patient_data.csv", "r") as file:
    reader = csv.reader(file)
    header = next(reader) # skip header if needed
    for row in reader:
        # Convert row to a comma-separated string
        message = ",".join(row) 
        producer.send(KAFKA_TOPIC, value = message.encode("utf-8")) 
        print(f"Sent: {message}")
        print(time.sleep(1))

##### ---------- Clean Up -------- #####
producer.flush() # Ensure all messages are sent
producer.close()

```

---

**Kafka Consumer()**

Consumers connect Kafka and downstream applications. Its job is to:

* Set up connections and subscribe to one or more topics
* Join a consumer group
* Read messages from Kafka (in order)
* Decode and process messages
* Track progress using offsets

``` python
from kafka import KafkaConsumer

KAFKA_TOPIC = "patient-monitoring"
KAFKA_BROKER = "localhost:9092" 

# create kafka consumer
consumer = KafkaConsumer(
    KAFKA_TOPIC, # topic to subscribe to
    bootsrap_servers = KAFKA_BROKER,
    auto_offset_reset = "earliest", 
    anable_auto_commit = True, # consumer automatically commits its offset after reading messages
    grou_id = "csv-group", # consumer group that the consumer belongs to
)
print("Listening for messages... ")

for message in consumer:
    print(f"Received: {message.value.decode("uft-8")}")
```
---

**Consumer Groups**

Consumers usually run in groups. Kafka ensures that:

* each message is processed by only one consumer in the group
* load is automatically balanced across consumers

This is how Kafka scales horizontally.

---

Architecturally, this separation between Producers() and Consumers() is critical and very powerful. In fact, is one of the core reasons why Kafka fits so well with event-driven and microservices architectures:

* producers don’t know who consumes the data
* consumers don’t care where the data cames from
* new services can be added without touching existing ones