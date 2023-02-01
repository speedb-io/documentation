# Speedb Example

```
1.$git clone speedb
2.$cd path/to/speedb
3.$make static_lib
4.$cd examples
5.$make speedb_is_awesome_example
6.$./speedb_is_awesome_example
```

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
