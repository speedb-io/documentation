# Log Parser

* [Overview](log-parser.md#speedbslogparsertool-overview)
* [Terminology](log-parser.md#speedbslogparsertool-terminology)
* [Major Capabilities](log-parser.md#speedbslogparsertool-majorcapabilities)
* [Usage](log-parser.md#speedbslogparsertool-usage)
* [Installation / Getting Started / Prerequisites](log-parser.md#speedbslogparsertool-installation-gettingstarted-prerequisites)
* [Tool’s Outputs Description](log-parser.md#speedbslogparsertool-toolsoutputsdescription)
  * [Abbreviations, Conventions and Terms used in the log parser’s outputs](log-parser.md#speedbslogparsertool-abbreviations-conventionsandtermsusedinthelogparsersoutputs)
  * [Timestamps](log-parser.md#speedbslogparsertool-timestamps)
  * [Availability of Data](log-parser.md#speedbslogparsertool-availabilityofdata)
    * [Log level](log-parser.md#speedbslogparsertool-loglevel)
    * [Availability of the information to parse and process](log-parser.md#speedbslogparsertool-availabilityoftheinformationtoparseandprocess)
  * [Options](log-parser.md#speedbslogparsertool-options)
    * [DB Options and CF Options](log-parser.md#speedbslogparsertool-dboptionsandcfoptions)
    * [Defaults](log-parser.md#speedbslogparsertool-defaults)
    * [Writing the options to the log](log-parser.md#speedbslogparsertool-writingtheoptionstothelog)
    * [Log Rolling](log-parser.md#speedbslogparsertool-logrolling)
    * [Opening an existing DB (DB Recovery)](log-parser.md#speedbslogparsertool-openinganexistingdb-dbrecovery)
    * [Number of Column Families and its implications](log-parser.md#speedbslogparsertool-numberofcolumnfamiliesanditsimplications)
    * [Unavailability of CF Names in Rolled Logs (Auto-Generated CF Names)](log-parser.md#speedbslogparsertool-unavailabilityofcfnamesinrolledlogs-auto-generatedcfnames)
* [The Output Folder](log-parser.md#the-output-folder)
* [Console Output](log-parser.md#speedbslogparsertool-consoleoutput)
* [Console Short Output Description](log-parser.md#speedbslogparsertool-consoleshortoutputdescription)
  * [Example Output](log-parser.md#speedbslogparsertool-exampleoutput)
  * [Output Fields Description](log-parser.md#speedbslogparsertool-outputfieldsdescription)
* [JSON File](log-parser.md#speedbslogparsertool-jsonfile)
  * [Top Level JSON Objects](log-parser.md#speedbslogparsertool-topleveljsonobjects)
  * [JSON Top-Level Objects Detailed Description](log-parser.md#speedbslogparsertool-jsontop-levelobjectsdetaileddescription)
    * [General](log-parser.md#speedbslogparsertool-general)
    * [Options](log-parser.md#options)
      * Notes about the display of Options
      * Options Diff
        * Overview
        * DB Options Diff
        * CF-s Options Diff - CF-s (Common) Sub-Object
        * CF-s Options Diff - CF-s (Specific) Sub-Object
      * All Options
    * [DB-Size Sub-Object](log-parser.md#speedbslogparsertool-db-sizesub-object)
  * [Flushes Object](log-parser.md#speedbslogparsertool-flushesobject)
  * [Compactions Object](log-parser.md#speedbslogparsertool-compactionsobject)
  * [Reads Object](log-parser.md#speedbslogparsertool-readsobject)
    * Get Histogram
    * Multi-Get Histogram
    * Per CF Read Latency
    * Filter Effectiveness
  * [Seeks Object](log-parser.md#speedbslogparsertool-seeksobject)
  * [Warnings Object](log-parser.md#speedbslogparsertool-warningsobject)
  * [Block-Cache-Stats Objects](log-parser.md#speedbslogparsertool-block-cache-statsobjects)
    * Top-level sub-objects
    * Caches top-level sub-object
    * DB Counters top-level sub-object
    * Detailed top-level sub-object
* [CSV Files](log-parser.md#speedbslogparsertool-csvfiles)
  * [counters.csv](log-parser.md#speedbslogparsertool-counters.csv)
  * [Counters Histograms CSV-s](log-parser.md#speedbslogparsertool-countershistogramscsv-s)
    * histograms\_human\_readable.csv
    * histograms\_tools.csv
  * compactions\_stats.csv
  * compactions.csv
  * flushes.csv
* [Testing](log-parser.md#testing)
* [Miscellaneous](log-parser.md#speedbslogparsertool-miscellaneous)
  * Reporting Bugs and Proposing
  * Known Bugs
  * Contributing to the tool’s development
  * Getting Advice or Discussing the Tool
  * The Future (the Features)

## Overview <a href="#speedbslogparsertool-overview" id="speedbslogparsertool-overview"></a>

Speedb's Log Parser is a tool that may be used to parse and process Speedb and RocksDB log files.

The tool extracts useful information from these logs and aids users in gaining insights about their systems with ease. It is expected to be a valuable tool for novices and experts alike.

The tool is written in the Python language and consists of a set of Python scripts.

It resides in a GitHub repository ([https://github.com/speedb-io/log-parser](https://github.com/speedb-io/log-parser)). There is a [README.md](http://readme.md) file in the root folder of the repository that describes how to install the tool, contribute to its development, etc.

## Terminology <a href="#speedbslogparsertool-terminology" id="speedbslogparsertool-terminology"></a>

The following terms and abbreviations are used throughout this document:

·       CF: Column-Family

·       DB / DB-Wide: Applicable to the entire DB rather than to a specific CF.

## Major Capabilities <a href="#speedbslogparsertool-majorcapabilities" id="speedbslogparsertool-majorcapabilities"></a>

* Parses a single Speedb or RocksDB log file (parsing multiple logs in in the **road-map**)
* &#x20;Parses the log and processes the information about the following elements and entities (detailed description may be found in the sections below):
  * Metadata information about the instance that has generated this log file (e.g., library version generating the log) (General Section).
  * Speedb / RocksDB Options (Options Section):
    * All of the options with their values (db-wide and per column family).
    * Displays the difference between the options in the log and an applicable baseline.
  * &#x20;Data about the size of the DB (DB-Size Section).
  * Flushes and Compactions (Flushes and Compactions sections respectively)
  * Information regarding DB read operations (Reads Section).
  * Information regarding seek DB operations (Seeks Section).
  * Warnings issued by the DB (Warnings Section).
  * Block cache statistics (Block-Cache-Stats Section)
  * Various Statistics: Counters, histograms, compaction stats, etc.
* The tool generates multiple outputs. Details may be found in the sections that follow. The outputs include:
  * A short console output (the default output format)
  * A JSON file with detailed information.
  * CSV files containing information about the counters, counters histograms, compactions, and flushes statistics.
  * &#x20;A detailed console output (the JSON file printed to the console).

## Usage <a href="#speedbslogparsertool-usage" id="speedbslogparsertool-usage"></a>

```
udi@udi-speedb:~/log-parser$ python3 log_parser.py -h
usage: log_parser.py [-h] [-c {short,long}] [-j] [-o OUTPUT_FOLDER] [-l] log-file-path

positional arguments:
  log-file-path         A path to a log file to parse (default: None)

optional arguments:
  -h, --help            show this help message and exit
  -c {short,long}, --console {short,long}
                        Print to console a summary (short) or a detailed (long) output (default: None)
  -j, --generate-json   Optionally generate a JSON file in the output folder's run-folder, with detailed information. If generated, it will be called log.json.(default: False)
  -o OUTPUT_FOLDER, --output-folder OUTPUT_FOLDER
                        The name of the folder where output files will be stored in SUB-FOLDERS named run_dddd, where 'dddd' is the run number (default: output_files)
  -l, --generate-log    Generate a log file for the parser's log messages (default: False)

Notes:
- The default it to print to the console in a short format.
- It is possible to specify both json and console outputs. Both will be generated.

```

## Installation / Getting Started / Prerequisites <a href="#speedbslogparsertool-installation-gettingstarted-prerequisites" id="speedbslogparsertool-installation-gettingstarted-prerequisites"></a>

Please see the [README.md](http://readme.md) file in the tool’s repository for this information.

## Tool’s Outputs Description <a href="#speedbslogparsertool-toolsoutputsdescription" id="speedbslogparsertool-toolsoutputsdescription"></a>

### Abbreviations, Conventions and Terms used in the log parser’s outputs <a href="#speedbslogparsertool-abbreviations-conventionsandtermsusedinthelogparsersoutputs" id="speedbslogparsertool-abbreviations-conventionsandtermsusedinthelogparsersoutputs"></a>

·       **tool:** The log parser

·       **Log file / log: A RocksDB / Speedb information log. Not the WAL (Write-Ahead-Log)**

·       **Parsed Log: The log file given to the log parser for parsing.**

·       **db:** Database

·       **db options**: Options that are not specific to any cf. Applicable to the entire db.

·       **cf options**: Options that are specific to an individual cf.

·       **cf (cf-s)**: Column Family (Column Families)

·       **Units:**

o   **Size Units**: B (Bytes), KB (Kilobytes), MB (Megabytes), TB (Terabytes)

o   **Numbers Units**: K (Thousands), M (Millions), G (Billions).

### Timestamps <a href="#speedbslogparsertool-timestamps" id="speedbslogparsertool-timestamps"></a>

**Log timestamps** (e.g., ‘2023/01/04-08:54:59.130735’) **are in local time**. The resolution is microseconds. All of the information displayed by the tool is using the timestamps from the parsed log as is.

### Availability of Data <a href="#speedbslogparsertool-availabilityofdata" id="speedbslogparsertool-availabilityofdata"></a>

#### Log level <a href="#speedbslogparsertool-loglevel" id="speedbslogparsertool-loglevel"></a>

The `info_log_level` db option controls the minimal level of issued log traces that will actually be written to the log (`INFO_LEVEL` by default in production library builds). Using a higher logging level will result in a log file that has almost no useful information. In that case, the tool (as well as the log file itself) will be of little use in practice.

#### Availability of the information to parse and process <a href="#speedbslogparsertool-availabilityoftheinformationtoparseandprocess" id="speedbslogparsertool-availabilityoftheinformationtoparseandprocess"></a>

The tool relies solely on the information contained in the parsed log file. Consequently, the information it displays reflects that. For example, the tool can’t display the average size of a key or a value in the entire DB, or the total number of keys in the DB, as this information is not printed to the log file.

Other common cases in which data may not be available:

·       Not having statistics (a configuration options)

·       Rolled logs (see “Log Rolling” below)

·       The number of cf-s (see “Number of Column Families and its implications” section below)

·       Lack of applicable activity. For example, no flush or compaction for a cf.

When data is not available, the output of the log parser will reflect that. For example:

·       “Filter Data Not Available”: When there is no information about the filter policy that was configured for a cf.

·       "Data Unavailable (No Stats Available)": When the DB was configured not to use statistics.

In the sections that follow, where applicable, there is a description of the information elements in the log file that were used to generate the associated output. For example:

·       Flush events and associated flush log traces

·       DB Statistics Dump

·       Counters and Histogram dumps

### Options <a href="#speedbslogparsertool-options" id="speedbslogparsertool-options"></a>

#### DB Options and CF Options <a href="#speedbslogparsertool-dboptionsandcfoptions" id="speedbslogparsertool-dboptionsandcfoptions"></a>

There are options that may be configured individually for every cf. These are called **cf options**. Options that are not specific to a cf are called **DB (or db-wide) options**. DB Options apply to the entire DB.

#### Defaults <a href="#speedbslogparsertool-defaults" id="speedbslogparsertool-defaults"></a>

Speedb and RocksDB come with defaults for every option. A user may override any of these options when opening a db. A user may also override any of the cf options when creating a new cf explicitly.

#### Writing the options to the log <a href="#speedbslogparsertool-writingtheoptionstothelog" id="speedbslogparsertool-writingtheoptionstothelog"></a>

The options are written to the log in multiple cases:

·       When a db is opened, the db and cf options are written to the log. This may occur when opening a new DB, or when recovering an existing db (with its existing cf-s and potentially newly created cf-s) from persistent storage.

·       When a new cf is created, its options are written to the log.

·       When a log is rolled, the db and cf options are written to the new log.

However, please see below for important information regarding applicable limitations.

#### Log Rolling <a href="#speedbslogparsertool-logrolling" id="speedbslogparsertool-logrolling"></a>

Log rolling (also called log rotation) is the process used to stop writing to an active log file, renaming it, and opening a new log file to which logging will be directed until the time comes to roll/rotate the log again. There are a few options that control this mechanism. In this document, log files that were rolled, are called **rolled logs**.

At the time of this writing, the defaults are to use a single log file (`max_log_file_size` option), effectively avoiding log rolling altogether. Some more information may be found [here](https://github.com/facebook/rocksdb/wiki/Logger).

#### Opening an existing DB (DB Recovery) <a href="#speedbslogparsertool-openinganexistingdb-dbrecovery" id="speedbslogparsertool-openinganexistingdb-dbrecovery"></a>

When the DB is opened as part of db recovery (opening an existing db), a new log file is created. This is unrelated to log rolling described above, but results in multiple log files nevertheless.

#### Number of Column Families and its implications <a href="#speedbslogparsertool-numberofcolumnfamiliesanditsimplications" id="speedbslogparsertool-numberofcolumnfamiliesanditsimplications"></a>

A user may create any number of column families (cf-s).

However, while opening a DB, **only the options of the first 10 (hard coded) cf-s will be printed to the log**.

The log will contain the following information for the 11th cf onwards:

{% code fullWidth="true" %}
```
2023/06/06-12:56:30.438743 322453 [/db_impl/db_impl.cc:3317] Created column family [column_family_name_000009] (ID 9)
2023/06/06-12:56:30.806215 322453 [/column_family.cc:625] 	(skipping printing options)
2023/06/06-12:56:30.806365 322453 [/db_impl/db_impl.cc:3317] Created column family [column_family_name_000010] (ID 10)
2023/06/06-12:56:31.191732 322453 [/column_family.cc:625] 	(skipping printing options)
2023/06/06-12:56:31.191836 322453 [/db_impl/db_impl.cc:3317] Created column family [column_family_name_000011] (ID 11)
2023/06/06-12:56:31.635102 322453 [/column_family.cc:625] 	(skipping printing options)
2023/06/06-12:56:31.635291 322453 [/db_impl/db_impl.cc:3317] Created column family [column_family_name_000012] (ID 12)
2023/06/06-12:56:32.053358 322453 [/column_family.cc:625] 	(skipping printing options)
2023/06/06-12:56:32.053527 322453 [/db_impl/db_impl.cc:3317] Created column family [column_family_name_000013] (ID 13)
2023/06/06-12:56:32.508987 322453 [/column_family.cc:625] 	(skipping printing options)
2023/06/06-12:56:32.509108 322453 [/db_impl/db_impl.cc:3317] Created column family [column_family_name_000014] (ID 14)

```
{% endcode %}

**So, we know that there are additional cf-s, and we know their names and id-s, but we do not know their options.**

#### Unavailability of CF Names in Rolled Logs (Auto-Generated CF Names) <a href="#speedbslogparsertool-unavailabilityofcfnamesinrolledlogs-auto-generatedcfnames" id="speedbslogparsertool-unavailabilityofcfnamesinrolledlogs-auto-generatedcfnames"></a>

When a new DB is opened, or when a cf is created explicitly (via the `CreateColumnFamily` or `CreateColumnFamilies` APIs) , this is what the log contains (Only the first lines are shows):

{% code fullWidth="true" %}
```
2023/06/06-12:56:28.376394 322453 [/column_family.cc:620] --------------- Options for column family [default]:
2023/06/06-12:56:28.376399 322453               Options.comparator: leveldb.BytewiseComparator
2023/06/06-12:56:28.376400 322453           Options.merge_operator: 0x7fdfd8068640
2023/06/06-12:56:28.376402 322453        Options.compaction_filter: None
2023/06/06-12:56:28.376403 322453        Options.compaction_filter_factory: None

```
{% endcode %}

**The first line contains the name of the cf (“default” in this case)**

However, when a new log file is created as part of log rolling, **this first line is not printed to the log** => the tool has no (simple and safe) way of knowing to which cf the options belong. As the “default” cf always exists and is the first one, the tool assumes the first to be “default”, but not for the rest.

**In addition, there is no information about the skipped cf-s (those that are > 10, see above).**

The rest of the log contains traces that may contain the names of the cf-s, but that is of no use in the association of options with their cf-s.

The log parser handles this by **auto-generating cf names** for the cf-s whose names are unknown, but for which there are options.

**The auto generated names have the following format: UNKNOWN-CF-#\<Number>**

The following snapshot shows an example:

![](<../.gitbook/assets/Screen Shot 2023-06-21 at 14.33.00.png>)

As a consequence of what is described above, **when the tool parses a log that is created as part of log rolling and there are more than 10 cf-s, the tool doesn’t know how many cf-s there are**.

This will be indicated to the user by displaying the following information in the console short output, or the json’s General object:

{% code lineNumbers="true" fullWidth="true" %}
```python
Num CF-s (*)             : Can't be accurately determined
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------
|       Column Family       |      Size        | Avg. Key Size | Avg. Value Size |    Compaction Style   | Compression |                Filter-Policy                 |
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------
|          default          | Data Unavailable |      24 B     |      100 B      | kCompactionStyleLevel |    Snappy   | rocksdb.internal.FastLocalBloomFilter (24.0) |
| column_family_name_000001 | Data Unavailable |      24 B     |      100 B      |        UNKNOWN        |   UNKNOWN   | rocksdb.internal.FastLocalBloomFilter (24.0) |
| column_family_name_000002 | Data Unavailable |      24 B     |      100 B      |        UNKNOWN        |   UNKNOWN   | rocksdb.internal.FastLocalBloomFilter (24.0) |
| column_family_name_000003 | Data Unavailable |      24 B     |      100 B      |        UNKNOWN        |   UNKNOWN   | rocksdb.internal.FastLocalBloomFilter (24.0) |
| column_family_name_000004 | Data Unavailable |      24 B     |      100 B      |        UNKNOWN        |   UNKNOWN   | rocksdb.internal.FastLocalBloomFilter (24.0) |
| column_family_name_000005 | Data Unavailable |      24 B     |      100 B      |        UNKNOWN        |   UNKNOWN   | rocksdb.internal.FastLocalBloomFilter (24.0) |
| column_family_name_000006 | Data Unavailable |      24 B     |      100 B      |        UNKNOWN        |   UNKNOWN   | rocksdb.internal.FastLocalBloomFilter (24.0) |
| column_family_name_000007 | Data Unavailable |      24 B     |      100 B      |        UNKNOWN        |   UNKNOWN   | rocksdb.internal.FastLocalBloomFilter (24.0) |
| column_family_name_000008 | Data Unavailable |      24 B     |      100 B      |        UNKNOWN        |   UNKNOWN   | rocksdb.internal.FastLocalBloomFilter (24.0) |
| column_family_name_000009 | Data Unavailable |      24 B     |      100 B      |        UNKNOWN        |   UNKNOWN   | rocksdb.internal.FastLocalBloomFilter (24.0) |
| column_family_name_000010 | Data Unavailable |      24 B     |      100 B      |        UNKNOWN        |   UNKNOWN   | rocksdb.internal.FastLocalBloomFilter (24.0) |
| column_family_name_000012 | Data Unavailable |      24 B     |      100 B      |        UNKNOWN        |   UNKNOWN   | rocksdb.internal.FastLocalBloomFilter (24.0) |
| column_family_name_000013 | Data Unavailable |      24 B     |      100 B      |        UNKNOWN        |   UNKNOWN   | rocksdb.internal.FastLocalBloomFilter (24.0) |
| column_family_name_000014 | Data Unavailable |      24 B     |      100 B      |        UNKNOWN        |   UNKNOWN   | rocksdb.internal.FastLocalBloomFilter (24.0) |
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------

(*) Please see the 'Ability to determine the number of cf-s' section in the log parser's documentation for more information
```
{% endcode %}

<figure><img src="../.gitbook/assets/Screen Shot 2023-06-22 at 9.22.32.png" alt=""><figcaption></figcaption></figure>

### The Output Folder

Every run of the log parser generates multiple output files. All of the files generated in a single run are placed under a single folder. The folder’s location and name are determined as follows:

·       The user may specify a parent folder via the ‘-o’ command line parameter.

·       If the user doesn’t specify an output parent folder, ‘output\_files’ will be used by default.

·       If the parent folder doesn’t exist, it will be created.

·       Under the parent folder, the parser will create a folder named ‘run\_dddd’ and place the run’s output files under that folder. ‘dddd’ are 4 digits that compose the run’s number. They are incremented every run and wrap around if reaching ‘9999’.

·       If the parent folder already contains ‘run\_dddd’ folders, the tool will detect the largest one (converting ‘dddd’ to its numeric equivalent - N), create a new folder ‘run\_mmmm’ where int('mmmm') = N+1

An example to put it all together:

The user specified ‘-o Output' as an argument when running the tool. There is no folder named ‘Output’ => The tool will create a new folder named ‘Output’, and the output files will be under ‘Output/run\_0001/’

The user re-runs the tool, again specifying ‘-o Output’:

The run’s output files will be under ‘Output/run\_0002’

### Console Output <a href="#speedbslogparsertool-consoleoutput" id="speedbslogparsertool-consoleoutput"></a>

There are 2 console output flavors:

1\.     **Short**: A concise summary of the major information elements. May be used to get a basic understanding of the db’s operation according to the parsed log.

2\.     **Detailed**: The contents of the JSON file, printed on the console (please see the JSON file description section for more details). This option is available for power users that wish to use only the console. To be of use, these users will probably use JSON command-line filtering tools such as [JQ](https://stedolan.github.io/jq/).

### Console Short Output Description <a href="#speedbslogparsertool-consoleshortoutputdescription" id="speedbslogparsertool-consoleshortoutputdescription"></a>

#### Example Output <a href="#speedbslogparsertool-exampleoutput" id="speedbslogparsertool-exampleoutput"></a>

{% code lineNumbers="true" fullWidth="true" %}
```
udi@udi-speedb:~/log-parser$ python3 log_parser.py  test/input_files/LOG_speedb 
Log file: file:///home/udi/log-parser/test/input_files/LOG_speedb
Baseline Log: file:///home/udi/log-parser/baseline_logs/LOG-speedb-2.3.0
Counters CSV Is in file:///home/udi/log-parser/output_files/run_0016/counters.csv
Human Readable Counters Histograms CSV Is in file:///home/udi/log-parser/output_files/run_0016/histograms_human_readable.csv
Compactions Stats CSV Is in file:///home/udi/log-parser/output_files/run_0016/compactions_stats.csv
Compactions CSV Is in file:///home/udi/log-parser/output_files/run_0016/compactions.csv
Flushes CSV Is in file:///home/udi/log-parser/output_files/run_0016/flushes.csv

Parsing of: /home/udi/log-parser/test/input_files/LOG_speedb
============================================================
Name                     : /home/udi/log-parser/test/input_files/LOG_speedb
Start Time               : 2023/04/09-12:27:35.901086
End Time                 : 2023/04/09-12:37:36.500378
Log Time Span            : 0d 00h 10m 00s
Creator                  : Speedb
Version                  : 2.4.0 [8ef7469a7f8d1d100a7a187b3c682a48777b7047]
DB Size (*)              : 111.6 MB
Num Keys Written         : 48.0 M
Avg. Written Key Size    : 24 B
Avg. Written Value Size  : 64 B
Num Warnings             : 0
Error Messages           : No Error Messages
Fatal Messages           : No Fatal Messages
Ingest (*)               : 3.9 GB
Ingest Rate              : 6.76 MBps
Statistics               : Available
Writes                   : 10.0% (48033559/480337639)
Reads                    : 90.0% (432304080/480337639)
Seeks                    : 0 (No Seek Operations)
Deleted (Flushed) Entries: 0 (No Delete Operations)
Num CF-s (**)            : 3
---------------------------------------------------------------------------------------------------------------------------------------
|       Column Family       | Size (*) | Avg. Key Size | Avg. Value Size |    Compaction Style   |  Compression  |   Filter-Policy    |
---------------------------------------------------------------------------------------------------------------------------------------
|          default          | 40.8 MB  |      24 B     |       64 B      | kCompactionStyleLevel | NoCompression | bloomfilter (10.0) |
| column_family_name_000001 | 38.3 MB  |      24 B     |       64 B      | kCompactionStyleLevel | NoCompression | bloomfilter (10.0) |
| column_family_name_000002 | 32.6 MB  |      24 B     |       64 B      | kCompactionStyleLevel | NoCompression | bloomfilter (10.0) |
---------------------------------------------------------------------------------------------------------------------------------------

(*) Data is calculated at: 2023/04/09-12:37:27.398344
(**) Please see the log parser's documentation for more information
```
{% endcode %}

#### Output Fields Description <a href="#speedbslogparsertool-outputfieldsdescription" id="speedbslogparsertool-outputfieldsdescription"></a>

<table data-header-hidden data-full-width="true"><thead><tr><th width="79"></th><th></th><th></th><th></th></tr></thead><tbody><tr><td></td><td><strong>Field Name</strong></td><td><strong>Meaning</strong></td><td><strong>Comments</strong></td></tr><tr><td>1</td><td>Title (“Parsing of:…”)</td><td>Parsed Log full path</td><td></td></tr><tr><td>2</td><td>Name</td><td>Parsed Log full path</td><td></td></tr><tr><td>3</td><td>Start Time</td><td>Time of first log entry</td><td></td></tr><tr><td>4</td><td>End Time</td><td>Time of last log entry</td><td></td></tr><tr><td>5</td><td>Log Time Span</td><td>The time difference between the Start Time and the End Time</td><td>The value is expressed in number of days, hours, minutes, and seconds.</td></tr><tr><td>6</td><td>Creator</td><td>The creator of the library that generated the parsed log</td><td>Currently may be either Speedb or RocksDB</td></tr><tr><td>7</td><td>Version</td><td>The library’s version [Git Hash]</td><td></td></tr><tr><td>8</td><td>DB Size (*)</td><td>The total size of all of the SST files in the database</td><td>The point in time in which the value was calculated is given below the per-cf table: “(*) Data is calculated ….”</td></tr><tr><td>9</td><td>Num Keys Written</td><td>The total number of keys written to the DB</td><td><p>The value of the <code>rocksdb.number.keys.written</code> counter if available,</p><p>otherwise, extracted from cumulative writes information (DB Stats)</p><p>If none of the above is available: “Data Unavailable”</p></td></tr><tr><td>10</td><td>Avg. Written Key Size</td><td>The average size of a written key</td><td><p>Calculated from information in <code>table_file_creation</code> events</p><p>If there are no such events, “Data Unavailable”</p></td></tr><tr><td>11</td><td>Avg. Written Value Size</td><td>The average size of a written value</td><td><p>Calculated from information in <code>table_file_creation</code> events</p><p>If there are no such events, “Data Unavailable”</p></td></tr><tr><td>12</td><td>Error Messages</td><td>The messages in severity “ERROR” as they appear in the log</td><td>“No Error Messages” if there are no errors in the log</td></tr><tr><td>13</td><td>Fatal Messages</td><td>The messages in severity “FATAL” as they appear in the log</td><td>“No Fatal Messages” if there are no errors in the log</td></tr><tr><td>14</td><td>Ingest (*)</td><td>Total ingest</td><td><p>Calculated from cumulative writes information.</p><p>The point in time in which the value was calculated is given below the per-cf table: “(*) Data is calculated ….”</p><p>If data is not available: “No Ingest Info Available”</p></td></tr><tr><td>15</td><td>Ingest Rate</td><td>Ingest Rate in Mega-Bytes per second</td><td><p>Calculated from cumulative writes information.</p><p>The point in time in which the value was calculated is given below the per-cf table: “(*) Data is calculated ….”</p><p>If data is not available: “No Ingest Info Available”</p></td></tr><tr><td>16</td><td>Statistics</td><td>Whether statistics are available or not</td><td></td></tr><tr><td>17</td><td>Writes</td><td><p>The total number of <strong>write</strong> operations</p><p>&#x3C;Percentage> (&#x3C;Count> / &#x3C;Total Ops>)</p><p><strong>Total Ops</strong>: Total number of operations (write + read + seek)</p><p><strong>Count</strong>: Total number of writes</p><p><strong>Percentage</strong>: Percentage of writes out of Total Ops</p></td><td><p>Calculated from the <code>rocksdb.number.keys.written</code>, <code>rocksdb.number.keys.read</code>, and <code>rocksdb.number.db.seek</code> counters.</p><p>If statistics are not available: “Data Unavailable (No Statistics)”</p></td></tr><tr><td>18</td><td>Reads</td><td><p>The total number of <strong>read</strong> operations</p><p>(Same as for the Writes field, but for reads)</p></td><td>Same as for Writes</td></tr><tr><td>19</td><td>Seeks</td><td><p>The total number of <strong>seek</strong> operations</p><p>(Same as for the Writes field, but for seeks)</p></td><td>Same as for Writes</td></tr><tr><td>20</td><td>Deletes</td><td><p>&#x3C;Percentage> (&#x3C;Num Deletes> / &#x3C;Num Entries>)</p><p><strong>Num Entries</strong>: The total number of flushed entries</p><p><strong>Num Deletes</strong>: The total number of deletes in</p><p><strong>Percentage</strong>: Percentage of deletes out of Num Entries</p></td><td><p>Information is gathered from Flush events in the log.</p><p>If no such events exist: “Data Unavailable (No Flush Started Events)”</p></td></tr><tr><td>21</td><td>Num CF-s</td><td><p>Number of cf-s or</p><p>“Can't be accurately determined”</p></td><td>The log parser may not be able to know for sure the number of cf-s in the DB using the parsed log file. In that case it will show the text “Can't be accurately determined”. Please see the “Ability to determine the number of cf-s“ for more information.</td></tr><tr><td>22</td><td></td><td></td><td></td></tr><tr><td>23</td><td><strong>Column Families Information Table</strong></td><td></td><td></td></tr><tr><td>24</td><td>Column Family</td><td>The name of the CF</td><td>Will display any CF for which there was information in the log (TBD - See XXXX)</td></tr><tr><td>25</td><td>Size</td><td>The total size of all of the SST files of the CF</td><td>If no data available to calculate the size: “Data Unavailable”</td></tr><tr><td>26</td><td>Avg. Key Size</td><td>Average value of a key in newly created SST-s of the CF</td><td>Gathered from table_file_creation events in the log</td></tr><tr><td>27</td><td>Avg. Value Size</td><td>Average value of a value in newly created SST-s of the CF</td><td>Gathered from table_file_creation events in the log</td></tr><tr><td>28</td><td>Compaction Style</td><td>The compaction style used in this CF</td><td><p>Taken from the options for the CF. The values are as they appear in the log.</p><p>If not known: “UNKNOWN”</p></td></tr><tr><td>29</td><td>Compression</td><td>The compression type used in this CF</td><td><p>Taken from the options for the CF. The values are as they appear in the log.</p><p>If not known: “UNKNOWN”</p></td></tr><tr><td>30</td><td>Filter-Policy (&#x3C;BPK>)</td><td><p>The filter policy used in this CF (If any)</p><p>&#x3C;BPK> - The average BPK for the filter.</p></td><td><p>Taken from the options for the CF. The values are as they appear in the log.</p><p>If not known: “UNKNOWN”</p></td></tr></tbody></table>

## JSON File <a href="#speedbslogparsertool-jsonfile" id="speedbslogparsertool-jsonfile"></a>

Upon the user’s request, a JSON file is generated in the output folder.

The JSON file is a text file. It is best viewed by a JSON viewer. The following screen shots were taken from the Firefox browser that has a built-in JSON viewer.

JSON is a hierarchical format. The following sections describe the contents of the JSON file accordingly.

### Top Level JSON Objects <a href="#speedbslogparsertool-topleveljsonobjects" id="speedbslogparsertool-topleveljsonobjects"></a>

The JSON file contains the following top-level objects (please see the following sections for details on every object):

<figure><img src="../.gitbook/assets/Screen Shot 2023-06-22 at 9.25.56.png" alt=""><figcaption></figcaption></figure>

* &#x20;General: The same information that is displayed in the short console output.
* &#x20;  Options:
  * The differences between the options in the log and an applicable baseline version (if available).
  * All of the DB-Wide and per-cf options in the log
* &#x20;DB-Size: Summary information of ingest data and per-cf and level size growth.
* &#x20; Flushes: Per CF flush-related information..
* &#x20;Compactions: Per CF compactions-related information.
* &#x20;Reads: Get / Multi-Get operations related information
* &#x20;Seeks: Seek operations related information
* &#x20;Warnings: Warnings statistics.
* &#x20;Block-Cache-Stats: Statistics about the use of the block-cache.
* &#x20;CSV-s: Paths of generated CSV-s.

### JSON Top-Level Objects Detailed Description <a href="#speedbslogparsertool-jsontop-levelobjectsdetaileddescription" id="speedbslogparsertool-jsontop-levelobjectsdetaileddescription"></a>

#### General <a href="#speedbslogparsertool-general" id="speedbslogparsertool-general"></a>

This includes the same information that is displayed in the short console output. Please see that section for more details.

<figure><img src="../.gitbook/assets/Screenshot from 2023-06-14 16-34-03 (1).png" alt=""><figcaption></figcaption></figure>

#### Options

The options object consists of 2 sub-objects:

1\.     Diff

2\.     All Options

<figure><img src="../.gitbook/assets/Screen Shot 2023-06-22 at 9.27.50.png" alt=""><figcaption></figcaption></figure>

**Notes about the display of Options**

·       **WBM Pseudo-Options**: The DB-Wide options contain pseudo-options for the Write Buffer Manager. They are not true Speedb / Rocksdb options. They are configuration parameters given to the WBM during its construction. They are displayed in the log together with the “official” options as follows:

{% code lineNumbers="true" fullWidth="true" %}
```
2023/04/09-12:27:35.901291 30528                    Options.db_write_buffer_size: 0
2023/04/09-12:27:35.901299 30528                    Options.write_buffer_manager: 0x55ded4834000
                                                                             wbm.size: 107374182
                                                                            wbm.cache: 0x55deca6980f0
                                                                     wbm.allow_stalls: 1
                                                                 wbm.initiate_flushes: 1
2023/04/09-12:27:35.901304 30528         Options.access_hint_on_compaction_start: 1
```
{% endcode %}

And this is how they are displayed in the JSON (the ‘write\_buffer\_manager\_’ prefix was added):

<figure><img src="../.gitbook/assets/Screen Shot 2023-06-22 at 9.28.49.png" alt=""><figcaption></figcaption></figure>

·       **Block-Based-Table-Options (CF Options sub-object)**

CF Options consist of a set of top-level options. One of these options is the table\_factory. The tool assumes the table\_factory is of type `BlockBasedTableFactory` (**Block-Based-Table-Factory**). The options for this entity are displayed separately, under a sub-object called “**Block-Based Table**”.

This is an example from a log file that shows how these options are printed to the log:

{% code lineNumbers="true" fullWidth="true" %}
```
2023/04/09-12:27:35.901772 30528            Options.table_factory: BlockBasedTable
2023/04/09-12:27:35.901929 30528            table_factory options:   flush_block_policy_factory: FlushBlockBySizePolicyFactory (0x55deca6d9c60)
  cache_index_and_filter_blocks: 1
  cache_index_and_filter_blocks_with_high_priority: 1
  pin_l0_filter_and_index_blocks_in_cache: 0
  pin_top_level_index_and_filter: 1
  metadata_cache_options:
    top_level_index_pinning: 3
    partition_pinning: 0
    unpartitioned_pinning: 0
  index_type: 0
  data_block_index_type: 0
  index_shortening: 1
  data_block_hash_table_util_ratio: 0.750000
  checksum: 1
  no_block_cache: 0
  block_cache: 0x55deca6980f0
  block_cache_name: LRUCache
```
{% endcode %}

* **block\_cache\_options:** These are displayed in the log as follows:

{% code lineNumbers="true" fullWidth="true" %}
```
block_cache_name: LRUCache
block_cache_options:
  capacity : 2147483648
  num_shard_bits : 4
  strict_capacity_limit : 0
  memory_allocator : None
  high_pri_pool_ratio: 0.600
  low_pri_pool_ratio: 0.000
block_cache_compressed: (nil)
```
{% endcode %}

In the JSON, a ‘block\_cache\_’ prefix is used as follows:

<figure><img src="../.gitbook/assets/Screen Shot 2023-06-22 at 9.31.06.png" alt=""><figcaption></figcaption></figure>

·       **metadata\_cache\_options:** These are displayed in the log as follows:

{% code lineNumbers="true" %}
```
metadata_cache_options:
  top_level_index_pinning: 3
  partition_pinning: 0
  unpartitioned_pinning: 0
index_type: 0
```
{% endcode %}

In the JSON, a ‘metadata\_cache\_’ prefix is used as follows:

<figure><img src="../.gitbook/assets/Screen Shot 2023-06-22 at 9.31.54.png" alt=""><figcaption></figcaption></figure>

·       **Options that are pointers**

Some options are pointers (values that start with ‘0x’ and only contain hexadecimal digits). It is impossible to know from the log the real entity that the pointer points to. A pointer may or may not be initialized. Uninitialized pointers are displayed in the log in multiple ways (e.g., ;(nil)', ‘None’, etc.).

The value of an initialized pointer is meaningless in and of itself. Its value is the address of the associated entity in the address space of its containing process, and is unique within that process. Its only use, in the context of log files, is the ability to understand that the same entity is shared. For example, if the same block cache is shared between multiple cf-s, then all of them will have the same value for the block cache’s pointer option.

When the value of an initialized pointer is displayed in the JSON file, it will be displayed as “Pointer (\<pointer value>).

Uninitialized pointers will be displayed as “Pointer (Uninitialise)”.

**Options Diff**

**Overview**

<figure><img src="../.gitbook/assets/Screen Shot 2023-06-22 at 9.32.14.png" alt=""><figcaption></figcaption></figure>

This object contains the differences between the options in the log and the default options in an applicable baseline version. The baseline version is the closest available version of the applicable creator of the library (RocksDB or Speedb).

This object contains the following sub-objects:

1\.     Baseline: The version that is used as the baseline with its creator in parentheses.

2\.     Baseline Log: The full path to the baseline log file.

3\.     DB: The diff in the db options.

4\.     CF-s: Per CF options diff. This object consists of 2 sub-objects:

1\.     CF-s (Common): Contains the options that are identical in all of the cf-s in the parsed log but are different than the corresponding option in the baseline.

2\.     CF-s (Specific): Contains the options that are not identical in the parsed log, and the corresponding option in the baseline.

Please see the section “CF-s Options Diff - CF-s (Common) Sub-Object” below for more details.

**There are 3 cases with respect to a diff between the baseline and the parsed log:**

1\.     The option **exists** in the baseline but was removed in the version that generated the parsed log.

2\.     The option **doesn’t** **exist** in the baseline but was added to the version that generated the parsed log.

3\.     The option **exists in both**, but the values are different.

{% hint style="info" %}
**Notes**

·       The Options Diff object will only show options in which there is a difference.

·       Every entry in the diff contains 2 lines, the first for the baseline (called “Baseline”), and the second for the parsed log (called “Parsed Log”).

·       When an option doesn’t exist, “Missing” will be displayed.

·       If an option was **renamed** in a version, it will be deemed as a new option and it will appear twice, first for the old name (missing in parsed logs) and then for the new name (missing in the baseline).

·       Pointers handling:

o   The value of initialized pointers will always be different in the baseline and the parsed log. They will be deemed **equal** for the purposes of the comparison.

o   If both pointers are uninitialized, they will be deemed **equal**.

·       In all other cases, the values will be displayed as they appear in the log
{% endhint %}



**DB Options Diff**

This is an example of a DB options diff:

<figure><img src="../.gitbook/assets/Screenshot from 2023-06-15 05-24-53.png" alt=""><figcaption></figcaption></figure>

**CF-s Options Diff - CF-s (Common) Sub-Object**

The following two snapshots are an example of this sub-object:

<figure><img src="../.gitbook/assets/Screenshot from 2023-06-15 05-36-31.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Screenshot from 2023-06-15 05-34-47.png" alt=""><figcaption></figcaption></figure>

**Notes**:

·       As noted above, the Common sub-object contains options whose value is identical in all of the cf-s of the parsed log. So, for example, the memtable\_factory is "speedb.HashSpdRepFactory" in all the cf-s of the parsed log. Its value is “SkipListFactory” in the baseline.

·       It consists of two sub-objects:

o   CF: The cf options that are not part of the block-based-table-format options

o   Block-Based Table: The block-based-table-format options.

**CF-s Options Diff - CF-s (Specific) Sub-Object**

<figure><img src="../.gitbook/assets/Screenshot from 2023-06-15 05-45-31.png" alt=""><figcaption></figcaption></figure>

**Notes:**

* &#x20;As noted above, this sub-object only contains options that are not identical in all of the cf-s of the parsed log. Please note however that all such options are included in this sub-object, even if its value is the same as the corresponding option in the baseline.
* “**Unknown-CF-#\<I>**”: As explained in the “Ability to determine the number of cf-s” section, these represent cf-s whose name can’t be determined, that have options at the top of the parsed log.

**All Options**

This object lists all of the options that appear in the log file, in the same order:

<figure><img src="../.gitbook/assets/Screen Shot 2023-06-22 at 9.34.55.png" alt=""><figcaption></figcaption></figure>

As in the options diff sub-object, there are sub-objects for the db-wide options and the cf-s options:



<figure><img src="../.gitbook/assets/Screen Shot 2023-06-22 at 9.35.28.png" alt=""><figcaption></figcaption></figure>

The sub-objects themselves, contain additional sub-objects, using the same principles described above for the options diff sub-object:

<figure><img src="../.gitbook/assets/Screenshot from 2023-06-15 07-03-35.png" alt=""><figcaption></figcaption></figure>

The **CF-s (Common)** sub-object contains all of the options that are identical in all of the cf-s of the parsed log.

The **CF-s (Specific)** sub-object contains all of the options that are not identical in all of the cf-s of the parsed log. For example:

<figure><img src="../.gitbook/assets/Screenshot from 2023-06-15 07-06-13.png" alt=""><figcaption></figcaption></figure>

As may be seen in this example, the “default” cf has options that are not the same as the corresponding options in the “column\_family\_name\_000001” and the “column\_family\_name\_000002” cf-s.

#### DB-Size Sub-Object <a href="#speedbslogparsertool-db-sizesub-object" id="speedbslogparsertool-db-sizesub-object"></a>

<figure><img src="../.gitbook/assets/Screenshot from 2023-06-15 07-48-57.png" alt=""><figcaption></figcaption></figure>

This sub-object consists of the following:

* &#x20;Ingest: Ingest information taken from the last cumulative writes log trace:
  * Ingest:
  * Ingest Rate:
  * Ingest Time:
* &#x20;CF-s Growth: Per CF and level report on the difference in the size from the start of the log to its end. The information is obtained from compaction stats dumps.
  * Per CF and level, the information is displayed as \<Start Size> **->** \<End Size> (\<Difference>)
  * The “Sum” entry shows the total for the CF (sum of all levels of the CF).

#### Flushes Object <a href="#speedbslogparsertool-flushesobject" id="speedbslogparsertool-flushesobject"></a>

<figure><img src="../.gitbook/assets/Screenshot from 2023-06-15 14-00-48.png" alt=""><figcaption></figcaption></figure>

This Object displays per-cf information about flushes.

Per CF, there is information **for** **all of the flushes that occurred in the CF**.

The following information is displayed, per CF:

| **Name**                           | **Meaning**                                                                                                                                                                                                                                                                                     | **Source**       |
| ---------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------- |
| L0->L1 Write-Amp                   | The write-amplification for Level0 → Level1 for this CF.                                                                                                                                                                                                                                        | Compaction Stats |
|                                    |                                                                                                                                                                                                                                                                                                 |                  |
| **Per \<Flush Reason> Sub-Object** | <p>Sub-object per flush reason in the flushes for this CF</p><p><strong>All of the fields below are in the context of a CF and flush reason</strong>.</p>                                                                                                                                       | Flush Events     |
| Sizes Histogram                    | <p>A histogram of the number of flushes per total data size range (bucket) in a flush.</p><p>The numbers are the number of flushes of all flushes for this cf and flush reason.</p><p>For example, in the snapshot above, there were 31 flushes whose total data size was more than 31 MB).</p> |                  |
| Num Flushes                        | The number of flushes                                                                                                                                                                                                                                                                           |                  |
| Min Duration                       | The minimum duration that a single flush took to complete                                                                                                                                                                                                                                       |                  |
| Max Duration                       | The maximum duration that a single flush took to complete                                                                                                                                                                                                                                       |                  |
| Min Num Memtables                  | The minimum number of memtables that were part of a single flush.                                                                                                                                                                                                                               |                  |
| Max Num Memtables                  | The maximum number of memtables that were part of a single flush.                                                                                                                                                                                                                               |                  |
| Min Total Data Size                | The minimum total data size in a single flush.                                                                                                                                                                                                                                                  |                  |
| Max Total Data Size                | The maximum total data size in a single flush.                                                                                                                                                                                                                                                  |                  |

#### Compactions Object <a href="#speedbslogparsertool-compactionsobject" id="speedbslogparsertool-compactionsobject"></a>

<figure><img src="../.gitbook/assets/Screen Shot 2023-06-22 at 9.38.59.png" alt=""><figcaption></figcaption></figure>

This Object displays per-cf information about compactions.

**All of the information in this sub-objects is based on traces of compaction jobs (events and associatedlog traces), and compaction level stats dumps.**

The sub-object consists of the following:

·       Largest compaction size of all compactions in the log.

·       Per CF compactions information (see below).

Per CF Information

The per-cf compactions information is based on **all of the compactions that occurred in the CF**.

The following information is displayed, per CF:

| **Name**                           | **Meaning**                                                                                                                                               |
| ---------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Per \<Flush Reason> Sub-Object** | <p>Sub-object per flush reason in the flushes for this CF</p><p><strong>All of the fields below are in the context of a CF and flush reason</strong>.</p> |
| Num Compactions                    | The number of compactions                                                                                                                                 |
| Min Compactions BW                 | Minimum write rate of a compaction                                                                                                                        |
| Max Compactions BW                 | Maximum write rate of a compaction                                                                                                                        |
| Comp                               | The elapsed time of all compactions                                                                                                                       |
| Comp Merge CPU                     | The elapsed CPU time of all compactions                                                                                                                   |
| Per Level Write-Amp                | Write amplication per level, and their total (SUM)                                                                                                        |

#### Reads Object <a href="#speedbslogparsertool-readsobject" id="speedbslogparsertool-readsobject"></a>

This object contains information about read operations performed by the user (Get and Multi-Get) and associated aspects.

<figure><img src="../.gitbook/assets/Screen Shot 2023-06-22 at 9.39.24.png" alt=""><figcaption></figcaption></figure>

**Get Histogram**

<figure><img src="../.gitbook/assets/Screen Shot 2023-06-22 at 9.39.42.png" alt=""><figcaption></figcaption></figure>

This is the last dump of the `rocksdb.db.get.micros` histogram.

It will be available only when statistics are enabled

**Multi-Get Histogram**

<figure><img src="../.gitbook/assets/Screen Shot 2023-06-22 at 9.40.01.png" alt=""><figcaption></figcaption></figure>

This is the last dump of the `rocksdb.db.multiget.micros` histogram.

It will be available only when statistics are enabled

**Per CF Read Latency**

<figure><img src="../.gitbook/assets/Screen Shot 2023-06-22 at 9.40.19.png" alt=""><figcaption></figcaption></figure>

Per CF, displays information about read performance across all of the CF’s levels. The information is obtained from “File Read Latency Histogram By Level” dumps:

·       Num Reads: Total number of reads.

·       Avg. Read Latency: The average read latency

·       Max Read Latency: The maximum latency

·       Read % of All CF-s: The percentage of reads performed on this CF relative to the total number of reads on all CF-s (the snapshot shows a single CF so the percentage is 100%).

**Filter Effectiveness**

<figure><img src="../.gitbook/assets/Screen Shot 2023-06-22 at 9.40.35.png" alt=""><figcaption></figcaption></figure>

Consists of:

* CF-s: Per CF:
  * Filter-Policy: The type of filter used (if any).
  * Avg. BPK: The average effective BPK for that filter
* Counters: Global filters counters for all filters (available only if statistics are available):
  * False-Positive Rate: The effective false positive rate of all of the filters. This is displayed as a “1 in N” as it is the convention.
  * False-Positives: The number of times the filters answered “Key May Exist”, but the key wasn’t actually found.
  * Negatives: The number of times the filters answered “Key Doesn’t Exist”
  * True-Positives: The number of times the filters answered “Key May Exist” and the key was actually found.

#### Seeks Object <a href="#speedbslogparsertool-seeksobject" id="speedbslogparsertool-seeksobject"></a>

<figure><img src="../.gitbook/assets/Screen Shot 2023-06-22 at 9.41.41.png" alt=""><figcaption></figcaption></figure>

This Object shows information about user seek operations. The information is gathered from counters so it will be available only when statistics are enabled.

The object consists of the following:

·       Num Seeks: The number of seek operations performed by the user \[`rocksdb.number.db.seek`counter]

·       Num Found Seeks: The number of calls to seek (`SeekToLast`, `Seek`, `SeekForPrev`, and `SeekToFirst`) that succeeded (iterator was placed on a valid value) \[`rocksdb.number.db.seek.found`counter]

·       Num Nexts: The number of calls to `Next` on the iterator \[`rocksdb.number.db.next`counter]

·       Num Prevs: The number of calls to `Prev` on the iterator \[`rocksdb.number.db.prev`counter]

·       Avg. Seek Range Size: (Num Nexts + Num Prevs) / Num Seeks

·       Avg. Seeks Rate Per Second: The average number of seek operation per second during the log’s time span.

·       Avg. Seek Latency: The average seek latency taken from the `rocksdb.db.seek.micros` histogram.

#### Warnings Object <a href="#speedbslogparsertool-warningsobject" id="speedbslogparsertool-warningsobject"></a>

<figure><img src="../.gitbook/assets/Screenshot from 2023-06-15 15-52-19.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Screenshot from 2023-06-15 15-55-59.png" alt=""><figcaption></figcaption></figure>

This object contains minimal summary information about the warnings found in the log.

The object consists of the following:

* DB: Warnings that are not associated with a specific cf. This will be displayed the number of such warnings. Currently all of the DB warnings are not categorized (“Other”).
* CF-s: Warning that are associated with a CF. Per CF:
  * A histogram of the number of warnings per category of warnings. Existing categories include:
    * Write-Stop: The database warns about a write stop
    * Write-Delay: The data base warns about a write delay
    * Other: Any warning not of one of the categories listed above.

#### Block-Cache-Stats Objects <a href="#speedbslogparsertool-block-cache-statsobjects" id="speedbslogparsertool-block-cache-statsobjects"></a>

This object contains block cache information and associated statistics found in the log (if available).

If you are unfamiliar with RocksDB’s block cache, please read [RocksDB Block Cache](https://github.com/facebook/rocksdb/wiki/Block-Cache).

Please note the following:

·       The block cache is optional and is configured per cf.

·       **Block Cache Sharing**:

o   Multiple cf-s may share the same block cache.

o   Multiple db-s may share the same block cache. However, there is no information in a single log file that provides any information about the sharing. RocksDB displays the information for the entire cache, so it’s impossible to know how it is used by the entities that share it (cf-s and db-s). Speedb adds per-cf information about the usage of the cache.

·       **Cache-Id**: A block cache is uniquely identified by its id (e.g., ‘LRUCache@0x7fde57dbf300#1’ in the image below). The cache’s id consists of 3 parts - \<name>@\<pointer>#\<process-id>

o   Name: The value of the block\_cache\_name field in the cf’s table options ('LRUCache').

o   Pointer: The value of the block\_cache field in the the cf’s table options ('0x7fde57dbf300'). This value may serve as a unique identifier for a specific cache entity within a process.

o   Process-id: The process id of the process that generated this log ('1'). The log parser may or may not display this part of the name.

·       **Cache Roles**: The cache classifies its contained blocks according to their type or role. Statistics are gathered for specific roles. All other roles are grouped under the ‘Misc’ role. The role’s name indicates its type (e.g., ‘FilterBlock’ is for Filter blocks). The ‘WriteBuffer’ role is shown when the Write Buffer Manager is configured to cost its reservations in the block cache.

·       The block cache is configured with a capacity. However, unless it is configured to use strict capacity (which is not done in practice), the usage may exceed the capacity.

**Top-level sub-objects**

<figure><img src="../.gitbook/assets/Screen Shot 2023-06-22 at 9.43.17.png" alt=""><figcaption></figcaption></figure>

There are 3 top-level sub-objects

1\.     Caches

2\.     DB Counters

3\.     Detailed

They are described in detail below.å

**Caches top-level sub-object**

<figure><img src="../.gitbook/assets/Screenshot from 2023-06-19 04-00-14.png" alt=""><figcaption></figcaption></figure>

This sub-object consists of the following:

* &#x20;Per cache-id (cache entity - “LRUCache@0x55deca6980f0”):
  * **Capacity**: The configured cache’s capacity.
  * **Num Shards**: The number of shards to which it is divided (2 ^ num\_shard\_bits).
  * **Shard Size**: The size of a single shared (capacity / number of shards)
  * **CF-s:**
    * Per-cf value of the `cache_index_and_filter_blocks` cf block based table option.
  * &#x20;**Index Block / Filter Block**: Statistics for the Index and Filter blocks that belong to cf-s that are part of this DB and share this cache. The data is obtained from `table_file_creation` events, so it is independent of its containment in the cache. The following data is displayed:
    * Avg. Size: The average size of a block of that type.
    * Max Size: The largest block of that type
    * Max Size At: At what time was the table that contained the largest block created.

**DB Counters top-level sub-object**

<figure><img src="../.gitbook/assets/Screenshot from 2023-06-19 04-21-44.png" alt=""><figcaption></figcaption></figure>

{% code lineNumbers="true" fullWidth="true" %}
```
2023/04/09-12:27:46.176881 30594 [/db_impl/db_impl.cc:807] STATISTICS:
 rocksdb.block.cache.miss COUNT : 61305
rocksdb.block.cache.hit COUNT : 25902952
rocksdb.block.cache.add COUNT : 60821
rocksdb.block.cache.add.failures COUNT : 0
rocksdb.block.cache.index.miss COUNT : 24
rocksdb.block.cache.index.hit COUNT : 6152077
rocksdb.block.cache.index.add COUNT : 24
rocksdb.block.cache.index.bytes.insert COUNT : 1538944
rocksdb.block.cache.index.bytes.evict COUNT : 0
rocksdb.block.cache.filter.miss COUNT : 24
rocksdb.block.cache.filter.hit COUNT : 13625205
rocksdb.block.cache.filter.add COUNT : 24
rocksdb.block.cache.filter.bytes.insert COUNT : 4558592
rocksdb.block.cache.filter.bytes.evict COUNT : 0
rocksdb.block.cache.data.miss COUNT : 61257
rocksdb.block.cache.data.hit COUNT : 6125677
rocksdb.block.cache.data.add COUNT : 60773
rocksdb.block.cache.data.bytes.insert COUNT : 253748240
```
{% endcode %}

This sub-object includes block cache miss and hit counters:

·       **cache\_add / cache\_miss cache\_hit**: Total number of cache misses / hits / adds (for all roles)

·       **index\_add / index\_miss / index\_hit**: Number of index block cache misses / hits / adds**.**

·       **filter\_add / filter\_miss / filter\_hit:** Number of filter block cache misses / hits / adds**.**

·       **data\_add / data\_miss / data\_hit:** Number of filter block cache misses / hits / adds**.**

**Detailed top-level sub-object**

<figure><img src="../.gitbook/assets/Screenshot from 2023-06-19 04-50-21.png" alt=""><figcaption></figcaption></figure>

This sub-object includes the individual processed block cache stats dumps such as the following:

{% code lineNumbers="true" fullWidth="true" %}
```
Block cache LRUCache@0x55deca6980f0#30528 capacity: 2.00 GB usage: 878.27 KB table_size: 256 occupancy: 16 collections: 1 last_copies: 2 last_secs: 6.2e-05 secs_since: 0
Block cache entry stats(count,size,portion): FilterBlock(7,459.22 KB,0.0218973%) IndexBlock(7,161.55 KB,0.00770316%) WriteBuffer(1,256.00 KB,0.012207%) Misc(1,0.00 KB,0%)
```
{% endcode %}

Per cache-id, the following **global** information:

·       **Capacity**: The block cache capacity (as configured)

·       **Usage**: The usage at the end of the log

This global information is followed by the individual dumps that are printed throughout the log. Per dump time, the following information is displayed, that reflects the state at that time:

·       **Usage**: The usage

·       **Per role** (Index / Filter / Write-Buffer / Misc):

o   **Count**: The number of blocks of that role currently in the cache

o   **Size**: The total size of those blocks

o   **Portion**: Their percentage of the total cache’s capacity

### CSV Files <a href="#speedbslogparsertool-csvfiles" id="speedbslogparsertool-csvfiles"></a>

During a run, the tool generates multiple CSV files. These files are best viewed with a spreadsheet viewing and editing software (e.g., LibreOffice Calc, Google Sheets, Microsoft Excel, etc.). These files are also an excellent input for further analysis and processing by applicable tools.

It may be that there is no applicable data in the parsed log to allow the generation of a CSV file (e.g., statistics are unavailable) . The tool reports, per CSV file, whether it was generated or not, and, if it was, the full path of the file.

The following table lists summarizes this:

| **CSV File name**                                             | **Source**                                                         | **Generation Info**        |
| ------------------------------------------------------------- | ------------------------------------------------------------------ | -------------------------- |
| counters.csv                                                  | Statistics dump                                                    | statistics must be enabled |
| <p>histograms_human_readable.csv /<br>histogram_tools.csv</p> | Statistics dump                                                    | statistics must be enabled |
| compaction\_stats.csv                                         | Compaction stats dumps                                             |                            |
| compactions.csv                                               | Compaction events (`compaction_started` and `compaction_finished`) |                            |
| flushes.csv                                                   | Flush events (`flush_started` and `flush_finished`)                |                            |

The sections that follow provide detailed information about the individual csv files.

#### counters.csv <a href="#speedbslogparsertool-counters.csv" id="speedbslogparsertool-counters.csv"></a>

This file contains the values of the dump of the counters in the log:

{% code lineNumbers="true" fullWidth="true" %}
```
2023/04/09-12:37:27.399291 30594 [/db_impl/db_impl.cc:807] STATISTICS:
 rocksdb.block.cache.miss COUNT : 2174509
rocksdb.block.cache.hit COUNT : 1626824082
rocksdb.block.cache.add COUNT : 2138479
rocksdb.block.cache.add.failures COUNT : 0
rocksdb.block.cache.index.miss COUNT : 881
```
{% endcode %}

<figure><img src="../.gitbook/assets/Screenshot from 2023-04-16 06-29-40.png" alt=""><figcaption></figcaption></figure>

This file is generated only when statistics are enabled.

The header line contains the names of the counters (e.g., ‘rocksdb.block.cache.miss’), in the order they appear in the log.

The row consists of the timestamp of the dump, followed by the values of all the counters at that time.

The dump to the log includes all of the tracked counters, even if their value is always 0. The CSV will **not** show counters whose values are always 0.

#### Counters Histograms CSV-s <a href="#speedbslogparsertool-countershistogramscsv-s" id="speedbslogparsertool-countershistogramscsv-s"></a>

These csv-s are for the dump of the counters histograms in the log. These are available when statistics are enabled

{% code lineNumbers="true" fullWidth="true" %}
```
rocksdb.db.get.micros P50 : 8.575028 P95 : 29.481907 P99 : 347.891977 P100 : 108208.000000 COUNT : 432304103 SUM : 8619564063
rocksdb.db.write.micros P50 : 299.321021 P95 : 1147.649366 P99 : 1865.648518 P100 : 159096.000000 COUNT : 48033546 SUM : 19528994334
rocksdb.compaction.times.micros P50 : 458512.396694 P95 : 702312.500000 P99 : 737699.000000 P100 : 737699.000000 COUNT : 174 SUM : 78440025
rocksdb.compaction.times.cpu_micros P50 : 418348.623853 P95 : 545480.000000 P99 : 545480.000000 P100 : 545480.000000 COUNT : 174 SUM : 67217901
rocksdb.subcompaction.setup.times.micros P50 : 0.000000 P95 : 0.000000 P99 : 0.000000 P100 : 0.000000 COUNT : 0 SUM : 0
rocksdb.table.sync.micros P50 : 2280.519481 P95 : 4275.000000 P99 : 9133.235294 P100 : 25031.000000 COUNT : 695 SUM : 1742121
rocksdb.compaction.outfile.sync.micros P50 : 2267.521368 P95 : 5060.000000 P99 : 9551.142857 P100 : 12668.000000 COUNT : 174 SUM : 416764
```
{% endcode %}

Two files are generated, one is aimed at a human reader (histograms\_human\_readable.csv) and the other at tools that will process the CSV (histograms\_tools.csv). Both files contain the same information, just arranged differently.

**histograms\_human\_readable.csv**

<figure><img src="../.gitbook/assets/Screenshot from 2023-06-13 14-46-50 (2).png" alt=""><figcaption></figcaption></figure>

**histograms\_tools.csv**

<figure><img src="../.gitbook/assets/Screenshot from 2023-06-13 14-42-08.png" alt=""><figcaption></figcaption></figure>

#### compactions\_stats.csv

This CSV includes the fields of the Compaction stats dumps:



{% code lineNumbers="true" fullWidth="true" %}
```
** Compaction Stats [column_family_name_000019] **
Level    Files   Size     Score Read(GB)  Rn(GB) Rnp1(GB) Write(GB) Wnew(GB) Moved(GB) W-Amp Rd(MB/s) Wr(MB/s) Comp(sec) CompMergeCPU(sec) Comp(cnt) Avg(sec) KeyIn KeyDrop Rblob(GB) Wblob(GB)
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  L0      4/0   982.88 MB   3.8      0.0     0.0      0.0       4.8      4.8       0.0   1.0      0.0    399.5     12.30              7.12        20    0.615       0      0       0.0       0.0
  L1      1/0   433.47 MB   1.7      4.6     3.8      0.7       4.5      3.8       0.0   1.2    543.1    540.4      8.59              7.37         5    1.718   4736K    23K       0.0       0.0
  L2      4/0    3.34 GB   1.3      6.4     2.9      3.4       6.3      2.9       0.5   2.2    261.6    259.6     24.89             14.18         3    8.297   6611K    51K       0.0       0.0
 Sum      9/0    4.73 GB   0.0     10.9     6.8      4.2      15.6     11.5       0.5   3.3    244.2    349.9     45.78             28.67        28    1.635     11M    75K       0.0       0.0
 Int      0/0    0.00 KB   0.0     10.9     6.8      4.2      15.6     11.5       0.5   3.3    244.2    349.9     45.78             28.67        28    1.635     11M    75K       0.0       0.0
```
{% endcode %}

<figure><img src="../.gitbook/assets/Screenshot from 2023-06-19 03-17-31.png" alt=""><figcaption></figcaption></figure>

#### compactions.csv <a href="#speedbslogparsertool-compactions.csv" id="speedbslogparsertool-compactions.csv"></a>

This CSV includes the fields of the compaction\_started and compaction\_finished events:

{% code lineNumbers="true" fullWidth="true" %}
```
2023/04/09-12:27:38.140657 30538 EVENT_LOG_v1 {"time_micros": 1681032458140646, "job": 4, "event": "compaction_started", "compaction_reason": "LevelL0FilesNum", "files_L0": [159, 150, 149, 146], "files_L1": [141], "score": 1, "input_data_size": 44710877, "oldest_snapshot_seqno": -1}
```
{% endcode %}

{% code lineNumbers="true" fullWidth="true" %}
```
2023/04/09-12:27:38.442191 30538 (Original Log Time 2023/04/09-12:27:38.442131) EVENT_LOG_v1 {"time_micros": 1681032458442117, "job": 4, "event": "compaction_finished", "compaction_time_micros": 300003, "compaction_time_cpu_micros": 297596, "output_level": 1, "num_output_files": 1, "total_output_size": 28850078, "num_input_records": 516430, "num_output_records": 333334, "num_subcompactions": 1, "output_compression": "NoCompression", "num_single_delete_mismatches": 0, "num_single_delete_fallthrough": 0, "lsm_state": [0, 1, 0, 0, 0, 0, 0]}
```
{% endcode %}

There is a large number of fields so only a subset is shown here. Please view an actual generated compactions.csv to see all of the fields.

<figure><img src="../.gitbook/assets/Screenshot from 2023-06-18 21-27-10.png" alt=""><figcaption></figcaption></figure>

#### flushes.csv <a href="#speedbslogparsertool-flushes.csv" id="speedbslogparsertool-flushes.csv"></a>

{% code lineNumbers="true" fullWidth="true" %}
```
2023/04/09-12:27:39.658098 30540 EVENT_LOG_v1 {"time_micros": 1681032459658093, "job": 6, "event": "flush_started", "num_memtables": 1, "num_entries": 85137, "num_deletes": 0, "total_data_size": 7662330, "memory_usage": 9538016, "flush_reason": "Write Buffer Manager Initiated"}
```
{% endcode %}

{% code lineNumbers="true" fullWidth="true" %}
```
2023/04/09-12:27:39.743132 30540 (Original Log Time 2023/04/09-12:27:39.743070) EVENT_LOG_v1 {"time_micros": 1681032459743064, "job": 6, "event": "flush_finished", "output_compression": "NoCompression", "lsm_state": [3, 1, 0, 0, 0, 0, 0], "immutable_memtables": 0}
```
{% endcode %}

Contains a subset of the fields from the flush\_started and flush\_finished events

<figure><img src="../.gitbook/assets/Screenshot from 2023-06-18 21-14-21.png" alt=""><figcaption></figcaption></figure>

## Testing

There are unit tests for the tool under the test folder. The tests are written in the pytest framework.

[pytest](https://docs.pytest.org/en/7.3.x/) installation

{% code lineNumbers="true" %}
```
pip install pytest
```
{% endcode %}

Running the unit tests:

{% code lineNumbers="true" %}
```
cd test
pytest
```
{% endcode %}

The tool should pass all of the unit tests.

A successful run should like like this:



{% code lineNumbers="true" fullWidth="true" %}
```
udi@udi-speedb:~/log-parser/test$ pytest
========================================================= test session starts ==========================================================
platform linux -- Python 3.8.10, pytest-7.2.1, pluggy-1.0.0
rootdir: /home/udi/log-parser/test
plugins: anyio-3.6.2
collected 139 items                                                                                                                    

test_baseline_log_files_utils.py ....                                                                                            [  2%]
test_cache_utils.py .                                                                                                            [  3%]
test_calc_utils.py .......                                                                                                       [  8%]
test_cfs_infos.py .........                                                                                                      [ 15%]
test_compactions.py ...                                                                                                          [ 17%]
test_counters.py .....                                                                                                           [ 20%]
test_csv_outputter.py ...                                                                                                        [ 23%]
test_db_files.py ..                                                                                                              [ 24%]
test_db_options.py ...............                                                                                               [ 35%]
test_events.py ...................                                                                                               [ 48%]
test_log_entry.py .....                                                                                                          [ 52%]
test_log_file.py ..............                                                                                                  [ 62%]
test_log_file_options_parser.py .......                                                                                          [ 67%]
test_regexes.py .....                                                                                                            [ 71%]
test_stats_mngr.py ....................                                                                                          [ 85%]
test_utils.py .................                                                                                                  [ 97%]
test_warnings_mngr.py ...                                                                                                        [100%]

========================================================= 139 passed in 0.26s ==========================================================
```
{% endcode %}

## Miscellaneous <a href="#speedbslogparsertool-miscellaneous" id="speedbslogparsertool-miscellaneous"></a>

### Reporting Bugs and Proposing <a href="#speedbslogparsertool-reportingbugsandproposing" id="speedbslogparsertool-reportingbugsandproposing"></a>

If you find a bug or wish to propose a new feature or an improvement in an existing feature, please open an issue in our [GitHub](https://github.com/speedb-io/log-parser/issues)

### Known Bugs <a href="#speedbslogparsertool-knownbugs" id="speedbslogparsertool-knownbugs"></a>

The major issues that and known bugs is listed in the project’s [GitHub](https://github.com/speedb-io/log-parser/issues).

### Contributing to the tool’s development <a href="#speedbslogparsertool-contributingtothetoolsdevelopment" id="speedbslogparsertool-contributingtothetoolsdevelopment"></a>

The log parser is an Speedb open source project. Contributions to its development are very welcome. Please see the [README.md](http://readme.md) in the repository for details.

### Getting Advice or Discussing the Tool <a href="#speedbslogparsertool-gettingadviceordiscussingthetool" id="speedbslogparsertool-gettingadviceordiscussingthetool"></a>

Please use the following [**Discord Channel**](https://discord.gg/gry235dZRh) to get assistance, ask questions, etc.

### The Future (the Features) <a href="#speedbslogparsertool-thefuture-thefeatures" id="speedbslogparsertool-thefuture-thefeatures"></a>

Requests for new features will be opened in [GitHub](https://github.com/speedb-io/log-parser/issues)
