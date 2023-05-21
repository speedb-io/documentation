# Reduce switch memtable latency

## Motivation&#x20;

The MemTableRepFactory class (each memtable representation inherits it) can be set as a “PrepareMemtableCreation” which  will create a switch background thread that is responsible to construct a new memtable object to be used when switch occurs. A new memtable will be created when we fetch this memtable, so we'll always have one memtable available for use.

In case we don't have a ready memtable (this can happen if the switch memtable happened before the previous trigger was triggered) we create the memtable immediately .

To be able support this ability , memtable representation needs to divide the memtable construction to pre/post phases . This allows us to get the correct memtable’s  parameters as defined in the mutable CF configuration.



## Implementation  &#x20;

The MemTableRepFactory class (each memtable representation inherits it) can be set as a “PrepareMemtableCreation” which  will create a switch background thread that is responsible to construct a new memtable object to be used when switch occurs. A new memtable will be created when we fetch this memtable, so we'll always have one memtable available for use.

In case we don't have a ready memtable (this can happen if the switch memtable happened before the previous trigger was triggered) we create the memtable immediately .

To be able support this ability , memtable representation needs to divide the memtable construction to pre/post phases . This allows us to get the correct memtable’s  parameters as defined in the mutable CF configuration.\


## Advantage&#x20;

1 - A switch memtable (size of the write buffer, threshold of the write buffer manager, wall size etc.) does not have a memory construction penalty and will allow for fluent acceptance of new writes. &#x20;

2 - Simplified code

## Disadvantage&#x20;

1 - Each CF will have two mutable memtables (one active and one on hold), so we multiply the memtable construct memory usage.

\
