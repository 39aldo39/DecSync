DecSync
=======

DecSync (Decentralized Synchronization) synchronizes key-value mappings using the file system in a conflict-free way. It uses a synchronized directory to synchronize the mappings. This can be done without a server by using for example [Syncthing](https://syncthing.net).

Currently, DecSync can be used to synchronize RSS, contacts and calendars using the following applications.

### RSS

* [FeedReader with DecSync plugin (Linux)](https://github.com/jangernert/FeedReader)
* [spaRSS DecSync (Android)](https://github.com/39aldo39/spaRSS-DecSync)

### Contacts/Calendars

* [DecSync for Evolution (Evolution plugin)](https://github.com/39aldo39/Evolution-DecSync)
* [DecSync CC (Android)](https://github.com/39aldo39/DecSyncCC)

To start using DecSync, all you have to do is install some of the applications above and synchronize the DecSync directories. By default, the DecSync directory is `~/.local/share/decsync` on Linux and `DecSync` in the primary external storage on Android.

Technical
---------

If you want to use DecSync in your own application, you can use the libraries [libdecsync-android](https://github.com/39aldo39/libdecsync-android) and [libdecsync-vala](https://github.com/39aldo39/libdecsync-vala).

The structure of the synchronized mappings used for RSS and contacts/calendars are described in [rss.md](rss.md) and [contacts-calendars.md](contacts-calendars.md).

For details about the internal implementation, see [internal.md](internal.md).
