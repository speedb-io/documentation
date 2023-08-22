# Speedb Tuning Function

## Overview

This feature aims to simplify the configuration process for Speedb storage engine by providing an easy way to enable and set Speedb options to optimized configurations. By introducing the SharedOptions class and specific functions, users can effortlessly configure Speedb's features with minimal manual intervention. Also when running multiple Speedb instances, it allows to configure all of the instances using the same method, and same configuration.&#x20;

## Motivation&#x20;

Speedb users currently configure the database manually. New Speedb users are required to spend time in order to understand how to enable/disable and actually configure all the relevant options to enjoy the capabilities.&#x20;

Not all of the capabilities are enabled by default and some need different tuning per workload. The user always has the flexibility to tune the database based on their needs, but in case a user would like to enjoy all of Speedb's benefits, we give the option to let us do the work for you and tune the database or databases to a default and improved configuration, following basic system parameters you provide while using this new method.&#x20;



## Technical Details

**SharedOptions Class**

The SharedOptions class has been introduced to enhance the usability of Speedb in multiple database scenarios. It organizes shared options, making configuration more streamlined and user-friendly.

### Configuration Parameters

To benefit from this feature, users need to provide three key configuration parameters:

* **Total RAM size for Speedb:** Specify the amount of RAM allocated for Speedb operations.
* **Total number of threads for background jobs**: Determine the number of threads allocated for handling background tasks.
* **Delayed write rate:** Set the rate at which delayed writes are performed.

By utilizing the provided functions, Speedb's features can be easily configured to default settings, simplifying the process for users. The default configuration includes the following features:

* Proactive Flushing
* Global Delayed Write
* Sorted Hash Memtable
* Dynamic Delayed Writes
* Paired bloom filter

### Implementation&#x20;



_Struct SharedOptions_&#x20;

* Contain the shared configuration for multiple databases group
  * Write buffer manager
  * Cache
  * Write controller
  * Env;
  * RateLimiter;
  * SstFileManager;
  * Logger;
  * EventListener;
  * FileChecksumGenFactory;
* Contain the methodology for configuring Speedb features&#x20;
  * total\_threads
  * total\_ram\_size\_bytes
  * delayed\_write\_rate
  * Write\_buffer\_size
  * Paired bloom filter



**class SharedOptions**

```cpp
class SharedOptions {
public:
SharedOptions();
SharedOptions(size_t total_ram_size_bytes, size_t total_threads,
size_t delayed_write_rate = 256 * 1024 * 1024ul);
size_t GetTotalThreads() { return total_threads_; }
size_t GetTotalRamSizeBytes() { return total_ram_size_bytes_; }
size_t GetDelayedWriteRate() { return delayed_write_rate_; }
// this function will increase write buffer manager by increased_by amount
// as long as the result is not bigger than the maximum size of
// total_ram_size_ /4
void IncreaseWriteBufferSize(size_t increase_by);


std::shared_ptr<Cache> cache = nullptr;
std::shared_ptr<WriteController> write_controller = nullptr;
std::shared_ptr<WriteBufferManager> write_buffer_manager = nullptr;
Env* env = Env::Default();
std::shared_ptr<RateLimiter> rate_limiter = nullptr;
std::shared_ptr<SstFileManager> sst_file_manager = nullptr;
std::shared_ptr<Logger> info_log = nullptr;
std::vector<std::shared_ptr<EventListener>> listeners;
std::shared_ptr<FileChecksumGenFactory> file_checksum_gen_factory = nullptr;


private:
size_t total_threads_ = 0;
size_t total_ram_size_bytes_ = 0;
size_t delayed_write_rate_ = 0;
};

```

**SharedOptions constructor**

* _Creates a LRU cache_\
  _Default values for cache’s construction is:_&#x20;
  * _Capacity = \_total\_ram\_size\_bytes_
* _Creates a write buffer manager (with proactive flushes)_ \
  _Default values for WBM’s construction is:_&#x20;
  * _\_write\_buffer\_size = 1;_
  * _kDfltMaxNumParallelFlushes = 4U;_
* _Creates a write\_controller_ \
  _default values for WC’s construction is:_&#x20;
  * _dynamic\_delay = true_
  * _\_delayed\_write\_rate = 256 \* 1024 \* 1024ul_



**EnableSpeedbFeatures function**

* _Call EnableSpeedbFeaturesDB_
* _Call EnableSpeedbFeaturesCF_

```cpp
Options* Options::EnableSpeedbFeatures(SharedOptions& shared_options){…
}

```

&#x20;\
**EnableSpeedbFeaturesDB function**

* _Set TotalThreads;_
* _Set delayed\_write\_rate;_
* _bytes\_per\_sync = 1ul << 20;_
* _use\_dynamic\_delay = true;_
* _Set write\_buffer\_manager;_
* _Set write\_controller;_\


**DBOptions\* DBOptions::EnableSpeedbFeaturesDB(SharedOptions& shared\_options)**

```cpp
DBOptions* DBOptions::EnableSpeedbFeaturesDB(SharedOptions& shared_options) {
 …
IncreaseParallelism((int)shared_options.GetTotalThreads());
delayed_write_rate = shared_options.GetDelayedWriteRate();
bytes_per_sync = 1ul << 20;
use_dynamic_delay = true;
write_buffer_manager = shared_options.write_buffer_manager;
write_controller = shared_options.write_controller;
 …
}
```



\
**EnableSpeedbFeaturesCF function**

* _Each new column family will ask the write buffer manager to increase the write buffer size by 512 \* 1024 \* 1024ul_
* _Set cf write\_buffer\_size min(db\_wbf\_size / 4, 64ul << 20);_
* _max\_write\_buffer\_number = 32;_
* _min\_write\_buffer\_number\_to\_merge = max\_write\_buffer\_number - 1;_
* _// set the pinning option for indexes and filters_
* _Set filter to FilterPolicy to speedb.PairedBloomFilter 10 bytes per key_
* _Set cache_
* _Pinned all tables info (index, filter, compression dictionary)_
  * _Set unpartitioned\_pinning to PinningTier::kAll_
  * _Set partition\_pinning to PinningTier::kAll_
  * _Pinned by set to CacheEntryRoleOptions::Decision::kEnabled_
    * &#x20;_CacheEntryRole::kFilterConstruction_
    * _CacheEntryRole::kBlockBasedTableReader_
    * _CacheEntryRole::kCompressionDictionaryBuildingBuffer_
    * _CacheEntryRole::kFileMetadata_
* _Make sure pinned memory is accounted toward the cache_
* _Set memtable type to speedb.HashSpdRepFactory_

\
\


**ColumnFamilyOptions\* ColumnFamilyOptions::EnableSpeedbFeaturesCF(SharedOptions& shared\_options)**

```cpp
ColumnFamilyOptions* ColumnFamilyOptions::EnableSpeedbFeaturesCF(
SharedOptions& shared_options) {
…
shared_options.IncreaseWriteBufferSize(512 * 1024 * 1024ul);
auto db_wbf_size = shared_options.write_buffer_manager->buffer_size();
// cf write_buffer_size
write_buffer_size = std::min<size_t>(db_wbf_size / 4, 64ul << 20);
max_write_buffer_number = 32;
min_write_buffer_number_to_merge = max_write_buffer_number - 1;
// set the pinning option for indexes and filters
…
Set config_options "speedb.PairedBloomFilter:10" filter_policy);
…
block_based_table_options.metadata_cache_options.unpartitioned_pinning =
PinningTier::kAll;
block_based_table_options.metadata_cache_options.partition_pinning =
PinningTier::kAll;
block_based_table_options.block_cache = shared_options.cache;
auto& cache_usage_options = block_based_table_options.cache_usage_options;
CacheEntryRoleOptions role_options;
role_options.charged = CacheEntryRoleOptions::Decision::kEnabled;
cache_usage_options.options_overrides.insert(
{CacheEntryRole::kFilterConstruction, role_options});
cache_usage_options.options_overrides.insert(
{CacheEntryRole::kBlockBasedTableReader, role_options});
cache_usage_options.options_overrides.insert(
{CacheEntryRole::kCompressionDictionaryBuildingBuffer, role_options});
cache_usage_options.options_overrides.insert(
{CacheEntryRole::kFileMetadata, role_options});
…
std::string memtablerep = "speedb.HashSpdRepFactory";
…
```



**SharedOptions constructor**

* _Creates a LRU cache_\
  _default values for cache’s construction is:_&#x20;
  * _Capacity = \_total\_ram\_size\_bytes_
* _Creates a write buffer manager (with proactive flushes)_ \
  _default values for WBM’s construction is:_&#x20;
  * _\_write\_buffer\_size = 1;_
  * _kDfltMaxNumParallelFlushes = 4U;_
* _Creates a write\_controller_ \
  _Default values for WC’s construction is:_&#x20;
  * _dynamic\_delay = true_
  * _\_delayed\_write\_rate = 256 \* 1024 \* 1024ul_



{% hint style="info" %}
Note: The user is still able to configure any non-shared parameter as she is currently doing.&#x20;
{% endhint %}

\


### Usage

Below is an example  demonstrating the usage of these functions and enabling Speedb features with default configurations. It is also available in the examples directory /enable\_speedb\_features\_example.cc". \


```cpp

#include <cstdio>
#include <iostream>
#include <string>

#include "rocksdb/compression_type.h"
#include "rocksdb/db.h"
#include "rocksdb/options.h"
#include "rocksdb/slice.h"

using namespace ROCKSDB_NAMESPACE;

#if defined(OS_WIN)
std::string kDBPath1 = "C:\\Windows\\TEMP\\enable_speedb_features_example1";
std::string kDBPath2 = "C:\\Windows\\TEMP\\enable_speedb_features_example2";
std::string kDBPath3 = "C:\\Windows\\TEMP\\enable_speedb_features_example3";
std::string kDBPath4 = "C:\\Windows\\TEMP\\enable_speedb_features_example4";
#else
std::string kDBPath1 = "/tmp/enable_speedb_features_example1";
std::string kDBPath2 = "/tmp/enable_speedb_features_example2";
std::string kDBPath3 = "/tmp/enable_speedb_features_example3";
std::string kDBPath4 = "/tmp/enable_speedb_features_example4";
#endif

int main() {
  DB *db1 = nullptr;
  DB *db2 = nullptr;
  DB *db3 = nullptr;
  DB *db4 = nullptr;
  Options op1;
  Options op2;
  Options op3;
  Options op4;
  size_t total_ram_size_bytes = 512 * 1024 * 1024;
  size_t delayed_write_rate = 256 * 1024 * 1024;
  size_t total_threads = 8;

  // define SharedOptions object for each databases group
  SharedOptions so1(total_ram_size_bytes, total_threads, delayed_write_rate);

  // customize each options file except SpeedbSharedOptiopns members
  // as listed in the definition of SpeedbSharedOptiopns in options.h
  op1.create_if_missing = true;
  op1.compression = rocksdb::kNoCompression;
  //...
  op1.EnableSpeedbFeatures(so1);

  op2.create_if_missing = true;
  op2.compression = rocksdb::kZlibCompression;
  //...
  op2.EnableSpeedbFeatures(so1);

  // open the databases
  Status s = DB::Open(op1, kDBPath1, &db1);
  if (!s.ok()) {
    std::cerr << s.ToString() << std::endl;
    return 1;
  }

  s = DB::Open(op2, kDBPath2, &db2);
  if (!s.ok()) {
    std::cerr << s.ToString() << std::endl;
    return 1;
  }
  std::cout << "DBs group 1 was created" << std::endl;

  // do the same for any group of databases
  total_ram_size_bytes = 1024 * 1024 * 1024;
  delayed_write_rate = 128 * 1024 * 1024;
  total_threads = 4;
  SharedOptions so2(total_ram_size_bytes, total_threads, delayed_write_rate);

  // again customize each options object except SharedOptiopns members
  op3.create_if_missing = true;
  op3.compaction_style = rocksdb::kCompactionStyleUniversal;
  //...
  op3.EnableSpeedbFeatures(so2);

  op4.create_if_missing = true;
  op4.compaction_style = rocksdb::kCompactionStyleLevel;
  //...
  op4.EnableSpeedbFeatures(so2);

  // open the databases
  s = DB::Open(op3, kDBPath3, &db3);
  if (!s.ok()) {
    std::cerr << s.ToString() << std::endl;
    return 1;
  }

  s = DB::Open(op4, kDBPath4, &db4);
  if (!s.ok()) {
    std::cerr << s.ToString() << std::endl;
    return 1;
  }
  std::cout << "DBs group 2 was created" << std::endl;

  // creation of column family
  rocksdb::ColumnFamilyOptions cfo3(op3);
  rocksdb::ColumnFamilyHandle *cf;
  // coustomize it except SpeedbSharedOptiopns members

  // call EnableSpeedbFeaturesCF and supply for it the same SharedOptions
  // object as the DB, so2 this time.
  cfo3.EnableSpeedbFeaturesCF(so2);
  // create the cf
  s = db3->CreateColumnFamily(cfo3, "new_cf", &cf);
  if (!s.ok()) {
    std::cerr << s.ToString() << std::endl;
    return 1;
  }
  std::cout << "new_cf was created in db3" << std::endl;

  s = db3->DropColumnFamily(cf);
  if (!s.ok()) {
    std::cerr << s.ToString() << std::endl;
    return 1;
  }
  db3->DestroyColumnFamilyHandle(cf);
  if (!s.ok()) {
    std::cerr << s.ToString() << std::endl;
    return 1;
  }
  std::cout << "new_cf was destroyed" << std::endl;

  s = db1->Close();
  assert(s.ok());
  s = db2->Close();
  assert(s.ok());
  s = db3->Close();
  assert(s.ok());
  s = db4->Close();
  assert(s.ok());

  delete db1;
  delete db2;
  delete db3;
  delete db4;

  return 0;
}
```

\
