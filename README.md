DecSync
=======

DecSync (Decentralized Synchronization) synchronizes RSS, contacts, calendars, tasks and more without requiring a server. It uses a synchronized directory to synchronize the data. This can be done without a server by using for example [Syncthing](https://syncthing.net), but any other method like Google Drive or Dropbox also works.

DecSync can also be used to synchronize custom key-value mappings, but currently synchronization of RSS, contacts, calendars, tasks and memos is implemented using the following applications.

### RSS

* [FeedReader with DecSync plugin](https://github.com/39aldo39/FeedReader) (Linux)
* [spaRSS DecSync](https://github.com/39aldo39/spaRSS-DecSync) (Android)

### Contacts/Calendars/Tasks/Memos

* [DecSync for Radicale](https://github.com/39aldo39/Radicale-DecSync) (Radicale plugin, Windows and Linux). Allows for synchronization with any CalDAV/CardDAV compatible client like [Thunderbird](https://thunderbird.net).
* [DecSync for Evolution](https://github.com/39aldo39/Evolution-DecSync) (Evolution plugin, Linux)
* [DecSync CC](https://github.com/39aldo39/DecSyncCC) (Android). Tasks are supported through [OpenTasks](https://opentasks.app) and [tasks.org](https://tasks.org), memos are not supported.

To start using DecSync, all you have to do is install some of the applications above and synchronize the DecSync directories.

Technical
---------

If you want to use DecSync in your own application, you can use the multiplatform library [libdecsync](https://github.com/39aldo39/libdecsync).

The structure of the synchronized mappings used for RSS and contacts/calendars are described in [rss.md](rss.md) and [contacts-calendars.md](contacts-calendars.md).

Information about the design of DecSync, and how to apply it is given in [design.md](design.md).

For details about the internal implementation, see [spec-general.md](spec-general.md).

Donations
---------

### PayPal
[![](https://www.paypalobjects.com/en_US/i/btn/btn_donateCC_LG.gif)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=4V96AFD3S4TPJ)

### Bitcoin
[`1JWYoV2MZyu8LYYHCur9jUJgGqE98m566z`](bitcoin:1JWYoV2MZyu8LYYHCur9jUJgGqE98m566z)
