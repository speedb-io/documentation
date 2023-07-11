---
description: A high-level overview of RocksDB objects and operations
---

# 🔦 RocksDB Basics

[RocksDB Overview](rocksdb-basics.md#rocksdb-overview)\
[LSM](rocksdb-basics.md#log-structure-merge-lsm)\
[Memtable](rocksdb-basics.md#memtable)\
[Write (Put)](rocksdb-basics.md#write-put)\
[Read (Get)](rocksdb-basics.md#read-flow-get)\
[Key tombstones (Delete)](rocksdb-basics.md#key-tombstones-delete)\
[LSM levels and compaction](rocksdb-basics.md#lsm-levels-and-compaction)\
[LSM compaction tradeoffs\
Leveld compaction  ](rocksdb-basics.md#lsm-compaction-tradeoffs)\
[General block-based SST structure](rocksdb-basics.md#general-block-based-sst-structure)\
[Column Families \
](rocksdb-basics.md#column-families)[WAL\
](rocksdb-basics.md#the-wal)[Iterators \
](rocksdb-basics.md#iterators)[Snapshots](rocksdb-basics.md#snapshots) \
[Merge Operators\
](rocksdb-basics.md#merge-operators)[Range Deletion \
](rocksdb-basics.md#range-delete)[Single delete ](rocksdb-basics.md#single-delete)

## RocksDB Overview&#x20;

RocksDB is a key value storage engine based on Log-Structures Merge (LSM) Tree. It is developed by Meta and a fork of LevelDB (Google).

### Log Structure Merge (LSM)

LSM tree is an append only data structure, optimized for write-heavy workloads.&#x20;

The LSM Tree is a variant of the traditional B-tree data structure and is commonly used in distributed storage systems and databases like Apache Cassandra, LevelDB, and RocksDB.

The LSM Tree organizes data into multiple levels, each level consisting of multiple sorted structures, typically called memtables and SSTables (Sorted String Tables). Here's a high-level overview of the LSM Tree process:

1. Write Path: When data is inserted or updated in an LSM Tree-based system, it is first written to an in-memory data structure called the memtable. The memtable allows for fast and efficient writes since it resides in memory. Once the memtable reaches a certain threshold, it is flushed to disk as an SSTable.
2. Compaction: Over time, multiple SSTables accumulate on disk as data is written and flushed from memtables. These SSTables can contain redundant or overlapping data. The compaction process merges and consolidates these SSTables to optimize storage space and improve read performance. During compaction, the SSTables are merged, sorted, and deduplicated, resulting in a compacted SSTable with only the latest and non-redundant data.
3. Read Path: When a read operation is performed, the LSM Tree checks the memtable first, as it contains the most recent writes. If the data is not found in the memtable, it looks for the data in the SSTables, starting from the most recent and going down the levels until the data is found or determined to be absent.

The LSM Tree structure and compaction process provide several advantages, including efficient write operations, high throughput, and scalability. However, it also introduces some trade-offs, such as increased read amplification due to the need to search through multiple SSTables during read operations. These trade-offs are carefully managed and optimized in LSM Tree-based systems to provide high-performance and reliable data storage and retrieval.



LSM consists of the following main two components:

1. Memtable, which is a memory-resident write buffer
2. SST (Sorted Static/String Table), which is a disk resident and includes immutable collection of key-value pairs

#### Memtable

A memory resident append only data structure that holds key-value pairs.  It acts  as a write buffer for incoming writes, and serves reads for data that it holds (more on that later)

Due to the need to support efficient ordered iteration, the default RocksDB memtable is an ordered singly-linked skip list, providing an average O(log N) complexity for both insertion and retrieval operations.

#### Write (Put)

* ncoming writes are applied to the memtable
* When certain conditions are met (more on that later) a memtable switch is performed, where the active memtable becomes immutable and flushed to disk, and a new memtable is created to accept new writes
* The flushed memtable is written as an ordered sequence of key-value pairs into an SST file

SST files are immutable after creation, and data in them cannot be modified or carved out of them (more on that later)

\


<figure><img src=".gitbook/assets/Screen Shot 2023-05-29 at 12.58.44.png" alt="" width="198"><figcaption></figcaption></figure>

#### Read flow (Get)

* Check the active memtable
* Check immutable memtables, if any
* Check SST files, from newest to oldest

This isn’t great for read performance, especially as SST files accumulate

<figure><img src=".gitbook/assets/Screen Shot 2023-05-29 at 13.01.21.png" alt="" width="374"><figcaption></figcaption></figure>

#### Key tombstones (Delete)

Because each SST file is immutable and we cannot just punch holes in existing SST files, to support key deletion, LSM stores employ a technique called tomb-stoning

A tombstone is a special record that only contains a key (without value) indicating that the key is deleted

A tombstone is said to “cover” an existing key record because a read always checks newer data first, so when a tombstone is encountered the read can complete immediately, knowing that the key was deleted

\
![](<.gitbook/assets/Screen Shot 2023-05-29 at 13.03.44.png>)

### LSM Levels and Compaction&#x20;

LSM stores data in multiple levels to enhance read performance. Since each level contains sorted runs, read operations can be efficiently performed by sequentially scanning the levels from the highest to lowest, gradually merging and retrieving the required data. This approach reduces random disk access and enhances overall read efficiency. (L0 to LN).

The SST files within each level aren't overlapping (except in L 0), so a GET only needs to do roughly as many reads as the number of levels.

Compaction is the process of moving data between levels in the background.

The process of compaction is called that way because it chooses two or more overlapping SST files from one or more levels and combines them into a new sorted collection of key-value pairs, discarding older versions of keys and tombstoned (possibly creating more than a single output SST).



<figure><img src=".gitbook/assets/Screen Shot 2023-06-05 at 9.28.59.png" alt="" width="183"><figcaption></figcaption></figure>

#### LSM compaction tradeoffs

* Space Amplification -- keeping multiple versions of the same key increases the amount of space taken by SST files
* Write Amplification -- compacting too frequently a range of keys increases the amount of data being written to disk, harming disk endurance and eating into the disk I/O bandwidth
* Read Amplification -- more overlapping sorted runs means more checks on every GET, incurring more slow disk read I/O

#### &#x20;Leveled compaction

A compaction strategy chooses an entire level once certain conditions are met (mainly, when the level reaches its size limit), and compacts it into a lower level by choosing the overlapping files from the lower level and combining them.

Since on L0 the SST files are just dumps of the data from the memtable, it doesn’t constitute a single sorted run and files can (and usually do) overlap, which effectively adds more “levels” that a GET request needs to check.  As a result, workloads that care about read performance should keep the number of files in L 0 to a minimum.

<figure><img src=".gitbook/assets/Screen Shot 2023-06-05 at 9.30.00.png" alt="" width="231"><figcaption></figcaption></figure>

\


#### More compaction Methods

* Universal Compaction (aka Tiered Compaction) -- trades lower read and space amplifications for lower write amplification as it tries to merge only sorted runs that are of similar size and cover roughly the same key ranges (as opposed to leveled compaction which always compacts a smaller sorted run into a larger one)
* FIFO Compaction (aka TTL compaction) -- discards whole SST files as their age exceeds a set threshold (useful for time-series data, for example)

### General block-based SST structure

The data on the SST is divided into logical blocks.

A block is just a single read-and-write unit, not defined by a hard size limit.

There are many block types, but these are the most relevant:

* Data
* Index
* Filter

![](<.gitbook/assets/Screen Shot 2023-06-05 at 9.31.16.png>)

#### The filter block (optional)

The filter block is used to avoid disk I/O during read by skipping sorted runs which do not contain the lookup key.

The filter block utilizes a probabilistic data structure which may return a false positive (key exists when it doesn’t), but never a false negative.&#x20;

RocksDB includes filter policy implementations based on Bloom Filter and on Ribbon Filter.

The filter block contains the serialized data used by the filter policy to determine if a lookup key exists or not.

RocksDB does not (currently) create a filter policy by default, because it consumes additional memory and storage, so SST files created with the default settings will not contain a filter block.

#### Data Block

Variable length blocks, broken after the first key-value pair which crosses the configured block\_size boundary.&#x20;

The data block contains the raw data of the key-value pairs encoded as variable-length strings.&#x20;

Keys are also delta-encoded by default to save space (which means that we can’t just seek into the beginning of a key and read it, because it’s only the delta from the previous key).

Data blocks contain some metadata in addition to the key-value pairs.&#x20;

#### &#x20;The index block

Because data blocks have variable lengths and in order to avoid doing expensive I/O when we search for the block containing the lookup key, an index block is created for each SST file.

Essentially a list containing the first key of each block and the offset at which that block begins.

By default the keys in the index block are delta-encoded as well.

### &#x20;Column Families

column family is a logical grouping of key-value pairs within the database. It allows for data organization and management, providing a way to group related data together while maintaining separate options and configurations for each group. \


With Column families you can group logically similar data together (akin to tables in a relational database), while still providing a holistic view of the data in the entire database.

Each column family is essentially a separate LSM-tree, with its own memtable, compaction and data layout configuration.

Column families share resources in the database, so they can be aware of what is happening in other column families when maintenance operations are required.

### &#x20; Write Ahead Log (WAL)

In order to prevent data loss upon premature shutdown (due to new data being written to the memory-resident memtable), every write batch is also written to disk in a Write Ahead Log file (aka the journal in some databases).

In order to uphold the atomicity guarantee of the write batch across column families, the WAL is shared between all column families in a database.

On each startup the WAL is replayed to arrive at the state of the database before shutdown.

RocksDB provides many configuration options for the WAL, including (but not limited to):

* Maximum size of a single WAL file
* Total size of the WAL per database
* Amount of data written to the WAL which will trigger a disk sync
* Avoiding writing to the WAL for a specific write batch
* How to deal with corruption in the WAL during replay

### The MANIFEST

The version information is written into the MANIFEST file.

Since memtables are memory-resident, the MANIFEST only holds the live WAL files.

Writing a full version on every change is inefficient, so instead RocksDB records a base version at the beginning of the MANIFEST, and from then on it only records version edits which contain only the delta between the new version and the previous one and on startup the MANIFEST is replayed to get the current version.

When the MANIFEST reaches a certain size (configurable), RocksDB creates a new MANIFEST file with the current version as base.

The current MANIFEST file name is stored in the CURRENT file.

### &#x20;Iterators

Used for iterating over an ordered range of keys (as opposed to a single key read).

Begins by seeking into the start of the range (calling Seek() on the iterator) to get the first key, and advancing the iterator (calling Next() on it) to get the next key.

Iterators require checking each memtable and every level of the LSM to see if the seek key falls in its range (for L0 this requires checking each file on the level).

When advancing the iterator, it needs to check all of the levels to find the next key (implemented using a min-heap).

Reverse iteration is also supported, though it may be less efficient.

\


### Iterators and resources consumption

Just as a read returns the key as seen at the specific point in time when it was initiated, an iterator presents the view of the key-value store as it was at the specific point in time when the iteration was initiated.

To present a point-in-time view of the LSM without SST files and memtables disappearing from under it, an iterator takes a reference on the current version, keeping it alive for the duration of the iteration.

This can lead to increased disk and memory usage, especially if the write rate is high.

### &#x20;Snapshots

For non-iterator use-cases, where having a view of the key-value store at a specific time point is needed, keeping a version alive is too expensive.

Instead, RocksDB allows preserving older versions of keys visible by a specific sequence number during compaction by taking a snapshot.

When trying to read from a snapshot, the snapshot should be provided as part of the read options, causing RocksDB to ignore keys with higher sequence numbers.

### Merge operators

In RMW (Read-Modify-Write) operations, such as incrementing an integer value, it's important to ensure that the value doesn't change between reading and writing (or the modification will lose information).

However, some modifications are commutative and mutually excluding them (such as doing the modification as part of a transaction) is too expensive.

RocksDB supports providing a merge operator for a column family, which allows writing a merge operand containing a value that would allow the operator to merge two or more operands into the final value.

The downside of using a merge operator is that a read might need to check all of the levels to gather all of the operands and return a value.



### Range Delete

Useful for cases where there’s a need to delete a large amount of keys in a specific range, which would require creating many deletion tombstones and sometimes also knowing which keys in the range were written.

Stores a special record which contains an end-exclusive range of keys to delete in the form \[first-key, last-key).

The range deletion records are kept in a separate structure for each memtable and SST file, and consulted at the beginning of the read from the memtable or SST file.

### Single delete

An optimisation of the deletion operation, to avoid having to propagate a deletion tombstone all the way to the bottom-most level even when no earlier versions of a key exist.

As soon as a key-value pair is encountered, a compaction discards the key-value pair and does not propagate the deletion tombstone further.

Only works when exactly one version of the key is visible in the LSM below the single delete marker (otherwise an older version of the key will resurface).

\
\


\
\




























\
