---
description: >-
  This document describes the new index size reporting per column family
  introduced in Speedb 2.3.0.
---

# Report Index Size per Column Family

## Overview&#x20;

Using this feature, you can see a breakdown of the  block cache size. Prior to Speedb v2.3.0, block cache size was reported for the cache without breakdown to column families.&#x20;

Speedb v2.3.0 adds granularity to the log by adding prints of the block cache breakdown of data, index and filter sizes per column family. It is also possible to query these new properties on demand.

## Motivation&#x20;

A clear understanding of the size of the index and filter held in the cache per column family can assist in re-configuring a system if the LRU cache runs out of memory. When identifying the column family with large index size, it is possible to change the block size, change the key size or the pinning configuration of this column family in order to mitigate the issue.&#x20;

## Technical details&#x20;

The feature consists of 2 aspects:

1. A new report in the log file that accompanies the existing report of the block cache entities.
2. Allowing the users to query the newly available information via the existing properties query API.

### Reporting in the Log:

The reporting about the block cache was augmented and now, per column-family, the report includes the total size of the data blocks, filter blocks, and index blocks in the block cache.

The following image shows an example of such a log printout

<figure><img src="https://lh6.googleusercontent.com/JqM4IxkO6lHqxsoW6K7_5w-Cx9VFprvnrTR6FKbOcP6u7KtLnzMojXSZl8N7-ZcL0wcSTnqWcWPHORvKiCjlfqosNPmIIgZMv6ETs1u3nka_WpbbnWgT2HgHS5kvRGeyod7MfaK1aFKHQG1VaDi8rB4" alt=""><figcaption></figcaption></figure>

The new line in the log is the 3rd one:

Block cache \[default]  DataBlock(0.00 KB) FilterBlock(288.08 KB) IndexBlock(160.16 KB)

In this case, it is for the “default” column family.



### Querying properties:

The new per-cf statistics may be queried programmatically via the existing properties querying API. A new key was defined: DB::Properties::kBlockCacheCfStats was added to support this.

The result is a map that has the following properties:

* The name of the column-family
* The block cache’s id associated with the column family (its DB actually)
* The total size in bytes used by every cache role. The supported roles are data, filter and index. For all other roles the size would always be 0.

The following code snippet (taken from the db\_block\_cache\_test.cc unit tests file demonstrates how to use this API in the context of the test:

## Example&#x20;

```cpp
void PrintCacheStats( DBImpl* db, const std::shared_ptr<Cache>& cache){
    if  (std::string(cache->Name()) != "LRUCache") {
      // The feature is only supported for LRU-Cache
      return;
    }


    auto cf_handle = db->DefaultColumnFamily();


    std::map<std::string, std::string> cf_values;
    db->GetMapProperty(cf_handle, DB::Properties::kBlockCacheCfStats, &cf_values);


    std::cout << "Block Cache Stats for the " << cf_values[BlockCacheCfStatsMapKeys::CfName()] << " column-family:\n";
    std::cout << "Cache Id:" << cf_values[BlockCacheCfStatsMapKeys::CacheId()] << '\n';
    std::cout << "Data Size:" << cf_values[BlockCacheCfStatsMapKeys::UsedBytes(CacheEntryRole::kDataBlock)] << '\n';
    std::cout << "Index Size:" << cf_values[BlockCacheCfStatsMapKeys::UsedBytes(CacheEntryRole::kIndexBlock)] << '\n';
    std::cout << "Filter Size:" << cf_values[BlockCacheCfStatsMapKeys::UsedBytes(CacheEntryRole::kFilterBlock)] << '\n';
  }
```



\
