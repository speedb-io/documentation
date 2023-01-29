# Speedb Example

1. Clone speedb
2. cd speedb/examples
3. make speedb\_is\_awesome\_example

**Code examples**

Full example can be found under speedb/examples/speedb\_is\_awesome\_example.ccp

```cpp
// Open the storage 
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
std::string value; db->Get(ReadOptions(), "key", &value);    
```

```cpp
// close DB 
db->Close();
```

\
