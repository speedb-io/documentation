# Paired Bloom Filter

The Paired Bloom Filter consumes less memory than FastLocalBloomFilterPolicy, RockDB’s default cache-local bloom filter, without increasing the rate of false positives.

The Paired Bloom filter algorithm introduced in Speedb release 2.1.0.&#x20;

## Overview

### **RocksDB’s default bloom filter**

Although RocksDB’s default cache-local bloom filter (FastLocalBloomFilterPolicy) offers faster CPU consumption than the theoretical standard bloom filter, its rate for false positives is significantly worse.

This trade-off deteriorates the performance/memory footprint even further as the amount of bit per key increases (typically from \~20 bits per key and up). This is especially problematic in use cases where high accuracy from the filter is required.

Moreso, once very high bits-per-key levels are reached, the rate for false positives can be extremely high.

### **Speedb’s Paired Bloom Filter**

Speedb’s goal was to ensure fast and low consumption but _without sacrificing accuracy_, offering a better trade-off in terms of memory footprint vs. the rate of false positives.

**The result:** In use cases where high accuracy is required, Speedb’s Paired Bloom Filter consumes 30% less memory, without an increase in the false-positive rate.

{% hint style="info" %}
The construction and use of the Paired Bloom Filter is \~2x slower than the default bloom filter. In most use cases the performance degradation will be insignificant. However, this should be reduced in the next version due to optimization and additional improvements.

**We recommend using the Paired Bloom Filter only for a bits per key rate that is larger than 10.**&#x20;

The current Paired Bloom Filter is suitable when all files contain a sufficient number of keys. The typical average number of keys per file should be above 50k keys. A lower number of keys will lead to a degradation in performance.
{% endhint %}





## Related algorithms

### **Standard Bloom Filter**

The standard bloom filter is used to check whether a certain key belongs to a predefined group of _**keys**  **S**_ of size _**n**_.&#x20;

The standard bloom filter is a vector of ** **_**m**_**  **_**bits**_ (denoted here as _**V**_), which are initially set to zero.&#x20;

Insertion of _key **e**_ in is done by mapping the key to _**k**_ locations in the bloom filter using _k_ independent hash functions (h\__1 (e),...h\__k(e) ).&#x20;

The bits in the corresponding locations are then set to 1.&#x20;

The process is repeated for all keys in **V**.&#x20;

A _**key e**_ belongs to _**S**_ only if the bits _**V(h\_i (e))**_ are 1 for **i=1..k**&#x20;

Therefore, in order to perform a query to check whether a key belongs to _**S,**_ you need to map the key to _**k**_ locations in the bloom filter using the _**k**_ independent hash functions, and check the bits in the bloom filter (**V**).&#x20;

If one of the bits is 0, the queried key is NOT in _**S**_. If all the bits are 1, the queried key might belong to _**S**_, but false positives may occur.&#x20;

The bits per key rate is defined as: **c = m/n**, and controls the false positive rate (FPR); The larger the bits per key, the lower the FPR. Hence, there is a trade-off between FPR and the memory footprint of the bloom filter.&#x20;

This is the optimal case:&#x20;

**k = ln(2)\*m/n** and\
**FPR = (½)^k**

### **Blocked Bloom Filter**

The Standard Bloom Filter (above) has the specific drawback of memory non-locality when performing a query.&#x20;

This is due to the query process of a _key **e**_, which includes accessing _**V**_ in the locations _**h\_i (e) (i=1..k)**_ producing on average 2 cache misses for negative keys and **k** cache misses for positive keys.&#x20;

In order to make the query process cache efficient, the Blocked Bloom Filter was proposed. In this filter, the elements are first mapped to a certain _block **V**_ out of blocks using a hash function _**h\_0 (e)**_.&#x20;

Each block is of the size of a cache line (512 bits) and is initialized to 0.&#x20;

After selecting the relevant _**** block **V\_l**_ , an element is inserted by mapping it to _**k**_ locations within the block and setting them to 1, similar to the standard bloom filter.&#x20;

To test whether an _**element**_ is in _**S**_, the element is first mapped to a block by using the hash _**** function **h\_0**_, then mapped to locations within the block using the _**k**_ independent hash functions. Finally, the bits in the corresponding locations are verified to all be 1 (similar to the standard bloom filter).&#x20;

The blocked bloom filter produces 1 cache miss for both negative and positive keys. However, the FPR of the blocked bloom filter is substantially worse than the FPR of the standard bloom filter for large bits-per-key rates (typically larger than 20).&#x20;

For small bits-per-key the blocked bloom filter and standard bloom filter have very similar FPR-s.&#x20;

For a larger bits-per-key rate the FPR of the blocked bloom filter may be several orders of magnitude worse than the standard bloom filter (Figure 1). The reason for this phenomenon is that the number of elements mapped for each block varies significantly across blocks.&#x20;

The distribution for the number of elements in a block follows a Binomial distribution.&#x20;

The variability in the number of elements per block gives rise to two effects:

1. The FPR changes for each block due to different bits-per-key rate, and since the FPR is exponential with the bits-per-key, overloaded blocks deteriorate the total FPR more than underloaded blocks improve it on average.&#x20;
2. The number of hash functions _**k**_ is optimal only for a small fraction of the blocks.

The Blocked bloom filter is the algorithm implemented in RocksDB.

![](<../.gitbook/assets/bloom filter.jpg>)

## **How to build the Paired Bloom Filter**

As described above, to tackle the performance deterioration of the Blocked Bloom filter for static databases, we propose the Paired Bloom Filter.

These are the steps required for building the filter:

1. Determine the number of blocks needed for the filter
2. Build a histogram of the number of keys mapped per block: The basic idea is to first build a histogram of the number of keys that are mapped to each block by applying **h\_0(e)** on all keys in S.
3. Partition the blocks into batches of consecutive blocks: \
   We partition the sequence of blocks to _**L**_ consecutive batches, each including 128 consecutive blocks (cache-lines). Currently, the batch size is 128 and the number of blocks is always a multiple of the number of blocks in a batch.&#x20;
4. Within each batch: \
   a) Sort the blocks according to the number of keys mapped to every block in the batch \
   b) Pair the sorted blocks based on their number of mapped keys: \
   \- The first block (smallest number of mapped keys) is paired with the last block (largest number of mapped keys), the second block is paired with the second to last, etc. This ensures an extremely narrow distribution of the total number of mapped keys of the pairs, balancing each other with respect to the number of mapped keys.&#x20;
5. Build the bloom bits in the blocks by pairs: \
   Map the keys to their blocks - each key should be mapped both to its block and to the paired block. However, we use only half of the hash functions in one block and the other half in the paired block. **We arbitrarily** decide that the first half of hash functions _**h\_1 (e),... h\_k/2 (e)**_ is used in the block with the smaller index (within the batch), while the other half is used in the block with larger index (we currently require k to be even). \
   The bits in the corresponding locations in each block are set to _**1**_, and the process is repeated for all keys in S. \
   We save the paired block location within the batch in the first 7 bits of the block (in the case of a 128 batch, the 7 bits represent a number between 0 and 127. In the case of a different number of blocks in a batch, the number of bits will change accordingly). This leaves 505 bits in each block for the bloom bits.

### Filter building pseudo-code

#### Constants & parameters

* Batch-Size = 128&#x20;
* L = Number of Batches&#x20;
* N = Number of Blocks (= 128 \* L)
* K = Number of Hash Functions (Probes) \[Even]

#### Data structures

* Batch-Histogram: Array\[128] of integers - Initially all 0
* Histogram: Array\[L] of Batch-Histogram elements
* Batch-Pairing-Table: Array\[128] containing:
  * The index of the pair in the batch
  * Hash set indicator - 0 (Use \[h_1 (e),... hk/2 (e)]) / 1 (Use \[hk/2+1 (e),... h\_k_ (e)])&#x20;
* Pairing-Table: Array\[L] of Batch-Pairing-Table
* Block: A sequence of 512 bits (initially all 0) partitioned into:
  * 7 bits: Index in batch of the pair block (0 - 127)
  * 505 bits: Bloom filter bits
* Hash sets:
  * h\_0 (e): Maps a key to its block (0 - \[N-1]) 0
  * h_\_1 (e),... h\__k/2 (e): Hash set 0 - Determine which bits to set in the primary block 1 k/2
  * h_k/2+1 (e),... h\_k_ (e): Hash set 1 - Determine which bits to set in the secondary block

#### Algorithm

Build the histogram:

```
 For each key e in S 
  ++Histogram[h0(e)] 
 End
```

Pair blocks in the batches:

```
For each batch i from 0 to L-1 
  Sort Histogram[i] 
   Pair corresponding blocks of Pairing-Table[i] as described above 
End
```

Build the block filters:

```
For each key e in S
 Use h0(e) to calculate i and map e to its Block[i] 
 Find the block’s pair index (j) via Pairing-Table[i][j] 
 Set the first 7 bits in Block[i] to j (the pair) 
 Use Hash-Set 0 to set k/2 bits in Block[i] 
 Set the first 7 bits in Block[j] to i (the pair) 
 Use Hash-Set 1 to set k/2 bits in Block[j] 
End
```

Filter testing pseudo-code:

To test whether _key **e**_ is positive or negative, apply the following algorithm:

```
Use h0(e) to calculate i and map e to its Primary Block[i] Extract the index of the pair (j)
Determine the Primary Hash Set (0 / 1): Hash Set = (i < i)? 0 : 1 The Secondary Hash Set is: 1 - Primary Hash Set 
For all k/2 hashes h in the Primary Hash Set 
 If the bit resulting from h(e) is not set in Block[i] 
 Return NEGATIVE 
End

// Repeat for Block[j] 
For all k/2 hashes h in the Secondary Hash Set 
 If the bit resulting from h(e) is not set in Block[j] 
 Return NEGATIVE 
End

Return POSITIVE
```

#### Results (#Hash Functions=16, bits-per-key=23.2-23.4):

**The algorithm was implemented in 4 environments:** \
****1. Simulation\
****2. Clean CPP code\
3\. Speedb POC: New filter type as part of the Speedb repo\
4\. RocksDB 7 / OSS repo POC&#x20;

In all 4 cases, with 23.4 bits-pre-key, the new filter achieved an FPR of \~1/55,000, which is close to the theoretical limit of a standard bloom filter.&#x20;

### Optimal use case/workload for the paired filter

1. The proposed filter should be most beneficial when there is a need for a very small FPR. This typically occurs when the penalty of a false positive is very big compared to the filter test time (database on the disk), and when true positives are rare. Typically, a true positive rate of 1/1000 will require an FPR of <1/100,000. When true positives aren't rare, a higher FPR can be used, and the benefit of the new algorithm compared to the blocked bloom filter will be less conspicuous.
2. A common real-world example of the optimal use case is a DB that performs multiple insertions into RocksDB. This type of DB often issues a Get query before every insertion, since it must verify that the inserted key doesn't already exist in RocksDB.

### **Comparison with the Ribbon filter**

Even with our improvements**,** the Ribbon filter is superior in terms of memory-FPR trade-off by \~40%. However, the time to test a key and build the filter is expected to be 4-6 times slower than our algorithm. This means that Speedb's new algorithm offers superior results when memory is sufficient and high-speed performance is the crucial requirement.

### Forecasts and estimations

#### **1. Performance**

a) Filter building:&#x20;

It should take \~2 times to build this filter compared to the current filter. However, it might have a minor negative impact on the total flush/compaction time.

b) Filter use:&#x20;

i) For Positive queries, there's an additional cache miss, however, this cache miss should be negligible in the overall flow of a positive query. There should be a total degradation of no more than 2% in point queries iops/sec.

ii) For negative queries we expect the influence on performance to be negligible.

**Memory footprint**

1. The proposed filter with 23.4 BPK is equivalent (FPR-wise) to 28 BPK in the RocksDB’s bloom => \~16% savings.
2. Potentially, the new algorithm may improve the memory footprint by 30% for FPR of 1/1,000,000, and even more when lower FPRs are needed.

### Usage

There are 4 cases in which a user may wish to customize the type of filter used:

1. filter\_bench
2. db\_bench
3. db\_stress
4. Configuring the filter in the user’s application code

The new filter policy type is a RocksDB plug-in. To use it, you must specify its - _**speedb.PairedBloomFilter**_ - name in the plug-in mechanism or create it directly.&#x20;

{% hint style="info" %}
The above string is case-sensitive, and must be used exactly as spelled.
{% endhint %}

As with other types of bloom filter policy, the user needs to configure the number of bits-per-key to use. Depending on the case, that number is either mandatory or optional. When optional and not specified, a default will be used.&#x20;

{% hint style="info" %}
The bits-per-key is a floating-point number (double).
{% endhint %}

#### Filter bench

./filter\_bench -impl=speedb.PairedBloomFilter \[-bits\_per\_key=] \[-average\_keys\_per\_filter=] \[-allow\_bad\_fp\_rate]

Default bits-per-key: 10

Default average-keys-per\_filter: 10000

Default allow-bad-fp-rate: false (don’t allow)

**Note:** As explained in section XXXXXXX (please refer to the appropriate section), using these defaults results in an effective FP rate that is intolerable, and will result in the following assertion:

_util/filter\_bench.cc: 472: Assertion prelim\_rate < tolerable\_rate failed_

This indicates that the benchmark should not be run under this effective FP rate.

To allow filter\_bench to run, you should do one or more of the following:

* Increase the bits per key
* Increase the average keys per filter&#x20;
* Allow bad FP rate

#### db\_bench

./db\_bench -filter\_uri=speedb.PairedBloomFilter:23.4

The bits-per-key value is mandatory in this case. Failure to specify this number will result in the following error message:

_failure creating filter policy\[speedb.PairedBloomFilter]: Not implemented: Could not load FilterPolicy: speedb.PairedBloomFilter_

#### db\_stress

./db\_stress -filter\_uri=speedb.PairedBloomFilter \[-bloom\_bits=]&#x20;

Default bits-per-key: 10

#### Configuring the filter in the user’s application code

Include the filter policy header:&#x20;

_#include "rocksdb/filter\_policy.h"_

Set the filter in the table options:

```
Options options;
BlockBasedTableOptions bbto;
ConfigOptions config_options;
config_options.ignore_unknown_options = false;
config_options.ignore_unsupported_options = false;
Status s = FilterPolicy::CreateFromString(config_options, "speedb.PairedBloomFilter:23.2", &bbto.filter_policy);
assert(s.ok());
Options.table_factory = NewBlockBasedTableFactory(bbto);

```

You may _view_ an example in the file called "speedb\_db\_bloom\_filter\_test.cc" (plugin/speedb/paired\_filter/speedb\_db\_bloom\_filter\_test.cc).

### Testing&#x20;

#### Unit testing&#x20;

Add applicable unit testing for the following aspects:&#x20;

1. Customizable support: Tests like those in customizable\_test.cc&#x20;
2. Functional tests: Tests like those in full\_filter\_block\_test.cc&#x20;
3. FPR Test: A test of the FPR of the filter under certain conditions (mainly the bits-per-key). By default, this will test on a relatively small number of keys to reduce the time required to complete the test.&#x20;

#### Performance testing&#x20;

**Optimal use-case**

To test the optimal scenario, the following is recommended:&#x20;

1. Random fill of a large number N of keys (eg., N = 100 Million) followed by \
   Read random of 100 X N keys (eg., 10 Billion)&#x20;
2. Run this scenario 3 times: \
   New paired block bloom filter, BPK = 23.4 \
   Default RocksDB Blocked bloom filter, BPK = 23.4 \
   Ribbon filter, BPK = 23.4&#x20;

**Additional use-cases**&#x20;

Run the usual performance tests with the default filter and the new filter. Use the default bits per key for both, then compare the results. This is to verify that the new filter doesn’t cause an unexpected performance degradation.&#x20;



### Performance benchmark

#### Test 1 - same FPR, less memory

The benchmark compares between the local bloom filter and the new paired bloom filter. \
The test was running with the following configuration:

* Number of objects: 1 Billion
* Value size: 256 bytes
* Write buffer size: 122MB (default)
* Number of threads: 4
* Max number of write buffer: 1
* Number of Column families: 1 (default)
* Number of CPU cores: 16
* Compression mode: None&#x20;

#### Results&#x20;

As compared to the traditional bloom filter, the new Paired bloom filter showed a reduction in memory consumption while keeping the same false positive rate.&#x20;

Memory usage decreases as the number of bits per key increases.&#x20;

|                           | Paired Bloom | Local Bloom |
| ------------------------- | ------------ | ----------- |
| Bits per key              | 40           | 29          |
| Total memory consumption  | 6200MB       | 7700MB      |

This test resulted in a 23% reduction in memory consumption while keeping the same performance for random read workload.&#x20;

<figure><img src="../.gitbook/assets/Screen Shot 2022-10-25 at 14.00.10.png" alt=""><figcaption><p>100% random read workload </p></figcaption></figure>

#### Test2 - same memory usage, improved FPR

As compared to the traditional bloom filter, the new Paired bloom filter in this test improved the false positive rate while using the almost the same amount of memory.&#x20;

Test configuration:

* Number of objects: 1 Billion
* Value size: 256 bytes
* Write buffer size: 256MB (default)
* Number of threads: 4
* Max number of write buffer: 1
* Number of Column families: 1 (default)
* Number of CPU cores: 8
* Compression mode: None

|                          | Paired Bloom | Local Bloom |
| ------------------------ | ------------ | ----------- |
| Bits per key             | 20           | 20          |
| Total memory consumption | 4900B        | 4800MB      |

This test simulates high ratio of non-existing keys and the disk is the bottleneck. \
The graph below demonstrate the significant improvement with random read in this type of workload.&#x20;

<figure><img src="../.gitbook/assets/Screen Shot 2022-10-25 at 13.58.23.png" alt=""><figcaption></figcaption></figure>
