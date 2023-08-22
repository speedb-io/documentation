---
description: >-
  This feature related to issue 363
  https://github.com/speedb-io/speedb/issues/363
---

# Range Delete Improvement

## Introduction:

This feature improves the Range Delete and not flushing the deleted element to the SST. It only  keeps the deleted record and reduces SST size for later compaction.



The Range Delete improvement aims to reduce the resources consumed during the flush process of immutable memory tables. Specifically, this feature avoids writing value key records to the SST file if they have a corresponding match tombstone record and are not part of any existing snapshot. By skipping these records, we can reduce the size of the SST file and minimize the workload for future compaction.

In the old implementation, when flushing the memtable, if a key matches a key in the delete range table and there are no associated snapshots for that record, the key is still written to the SST file along with its value.



## Motivation&#x20;

When a group of immutable memory tables is flushed and a new SST file is created, a value key record is still written to the SST file even though it has a match tombstone record. This is an expensive resource that we can save. The SST file will be smaller, and later compaction will have less work to do.

## Implementation&#x20;

To implement this feature, we iterate over the immutable memtables group records during the building of the context of an SST file as part of the WriteLevel0Table function. Before adding a record to the SST write buffer, we check whether any action needs to be taken. For value key records, we check if the key is in the range of the match immutable memtables tombstone record. If the key is in the range and not part of an existing snapshot (i.e., its sequence is not greater than the snapshot sequence), we skip the value key record and keep only the tombstone record.



## How to use it?

This option is a new DB options called:

"use\_clean\_delete\_during\_flush"

\
\
\
