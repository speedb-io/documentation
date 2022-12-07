# Getting Started

### **How to get up and running with Speedb**

Speedb can be installed on a new environment or replaced with your existing key value store (Rocksdb) in three easy steps.



### Install

#### Prebuilt Binaries

1. Download and extract the [Speedb package](https://github.com/speedb-io/speedb/releases). You should have a `usr` directory with `include` and `lib` directories under it.
2. Add the path of the `include` directory to the compiler command line (this may depend on your build system; `-I` for GCC/Clang. Assuming you extracted the archive to `/home/user/speedb`, the flag should be `-I /home/user/speedb/usr/include`
3. Add the path to the `lib` directory and the library to the linker command line (this may depend on your build system. Assuming you extracted the archive to `/home/user/speedb`, for GCC/Clang this would be `-L /home/user/speedb/usr/lib` and `-lspeedb`, respectively)

#### From Source

To install Speedb from source, [follow our full guide](install.md) on how to clone, compile, and use the library.&#x20;

#### Code example

<pre class="language-cpp" data-line-numbers><code class="lang-cpp">#include "rocksdb/db.h" 

<strong>int example() { 
</strong><strong>    // Open the storage 
</strong>    DB* db; Options options;
    DB::Open(options, "/speedb", &#x26;db); 
    
    // append new entry 
    std::string key = "key";
    std::string val = "Speedb is awesome!"; 
    db->Put(WriteOptions(), key, val);
<strong>    
</strong><strong>    // retrieve entry 
</strong><strong>    std::string value; db->Get(ReadOptions(), "key", &#x26;value);
</strong>    
    // close DB 
    db->Close(); return 0;
}
</code></pre>

###

### Replace your existing Rocksdb with Speedb

1. [Download the package](https://github.com/speedb-io/speedb/releases)
2. Copy the library along with the include folder to your project&#x20;
3. Link it&#x20;

That's it  - you can now start using using Speedb!

Note: Speedb is fully compatible with Rocksdb. The changes can be reverted at any time, although we'd appreciate your feedback if, for any reason, you choose to revert.



Questions? Please feel free to contact us via [GitHub](https://github.com/speedb-io/speedb/discussions) or [Discord](https://discord.gg/52yzKZ5G9D).

{% hint style="info" %}
Please be sure to read and follow our [Code of Conduct](https://github.com/speedb-io/speedb/blob/main/CODE\_OF\_CONDUCT.md).
{% endhint %}

