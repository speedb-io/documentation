---
description: Speedb Interactive tool
---

# Beezcli Tool

## Introduction

The beezcli command line tool offers multiple data access and database admin commands. beezcli supports interactive mode with history and it will remember the --db and --secondary\_path once the interactive mode is being enabled.

beezcli is wrapping RocksDB's and LevelDB's ldb tool.

Some examples are listed below. For more information, please consult the help message displayed when running beezcli without any arguments or with --help.

\
How to use it?
--------------

The beezcli tool is available under the [tools](https://github.com/speedb-io/speedb/tree/main/tools) directory in Speedb repository.&#x20;

By default, \`beezcli\` can only be used against a DB that is offline. Operating the DB, even for read-only operations, might make changes to the DB directory, e.g. info logs.

#### Open as secondary

An option \`--secondary\_path=\<secondary\_path>\` would open the DB as a \[\[Secondary instance]], which can be used to open a running DB and/or to minimize impacts to the DB directory. This argument can be used for any beezcli command, but since not all operations can be done against secondary instances, some operations will fail. Besides write operations which would definitely fail with secondary instances, some read operations might also fail.

## Compilation and dependencies

### &#x20;Dependencies

\- cmake

\- make

\- gflags

\- snappy

\- zlib

\- bzip2

\- zstandard

\- lz4

\- gnu readline

### &#x20;How to compile

For mac os and Linux:

````
```zsh
git clone https://github.com/ofriedma/speedb beezcli
cd beezcli
mkdir build
cmake .. -DCMAKE_BUILD_TYPE=Release -DWITH_CORE_TOOLS=ON -DWITH_TOOLS=ON
make -j $(nproc) beezcli 
````



{% hint style="info" %}
Notice: Currently the tool is not supported on Windows
{% endhint %}



#### Example for interactive mode

Example data access:

```bash
$./beezcli --interactive --db=/tmp/test_db
/tmp/test_db
beezcli> put a1 b1 --create_if_missing
OK
beezcli> get a1
b1
beezcli> delete a1
OK
beezcli> get a1
Failed: Get failed: NotFound:
beezcli> quit


Ciao

```

Example dump database:

```bash
$./beezcli --interactive --db=/tmp/test_db
/tmp/test_db
beezcli> put a1 b1 --create_if_missing
OK
beezcli> dump
a1 ==> b1
Keys in range: 1
beezcli> dump --no_value
a1
Keys in range: 1
beezcli> dump --count_only

beezcli> idump
'a1' seq:0, type:1 => b1
Internal keys in range: 1
beezcli> delete a1
OK
beezcli> idump
'a1' seq:4, type:0 => 
'a1' seq:0, type:1 => b1
Internal keys in range: 2
```



Example data access sequence:

```bash
$./beezcli --db=/tmp/test_db --create_if_missing put a1 b1
    OK 
    $ ./beezcli --db=/tmp/test_db get a1
    b1
 
    $ ./beezcli --db=/tmp/test_db get a2
    Failed: NotFound:
    
     $ ./beezcli --db=/tmp/test_db scan
    a1 : b1
 
    $ ./beezcli --db=/tmp/test_db scan --hex
    0x6131 : 0x6231
 
    $ ./beezcli --db=/tmp/test_db put --key_hex 0x6132 b2
    OK
 
    $ ./beezcli --db=/tmp/test_db scan
    a1 : b1
    a2 : b2
  
    $ ./beezcli --db=/tmp/test_db get --value_hex a2
    0x6232
 
    $ ./beezcli --db=/tmp/test_db get --hex 0x6131
    0x6231
 
    $ ./beezcli --db=/tmp/test_db batchput a3 b3 a4 b4
    OK
 
    $ ./beezcli --db=/tmp/test_db scan
    a1 : b1
    a2 : b2
    a3 : b3
    a4 : b4
 
    $ ./beezcli --db=/tmp/test_db batchput "multiple words key" "multiple words value"
    OK
 
    $ ./beezcli --db=/tmp/test_db scan
    Created bg thread 0x7f4a1dbff700
    a1 : b1
    a2 : b2
    a3 : b3
    a4 : b4
    multiple words key : multiple words value
```



To dump an existing speedb database in HEX:

```bash
$ ./beezcli --db=/tmp/test_db dump --hex > /tmp/dbdump
```



To load the dumped HEX format data to a new Speedb database:

```bash
$ cat /tmp/dbdump | ./beezcli --db=/tmp/test_db_new load --hex --compression_type=bzip2 --block_size=65536 --create_if_missing --disable_wal
```

\


To compact an existing Speedb database:

```bash
$ ./beezcli --db=/tmp/test_db_new compact --compression_type=bzip2 --block_size=65536
```

You can specify the command line \`--column\_family=\<string>\` for which column family your query will be against.



\`--try\_load\_options\` will try to load the options file in the DB to open the DB. It is a good idea to always try to have this option on when you operate the DB. If you open the DB with default options, it may mess up LSM-tree structure which can't be recovered automatically.

\


\
