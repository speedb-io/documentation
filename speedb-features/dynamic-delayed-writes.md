---
description: >-
  The purpose of this document is to describe how Speedb v2.2.0 has achieved
  performance stabilization with its dynamic delayed write mechanism
---

# Dynamic Delayed Writes

## Overview

This feature offers an alternative method to calculate the delayed write rate using the flag - use\_dynamic\_delay. Based on the CF slowdown limits, the new calculation reduces the delayed write rate (DWR) in a linear manner.

Here is where the DWR is calculated and set:

Whenever the state of the LSM changes, which happens in every call to InstallSuperVersion, RecalculateWriteStallConditions checks the state of the CF and checks if its in a delayed state if one of the following happens:&#x20;

1. Writing to the last memtable when there's more than 3
2. L0 files > level0\_slowdown\_writes\_trigger
3. compaction pending bytes > soft\_pending\_compaction\_bytes\_limit

## Motivation

In the current calculation, the DWR is determined based on changes in compaction pending bytes between the current call to RecalculateWriteStallConditions and the previous call (which is called on every InstallSuperVersion). Whenever compaction pending bytes are less than the previous value, the current DWR is increased by 25%, while when they are equal to or greater than the previous value, the DWR is reduced by 20%.

Thus, regardless of how much the state has changed, the DWR will always change by the same amount, which results in an unjust increase or decrease in the DWR. This in turn, causes spikes in write performance as can be seen in the following analysis - [https://github.com/speedb-io/speedb/issues/154#issuecomment-1310053858](https://github.com/speedb-io/speedb/issues/154#issuecomment-1310053858) .

Furthermore, these changes are larger the closer we are to the original (user) DWR, e.g. reduce by 20Mb/s from 100Mb/s versus reduce by 2 Mb/s from 10MB/s. As a consequence, the closer we get to the stop limitation, the less we change the amount by which we delay, which is not what we want.\
We would like the delay to be small for small violations of the limitations (L0 files, compaction pending bytes, memory) and increase the delay by bigger quantities as we get closer to the stop limitations but for the meanwhile we’ll settle for equal amounts of change.



When use\_dynamic\_delay = false, the write rate is calculated in SetupDelay in db/column\_family.cc. (called from RecalculateWriteStallConditions). \


We propose to calculate the DWR based on how much we're currently in a violation of the limitation and calculate according to the user DWR and not the previously calculated DWR. e.g. if the user DWR is 100MB/s, L0 slowdown is set to 10 and stop is set to 20 files, and there are 14 L0 files:

There are 10 steps until the write stop. So each step toward the stop is a 10MB/s reduction from the delayed rate the user specified and the other way when increasing the write rate.

10 L0 files - 100 Mb/s

11 L0 files - 90 Mb/s

until total stop at 20 L0 files.

Same solution for compaction pending bytes.&#x20;

For memtables, set up a 10X delay once the last memtable is being written to.

The biggest violation out of all 3 cases (L0 files, pending bytes or memtables) will be the one deciding the delay factor.&#x20;

The new calculation is done in CalculateWriteDelayDivider which is called fromRecalculateWriteStallConditions through DynamicSetupDelay.\


### Test&#x20;

We tested the new write delay mechanism on a database with 80 million objects, while the value size is 1KB, We used 50 threads and 16 CPU cores Intel(R) Xeon(R) Platinum 8370C CPU @ 2.80GHz.

We were running a massive write workload to trigger the delayed write mechanism.&#x20;

### Test Results

Based on the graph below, we can see that the new write delay mechanism provides consistent performance compared with the previous mechanism.&#x20;

\


<figure><img src="../.gitbook/assets/_                                                                                              Random Writes .png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/_                                                                                                                Seek random while write.png" alt=""><figcaption></figcaption></figure>
