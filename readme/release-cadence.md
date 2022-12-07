# Release cadence

Speedb releases follow the usual major.minor.patch semantic versioning schema.&#x20;

**Major versions**

Major versions introduce new capabilities and significant changes. It's possible that these versions don't maintain backward compatibility.&#x20;

{% hint style="info" %}
Upgrading to a new major version may require some code changes.&#x20;
{% endhint %}

#### Minor versions

Minor versions usually contain extended functionality, without breaking compatibility.&#x20;

{% hint style="info" %}
Upgrade to a new minor version is expected to be seamless&#x20;
{% endhint %}

#### Patches

Patch-level versions consist primarily of backward-compatible bug fixes.&#x20;

Default values may also be changed with patch versions, as long as they don't impact the system.&#x20;

{% hint style="info" %}
Upgrading from the previous patch version is expected to be seamless.
{% endhint %}



### Release schedule

Our goal is to release a new version approximately every two months.

* The version number will be based on the content and semantic versioning
* Any critical bug fixes will be resolved outside of the release cycle.
* We will rebase to a newer Rocksdb version occasionally, changing the release number accordingly
