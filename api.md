API
===

This document lists the primary functions for interacting with the DecSync data stored on the file system.
The data is represented as a collection of entries, each with a `path` (list of strings), a `key` (arbitrary JSON value) and a `value` (arbitrary JSON value).
Every `path` and `key` combination is associated with a single `value`.

### `Constructor(decsyncDir, syncType, collection, ownAppId)`

Creates a DecSync instance required for most interactions.

- The `decsyncDir` is the directory in which the synchronized DecSync files are stored.
- The `syncType` is the type of data to sync. For example, `"rss"`, `"contacts"` or `"calendars"`.
- The `collection` is an optional collection identifier when multiple of the `syncType` are supported.
  For example, this is the case for `"contacts"` and `"calendars"`, but not for `"rss"`.
  The collections can be obtained by using `listDecsyncCollections(decsyncDir, syncType)`.
- The `ownAppId` is the unique application ID corresponding to the application.
  There must not be two simultaneous instances with the same `appId`.
  However, if an application is reinstalled, it may reuse its old `appId`.

Methods
-------

### `setEntry(path, key, value)`

Updates the entry corresponding to the given `path` and `key`.
Its new value will be `value`.

There are also some variants that allow for multiple simultaneous updates.
Those can be more convenient or efficient.

### `addListener(subpath, onEntryUpdate(path, key, value, extra))`

Adds a listener for the entries whose paths start with `subpath`.
Whenever a relevant entry is updated, `onEntryUpdate(path, key, value, extra)` is called, where `path`, `key` and `value` are the parts of the entry (with `subpath` removed from the path) and where `extra` is some extra data passed by the calling method.

### `executeAllNewEntries(extra)`

Gets all updated entries and calls the corresponding listeners.
The `extra` data is passed to all the listeners.

There is also the variant `initStoredEntries()` that also gets all updated entries, but it does not call any listeners.

### `executeStoredEntry(path, key, extra)`

Gets the stored entry corresponding to the given `path` and `key` and calls the corresponding listener.
The `extra` data is passed to the listener.

There are also some variants that allow multiple entries to be processed at once.
Some even allow for wildcards in the `path` or `key`.

Static methods
--------------

The following methods do not require a DecSync instance.

### `checkDecsyncInfo(decsyncDir)`

Checks whether the DecSync info marker in `decsyncDir` is of the right format and contains a supported version.
If it does not exist, a new one with version 1 is created.

### `listDecsyncCollections(decsyncDir, syncType)`

Lists all DecSync collections inside `decsyncDir` corresponding to `syncType`.
This function does not apply for sync types with single instances like `"rss"`.

### `getStaticInfo(decsyncDir, syncType, collection)`

Gets the most up-to-date entries stored in the path `["info"]` corresponding to the given `decsyncDir`, `syncType` and `collection`.
Entries stored in `["info"]` are about the collection, like its name and whether it is deleted or not.
