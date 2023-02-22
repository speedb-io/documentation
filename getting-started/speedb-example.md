# Speedb Example

```
git clone https://github.com/speedb-io/speedb
cd path/to/speedb
make static_lib
cd examples
make speedb_is_awesome_example
./speedb_is_awesome_example
```

**Code examples**

Full example can be found under https://github.com/speedb-io/speedb/blob/main/examples/speedb_is_awesome_example.cc

```cpp
#include <iostream>

#include "rocksdb/db.h"
#include "rocksdb/options.h"

using namespace ROCKSDB_NAMESPACE;

#if defined(OS_WIN)
std::string kDBPath = "C:\\Windows\\TEMP\\speedb_is_awesome_example";
#else
std::string kDBPath = "/tmp/speedb_is_awesome_example";
#endif
```

```cpp
// Open the storage
DB* db = nullptr;
Options options;
// create the DB if it's not already present
options.create_if_missing = true;
DB::Open(options, "/speedb", &db);     
```

```cpp
// append new entry 
std::string key = "key";
std::string val = "Speedb is awesome!"; 
db->Put(WriteOptions(), key, val);
```

```cpp
// retrieve entry 
std::string value; 
db->Get(ReadOptions(), "key", &value);    
```

```cpp
// close DB 
db->Close();
```
