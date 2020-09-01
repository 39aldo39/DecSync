Specification (v2, draft)
=========================

File structure
--------------

Internally, all key-value mappings are stored in files in the DecSync directory.
Each collection is stored in the subdirectory `syncType/collectionId`, or just `syncType` if there is just one collection for the sync type.
Every application stores its own files in the subdirectory `"v2"/appId`.
So, every application only writes files inside `syncType/collectionId/"v2"/appId`.
Inside these directories there are two types of files: entry files of the form `00` to `ff` or `info`, and a `sequences` file.

In addition to the `"v2"/appId` directories, there are also the `"local"/appId` ones.
These directories store information only relevant to the application itself.
Its place inside the DecSync directory is just the (current) default location, but this is not required.
It could be placed anywhere, even outside the DecSync directory.

### Example

```
DecSync
├── calendars
│   └── colID12345
│       ├── local
│       │   ├── appId1
│       │   │   ├── info
│       │   │   └── sequences
│       │   └── appId2
│       │       ├── info
│       │       └── sequences
│       └── v2
│           ├── appId1
│           │   ├── 2f
│           │   ├── a7
│           │   ├── info
│           │   └── sequences
│           └── appId2
│               ├── 2f
│               ├── a7
│               ├── info
│               └── sequences
└── rss
    ├── local
    │   ├── appId1
    │   │   ├── info
    │   │   └── sequences
    │   └── appId2
    │       ├── info
    │       └── sequences
    └── v2
        ├── appId1
        │   ├── 0b
        │   ├── b9
        │   └── sequences
        └── appId2
            ├── 0b
            ├── b9
            └── sequences
```

With for example `DecSync/rss/v2/appId1/b9`:

```
[["feeds", "subscriptions"], "2020-07-17T12:34:56", "https://foo.example.com/rss", true]
[["feeds", "subscriptions"], "2020-07-17T12:35:56", "https://bar.example.com/rss", false]
[["feeds", "subscriptions"], "2020-07-17T12:36:56", "https://baz.example.com/rss", true]
```

`DecSync/rss/v2/appId1/sequences`:

```
{"0b": 1, "b9": 4}
```

`DecSync/rss/local/appId2/info`:

```
{"version": 2, "last-active": "2020-07-17"}
```

`DecSync/rss/local/appId2/sequences`:

```
{"appId1": {"0b": 1, "b9": 2}}
```

### Entry files

The entry files contain a list of entries separated by newlines.
Every entry is expressed by the JSON value `[path, datetime, key, value]`, with `path` an array of strings, `datetime` a string and `key` and `value` arbitrary JSON values.
The name of the file is equal to the hash of `path`, whose definition is specified below.

### Sequence files

The `sequences` file contains a JSON object mapping the hashes to their sequence numbers.

### Local files

Inside the local directory the file `sequences` is stored.
This file contains a JSON object, mapping all other appIds to their last read sequence numbers.
Furthermore, there is a `info` file stored inside this directory, which is used for general operation of the DecSync directory unrelated to the current version.

### Hash function

The hash function hashes a `path` to an integer in `[0, 256)` expressed as two lowercase hexadecimal digits.
It is the polynomial hash modulo 256 of its hashed parts evaluated at 199.
That is, `[s1, s2, ..., sn]` is hashed to `199^(n-1)*h_s(s1) + 199^(n-2)*h_s(s2) + ... + 1*h_s(sn)`.
Here, `h_s` is the hash for the strings.
This is also a polynomial hash modulo 256, but over the UTF-8 bytestring and evaluated at 19.
That is, `[b1, b2, ..., bn]` is hashed to `19^(n-1)*b1 + 19^(n-2)*b2 + ... + 1*bn`.
There is one exception to this: the hash of `["info"]` is equal to `info`.

Methods
-------

Here, the implementations of the most important methods are given, see also [api.md](api.md).

### `setEntry(path, key, value)`

- Remove any entry with the same `path` and `key` in the file `hash(path)`.
- Append the entry `[path, datetime-now, key, value]` to the file `hash(path)`, with `datetime-now` the current datetime.
- Increase the corresponding sequence number in `sequences` by 1, with a non-existing one treated as 0.

### `executeAllNewEntries(extra)`

- Read the `sequences` file in our local directory.
- List all the files with updates by reading all other `sequences` files.
- Read all the entries in the updates files.
- Filter out already up-to-date entries.
- Execute all the new entries.
- Add the new entries to our own entry files.
- Update the `sequences` file in our local directory.
- Do not update our own `sequences` file, as the entries do not originate from us.

### `executeStoredEntry(path, key, extra)`

- Read the file `hash(path)`.
- Filter out the entry corresponding to `path` and `key`.
- Execute the entry, if it exists.

Static methods
--------------

### `getStaticInfo(decsyncDir, syncType, collection)`

- Read all the `info` files in all directories.
- Combine the results to get all most up-to-date entries.
