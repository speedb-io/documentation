# Performance testing

We run performance tests for several reasons:

**Dev:** This is to ensure there's no degradation due to additional code and to ensure stabilization. \
In the case of a performance feature, this type of testing ensures that we achieve the expected performance gain.

{% hint style="info" %}
Dev tests are a combination of regression tests and larger benchmarks.&#x20;
{% endhint %}

**Stress test:** By running our code over a longer period of time and under stress, we ensure there's no additional degradation. &#x20;

**Competitive analysis:** This enables us to compare our performance to our competitors and show our achievements.&#x20;

{% hint style="info" %}
For each test we run, it's important to understand the reason for running this specific test, in addition to the configuration and pattern.
{% endhint %}

### Test coverage&#x20;

When testing, we aim to get maximum coverage. This includes variations on configurations and io patterns.

### Terminology&#x20;

This page uses the below terms.&#x20;

For the configuration key size default 16B the below are value sizes.

* Small obj – 64B&#x20;
* Large obj – 1000B&#x20;
* Small DB – Below the instance RAM < 100GB (consider, smaller instance and smaller DB )
* Large DB – Above the instance RAM > 150GB&#x20;
* Huge DB – Factor larger than large
* db\_bench benchmark, or benchmark – a set of tests running on a specific configuration
* Test - a single result within a benchmark

### Performance instance&#x20;

* type i3.4xlarge&#x20;
* Intel(R) Xeon(R) CPU E5-2686 v4 @ 2.30GHz&#x20;
* 16 cores&#x20;
* RAM 122GB

### Current test configurations&#x20;

* Large obj, small DB&#x20;
* Small obj small DB&#x20;
* Large obj, large DB&#x20;
* Large obj huge DB – in this configuration, multiple CF are also tested&#x20;

{% hint style="info" %}
It's possible to run tests in other configurations.
{% endhint %}

### Test IO patterns&#x20;

Currently, each benchmark includes fillup + 10 patterns:

1. Fillup 100% random writes&#x20;
2. 100% random reads&#x20;
3. 100% random write (rewrite)&#x20;
4. Mix load reads writes&#x20;
5. Seeks

{% hint style="info" %}
The tests are run in order; the state of the DB is affected by previous tests.
{% endhint %}

### Performance optimization settings

We added the following settings to our db\_bench testing:

* Reduce the CPU bottlenecks on compaction
* max\_background\_flushes=4
  * Allow better handling of faster workflows
* max\_write\_buffer\_number=4
  * Allow better handling of faster workflows
* write\_buffer\_size=265MB&#x20;
  * In some cases, improves memory handling
* bloom\_bits=10

### Performance test results

The performance benchmark result will be uploaded to a dashboard (AKA web admin). The result will also be compared to a baseline either in graphs or tables. We aim to get simple results in the form of:

* Better - Improvement with >X% across all benchmarks & tests, with memory & disk space usage the same or less&#x20;
* Same - +- X% across all benchmarks & tests, memory & disk space&#x20;
* Degraded – >X% less across all benchmarks & tests, memory & disk space same or more
* Inconclusive – Improvement in some tests and degradation in others

{% hint style="info" %}
In addition to the IOPS results, we will monitor and report the memory usage, the disk space consumption, and CPU stats (although at this stage, we don't evaluate them).
{% endhint %}

### Additional parameter tests

Running the same tests with different parameters to increase coverage.&#x20;

Suggested:

1. Compression - off/LZ4/Snappy&#x20;
2. Wal (redo log) - off/on&#x20;
3. Number of threads – 50/16/4/1&#x20;
4. Optional feature active/disabled&#x20;
