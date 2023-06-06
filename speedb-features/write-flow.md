---
description: This feature is experimental in v2.4.0
---

# Write Flow

## Introduction:

The SPDB Write Flow introduces a significant improvement in the performance of data writing in RocksDB. The SPDB Write Flow is a new approach to writing data in RocksDB that aims to reduce the amount of time spent holding global database mutexes and increase the level of parallelism for IO writes. This new approach is implemented in a dedicated SPDB write thread, which handles the memtable switch, flush requests, WAL switch, WBM threshold limit, write delay, and WAL trim.

## Background:

In the previous write flow, each write was inserted into a matching write queue, and a relevant thread queue picked a write group leader. Other writes were then connected to this leader as group members, and the parallel work was done until the limit was reached. The leader was responsible for WAL writing and waiting for all group members to complete their task. The version progressed when the group completed its task. However, this approach had several disadvantages, such as the blocking of new writes due to the memtable switch status check and the serial execution caused by the many DB mutex points.

## Algorithm:

The new SPDB Write Flow algorithm consists of a thread dedicated to handling the write flow. This thread wakes up at a specified time and handles quiesce, which includes the memtable switch, flush requested, WAL switch, WBM threshold limit, write delay, and WAL trim. The thread then proceeds to handle the batch writes by inserting them into two containers, one for the current batch writes and one for the new batch writes that are being inserted.

Each batch write is responsible for writing to the memtable (if needed) and waiting for the complete batch group. The batch group leader is, by default, the first one unless new batches are inserted in parallel, and the one batch group limit size is not reached. The batch leader is responsible for the merged group WAL write and writing to WAL with or without synchronization. It is important to note that WAL writes can be completed before all the container memtable writes are completed, which is okay. However, we must wait for the memetable writes to be completed to progress the version. The flow handles merge/none memtable writes/none WAL writes in the same container, allowing for a fluent flow. Still, writes to the WAL should consider that we build the merged WAL batch (seq number).

\
The table below summarize the differences between RocksDB write flow and the Speedb write flow:

|                                                 | RocksDB                                                     | Speedb                                                             |
| ----------------------------------------------- | ----------------------------------------------------------- | ------------------------------------------------------------------ |
| General                                         | DBmutex on every write batch                                | RW lock - write lock when needed                                   |
| Writes to the WAL                               | Append, sync writes                                         | <p>Writing to a specific address space, parallel writes</p><p></p> |
| Checking the triggers                           | <p>Part of the DB mutex on every write</p><p></p>           | <p>Background, write lock only when needed </p><p></p>             |
| <p>Switch memtable/switch WAL/Trim WAL <br></p> | <p>Part of the DB mutex on every write</p><p></p>           | <p>Background, without any locks </p><p></p>                       |
| Writes  rollback                                | <p>Don’t needed (since writing first to the WAL)</p><p></p> | <p>Needed when write to the memtable failed </p><p></p>            |

To summaries, the new write flow enables parallel writes by the following changes:

1. In the current write flow, the DBmutex is global for the entire database. With Speedb write flow the db mutex is split to wal mutex and other operations. So every write to the WAL is not not holding the entire db mutex. Other internal operations can run in parallel to writing to the WAL.
2. Speedb write flow allows parallel writes to the memtable and the WAL. Ack is sent only after the data is written to both, but the writes are no longer serial.&#x20;
3. The wall in the previous write flow is using append, meaning only single write is allowed at a time. Speedb's new write flow changed the way data is written to the WAL and now writes to a specific address in the file, a fact that enables the parallel writes to the WAL.&#x20;

##

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


<figure><img src="../.gitbook/assets/_                                                    readrandomwriterandom_90.png" alt=""><figcaption></figcaption></figure>

\
