---
description: >-
  This page provides a basic introduction and guides on how to contribute to
  Speedb.
---

# Contribute code

## Code Style

Speedb’s code follows the [Google C++ style guide](https://google.github.io/styleguide/cppguide.html), which you can read [more about here](https://google.github.io/styleguide/cppguide.html).

For formatting, we limit each line to 80 characters. Most formatting can be done automatically by running:

```
build_tools/format-diff.sh
```

Alternatively, if you use [GNU make](https://www.gnu.org/software/make/), simply run `make format`. If you lack any of the dependencies required to run the script, the script will print out instructions for you to install them.

## Contribution workflow

Like most open-source projects in GitHub, each Speedb contributor works on their own fork, sending pull requests to Speedb’s repo. Once a reviewer approves the pull request, a Speedb team member will merge it.

Read more about the [Pull Request](submit-a-pull-request.md) process here.

## Unit tests <a href="#unit-tests" id="unit-tests"></a>

If you make a code-related change, be sure to add a unit test for validation.

For new features, new unit tests or test scenarios must be added, even if the changes have been validated manually. This is to make ensure future contributors can rerun the tests to validate that their changes don't cause any issues.

Speedb uses [GTest](https://github.com/google/googletest) for the C++ unit tests and [JUnit](https://junit.org/) for the Java unit tests.&#x20;

### C++ unit tests

For  C++ unit tests, it's preferable to add a test to an existing unit test suite (in files that end with `_test.cc`) in order to keep build and test time to a minimum.&#x20;

That said, if you're adding a test for a new feature and it doesn't belong in any of the existing test suites, you can add a new file.&#x20;

Be sure to update the `TEST_MAIN_SOURCES` variable in `[`[`src.mk`](https://github.com/speedb-io/speedb/blob/main/src.mk)`](<https://www.notion.so/zigel/src.mk>)` (note the backslashes at the end of each line) as well as the `TESTS` variable in `[`[`CMakeLists.txt`](https://github.com/speedb-io/speedb/blob/main/CMakeLists.txt)`](<https://www.notion.so/zigel/CMakeLists.txt>)`.

You can run the C++ unit tests using the Makefile as explained below, or, if you're using CMake, using `ctest`.&#x20;

The Makefile supports running the unit tests in parallel using GNU Parallel, so it's recommended that you install GNU Parallel first using your system's package manager (refer to the GNU Parallel [official webpage](https://www.gnu.org/software/parallel/) for more information).

In order to run unit tests execute the following command:

```
make check
```

This will build Speedb and run the tests. For better CPU utilization and to speed up the build, you can use the `-j` flag.

Note that this flag only affects the build, not the tests themselves. If you have GNU Parallel installed, you can control the number of parallel tests to run using the environment variable `J`. \
For example, to build on a 64-core CPU and run the tests in parallel, you can run:

```
make J=64 check -j64
```

Unlike `-j`, which, if not provided defaults to 1, if `J` isn't provided, one job will be run per core.

If you switch between release and debug build, normal or lite build, or compiler or compiler options, call `make clean` first.&#x20;

Here's a safe routine to run all tests:

```
make clean && make check -j64
```

### Debug single unit test failures

You can run a specific unit test by running the test binary that contains it.&#x20;

If you use GNU make, the test binary will be in the root directory of the repository. Note: If you use CMake, the test binary will be in your build directory.&#x20;

For example, the test `DBBasicTest.OpenWhenOpen` is in binary `db_basic_test`, so simply running the following will run all tests in the binary:

```
./db_basic_test
```

GTest provides some useful command line parameters. To view them, call `--help`:

```
./db_basic_test --help
```

The command line parameter that you're most likely to use is probably `--gtest_filter`, which allows you to specify a subset of the tests to run.&#x20;

For example, if you only want to run `DBBasicTest.OpenWhenOpen`:

```
./db_basic_test --gtest_filter="*DBBasicTest.OpenWhenOpen*"
```

By default, the test DB created by tests is cleared even if test fails. You can try to preserve it by using `--gtest_throw_on_failure`.&#x20;

If you want to stop the debugger when assert fails, specify `--gtest_break_on_failure`.

The `KEEP_DB=1` environment variable is another way to preserve the test DB from being deleted at the end of a unit-test run, regardless of whether the test fails or not:

```
KEEP_DB=1 ./db_basic_test --gtest_filter=DBBasicTest.Open
```

By default, the temporary test files will be under `/tmp/rocksdbtest-<number>/` (except when running in parallel, in which case they are under `/dev/shm`).&#x20;

You can override the location by using environment variable `TEST_TMPDIR`. For example:

```
TEST_TMPDIR=/dev/shm/my_dir ./db_basic_test
```

### Java unit tests

To run Java unit tests, run:

```
make jclean rocksdbjava jtest
```

Running with `-j` can sometimes cause issues. If this occurs, try removing `-j.`

### Additional build flavors

For more complicated code changes, we ask contributors to run more build flavors before sending the code for review.

To build with _AddressSanitizer (ASAN)_, set the environment variable `COMPILE_WITH_ASAN`:

```
COMPILE_WITH_ASAN=1 make check -j64
```

To build with _ThreadSanitizer (TSAN)_, set the environment variable `COMPILE_WITH_TSAN`:

```
COMPILE_WITH_TSAN=1 make check -j64
```

To run _UndefinedBehaviorSanitizer (UBSAN)_, set the environment variable `COMPILE_WITH_UBSAN`:

```
COMPILE_WITH_UBSAN=1 make check -j64
```

To run LLVM's analyzer, run:

```
make analyze
```

### Crash tests

For changes with higher risks, other than running all of the tests with multiple flavors, a crash test cycle needs to be executed without failure. If the crash test doesn't cover the new feature, add it there.

To run all crash tests, run:

```
make crash_test -j64
make crash_test_with_atomic_flush -j64
```

If you aren't able to use GNU make, you can manually build the `db_stress` binary, and run the following commands manually:

```
  python -u tools/db_crashtest.py whitebox
  python -u tools/db_crashtest.py blackbox
  python -u tools/db_crashtest.py --simple whitebox
  python -u tools/db_crashtest.py --simple blackbox
  python -u tools/db_crashtest.py --cf_consistency blackbox
  python -u tools/db_crashtest.py --cf_consistency whitebox
```

### Commit changes

Please keep your commits:

* Standalone - The code must compile and run successfully after each commit (no breaking commits!)
* Minimal - Break your code into minimal, logically-complete chunks
* Self-reviewed - Always double-check yourself before submitting

Commit messages should:

* Start with a component name followed by a colon. For example, if you made changes to the documentation, prefix the commit message with `docs:` \
  If you only updated tests, prefix the commit message with `tests:`\
  ``For build-related changed use `build:` , etc.
* Reference a relevant issue, if any. This is especially relevant for bug fixes and new features. The issue should be referenced at the end of the first line as a hash sign followed by the issue number. For example, `#23`.
* Have the line length limited to 100 characters or less. This restriction does not apply when quoting program output, etc.
* Please use clear and grammatically-correct language, and use present tense ("add feature", not "added feature".)

###

