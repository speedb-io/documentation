# Sorted Hash Memtable

### Overview

There are currently 5 RocksDB memtable representation options, with the default being _**SkipList**_.&#x20;

We designed a new memtable representation algorithm that allows parallel writes without requiring the insertion of synchronization locks, also improving read and seek operations.

### Algorithm

The Speedb memtable combines hash and sorted vectors (which are created in background thread).

The default hash size is 1,000,000 buckets, and the maximum size for each vector is 10,000 keys. (this can be tuned as part of CF creation properties).

The goal of the hash is to achieve direct access on read operations **(O(1))**.&#x20;

The goal of the sorted vector is to improve seek operations, taking into consideration that writes are still progressing.&#x20;

A new write is added to the hash and atomic write list.

### Write operation

#### **Hash insertion:**

Since the default hash size is 1000000 buckets, the possibility of more than 1 write on the same bucket is reduced (not including an overwrite operation). This gives us direct access without synchronization.

#### Write list insertion:

Elements are inserted into the write list atomically (the anchor pointer is an atomic exchange). The write list is not sorted - this occurs in the background.

#### Sorted thread:

In the background, the sorted thread divides the write list into sorted vectors if at least one of the following conditions occurs:

1. The vector size limit is reached (default 10000)&#x20;
2. A seek operation occurs

The vector will only be valid if it is sorted.

#### Read operation:

Read is done by direct access to the match memtable hash bucket. No synchronized lock is required.

#### Seek operation:

The seek request signals the sorted thread to divide a new sorted vector by adding a special entry to the write list.&#x20;

{% hint style="info" %}
Note: New writes are not suspended!
{% endhint %}

The seek request waits for the cutoff vector to be sorted. Each seek iterator has its own binary heap of shared sorted vectors relevant to the seek time creation, so there's no dependency on other iterators' progress.

{% hint style="info" %}
If no write occurs, the last sorted vector is used (a new one isn't created).&#x20;
{% endhint %}

#### Merge sorted vectors:

When there are many seek operations, a situation could arise with many small sorted vectors.&#x20;

In this case, the sorted thread's responsibility is to merge small continuous sorted vectors, so the seek iterator will be created from a small number of sorted vectors.

### Preparing the memtable&#x20;

As mentioned above, the creation of this memtable is expensive. As such, we've created a CF background thread that creates a standby memtable, which allows us to switch memtables without wasting time.&#x20;

### Advantages

1. High performance in a heavy write workload &#x20;
2. A write request during a seek request is not blocked.&#x20;
3. Sorted vectors are used on each seek operation without being copied
4. If no write occurs between two seek operations, both seek iterators will have the same sorted vector&#x20;

### Disadvantages

1. The memtable constructor size is about 8mg (depends on the bucket size)&#x20;
2. memtable creation is expensive (creating 1000000 buckets takes time)

### Performance results&#x20;

The new sorted hash memtable was tested with db\_bench, using the configuration below.&#x20;

The following benchmark results compare the performance of Rocksdb v7.2.2 with skiplist compared to Speedb with the new sorted hash memtable code.\


Note: More details about how we test performance you can read in the Performance Testing chapter.&#x20;

Configuration:

* Number of objects: 1Billion&#x20;
* Value size: 1KB
* Write buffer size 64MB (default)
* Number of Threads: 4
* Max number of write buffers: 4
* Number of CF: 1 (default)
* Number of CPU cores: 16
* Compression mode: none

&#x20;**\*Up to 155% improvement in overwrite workload**&#x20;

<figure><img src="../.gitbook/assets/Overwrite - memtable" alt=""><figcaption><p>100% Overwrite workload <br></p></figcaption></figure>

**Up to 68% improvement in 50% random read workload**&#x20;

<figure><img src="../.gitbook/assets/random read memtable" alt=""><figcaption><p>50% Random Read performance results</p></figcaption></figure>

**Up to 15% improvement in random seek workload**&#x20;

<figure><img src="../.gitbook/assets/random seek memtable" alt=""><figcaption><p>100% random seek workload </p></figcaption></figure>

### Usage

#### db\_bench/db\_stress

./db\_bench--memtablerep=speedb.HashSpdRepFactory\
It will use the default bucket size = 1000000 In order to change the default bucket size, use the following syntax: ./db\_bench --memtablerep=speedb.HashSpdRepFactory:1000 (where 1000 is the bucket size)

#### Configuring in the user’s application code

This should be set in the DB Options object:&#x20;

```
Options options;
ConfigOptions config_options;
config_options.ignore_unknown_options = false;
config_options.ignore_unsupported_options = false;
Status s = MemTableRepFactory::CreateFromString(config_options, "speedb.HashSpdRepFactory", &options.memtable_factory);
assert(s.ok());

```

****



