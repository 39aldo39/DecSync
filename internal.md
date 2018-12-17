Internal
========

Internally, all key-value mappings are stored in files in the DecSync directory. Each collection is stored in the subdirectory `syncType/collectionId`, or just `syncType` if there is just one collection for the sync type.

In each collection, there are four directories: `"new-entries"`, `"stored-entries"`, `"read-bytes"` and `"info"`. Each `appId` has an own subdirectory in each of these: `"new-entries"/appId`, `"stored-entries"/appId` etc. This way, file conflicts are impossible, since concurrent updates modify different directories.

### New entries
In the directory `"new-entries"` updated entries are stored. When an application sets a `key` at a `value` in a `path` at a `datetime`, the JSON value `[datetime, key, value]` is appended to the file `"new-entries"/appId/path`.

### Stored entries
In the directory `"stored-entries"` already received entries are stored. The file format is equal to that in `"new-entries"`: each file contains lines of the format `[datetime, key, value]` and is stored at `"stored-entries"/appId/path`. However, in each path, only the most recent entry for each key is stored in the file.

When an entry is updated (either by the current application, or another one), its JSON value may be appended to the file with the right path. If the new entry is newer than the previous one stored (or there is no previous entry), the new entry is appended and the previous one is removed. If the new entry is older than the previous one stored, nothing happens to the file. Furthermore, when the entry was successfully updated by another application, the application is notified about the update and executes the corresponding action.

### Read bytes
In the directory `"read-bytes"` the amount of processed entries are stored. The locations of the files have the format `"read-bytes"/ownAppId/otherAppId/path` and contains an integer indicating the number of bytes read from the file `"new-entries"/otherAppId/path`. When an application looks for additions to that file, it can skip to the new part of the file. Even better, if the number of read bytes is equal to the file size, it does not even have to open the file.

### Info
In the directory `"info"` some additional information is stored. Currently, only the datetime of the most latest stored entry is stored in `"info"/appId/"latest-stored-entry"`. This is used to get the most up-to-date appId (approximately) for an initial sync.

### Sequence numbers
In addition to storing the number of read bytes of each file in `"new-entries"`, the sequence numbers of the subdirectories are stored as well. Every subdirectory contains the hidden file `".decsync-sequence"`, which contains the number of times a file in the subdirectory has been updated. For example, when an entry is updated at the path `"new-entries"/appId/"dir1"/"dir2"/"file"`, the integers stored in `"new-entries"/appId/"dir1"/"dir2"/".decsync-sequence"`, `"new-entries"/appId/"dir1"/".decsync-sequence"` and `"new-entries"/appId/".decsync-sequence"` are increased by 1. Additionally, these files are stored in `"read-bytes"` as well. Using this, an application can skip a directory in `"new-entries"` if its sequence number is equal to its stored one. In particular, it only has to read the file `"new-entries"/appId/".decsync-sequence"` when there are no updates.

### File observer
A different way of getting notified about updated entries is by using a file observer. A basic one which recursively watches every file in `"new-entries"` is implemented. However, it should be possible to implement one which just watches every `"new-entries"/appId/".decsync-sequence"` for updates, and `"new-entries"` for new applications. Nevertheless, it is not clear when the file syncing program is done syncing. It may update the sequence number before the update itself, in which case the update is lost.

### Path encoding
Since we want all characters to be allowed in the path, we apply a [urlencoding](https://en.wikipedia.org/wiki/Percent-encoding) to the path segments, which makes the strings valid filenames. However, this does not solve all limitations on the file systems, like length limits or case insensitivity. Case insensitivity is not a big problem, since the case is usually still stored. However, multiple files differing in case are not supported.
