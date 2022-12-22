# DB\_bench: Groups

## Overview

This is a usability enhancement for db\_bench.

Currently,  db\_bench allows running multiple benchmarks in a single invocation, for example:

./db\_bench -benchmarks fillseq,flush,overwrite …. \[other flags as applicable]

However, all of the benchmarks share the same configuration.

A user could run db\_bench multiple times on an existing Database(s), each run with its own configuration, by specifying the ‘-use\_existing\_db’ flag. For example, say the user wanted to do a long fillup of a database, followed by multiple benchmarks, on the filled up database. That user could do that by running the following commands:

./db\_bench -benchmarks fillrandom -duration 3600 -threads 50 …

./db\_bench -benchmarks overwrite -use\_existing\_db -duration 60 -report\_file overwrite.csv …

./db\_bench -benchmarks readrandom  -use\_existing\_db -duration 60 -report\_file readrandom.csv …

However, whenever db\_bench exits (at the end of each run), it closes the database (and flushes the memtables).

This new feature allows a user to run db\_bench once, but specify groups of benchmarks, each with its own configuration, as if they were run separately. For example, the above 3 commands could be combined using the new feature as follows:

./db\_bench -groups “-benchmarks fillrandom -duration 3600 -threads 50 …“ “-benchmarks overwrite -duration 60 -report\_file overwrite.csv …“ “-benchmarks readrandom  -duration 60 -report\_file readrandom.csv …“

The same 3 benchmarks would run (each has its own group), each with its own desired configuration. The difference is that the DB(s) remain open between groups. So, when one group ends and a new one starts, the database retains its state. Specifically, the database stays open, and the memtables are not flushed.

## Syntax

&#x20;./db\_bench -groups '\<group1>' '\<group2>' '\<group3>' …

## Notes

* Each group has its own configuration. A group’s configuration is a set of valid db\_bench flags, exactly as if they were given in the normal db\_bench invocation (single-group mode).
* &#x20;The first group (the MASTER group) sets the initial configuration for all subsequent groups. Subsequent groups may override the initial configuration (some limitations apply, see below). Please note that the master group’s configuration is the default for any of the following groups. So, each group following the first starts with the master’s configuration, and may override or add to that.
* As noted above, the database(s) are opened when running the first group and  remain open thereafter. They are closed, as always, when db\_bench exits. However, please note that:
*
  * Any flag that affects the options may only be set during the first group. These flags will be silently ignored in subsequent groups (e.g., flags such as ‘-write\_buffer\_size’, or ‘max\_write\_buffer\_number’).
  * The following flags may only be set for the first group. Attempting to override them in subsequent groups will cause db\_bench to abort when running the violating group:
  *
    * \-db
    * \-statistics
    * \-statistics\_string
    * \-env\_uri
    * \-fs\_uri
    * \-simulate\_hdd
    * \-simulate\_hybrid\_fs\_file
    * \-simulate\_hybrid\_fs\_file;
    * \-simulate\_hybrid\_hdd\_multipliers
    * \-simulate\_hybrid\_hdd\_multipliers;
    * &#x20;\-seed
* The ‘-groups’ or ‘--groups’ keyword indicates that the user works in groups mode.
* The ‘groups’ keyword is not a gflags flag. It is displayed in the db\_bench’s help as do all other flags, but, again, is not a gflags flag:
*
  * It must be the first “flag”, right after the db\_bench executable’s name
  * It is not passed to gflags for parsing as do all other flags.
  * If the “-groups” keyword is missing or misplaced, db\_bench assumes the default, “single-group” mode, that was the only one thus far.
* Every group is delimited by ‘’ (single-quotes) or “” (double-quotes). When values of flags themselves need to use quotes, the user should just use one type of quotes as  the group’s delimiter, and the other for the values.

### Additional Examples

&#x20;./db\_bench -groups '-num 100 -benchmarks "fillseq,readrandom"' '-num 200  -benchmarks readrandom' '-benchmarks readrandom -reads 10000'

Group 1

* The fillseq,readrandom benchmarks will run.
* \-num = 100
* All other flags have their default values as usual.

Group 2

* The readrandom benchmark will run.
* \-num = 200 (set by the group 1 and not overridden).
* All other flags retain their (default) values from group 1

Group 3

* The readrandom benchmark will run
* \-num = 1000 (overriding -num of group 1).
* \-reads = 1000 (overriding the default (implicit) value of -reads set internally by db\_bench)

&#x20;

\
