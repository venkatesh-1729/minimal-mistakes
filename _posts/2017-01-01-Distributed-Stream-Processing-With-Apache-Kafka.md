---
title: "Distributed Stream Processing With Apache Kafka"
tags: Kafka-Streaming 
markdeep: true
---

> Kafka first came into existence as distributed publish-subscribe messaging system at [Linkedin](https://engineering.linkedin.com/27/project-kafka-distributed-publish-subscribe-messaging-system-reaches-v06) which on later evolved into a [distributed streaming platform](https://kafka.apache.org/). Currently, it is the backbone for many real-time stream processing infrastructures across [many](https://cwiki.apache.org/confluence/display/KAFKA/Powered+By) companies.

Introduction
============
Kafka as a distributed streaming platform has the following capabilities.

1. Publish and subscribe to streams of records - Messaging System.
2. Store streams of records in fault tolerant way - Storage System.
3. Lets you process streams of records as they occur - Real-time stream Processing.

It is used for building two broad classes of applications.

1. Real-time data pipelines to move data reliably between systems.
2. Real-time streaming applications that react to or transform streams of data.


<div class="markdeep-diagram">
*****************************************************************************
*                              Producers                                    *
*                 .------.      .------.      .------.                      *
*                |        |    |        |    |        |                     *
*                |Producer|    |Producer|    |Producer|                     *
*                |        |    |        |    |        |                     *
*                 '------'      '------'      '------'                      *
* .--------.          |            |            |             .-------.     *
*|          |          '-----.     |      .----'             |         |    *
*|'--------'|---.             |    |     |              .--> |Streaming|    *
*| Database |    |            |    |     |             |     |   Apps  |    *
*|          |    |            v    v     v             |     |         |    *
* '--------'     |       .---------------------.       |      '-------'     *
*                 '---> |                       | <---'                     *
* Connectors            |         Kafka         |          Stream Processors*
*                 .---- |                       | <---.                     *
* .--------.     |       '---------------------'       |      .-------.     *
*|          |    |               |   |                 |     |         |    *
*|'--------'|    |               |   |                 |     |Streaming|    *
*| Database |<--'          .----'     '-----.           '--> |   Apps  |    *
*|          |             |                  |               |         |    *
* '--------'              v                  v                '-------'     *
*                      .------.          .------.                           *
*                     |        |        |        |                          *
*                     |Consumer|        |Consumer|                          *
*                     |        |        |        |                          *
*                      '------'          '------'                           *
*                               Consumers                                   *
*****************************************************************************
</div>

Kafka Terminology
=================
* **Record** - Unit of data stored in Kafka.
  + A record consists of *Key*, *Value*, *Time Stamp*.
* **Topic** - A category or feed name to which records are published.
  + Core abstraction of the stream of records in Kafka.
* **Publisher** \| **Producer** - A Kafka client that publishes records to the Kafka cluster.
* **Subscriber** \| **Consumer** - A Kafka client that consumes records from the Kafka cluster.
  + Consumers label them shelves with *consumer group name*.
* **Broker** - A node in Kafka cluster.
  + More than one broker can exist on a machine.
* **Partitions** - Division of data present in a topic.
  + All the data of a topic may not fit in one machine.
  + They allow data to be consumed in parallel by consumers in a consumer group.
* **Partitioned Logs** - an Immutable sequence of records in each partition of a topic.
* **Offset** - Unique sequential index of data in partition.
  + Local to partition i.e no global ordering among records in a topic.
  + Oldest will be at the head (offset 0) and the newest record will be at the tail.

<div class="markdeep-diagram">
******************************************************************************************
*               +--------------------+                                                   *
*               |   .------------.   |                                                   *
*               |   |  Consumer  |   |                       Anatomy Of Topic            *
*       .------>|   '------------'   | <------.                                          *
*      |        +--Consumer Group 2--+         |                                         *
*      |                                       |         +---+---+---+---+---+---+       *
*      |                   ^                   |         | 0 | 1 | 2 | 3 | 4 |     P1    *
*      |                   |                   |         +---+---+---+---+---+---+       *
*      |                   |                   |                                         *
*    .-----------.   .-----------.   .-----------.       +---+---+---+---+---+---+       *
*   |   Broker 1  | |   Broker 2  | |   Broker 3  |      | 0 | 1 | 2 |   |   |     P2    *
*   |             | |             | |             |      +---+---+---+---+---+---+       *
*   |             | |             | |             |                                      *
*   |  ---MP1---  | |  ---SP1---  | |  ---SP1---  |      +---+---+---+---+---+---+       *
*   |             | |             | |             |      | 0 | 1 | 2 | 3 |   |     P3    *
*   |  ---SP2---  | |  ---MP2---  | |  ---SP2---  |      +---+---+---+---+---+---+       *
*   |             | |             | |             |                                      *
*   |  ---SP3---  | |  ---SP3---  | |  ---MP3---  |       Partitions Log Of A topic      *
*   |             | |             | |             |             With offsets             *
*   |             | |             | |             |                                      *
*    '-----------'   '-----------'   '-----------'                                       *
*          |               |               |                                             *
*          |               |               |                   Configuration             *
*          |               |               |                                             *
* +--------v---------------v---------------v--------+    Replication Factor - 3          *
* | .-------------. .-------------. .-------------. |    Partitions - 3                  *
* | |  Consumer   | |  Consumer   | |  Consumer   | |                                    *
* | '-------------' '-------------' '-------------' |    MPn - Master Of Part. n.        *
* +--------------- Consumer Group 1  ---------------+    SPn - Replica/Slave of Part. n. *
******************************************************************************************
</div>

Kafka Features And Promises
===========================
* Kafka retains all the published records irrespective of their consumption.
* Communications in Kafka use a simple, high performant, language agnostic TCP protocol.
* Consumer 
  + Consumer are grouped into consumers groups for fault tolerance and scalability.
  + Each record in a topic will be delivered to one consumer in a consumer group.
  + Only meta data retained (in ZooKeeper) for a consumer is its offset.
  + Offset is controlled by the consumer - can consume records in any order.
  + These features makes consumer very cheap.
  + They can come and go without affecting Kafka cluster.
* Producer
  + Producer is can choose partition for a record.
  + For example round robin for load balancing and hash partitioning based on the key.
* Partitions
  + They are distributed across brokers.
  + They are replicated for fault tolerance.
  + Each partition has a leader.
  + All the read and writes are handled by the leader.
  + Each broker will act as a leader for some of its partitions for load balancing.

* Total Order is guaranteed withing a partition.
* Messages sent by the producer to a partition will be appended in the order they are sent.
* Consumer sees the-the records in the order they are stored in the log.
* A topic with replication factor N will tolerate N-1 server failures.

Kafka Configuration
===================
Kafka has a lot of configurable options which can be used to improve performance add features.

* Producer - bootstrap.servers, key.serializer, value.serializer, acks, compression.type, retries
* Consumer - bootstrap.servers, key.deserializer, value.deserializer, enable.auto.commit
* Broker - compression.type, delete.topic.enable, auto.create.topics.enable

Apache ZooKeeper Role
=====================
> ZooKeeper is a centralized service for maintaining configuration information, naming, providing distributed synchronization, and providing group services. All of these kinds of services are used in some form or another by distributed applications. -- [Zookeeper](https://zookeeper.apache.org/)

Few places where Kafka uses ZooKeeper :

* Topic registration info - /brokers/topics/[topic]
* Partition state info - /brokers/topics/[topic]/partitions/[partitionId]/state
* Broker registration info - /brokers/ids/[brokerId]

You'll find all data structure of Kafka used in ZooKeeper [here](https://cwiki.apache.org/confluence/display/KAFKA/Kafka+data+structures+in+Zookeeper)

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
path = /brokers/topics/TestTopic
val={"version":1,"partitions":{"2":[2,0,1],"1":[1,2,0],"0":[0,1,2]}}

path = /brokers/topics/TestTopic/partitions/0/state 
val = {"controller_epoch":8,"leader":0,"version":1,"leader_epoch":8,"isr":[0]}

path = /brokers/topics/TestTopic/partitions/1/state
val = {"controller_epoch":8,"leader":1,"version":1,"leader_epoch":7,"isr":[1]}

path = /brokers/topics/TestTopic/partitions/2/state
val = {"controller_epoch":8,"leader":2,"version":1,"leader_epoch":6,"isr":[2]}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Screencast
================
[Asciicast Video](https://asciinema.org/a/97315)


Integration
================
![Streaming Architecture](/assets/images/streaming-architecture.svg)




References
==========
Most of this presentation covers Introduction and Quick Start sections of Official Kafka project website among many other things.
If you want to know more internal of log replication etc. refer links below.

1. [Apache Kafka Website](https://kafka.apache.org/)
2. [Papers and presentations](https://cwiki.apache.org/confluence/display/KAFKA/Kafka+papers+and+presentations)
3. [Apache ZooKeeper](https://zookeeper.apache.org/)
4. [Kafka data structures in Zookeeper](https://cwiki.apache.org/confluence/display/KAFKA/Kafka+data+structures+in+Zookeeper)

