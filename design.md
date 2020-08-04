Design
======

The goal of DecSync is to be able to synchronize data between different devices, without any explicit server.
It uses the file system as a tool, as there are already many ways to synchronize files.
A notable example is [Syncthing](https://syncthing.net) which synchronizes directly between devices, but any file synchronization tool can be used.

The biggest problem when synchronizing are conflicts.
This occurs when device A and B both change some data when they are offline.
When they become online, we would like to get the updates from both A and B.
Additionally, we have to avoid file conflicts, as we are using the file system.
The latter problem is solved by giving every device its own subdirectory, which means that every file has a unique device that can modify it.
The problem of processing all updates is solved by having additional requirements:

1. We only support key-value pairs (entries).
2. The order in which the entries are processed does not matter.
3. Store as little information as possible in every entry.

With these requirements, there are no problems when A and B update different keys, as B can just process the update from A and vice versa.
Requirement (2) guarantees that their states will be the same afterwards.
However, there will still be a conflict when A and B update the same keys.
This conflict is solved by having timestamps and choosing the most recent update.
The information loss is minimized by (3).
Moreover, there will be no file conflicts when this situation occurs.

Satisfying the Requirements
---------------------------

The main challenge when synchronizing data using DecSync is expressing our data in such a way that the requirements are satisfied.
The main tool for this will be replaying updates.
We will illustrate this by using the categories of [RSS](rss.md) as an example.
In RSS, we have a list of subscribed URLs.
Every subscription has a name and can be part of a category.
Every category has a name as well and can also be part of another category.
For example, we may have the following structure:

```
.
├── Cat1
│   ├── Bar: bar.example.com
│   └── Cat2
│       └── Baz: baz.example.com
└── Foo: foo.example.com
```

### Subscriptions

First, we look at the subscriptions.
They are the following: `[Foo: foo.example.com, Bar: bar.example.com, Baz: baz.example.com]`.

#### Expressing as Key-Value Pairs

We express them as key-value pairs by creating two different kinds of entries: one kind for whether we are subscribed, and another for the name of the subscription.
This results in the following entries:

```
feeds/subscriptions/foo.example.com: true
feeds/subscriptions/bar.example.com: true
feeds/subscriptions/baz.example.com: true

feeds/names/foo.example.com: Foo
feeds/names/bar.example.com: Bar
feeds/names/baz.example.com: Baz
```

#### Replaying Updates

Although it is a simple mapping, it does not satisfy requirement (2) directly!
It works fine when we process a subscription update first and process the name afterwards, but it does not work the other way around.
When we get a name for a non-existing feed, we cannot perform the update, so the name of the feed is lost.
In this case, we could store the name anyway, but that may not be straightforward and in other situations it is not possible.

What we do instead, is that we replay an update when necessary.
When we add the subscription `foo.example.com`, we execute the entry `feeds/names/foo.example.com: Foo` retroactively.
This way, it does not matter in which order the updates are executed, the name will be correct in all cases.
In general, when we cannot perform an update because some required data is missing, we perform it retroactively when the missing data is added.

### Categories

Now we look at the categories.

#### Expressing as Key-Value Pairs

This time we have three kind of entries: first one for the categories of the feeds, a second one for the names of the categories, and a third one for the parents of the categories.
This leads to the following entries:

```
feeds/categories/foo.example.com: null
feeds/categories/bar.example.com: catID1
feeds/categories/baz.example.com: catID2

categories/names/catID1: Cat1
categories/names/catID2: Cat2

categories/parents/catID1: null
categories/parents/catID2: catID1
```

Note that we use identifiers for the categories.
We cannot just use names, as that gives trouble for requirement (2).
On the other hand, we were able to just use the URLs for the subscriptions, as those are already unique identifiers for them.

#### Replaying Updates

Like in the case of the subscriptions, we have to replay updates in order to satisfy (2).
Unlike the previous case, we do not have an entry that creates or deletes a category.
It is a valid strategy to have additional entries for that.
However, it creates some additional complexity, as it is not clear what to do when a feed is added to a deleted category.

Instead, we create and delete categories on the fly.
Whenever we reference a non-existing category, we create it, and when a category becomes empty, we delete it.
Note that we only automatically delete a category when it becomes empty programmatically, not when the user makes it empty, as that would be confusing.

Regarding the replaying updates, we execute the entries `categories/names/catID1` and `categories/parents/catID1` retroactively when the category `catID1` is created.
Additionally, we have to execute the entry `feeds/categories/foo.example.com` when the subscription `foo.example.com` is added, in addition to the entry we had to execute in the previous section.
