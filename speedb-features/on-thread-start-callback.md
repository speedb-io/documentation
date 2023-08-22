# On Thread Start Callback

## Motivation:

This is a community driven feature following issue Github [#476](https://github.com/speedb-io/speedb/issues/476).&#x20;

This feature provides the ability to limit the Speedb threads to run on specific cores in order to gain performance or better control CPU usage.



## Solution:

Speedb introduces a new capability that allows Speedb users to configure a callback function that will run for each Speedb thread that is being spawned.

Using this callback a user could configure the thread affinity the way the user wants.\


## How to use it?

An example is also available under: examples/on\_thread\_start\_callback\_example.cc

{% code fullWidth="false" %}
```cpp
//  This example will pin all Speedb threads to the first core


#include <functional>
#include <iostream>
#include <memory>

#include "rocksdb/db.h"
#include "rocksdb/options.h"

using namespace ROCKSDB_NAMESPACE;

#if defined(OS_WIN)
std::string kDBPath = "C:\\Windows\\TEMP\\speedb_thr_affinity";
#else
std::string kDBPath = "/tmp/speedb_thr_affinity";
#endif

int main() {
  // Open the storage
  DB* db = nullptr;
  Options options;
  // create the DB if it's not already present
  options.create_if_missing = true;
  auto f = [](std::thread::native_handle_type thr) {
// callback to pin all Speedb threads to the first core.
#if defined(OS_WIN)
#include "winbase.h"
    SetThreadAffinityMask(thr, 0);
#else
#include "pthread.h"
    std::cout << "thread spawned, thread_id: " << thr << std::endl;
    cpu_set_t cpuset;
    CPU_ZERO(&cpuset);
    CPU_SET(0, &cpuset);
    pthread_setaffinity_np(thr, sizeof(cpu_set_t), &cpuset);
#endif
  };
  options.on_thread_start_callback =
      std::make_shared<std::function<void(std::thread::native_handle_type)>>(f);
  Status s = DB::Open(options, kDBPath, &db);
  assert(s.ok());

  // append new entry
  std::string key = "key_1";
  std::string put_value = "Speedb is awesome!";
  s = db->Put(WriteOptions(), key, put_value);
  assert(s.ok());

  // retrieve entry
  std::string get_value;
  s = db->Get(ReadOptions(), key, &get_value);
  assert(s.ok());
  assert(get_value == put_value);
  std::cout << get_value << std::endl;

  // close DB
  s = db->Close();
  assert(s.ok());
  return 0;
}

```
{% endcode %}



\
\
\
