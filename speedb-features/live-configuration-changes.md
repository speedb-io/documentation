---
description: >-
  This document describes the Live Configuration Changes functionality in Speedb
  introduced in v2.3.0
---

# Live Configuration Changes

## Overview&#x20;

Live Configuration Changes allows a user to change the mutable options of a Speedb database without restarting the process.  Mutable options allow the user to change certain behavior without closing the database.   For example, the user may wish to enable the Blob DB or change the compression algorithms without restarting their running database.  Prior to this feature, this could be accomplished by invoking the DB::SetOptions or DB::SetDBOptions methods.  Live Configuration Changes allow such changes to be made to a running database process without any downtime and without the programmer explicitly calling these methods.\


## Motivation&#x20;

Without the live configuration changes feature, in order to change a mutable option, a restart is required. Since configuration changes are sometimes required unexpectedly, this feature enables the user to update the options without any downtime.

## How does it work?

Live Configuration Changes works by periodically checking to see if a Speedb Options file (the **refresh\_options\_file**) exists and, if so, updating the Options to the values contained in the refresh\_options\_file. The frequency to check for the refresh options file is controlled by the Options.refresh\_options\_sec parameter.  If greater than zero, this parameter controls how often a background task will check if the file exists (by default, the refresh\_options\_file is checked hourly).

The name of the options file is specified in the Options.refresh\_options\_file parameter.  If the parameter points to an absolute path, the full path is used.  Otherwise, the path is relative to the database directory.  If no file name is set, the “Options.new” file in the database directory is checked.

The refresh options file will be checked periodically (every **refresh\_options\_sec** seconds).  If a file is found, it is read as an OPTIONS file. The file must be a valid options file, containing at a minimum the DBOptions and ColumnFamilyOptions “default” sections (though both may be empty and contain no parameters). &#x20;

Only the parameters that you would like to be changed should be specified in the file – other parameters will retain their current value.  Only parameters that are mutable (those which can be changed via SetOptions or SetDBOptions) may be in the file; non-mutable options will cause an error to be generated. &#x20;

The “table” section of the options file is ignored.  To update mutable table options, the options should be specified as “_table\_factory.option = new\_value_”, where “option” is the name of the option (e.g. “block\_size”) being changed.  Once processed, the refresh options file will be removed.

Because the refresh options job runs in the background asynchronously, no errors are reported to the user.  The log file should be checked to see that the values were successfully read and updated.  If there are errors while performing the update, the options may be partially applied (e.g. to the DBOptions or some of the column families). &#x20;

The name of the options file and the refresh time can be changed dynamically.  If the refresh\_options\_sec=0, this feature will be disabled.  If disabled, it can only be enabled by calling SetDBOptions with a non-zero refresh\_options\_sec or restarting the database with a non-zero time.&#x20;

## How to use it?&#x20;

1. Create an options file that includes all the options that you would like to update (see example below).  Name it as set in the Options.refresh\_options\_file parameter (default Options.new)&#x20;
2. Store the file in the database directory&#x20;
3. After the defined seconds passed, the options should be updated&#x20;

Note: The default of refresh\_options\_sec is 0.&#x20;

## Example:&#x20;

The example below shows an example refresh options file updating three properties.

&#x20;       \[DBOptions]

&#x20;       max\_background\_jobs = 32

&#x20;       \[CFOptions “default”]

&#x20;       table\_factory.block\_size = 32768

&#x20;       enable\_blob\_files = true

This example updates the DBOptions property “max\_background\_jobs”, the table property “enable\_blob\_files” and the ColumnFamilyOptions property “block\_size”.&#x20;

{% hint style="info" %}
**Note:** The \[DBOptions] section is required and can be left empty if no DBOptions are being updated.  The \[CFOptions “default”] section is also required and may be empty.  If properties in other column families were being updated, they would be updated in a corresponding \[CFOptions cf\_name] section.
{% endhint %}



\
