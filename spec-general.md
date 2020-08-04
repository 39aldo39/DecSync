Specification (general)
=======================

This document describes some behavior of DecSync unrelated to a specific version.
The specifications for specific versions are given in [spec-v1.md](spec-v1.md).
The main functions for interacting with the DecSync data are given in [api.md](api.md).

Static methods
--------------

The static methods do not require a DecSync instance, hence they cannot rely on a certain version.
Here, their implementations are described.
See [api.md](api.md) for their behavior.

### `checkDecsyncInfo(decsyncDir)`

There is a file `decsyncDir/.decsync-info` that contains a JSON object with information about the DecSync instance.
Currently, it contains just one mapping, namely `"version"` that gives the current DecSync version.
This method checks whether the given version is supported.

### `listDecsyncCollections(decsyncDir, syncType)`

This simply lists all the directories that exist under `decsyncDir/syncType`.

### `getStaticInfo(decsyncDir, syncType, collection)`

This method calls `getStaticInfo` for all specific versions (up to the current version) and combines their results into one.
It is necessary to also get the old versions, as the collection may not be upgraded to the latest version.
For example, if a collection is deleted, it will (probably) never be upgraded, but this method should still say that it is deleted.
Even worse, it is possible that an application is (temporarily) working on an outdated version, but writes a new entry to it.

Version management
------------------

As we want to avoid file conflicts, we cannot simply upgrade to whole DecSync directory to a new version.
Instead, every application has to upgrade its own subdirectories.
The most recent version is given by `.decsync-info`.
This is the only version that is guaranteed to be up-to-date.
An applications may still (temporarily) work with an outdated version, and even write new entries to it, but it cannot expect to have the latest information.
This can be useful when the application is in the foreground, as an upgrade is preferably executed in the background.
When an application has time to upgrade, it does the following:

- Call `executeAllNewEntries` to be up-to-date relative to the old version.
- Get all the entries from the old version.
- Write all the entries to the new version.
- Delete the entries from the old version.
- Call `executeAllNewEntries` to also be up-to-date relative to the new version.

Consequently, a version upgrade is as simple as modifying the `"version"` field inside `.decsync-info`.

Local directories
-----------------

In addition to the normal DecSync directory, applications also store data that does not have to be synchronized with other devices.
Currently, these local directories are simply stored under `syncType/collectionID/"local"/appId` inside the DecSync directory.
Its place inside the DecSync directory is just the (current) default location, but it is not required.
It could be placed anywhere, even outside the DecSync directory.

This local directory has a file `info` with a JSON object containing some general information.
Currently it contains two keys:

- `"version"`, denoting the current DecSync version that the application uses.
  It could deduce this from the current file structure, but storing it here works as an easier cache.
- `"last-active"`, denoting the last date as `"YYYY-MM-DD"` that this application was active.
  This is written as the entry with path `["info"]` and key `"last-active-appId"`.
  It is used for data management, as it allows for detection of out-of-date applications.
