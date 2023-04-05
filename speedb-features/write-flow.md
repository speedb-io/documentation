---
description: This feature is experimental in v2.4.0
---

# Write Flow

## Introduction:

The SPDB Write Flow is a new approach to writing data in RocksDB that aims to reduce the amount of time spent holding global database mutexes and increase the level of parallelism for IO writes. This new approach is implemented in a dedicated SPDB write thread, which handles the memtable switch, flush requests, WAL switch, WBM threshold limit, write delay, and WAL trim.

## Background:

In the previous write flow, each write was inserted into a matching write queue, and a relevant thread queue picked a write group leader. Other writes were then connected to this leader as group members, and the parallel work was done until the limit was reached. The leader was responsible for WAL writing and waiting for all group members to complete their task. The version progressed when the group completed its task. However, this approach had several disadvantages, such as the blocking of new writes due to the memtable switch status check and the serial execution caused by the many DB mutex points.

## Algorithm:

The new SPDB Write Flow algorithm consists of a thread dedicated to handling the write flow. This thread wakes up at a specified time and handles quiesce, which includes the memtable switch, flush requested, WAL switch, WBM threshold limit, write delay, and WAL trim. The thread then proceeds to handle the batch writes by inserting them into two containers, one for the current batch writes and one for the new batch writes that are being inserted.

Each batch write is responsible for writing to the memtable (if needed) and waiting for the complete batch group. The batch group leader is, by default, the first one unless new batches are inserted in parallel, and the one batch group limit size is not reached. The batch leader is responsible for the merged group WAL write and writing to WAL with or without synchronization. It is important to note that WAL writes can be completed before all the container memtable writes are completed, which is okay. However, we must wait for the memetable writes to be completed to progress the version. The flow handles merge/none memtable writes/none WAL writes in the same container, allowing for a fluent flow. Still, writes to the WAL should consider that we build the merged WAL batch (seq number).

\


## Limitations and known issues&#x20;

In the 2.4 release, the write flow feature is experimental. It currently consumes slightly more memory than usual and this will be fixed in the next release.&#x20;

## Test Results&#x20;

A comparison of RocksDB 7.7, Speedb 2.3 and Speedb 2.4 +write flow  was performed.

This was tested with db\_bench, using the configuration below:&#x20;



* Number of objects: 1Billion
* Value size: 64b
* Write buffer size: 268MB
* Number of Threads: 50
* Number of CPU cores: 16\


The graph illustrates the dramatic increase in writing performance when using small objects: \


![](https://lh5.googleusercontent.com/ueDtl8husENo6aP-f4MLuELq48\_spP5LFDKTPCW\_Le139UWPOunhgyh5a12kKKdY1M\_TJ\_-D5jyKjZMkWgd3liekIaoxAcHARARArbycwRE9K5CH\_OlVsETPsJzWyL92aD0PejywzvCMRo8O7xkVlHA)

\
