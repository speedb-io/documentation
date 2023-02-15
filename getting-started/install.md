# Use prebuilt binaries

1. Download and extract the [Speedb package](https://github.com/speedb-io/speedb/releases). You should have a `usr` directory with `include` and `lib` directories under it.
   * Add the path of the `include` directory to the compiler command line (this may depend on your build system; `-I` for GCC/Clang. Assuming you extracted the archive to `/home/user/speedb`, the flag should be `-I /home/user/speedb/usr/include`
   * Add the path to the `lib` directory and the library to the linker command line (this may depend on your build system. Assuming you extracted the archive to `/home/user/speedb`, for GCC/Clang this would be `-L /home/user/speedb/usr/lib` and `-lspeedb`, respectively)
