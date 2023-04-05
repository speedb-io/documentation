# Global Delayed write

## Overview

The Global delayed write feature is designed to address a problem where users running multiple databases on the same disk can experience disk bandwidth starvation, leading to uneven resource distribution. This feature converts the WriteController from a per-database object into a shared object that can record and enforce the live delay requirements of all the databases that are connected to it. In this document, we'll explain the problem, the solution, and how to use the global delayed write feature.

This feature was introduced with Speedb open source version 2.4.0.&#x20;

## Problem description:

When multiple databases are running on the same disk, some databases may experience disk bandwidth starvation. The starvation is caused by the following scenario:

Db1 enters a slowdown/stop condition because the disk can't keep up with the ingest rate.&#x20;

Other dbs using the same disk are running without delay (since they themselves are not in a delay state) and they are taking up all the disk bandwidth.

In order for Db1 to exit the slowdown/stop condition, it needs to run compactions but the disk bandwidth for these compactions is unavailable due to the other dbs which leaves Db1 in a delayed state indefinitely or until the other dbs release some disk bandwidth.



## Solution Description

The global delayed write feature addresses this problem by converting the WriteController from a per-database object into a shared object. This shared object records and enforces the live delay requirements of all the databases that are connected to it. It works by enforcing the most extreme delay (i.e., the minimum rate) on all databases.

In practice, the WriteController is now a shared\_ptr option in ImmutableDBOptions. It keeps track of and enforces all the delay requirements of the column families (cfs) in the databases to which those options were passed. The default behavior of having one WriteController per database will remain, but users can also set a WriteController in the options. Note that setting a WriteController in the options is only valid when use\_dynamic\_delay = true.

## How to Use the Global Delayed Write?&#x20;

To use the global delayed write  feature, users should set use\_dynamic\_delay = true in the options. By default, each database will have one WriteController, but users can set a WriteController in the options if desired..



### Example code:

```cpp
Options options;
options.use_dynamic_delay = true
options.write_controller.reset(new WriteController(
      options.use_dynamic_delay, options.delayed_write_rate));
  DB* db1 = nullptr;
  DB* db2 = nullptr;
  Status s = DB::Open(options, "db_path1", &db1);
  Status s2 = DB::Open(options, "db_path2", &db2);
```

&#x20;

## Test Results

The new global delayed write was tested with db\_bench, using the command below.

db\_bench --compression\_type=None -db=/data/ -num=200000000 -value\_size=1000 -key\_size=16 --delayed\_write\_rate=536870912 -report\_interval\_seconds=1 -max\_write\_buffer\_number=4 -num\_column\_families=3 -histogram -max\_background\_compactions=8 -cache\_size=8388608 -max\_background\_flushes=4 -bloom\_bits=10 -benchmark\_read\_rate\_limit=0 -benchmark\_write\_rate\_limit=0 -report\_file=fillrandom.csv --disable\_wal=true --benchmarks=fillrandom,levelstats --column\_family\_distribution=50,24,26 --level0\_slowdown\_writes\_trigger=8 -write\_buffer\_size=268435456



The graph shows that with the new global delayed write enabled the results are more stable by 30%.\


Without the global delayed write, there are many stalls observed during the test while with the feature enabled there were no stalls.&#x20;

\
<img src="../.gitbook/assets/Screen Shot 2023-04-05 at 23.49.25.png" alt="" data-size="original">\
\




\
