---
description: This page summarize information about the Speedb releases.
---

# Releases

Find out what's new in Speedb!

This page summarize information about new features, bugs fixes and enhancements per release.

<details>

<summary>v2.5.0</summary>

Release date: 14-June-2023 | Based on RocksDB 8.1.1

### New Features

* Dirty memory: connect wbm to global delay : Delay writes gradually based on memory usage of the WriteBufferManager (WBM) in order to gain stability. To use this feature, pass allow\_stall = true to the ctor of WBM and the db needs to be opened with options.use\_dynamic\_delay = true. by [@Yuval-Ariel](https://github.com/Yuval-Ariel) in [#423](https://github.com/speedb-io/speedb/pull/423)
* [Prevent flush entry followed delete operations currently during memtable flush](../enhancements/remove-single-delete-elements-during-memtable-flush.md) , if the key has a match key in the delete range table and this record has no snapshot related to it, we still write it with its value to the SST file. This feature keeps only the delete record and reduces SST size for later compaction. by [@ayulas](https://github.com/ayulas) in [#418](https://github.com/speedb-io/speedb/pull/418) ([#411](https://github.com/speedb-io/speedb/issues/411))

### Enhancements

* Log: Add the CF name and job ID to all compaction job traces by [@udi-speedb](https://github.com/udi-speedb) in [#511](https://github.com/speedb-io/speedb/pull/511)
* Log: Display cf names in rolled logs with their options by [@udi-speedb](https://github.com/udi-speedb) in [#518](https://github.com/speedb-io/speedb/pull/518)
* Log: Report the name of cf-s whose options are skipped in the log by [@udi-speedb](https://github.com/udi-speedb) in [#520](https://github.com/speedb-io/speedb/issues/520)
* db\_stress: Add cost\_write\_buffer\_to\_cache flag by [@udi-speedb](https://github.com/udi-speedb) in [#513](https://github.com/speedb-io/speedb/pull/513)

### Bug Fixes

* Fixed sorted hash memtable use after free bug by [@ayulas](https://github.com/ayulas) in [#553](https://github.com/speedb-io/speedb/pull/553) [#501](https://github.com/speedb-io/speedb/issues/501)
* Sanitize max\_num\_parallel\_flushes in WBM if 0 by [@udi-speedb](https://github.com/udi-speedb) in [#515](https://github.com/speedb-io/speedb/pull/515)
* WriteController: fix for stop while shutting down. Also switch to waiting a sec on the CV each time. This is required since a bg error \* doesn't signal the CV in the WriteController. by [@Yuval-Ariel](https://github.com/Yuval-Ariel) in [#499](https://github.com/speedb-io/speedb/pull/499)
* Fixed UnlockWALStallCleared test in utilities/transactions/transaction\_test.cc by [@Yuval-Ariel](https://github.com/Yuval-Ariel) in [#514](https://github.com/speedb-io/speedb/pull/514)
* Always assume optimize\_filters\_for\_memory=false when creating a paired bloom filter by [@udi-speedb](https://github.com/udi-speedb) in [#528](https://github.com/speedb-io/speedb/pull/528)
* db\_bench and stress: fixed WBM initiation by [@udi-speedb](https://github.com/udi-speedb) in [#510](https://github.com/speedb-io/speedb/pull/510)
* db\_bench: Create a WBM once for all db-s regardless of their use in different groups by [@udi-speedb](https://github.com/udi-speedb) in [#551](https://github.com/speedb-io/speedb/pull/551)
* Fixed Tombstone test failure as a result of not clear local variable by [@ayulas](https://github.com/ayulas) in [#561](https://github.com/speedb-io/speedb/pull/561)
* Makefile: Remove pycache artifacts after running gtest-parallel [#495](https://github.com/speedb-io/speedb/pull/495)
* AVX512: fix disabling other optimizations by [@Yuval-Ariel](https://github.com/Yuval-Ariel) in [#489](https://github.com/speedb-io/speedb/pull/489) [#489](https://github.com/speedb-io/speedb/pull/489)

### Miscellaneous

* Print optimize\_filters\_for\_memory option to the log by [@udi-speedb](https://github.com/udi-speedb) in [#537](https://github.com/speedb-io/speedb/pull/537)

**Full Changelog**: [https://github.com/speedb-io/speedb/commits/speedb/v2.5.0](https://github.com/speedb-io/speedb/commits/speedb/v2.5.0)

</details>

<details>

<summary>v2.4.1</summary>

Release date: 19-April-2023 | Based on RocksDB 7.7.8

### Enhancement

* Add the ability to create any Filter Policy in java (including ribbon filter and the Speedb paired bloom filter)  [#387](https://github.com/speedb-io/speedb/pull/387)

### Bug Fix

* Write Flow: Reduce debug log size. Note: the write flow is still experimental in this release [#472](https://github.com/speedb-io/speedb/pull/472)

</details>

<details>

<summary>v2.4.0</summary>

Release date: 5-April-2023 | Based on RocksDB 7.7.8

#### New Features

* [New beezcli: ](../speedb-features/beezcli-tool.md)Interactive CLI that offers data access and admin commands [#427](https://github.com/speedb-io/speedb/pull/427)
* [Global delayed write rate](../speedb-features/global-delayed-write.md): manage the delayed write rate across multiple CFs/databases [#392](https://github.com/speedb-io/speedb/pull/392)
* [New write flow](../speedb-features/write-flow.md): Major improvement of writing while reading. Note: This feature is experimental and it consumes slightly more memory in this release  [#445](https://github.com/speedb-io/speedb/pull/445)

#### Enhancements

* Skip expired object while using DBWithTtl [#403](https://github.com/speedb-io/speedb/pull/403)

#### Bug Fixes

* Dynamic delay writes: fix pending bytes rate calculation  [#451](https://github.com/speedb-io/speedb/pull/451)
* Global delay write: check again credits under mutex [#438](https://github.com/speedb-io/speedb/pull/438)

#### Miscellaneous

* Add back accidental revert in DropRandomUnsyncedData  [#402](https://github.com/speedb-io/speedb/pull/402)
* Add speedb licenses to code [#409](https://github.com/speedb-io/speedb/pull/409)
* Enforce writing licenses inside a source file [#410](https://github.com/speedb-io/speedb/pull/410)
* Makefile: Use speedb libs in `build_size` target  [#399](https://github.com/speedb-io/speedb/pull/399)
* Replace uint with unsinged int (Windows Build Failure)  [#421](https://github.com/speedb-io/speedb/pull/421)
* crashtest: dont reroll skip\_list or HashSpdRepFactory  [#452](https://github.com/speedb-io/speedb/pull/452)
* options: Forward declare WriteBufferManager [#433](https://github.com/speedb-io/speedb/pull/433)

</details>

<details>

<summary>v2.3.0</summary>

Release date: 15-February-2023 | Based on RocksDB 7.7.8

#### New Feature

* [New Live configuration changes: ](../speedb-features/live-configuration-changes.md)support changing immutable options on the fly [#294](https://github.com/speedb-io/speedb/pull/294)

#### Enhancements

* Improved performance while using the sorted-hash memtable [#299](https://github.com/speedb-io/speedb/pull/299)

<!---->

* Added prints and query option of Index size per CF - LRU Cache Only [#368](https://github.com/speedb-io/speedb/pull/368)
* Add F\_BARRIERFSYNC for Sync operations on MacOS (addresses the issue raised in [rocksdb#11035](https://github.com/facebook/rocksdb/issues/11035))  [#319](https://github.com/speedb-io/speedb/pull/319)
* Paired-Bloom-Filter: Balancing rounding to batches between the bottom-most level and other levels [#371](https://github.com/speedb-io/speedb/pull/371)
* db\_bench: recreate only specified DBs in a group of benchmarks  [#370](https://github.com/speedb-io/speedb/pull/370)
* &#x20;Use a NoSyncFileSystem to skip Sync/FSync to reduce test times ( based on RocksDB PR [9545](https://github.com/facebook/rocksdb/pull/9545))  [#380](https://github.com/speedb-io/speedb/pull/380)

#### Bug Fixes

* Delayed Writes: fix L0 calc bug [#311](https://github.com/speedb-io/speedb/pull/311)
* util: Fixed compilation failure on Fedora 35 with gcc 11.2.1 and gflag 2.2.2   [#396](https://github.com/speedb-io/speedb/pull/396)
* Fixed compilation failure on windows [#384](https://github.com/speedb-io/speedb/pull/384)
* Fixed compilation issues on Mac by [#393](https://github.com/speedb-io/speedb/pull/393)
* Use the Test Name for the dbname when running unit tests  [#353](https://github.com/speedb-io/speedb/pull/353)

#### Miscellaneous&#x20;

* add Speedb is awesome example to the getting started section  [#382](https://github.com/speedb-io/speedb/pull/382)
* unit tests: fix CompactionServiceTest.RemoteEventListener ([#314](https://github.com/speedb-io/speedb/issues/314)) [#354](https://github.com/speedb-io/speedb/pull/354)
* artifacts check tool - readme file was updated  [#293](https://github.com/speedb-io/speedb/pull/293)
* don't use AVX512 with asan  [#398](https://github.com/speedb-io/speedb/pull/398)

</details>

<details>

<summary>v2.2.1</summary>

Release date: 30-January-2023 | Based on RocksDB 7.7.8

#### Bug Fixes

* Delayed Writes: fixed L0 calculation bug [#311](https://github.com/speedb-io/speedb/pull/311)

#### Miscellaneous

* Added WBM's cache info to the log [#313](https://github.com/speedb-io/speedb/pull/313)
* db\_bench: set db\_bench defaults to Speedb [#322](https://github.com/speedb-io/speedb/pull/322)
* build: remove the dependency on GNU Parallel for running unit tests [#243](https://github.com/speedb-io/speedb/pull/243)

</details>

<details>

<summary>v2.2.0</summary>

Release date: 22-December-2022 | Based on RocksDB 7.7.3

#### New Features

* [Proactive flushes ](../speedb-features/proactive-flushing.md)for better resources utilization [#185](https://github.com/speedb-io/speedb/pull/185)
* [Dynamic delayed write ](../enhancements/dynamic-delayed-writes.md)mechanism for consistent performance [#2](https://github.com/speedb-io/speedb/pull/281)

#### Enhancements&#x20;

* Paired block bloom: Removed the bits-per-key limitation for better results  [#163](https://github.com/speedb-io/speedb/pull/163)
* [DB-bench groups support](../tools/db\_bench-groups.md): Allow running multiple benchmark, each with its own configuration [#250](https://github.com/speedb-io/speedb/pull/250)
* db\_bench: Support '--groups' in addition to '-groups'  [#295](https://github.com/speedb-io/speedb/pull/295)
* db\_stress enhancement: Support control over WBM's allow\_stall [#289](https://github.com/speedb-io/speedb/pull/289)
* [Shorten latency while switch](../enhancements/reduce-switch-memtable-latency.md) generic memtable  [#297](https://github.com/speedb-io/speedb/pull/297)

#### Bug Fixes

* db\_bench: bug fix inserted   [#265](https://github.com/speedb-io/speedb/pull/265)
* db\_bench: ErrorExit from static func bug  [#278](https://github.com/speedb-io/speedb/pull/278)
* Proactive Flushes: compilation warnings fix  [#307](https://github.com/speedb-io/speedb/pull/307)

#### Miscellaneous

* Added info to the log file for artifact testing  [#286](https://github.com/speedb-io/speedb/pull/286)
* Disable LoadCustomizableTest.LoadMemTableRepFactoryTest  [#305](https://github.com/speedb-io/speedb/pull/305)

</details>

<details>

<summary>v2.1.1</summary>

Release date: 15-November-2022 | Based on RocksDB 7.2.2

#### Bug Fixes

* Shorten latency while switch memtable ([#14](https://github.com/speedb-io/speedb/issues/14))
* Fixed a crash that occurred when using the hash memtable. ([#98](https://github.com/speedb-io/speedb/issues/98))
* memtable\_list: avoid rolling back memtable flush on CF drop ([#144](https://github.com/speedb-io/speedb/issues/144))
* crashtest: fix 0 value of data\_block\_hash\_table\_util\_ratio ([#214](https://github.com/speedb-io/speedb/issues/214))
* deletefile\_test: fix breakage caused by the compaction threads change ([#218](https://github.com/speedb-io/speedb/pull/218))
* cmake: clean up on successful runs and randomise test scheduling ([#202](https://github.com/speedb-io/speedb/issues/202))
* build: add a version build-tag for non-release builds ([#156](https://github.com/speedb-io/speedb/issues/156))
* build: support ccache and sccache in the Makefile build ([#170](https://github.com/speedb-io/speedb/issues/170))
* Update README.md
* docs: fix instructions for building Speedb in README.md and INSTALL.md
* readme typo fix by [@azmisaquib](https://github.com/azmisaquib) ([#223](https://github.com/speedb-io/speedb/pull/223))
* build\_version: apply the build tag to the Speedb version string ([#231](https://github.com/speedb-io/speedb/issues/231))
* build: correctly handle merge commits when calculating a build tag ([#207](https://github.com/speedb-io/speedb/pull/207))
* db\_test2: fix BackgroundPurgeTest ([#236](https://github.com/speedb-io/speedb/issues/236))
* Update HISTORY.md ([#239](https://github.com/speedb-io/speedb/pull/239))
* db\_bench: Fix a bug when destructing a Benchmark with multiple db-s ([#234](https://github.com/speedb-io/speedb/issues/234))
* db\_bench: add benchmark - seektodeletedranges ([#201](https://github.com/speedb-io/speedb/pull/201))

</details>

<details>

<summary>v2.1.0</summary>

Release date: 26-October-2022 | Based on RocksDB 7.2.2

#### New Features

* Added new [Paired bloom filter](../speedb-features/paired-bloom-filter.md) that reduces false positive rate with the same performance and memory. In some configurations, the memory consumption is even reduced by up to 30%.\
  Note: Paired bloom filter is recommended to use when the number of bits per key is larger than 10. ([#54](https://github.com/speedb-io/speedb/pull/54))
* Added Plugin Tests to builds ([#143](https://github.com/speedb-io/speedb/pull/143))

#### Enhancements

* The default value for the number of compaction threads has changed to 8 ([#194](https://github.com/speedb-io/speedb/pull/194))
* An infrastructure addition for a future feature: added API to retrieve the amount of immutable memory that can be freed. ([#113](https://github.com/speedb-io/speedb/issues/113))
* cmake: allow running the tests in parallel like in the Makefile ([#103](https://github.com/speedb-io/speedb/pull/103))
* build: fix the java test target dependencies ([#129](https://github.com/speedb-io/speedb/pull/129))
* flush\_job: do not roll back memtable flush on CF drop and DB shutdown ([#127](https://github.com/speedb-io/speedb/pull/127))
* When background purges are used, set their priority to low instead of high, ([#151](https://github.com/speedb-io/speedb/pull/151))
* Added db\_bench option to change the parameter: avoid\_unnecessary\_blocking\_io ([#184](https://github.com/speedb-io/speedb/pull/184))
* Allow construction of Filter Policy from uri to the tools ([#83](https://github.com/speedb-io/speedb/pull/83))

#### Miscellaneous

* Remove the GPL as an alternative license ([#119](https://github.com/speedb-io/speedb/pull/119))
* Fix shell tab-completions in makefile ([#148](https://github.com/speedb-io/speedb/pull/148))
* Added Speedb change-log to the HISTORY.md file ([#189](https://github.com/speedb-io/speedb/pull/189))
* makefile: rework the dependency graph for faster test runs startup ([#175](https://github.com/speedb-io/speedb/pull/175))
* Change the name of the output artifacts to Speedb ([#66](https://github.com/speedb-io/speedb/pull/66))



</details>

<details>

<summary>v2.0.0</summary>

Release date: 04-August-2022 | Based on RocksDB 7.2.2

#### New Features

* Added a new [hash based memtable ](../speedb-features/sorted-hash-memtable.md)that supports concurrent reads and writes
* Added ability to create MemTableFactory from URI/string to tools

#### Bug Fixes

* Avoid comparing Status using == as it compares only status codes. The comparison breaks when comparing against status::NoSpace() since it has a status code of `Code::kIOError` and only a subcode of `SubCode::kNoSpace`
* Fixed snapshots leak in optimistic\_transaction\_example: whenever the example is run under ASan, snapshots are acquired but not released, resulting in a memory leak error.
* ldb: fix get to print the entire value
* db\_bench: fix Rocksdb bug of last\_ref assertion. Test fails to delete multi-dbs correctly.
* db\_bench: fix SeekRandom and ReadRandomWriteRandom to work on all CFs instead of the default
* db\_bench to report accurate response time when using rate limit
* db\_test: add test for - forward the incomplete status on no\_io ([facebook/rocksdb#8485](https://github.com/facebook/rocksdb/pull/8485))
* CMake: use the old plugin infra and add support for \*\_FUNC registration

#### Miscellaneous

* LOG: Print write\_buffer\_manager size to LOG
* LOG: change log header to SpeeDB
* LOG & db\_bench: metadata\_cache\_options - print to LOG and support its configuration in db\_bench
* db\_impl: use unique\_ptr in DBImpl::Open for nicer memory management
* Explicitly compare the SuperVersion pointer in column\_family
* Rename rocksdb threads to speedb
* Add a version number to Speedb builds
* Clang-Format: Do not include third-party code as any changes are either version updates or fixes.
* Git: add clangd cache to .gitignore

</details>
