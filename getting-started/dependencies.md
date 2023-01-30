# Dependencies

Speedb's library should be able to compile without any dependency installed.

Although we recommend installing some compression libraries:

* [zlib](http://www.zlib.net/) - for data compression
* [bzip2](http://www.bzip.org/) - for data compression
* [lz4](https://github.com/lz4/lz4) - for extremely fast data compression
* [snappy](http://google.github.io/snappy/) - for fast data compression
* [zstandard](http://www.zstd.net) - for fast real-time compression algorithms
* All of our tools depend on [gflags](https://gflags.github.io/gflags/). This is a library that handles command line flag processing. Note that this is only required for building the tools, and that you can compile the Speedb library even if you don't have gflags installed.
* `make check` will also check code formatting, which requires [clang-format](https://clang.llvm.org/docs/ClangFormat.html)
* To build the RocksJava static target, then CMake is required for building Snappy
* If you want to run microbench (e.g, `make microbench`, `make ribbon_bench` or `cmake -DWITH_BENCHMARK=1`), Google benchmark >= 1.6.0 is needed.
* [Per Operating System](dependencies.md#supported-platforms)

## **Prerequisites:**

1. &#x20;gcc/clang with C++17 support (GCC >= 7, Clang >= 5).&#x20;

{% hint style="info" %}
Note: By default, the binary we produce is optimized for the platform you're compiling on (`-march=native` or the equivalent).
{% endhint %}

{% hint style="info" %}
Note: SSE4.2 will be enabled automatically if your CPU supports it.
{% endhint %}

{% hint style="info" %}
Note: To print a warning if your CPU doesn't support SSE4.2, build with `USE_SSE=1 make static_lib` or, if using CMake, `cmake -DFORCE_SSE42=ON`.
{% endhint %}

{% hint style="info" %}
If you want to build a portable binary, add `PORTABLE=1` before your make commands, as follows: `PORTABLE=1 make static_lib`, or `cmake -DPORTABLE=1` if using CMake.
{% endhint %}





## Supported Platforms

### **Linux - Ubuntu**

* Upgrade your gcc to version at least 7 to get C++17 support
* Install gflags:\
  First, try `sudo apt-get install libgflags-dev`\
  If this doesn't work and you're using Ubuntu, here's a good tutorial: (http://askubuntu.com/questions/312173/installing-gflags-12-04)
* Install Snappy:\
  This is usually done easily by running: `sudo apt-get install libsnappy-dev`.
* Install zlib:\
  Try `sudo apt-get install zlib1g-dev`
* Install bzip2: `sudo apt-get install libbz2-dev`
* Install lz4: `sudo apt-get install liblz4-dev`
* Install zstandard: `sudo apt-get install libzstd-dev`

### **Linux - CentOS / RHEL**

* Upgrade your gcc to version to at least 7 in order to get C++17 support
*   Install gflags:

    ```
        git clone https://github.com/gflags/gflags.git
        cd gflags
        git checkout v2.0
        ./configure && make && sudo make install
    ```

    **Notice**: Once installed, add the include path for gflags to your `CPATH` environment variable and the lib path to `LIBRARY_PATH`.\
    If installed with default settings, the include path will be `/usr/local/include` and the lib path will be `/usr/local/lib`
*   Install Snappy:

    ```
        sudo yum install snappy snappy-devel
    ```
*   Install zlib:

    ```
        sudo yum install zlib zlib-devel
    ```
*   Install bzip2:

    ```
        sudo yum install bzip2 bzip2-devel
    ```
*   Install lz4:

    ```
        sudo yum install lz4-devel
    ```
*   Install ASAN (optional for debugging):

    ```
        sudo yum install libasan
    ```
* Install zstandard:
  *   With [EPEL](https://fedoraproject.org/wiki/EPEL):

      ```
      sudo yum install libzstd-devel
      ```
  *   With CentOS 8:

      ```
      sudo dnf install libzstd-devel
      ```
  *   From source:

      ```
      wget https://github.com/facebook/zstd/archive/v1.1.3.tar.gz
      mv v1.1.3.tar.gz zstd-1.1.3.tar.gz
      tar zxvf zstd-1.1.3.tar.gz
      cd zstd-1.1.3
      make && sudo make install
      ```

### **OS X**

* Install the latest C++ compiler that supports C++ 17:
  * Update XCode: Run `xcode-select --install` (or install it from XCode App's settting).
  * Install via [homebrew](http://brew.sh/):
    * If you're first time developer in MacOS, you still need to run: `xcode-select --install` in your command line.
    * Run `brew tap homebrew/versions; brew install gcc7 --use-llvm` to install gcc 7 (or higher).

### **Windows** (Visual Studio 2017 to up)

* Read and follow the instructions at CMakeLists.txt
