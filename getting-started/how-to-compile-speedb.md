---
description: This page describes how to clone, compile and use the library.
---

# How to Compile Speedb

Check the prerequisites before you start compiling in the [dependencies](dependencies.md) page.&#x20;

### Compile in Release mode

* **Recommended:** `make static_lib` will compile the Speedb static library (`librocksdb.a`) in release mode.
* `make shared_lib` will compile the Speedb shared library (`librocksdb.so`) in release mode.

### Compile in Debug mode

{% hint style="info" %}
**Important**: If you plan to run Speedb in production, don't compile using the default `make` or `make all` invocations. This will compile Speedb in debug mode, which is much slower than release mode.
{% endhint %}



* `make check` will compile Speedb in debug mode and run all the unit tests.
* `make all` will compile Speedb's static library, and all tools and unit tests. These tools depend on gflags, so you'll need to have gflags installed to run `make all`. This will compile Speedb in debug mode. Also, please don't use binaries compiled by `make all` in production.





##
