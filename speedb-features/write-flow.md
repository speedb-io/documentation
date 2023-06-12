---
description: This feature is experimental in v2.4.0
---

# Write Flow

## Introduction:

The SPDB Write Flow introduces a significant improvement in the performance of data writing in RocksDB. The SPDB Write Flow is a new approach to writing data in RocksDB that aims to reduce the amount of time spent holding global database mutexes and increase the level of parallelism for IO writes. This new approach is implemented in a dedicated SPDB write thread, which handles the memtable switch, flush requests, WAL (Write Ahead Log) switch, WBM threshold limit, write delay, and WAL trim.

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

1. Speedb changed the DB mutex to read/write-lock
2. Speedb write flow allows parallel writes to the memtable and the WAL. Ack is sent only after the data is written to both, but the writes are no longer serial.&#x20;
3. The wall in the previous write flow is using append, meaning only single write is allowed at a time. Speedb's new write flow changed the way data is written to the WAL and now writes to a specific address in the file, a fact that enables the parallel writes to the WAL.&#x20;

### Transactions

RocksDB supports both optimistic and pessimistic concurrency controls. The pessimistic transactions make use of locks to provide isolation between the transactions. The default write policy in pessimistic transactions is WriteCommitted, which means that the data is written to the DB, i.e., the memtable, only after the transaction is committed. This policy simplified the implementation but came with some limitations in throughput, transaction size, and variety in supported isolation levels.&#x20;

This approach actually doesn't require any special treatment in the new spdb write flow since the txn (transaction) batches are kept in memory and are written to the WAL/memtable only as part of commit.

What is interesting for us is the WritePrepared and WriteUnprepared

In this propose we actually have for each txn:

**Prepare**

**Txn Writes**

**Commit**



In the prepare phase we write only a wal write that gets a seq as all batch writes does but this id is the identify of the txn

Any txn writes will be attached to this txn id.

The write flow will consist of a txn data structure that keeps track of the uncomplete txn.

The Txn write batches will be inserted to the memtable as the regular batch writes (can be with initial  txn id to all writes or increments one - if the second option this seq should be kept in the incomplete txn data)

And will be updated in the wal.&#x20;

The version seq will be update as well.&#x20;

Any read from the txn will try find first in the txn data and then from the memtable and  levels .

New regular writes will proceed with no effect

Read from the db and not txn read will skip incomplete txn writes, by checking if the memtable read result is part of the txn WF data.

When txn is complete a commit write executed and the relevant txn id is being removed from the txn WF data and the txn data is part of the DB and is accessible  to all.



This approach is simplify the rollback since any read will ignore the txn data,&#x20;

Memtable switch will be made ONLY when no open txn. As part of flush, if we have failed txn , the data is ignored and not part of the SST.

\


### Error handling

Because we wrote to memtable while writing to the wal, we might fail on part of the memtable writes/wal write, but the memtable already consists of the write. In such case, as like txn pending data, we keep a failed sequence, and will be ignored when read/flush.

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


<figure><img src="../.gitbook/assets/_                                                    readrandomwriterandom_90.png" alt=""><figcaption></figcaption></figure>

\
