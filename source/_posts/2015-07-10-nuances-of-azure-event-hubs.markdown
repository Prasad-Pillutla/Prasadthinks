---
layout: post
title: "Nuances of Azure Event Hubs"
date: 2015-06-18 22:43:27 +0530
comments: true
categories: Azure, EventHub, Service Bus
---

Event Hub, Microsoft’s PaaS offering to accept millions of events per second is built to handle traffic for Internet of Things scenarios. It (Event Hub) is an offshoot of Partitioned queues/topics and leverages same partitioning strategy. I know you would ask, why does Event Hub scale and Queues/Topics don’t? To achieve scale certain features like session based messaging, TTL were let go in Event Hub, which makes it light-weight and achieve the scale.

In this post I’ll not be writing about Event Hubs functionality, which is available on [MSDN](http://azure.microsoft.com/en-in/services/event-hubs/) instead I’ll be sharing subtle aspects which are important to know while using Event Hubs.

Event hub speed is measured in Throughput Units (TPU), which defines the maximum event rate (ingress and egress) i.e., 1 Throughput unit equals to 1 MB/sec ingress and 2MB/sec egress and 2 Throughput units equals to 2 MB/sec ingress and 4MB/sec egress and so on.

Event Hubs are designed for downstream parallelism which is why egress rate is double that of Ingress. To understand this better let’s do some math

Ingress rate per partition => (TPU / No of Partitions)

Egress rate per partition => ((TPU * 2) / No of Partitions); Egress is twice of Ingress for TPU

|         TPU  |		Partitions |            Ingress            |          Egress            |
|:------------:|:-----------------:|:-----------------------------:|---------------------------:|
|          1   |        8          |     0.125 MB/Sec (=> 1/8)     |  0.25 MB/Sec (=> 2/8)      |
|          2   |        8          |     0.250 MB/Sec (=> 2/8)     |  0.50 MB/Sec (=> 4/8)      |
|          1   |       32          |     0.03125 MB/Sec (=> 1/32)  |  0.0625 MB/Sec (=> 2/32)   |
|          2   |       32          |     0.0625 MB/Sec (=> 2/32)   |  0.125 MB/Sec (=> 4/32)    |

 **TPU** - _No of Throughput Units_

**Partitions** - _Eventhub partitions_

**Ingress** - _Ingress rate per partition (MB/Sec)_

**Egress** - _Egress rate per partition (MB/Sec)_


For example, if user selects 1 TPU and 8 partitions and all partitions see even load, each partition gets approximately *0.125 MB/Sec* ingress throughput for a total aggregate throughput of 1 MB/Sec(8 x 0.125) and if user selects 2 TPU and 8 partitions, each partition gets approximately *0.25 MB/Sec* (8 x 0.25).

However there is cap on maximum ingress and egress that a partition can deliver, which is 1 MB/sec ingress and 2MB/Sec Egress. For instance, I choose 2 TPUs and 8 Partitions for an Event Hub, and one partition is receiving all of traffic and other partitions are sitting idle. When this partition receives more than 1MB/Sec it starts throttling and doesn’t matter if maximum traffic that Event Hub with 2 TPUs can take is 2MB/Sec.

To summarize, irrespective of load patterns of individual partitions and maximum throughput defined by TPUs selected, *a partition cannot take more than 1 MB/Sec ingress*.

Let’s say user selects 8 partitions and 10 TPUs for an Event Hub; total ingress that it can support is 10 MB/Sec which means in case of even load distribution each partition will receive 1.25 MB/Sec. How?

=> Each TPU guarantees 1MB/Sec Ingress

=> 10 TPUs will deliver 10 * 1 MB/Sec, which is 10 MB/Sec

=> Total number of partitions to cater traffic is 8

=> Even distribution means 10 MB per Sec/ 8 Partitions, which is 1.25 MB/Sec per Partition

But there is cap on maximum throughput for a partition, which is 1MB/Sec. What does this imply? This means that partitions will throttle if Event Hub is operating at its full capacity. This brings us to most important decision point, Number of Partitions and TPUs have to be selected together and after evaluating load patterns. Event Hub allows changing TPUs anytime but not partitions. Once allocated partitions cannot be increased or decreased and to alter their count Event hub has to be recreated.

Too many parameters to consider aren’t they; let me make it simple with an example. I’ve a scenario where peak traffic volume is 2,000 transactions/Second and each transaction(message) is of size 15 KB. Net volume that Event Hub will have to deal with is

=> 2,000 * 15 KB => 30,000 KB/Sec => 30 MB/Sec (for simplicity 1 MB = 1000 KB)

I choose to have 8 partitions and in the case of even load distribution each partition will receive

=> 30 MB per Sec / 8 Partitions => 3.75 MB/Sec per Partition, which is way more than maximum ingress rate of a partition (1 MB/sec). 8 partitions Event Hub will throttle for above scenario.

Now I configure 32 partitions and in the case of even load distribution each partition will receive

=> 30 MB per Sec / 32 partitions => 0.9375 MB/Sec per Partition, which is less than maximum ingress rate of a partition (1 MB/sec). 32 partitions will cater my requirement (2000 transactions per Sec, each transaction is of 15 KB size) without throttling.

Azure portal allows user to create maximum of 32 partitions and provision upto 20 TPUs. If user requires more throughput units (*partitions implied*) he/she should contact [Azure support](http://azure.microsoft.com/en-in/support/options/).