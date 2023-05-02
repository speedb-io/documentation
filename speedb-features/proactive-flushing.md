---
description: >-
  This document describes the proactive flushing mechanism introduced in Speedb
  v 2.2.0
---

# Proactive Flushing

## Overview

In RocksDB, memtables are flushed for a number of reasons. Typically, this occurs when the size of a memtable reaches its maximum configuration. Flushing may also be triggered by the WriteBufferManager (or WBM, see [WBM's detailed description](https://github.com/EighteenZi/rocksdb\_wiki/blob/master/Write-Buffer-Manager.md)). The WBM is an optional component. A user who wishes to limit the maximum memory used by multiple databases for memtables (the Quota), will create a WBM instance and configure all associated databases to update the WBM as needed.

The WBM tracks both the total memtable memory and the total mutable memtable memory. The WBM performs the tracking through its associated databases. These databases notify their WBM when they allocate and release memtable memory, and when their memtables become immutable.

Flushing is handled passively by the WBM. The databases call the WBM’s ShouldFlush() API during their write flows. A DB that calls ShouldFlush() and receives a positive reply, triggers a flush. The WBM’s reply is based on the current mutable and immutable memory consumption, relative to the quota.

When a DB triggers a flush due to the WBM, it has to pick CF(s) to flush. In case of atomic flush, all CFs are flushed. Otherwise, the oldest CF (the one with the smallest sequence number) is selected.

## Problems with the current method

Currently, there are a few issues to address:

1. The WBM is passive. The DB checks the WBM’s state during a DB write.
2. The CFs chosen for non-atomic flushes are based solely on their "age" and are not considered based on the amount of unflushed memtable data they have accumulated.
3. There may also be problems with immutable memtables when they are not pending for flush, either because of the "min\_num\_write\_buffers\_to\_merge" setting or because they have been flushed and are not released.

&#x20;

These issues will be apparent in a multi-db scenario. There may be some databases which are very active (heavy writers) and others which are hardly active or inactive. In such a scenario, the inactive databases may consume memtable memory that will not be released (WBM flushing is a result of database writes), while the very active databases will flush very frequently, even if their memtables consume little memory. This will result in the creation of a large number of small level-0 files, as well as not free up any memory.

&#x20;

## Solution - Proactive Flushing

A proactive flushing approach is based on the principle that the WBM initiates flushes rather than waiting for its databases to query it.

As a secondary principle, CF-s are selected for flushing based on the size of their mutable memtables.

Proactive flushing is an optional new feature, introduced in release 2.2.1. Once enabled, proactive flushing will replace the existing WBM flushing mechanism. Thus, when its ShouldFlush() API is called, the WBM will always respond with 'no'.

### Usage

The feature is enabled by default. To disable it, set the initiate\_flushes parameter to false when creating the WBM.&#x20;



The following example creates a write buffer manager with proactive flushes enabled, with a maximum number of parallel flushes == 3

```cpp
size_t buffer_size = 1 << 30U;
std::shared_ptr<Cache> cache = ...;
bool initiate_flushes = true;
FlushInitiationOptions flush_initiation_options;
flush_initiation_options.max_num_parallel_flushes = 3U;
  
Options options;

// Options initialization code
  .
  .
  .

options.write_buffer_manager.reset(new WriteBufferManager(buffer_size,
                                                            cache,
                                                            allow_wbm_stalls,
                                                            initiate_wbm_flushes,
                                                            flush_initiation_options));

  .
  .
  .

// Open the database using options
```

### Registration with the WBM

In order for the WBM to initiate flushes, every DB will register with its WBM a callback function which the WBM may use to request the DB to initiate a flush.

### Tracking the number of flushes in progress

The WBM will now track the number of flushes that are running simultaneously across all of its databases. These flushes may be those initiated by the WBM or those triggered due to other reasons (e.g., memtable size limit, manual flush, etc.).

### Adapting the number of flushes

When using proactive flushing, the user sets a maximum number of flushes that may run simultaneously in all of the WBM’s databases together as demonstrated in the example above. The WBM attempts to maintain a number of simultaneous flushes that correspond to the percentage of used memory out of its quota, up to the maximum configured number.&#x20;

### Initiating a flush

The WBM has a new internal thread that will wake up whenever it needs to initiate new flushes. The thread will iterate over its DBs, requesting each DB in turn to initiate a flush. The WBM will also specify a minimum size for the flush, to reduce the chances of generating many small level-0 files. A DB will evaluate the request and may or may not initiate a flush. It will return the result (flush initiated or not) to the WBM.

The WBM will stop requesting DB-s when one of the following occurs:

* The desired number of flushes was reached; Or
* All of the DB-s were requested to flush.

### Picking a CF

When a DB receives the WBM’s request to flush, it will evaluate the request and see if there is a CF that may be flushed. The DB will select the oldest CF that meets the minimum size requirement. However, to avoid a case where there are multiple CF-s in a DB, some heavily active (writing) and some not at all, a CF that was not picked multiple times will be picked occasionally, regardless of the minimum size requirement.

### Performance results

The new proactive flushing algorithm was tested with db\_bench, using the configuration below.

**Test description :**

* Set the write buffer manager with size of 110,000,000
* Create 3 databases.
* Performing fillseq operations on 430,000 keys with a value size of 76B on 2 databases. This has resulted in the memtable becoming mostly full but not enough to require a flush.
* Writing 100000000 keys of the same size to the last DB.

**HW configuration:**&#x20;

16 CPU cores, Intel(R) Xeon(R) Platinum 8370C CPU @ 2.80GHz

128GB memory

**Results:**

Below are benchmark results comparing the performance of Speedb v2.2.0 with proactive flushing enabled against Speedb without proactive flushing enabled.&#x20;

Based on the graphs, it can be seen that without the proactive flushing, there were many small flushes, high write amplification, and a reduction in performance



<figure><img src="../.gitbook/assets/Proactive flushes overwrite.png" alt=""><figcaption></figcaption></figure>
