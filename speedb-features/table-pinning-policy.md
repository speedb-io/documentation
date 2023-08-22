# Table Pinning Policy

## Overview&#x20;

Speedb allows the user to specify an alternative policy to control how table memory is pinned.  Pinned memory is not subject to lookup or expiration by the cache and provides quicker access.  Because pinned memory is a more limited resource than cache, the use of it should be more controlled.\


## Motivation

Today there is an option to pin items to the cache, to increase performance.&#x20;

The user needs to decide what type of items to pin to the cache and what should be left to the block cache to manage.  The value of pinning is that it provides better performance while the downside is that the database may allocate memory more than the user intended and may cause a catastrophic out of memory event. Therefore, in many environments the users give up the performance gain of pinning, in order to get a predictable amount of memory while in others the users allocate a much smaller cache than it may in order to have safety margins. This new pinning control mechanism allows pinning index and filters up to a defined threshold so the user can enjoy the performance benefits of pinning without getting into out of memory condition.&#x20;



## Details&#x20;

Speedb provides a new scoped table pinning policy that pins data based on its level and how much data is currently pinned.  The policy has the following properties:

* Capacity: How much data can be pinned.  The capacity should be set to a percentage of the cache size;
* Bottom Percent: What percentage of the pinned capacity can be used by data at the bottom-most level of the LSM tree.  If pinned memory exceeds this capacity, data at the bottom-most level will no longer be pinned;
* Mid Percent: What percentage of the pinned capacity can be used by data in the mid levels (not L0 and not bottom most). &#x20;

This scoped pinning policy looks at the type and level of the data to be pinned and determines, based on this information and how much data is already pinned, if the new data should be pinned as well.



## Usage



A TablePinningPolicy is associated with a BlockBasedTableOptions.  If not specified, the default policy (matching the existing RocksDB behavior) is used.  The new Speedb scoped pinning policy can be created via TablePinningPolicy::CreateFromString.  For example, the call

<mark style="color:blue;">`TablePinningPolicy::CreateFromString(...,`</mark>&#x20;

&#x20; <mark style="color:blue;">`“id=speedb_scoped_pinning_policy; capacity=1G; bottom_percent=10;`</mark>

&#x20;  <mark style="color:blue;">`mid_percent=75”, …)`</mark>&#x20;

creates a new scoped pinning policy that will:

* Pin up to 1G of memory;
* Limit the bottom-most level to no more than 10% of 1G
* Limit the mid-level pinning to 75% of 1G.

By sharing the same pinning policy amongst the process.  If not specified, the default capacity is 3G, the default bottom and mid percentages are 10% and 80%, respectively.&#x20;



In order to get accurate accounting of memory, set “cache\_index\_and\_filter\_blocks=true” when using the scoped pinning policy.

\
