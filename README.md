# Apache Kafka 101


## Apache Kafka® is a distributed streaming platform

A streaming platform has three key capabilities:

* Publish and subscribe to streams of records, similar to a message queue or enterprise messaging system.
* Store streams of records in a fault-tolerant durable way.
* Process streams of records as they occur. 

Kafka is generally used for two broad classes of applications:

* Building real-time streaming data pipelines that reliably get data between systems or applications
* Building real-time streaming applications that transform or react to the streams of data 

To understand how Kafka does these things, let's dive in and explore Kafka's capabilities from the bottom up.

First a few concepts:

* Kafka is run as a cluster on one or more servers that can span multiple datacenters.
* The Kafka cluster stores streams of records in categories called topics.
* Each record consists of a key, a value, and a timestamp. 

Kafka has four core APIs:

* The Producer API allows an application to publish a stream of records to one or more Kafka topics.
* The Consumer API allows an application to subscribe to one or more topics and process the stream of records produced to them.
* The Streams API allows an application to act as a stream processor, consuming an input stream from one or more topics and producing an output stream to one or more output topics, effectively transforming the input streams to output streams.
* The Connector API allows building and running reusable producers or consumers that connect Kafka topics to existing applications or data systems. For example, a connector to a relational database might capture every change to a table. 

    ![Kafka Architecture](images/kafka-architecture.png)

In Kafka the communication between the clients and the servers is done with a simple, high-performance, language agnostic TCP protocol. This protocol is versioned and maintains backwards compatibility with older version. We provide a Java client for Kafka, but clients are available in many languages.


## Labs

Basic Kafka features and functionalities are explored during the lab.

* Lab01 - [Setup](Lab01/README.md)
* Lab02 - [Produce and Consume Messages to a Kafka Stream with Kafka CLI Tools](Lab02/README.md)
* Lab03 - [Produce and Consume Messages to a Kafka Stream with Spring Boot](Lab03/README.md)
* Lab04 - [Event Streams Admin API](Lab04/README.md)
* Lab05 - [Event Streams CLI](Lab05/README.md)


## References

* IBM Event Streams Documentation, https://ibm.github.io/event-streams/getting-started/using-kafka-console-tools/
* Apache Kafka, https://kafka.apache.org/intro
