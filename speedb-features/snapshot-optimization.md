# Snapshot Optimization

## Overview

This documentation provides an overview of a performance enhancement feature aimed at improving the performance of snapshots in transactional databases. The enhancement leverages previous snapshots to avoid unnecessary database mutex acquisition, leading to improved performance in scenarios where snapshot periods are short and few changes occur.

The goal of this enhancement is to optimize snapshot performance by utilizing previous snapshots when no changes have occurred since the last one. By avoiding the acquisition of a new database mutex and reusing the previous snapshot, performance is improved, especially in scenarios with read-intensive operations and a multithreaded environment.

## Motivation:

When taking a snapshot in a database, acquiring a database mutex has a performance impact on the entire system. In transactional databases, where maintaining data consistency is crucial, snapshots are taken for each transaction to ensure that operations work on a locked copy. This results in frequent acquisition of the database mutex, affecting overall performance.\


The improvement allows the database to use the previous snapshots taken, if  no changes were made since then and by that to avoid another db mutex. New snapshot would not be taken when the last snapshot has the same sequence number as a new one.

When transactional databases use mostly read operations, it improves performance when used with a multithreaded environment and as well other scenarios of taking large amounts of snapshots with mostly read operations.

The most important information inside a snapshot is its Sequence number, which allows the compaction to know if the key-value should be deleted or not.

The sequence number is being changed when modification happens in the db.

## Feature Details

Snapshot Reuse:&#x20;

The enhancement introduces the capability for the database to reuse the previous snapshot if no changes have been made since its creation. This is determined by comparing the sequence number of the last snapshot with the new snapshot. If they have the same sequence number, indicating no modifications, a new snapshot is not taken, and the previous snapshot is utilized.

Importance of Sequence Number

The sequence number plays a vital role in the snapshot performance enhancement. It allows the compaction process to determine whether a key-value pair should be deleted or retained. The sequence number is incremented whenever a modification occurs in the database, ensuring accurate tracking of changes.



## Prerequisites&#x20;

\*\*This Feature must have folly library installed.\*\*



To utilize this feature, the Folly library must be installed. Ensure that the necessary dependencies and configurations are in place before enabling the snapshot performance enhancement.\


Folly (acronymed loosely after Facebook Open Source Library) is a library of C++14 components designed with practicality and efficiency in mind. Folly contains a variety of core library components used extensively at Facebook. In particular, it's often a dependency of Facebook's other open source C++ efforts and a place where those projects can share code.



## How to install folly on Ubuntu:

```bash
sudo apt install libssl-dev libfmt-dev
git clone https://github.com/facebook/folly
cd folly
sudo ./build/fbcode_builder/getdeps.py install-system-deps --recursive
mkdir build_
cd build_
cmake .. -DBUILD_SHARED_LIBS=ON
make -j $(nproc) install

```

If there is an error with the compile process above that is related with fmt,&#x20;

Please run:

```bash
 sed -i "s/format_to/fmt::format_to/g" /usr/include/fmt/chrono.h
```

## **To compile Speedb with the snapshot optimization enhancement please compile it using cmake:**

```bash
git clone https://github.com/speedb-io/speedb
cd speedb
mkdir build
cd build
cmake .. -DWITH_SNAP_OPTIMIZATION=ON -DBUILD_SHARED_LIBS=ON -DCMAKE_BUILD_TYPE=Release
make -j $(nproc)
```



## How has it been implemented?

In order to cache the snapshots, there is last\_snapshot\_

(folly::atomic\_shared\_ptr, lock free atomic\_shared\_ptr) in order to

access the last\_snapshot\_ created and point to it.

For every GetSnapshotImpl call (where snapshots are being created), the

function checks if the sequence number is different than last\_snapshot\_,

if no, it creates new snapshot and inside this snapshot it adds a

reference to last\_snapshot\_ (the reference is cached\_snapshot), so this sequence number will remain inside

SnapshotList (SnapshotList is the list of the snapshots in the system and used in compaction to show which snapshots are being used), if there are still snapshots holding this sequence number. If the sequence number has changed or the last\_snapshot\_ is nullptr it will create the snapshot while acquiring db\_mutex.



For ReleaseSnapshotImpl (deleting a snapshot).

We will unref the last\_snapshot\_ (using comapre\_exchange\_weak) and if the refcount becomes 0, it will

call Deleter and remove this snapshot entirely from the SnapshotList and

continue with taking the db mutex.

If there are still references, it will return without taking it out from

the SnapshotList nor taking the db mutex



## Test

The test was running on Azure Standard\_L64s\_v3 machine.\
Each thread in every test is creating 1M snapshots and delete those.

## Test results&#x20;

The following graph compares the time in seconds between RocksDB and Speedb while running with different numbers of threads.

As can be seen in the graph below, Speedb performs the operation in less than 60 seconds, regardless of the number of threads running while RocksDB reached to almost 2 minutes when the number of threads increased.  \
Threads here are transactions that requires snapshots. \
\


<figure><img src="https://lh5.googleusercontent.com/oGBwfNXYg_50Ckc4U86y5cUsKIu7i_HURseldVRYOUXPd5x-ghOWFqQiv-mTd_RmjumH3mHtJE0y-cWLgOn4AlfKQbpfXvehyROwwYgNmyyYE1c7jcHZ4BdrAsslDFuqS9DNnBgSiF8MC5eKwlz_Iaw" alt=""><figcaption></figcaption></figure>

User and system time graph shows that with Speedb, the performance is much higher without taking any extra CPU resources.&#x20;



<figure><img src="https://lh4.googleusercontent.com/rAxcWrb9n7MggucPlbKEfiLrtJld14rdvOAIHcP1r_GrPPgwcJKsIc-_6FELKBUgNWb43J0jYWJWx54hFaqhDzIqFtYPbQbg4JuHvuw9cUhg0OaSCCk5nAZhtR3WoPDzcPs1Lo7kQ8H-atsmdrHLVc8" alt=""><figcaption></figcaption></figure>

## Best practices&#x20;

The feature is mostly beneficial when:

* You are using snapshots with mostly read workload&#x20;

Example: when using transactional databases

\
\
\
\
\
