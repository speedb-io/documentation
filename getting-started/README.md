# Getting Started

### **How to get up and running with Speedb**

Speedb can be installed on a new environment or replaced with your existing key value store (Rocksdb) in three easy steps.

#### lets run an example

1. [clone speedb](install.md) 
2. cd speedb/examples
3. make speedb_is_awesome_example


#### Code examples

Full example can be found under speedb/examples/speedb_is_awesome_example.ccp

<pre class="language-cpp" data-line-numbers><code class="lang-cpp"><strong>// Open the storage 
</strong>DB::Open(options, "/speedb", &#x26;db);     
</code></pre>

<pre class="language-cpp" data-line-numbers><code class="lang-cpp"><strong>// append new entry 
</strong>std::string key = "key";
std::string val = "Speedb is awesome!"; 
db->Put(WriteOptions(), key, val);
</code></pre>

<pre class="language-cpp" data-line-numbers><code class="lang-cpp"><strong>// retrieve entry 
</strong>std::string value; db->Get(ReadOptions(), "key", &#x26;value);    
</code></pre>

<pre class="language-cpp" data-line-numbers><code class="lang-cpp"><strong>// close DB 
</strong>db->Close();
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

