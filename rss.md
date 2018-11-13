RSS
===

For RSS, some properties of the articles, feeds and categories are stored.

For the articles, the read and marked/starred statuses are stored. The articles themselves are not stored, they have to be (periodically) retrieved by the application. The articles are identified by their guid.

There is also a list of subscribed feeds stored. Additionally, for each feed its name and its category are stored. The feeds are identified by their URL.

Finally, for each category, its name and its parent category are stored. The categories are identified by string ids. Furthermore, categories are created and removed on the fly. When a category is needed for an action, it is created. And when a category becomes empty, it is removed.

Mappings
--------

The following DecSync paths are used.

#### `["articles", "read", year, month, day]`

* key: String, guid of the article.
* value: Boolean, indicating the read status of the article.
* year/month/day: the year/month/day of the publication date, with leading zeros.

Since the article may not be known yet, we have to retroactively execute this action when we fetch an article.

#### `["articles", "marked", year, month, day]`

* key: String, guid of the article.
* value: Boolean, indicating the marked/starred status of the article.
* year/month/day: the year/month/day of the publication date, with leading zeros.

Since the article may not be known yet, we have to retroactively execute this action when we fetch an article.

#### `["feeds", "subscriptions"]`

* key: String, URL of the feed.
* value: Boolean, indicating the subscription status of the feed.

#### `["feeds", "names"]`

* key: String, URL of the feed.
* value: String, name of the feed.

Since the feed may not be known yet, we have to retroactively execute this action when we subscribe to a feed.

#### `["feeds", "categories"]`

* key: String, URL of the feed.
* value: String or null, category id of the category of the feed, null if the feed has no category.

Since the feed may not be known yet, we have to retroactively execute this action when we subscribe to a feed.

#### `["categories", "names"]`

* key: String, category id.
* value: String, name of the category.

Since the category may not be known yet, we have to retroactively execute this action when a category (with id) is added.

#### `["categories", "parents"]`

* key: String, category id.
* value: String or null, category id of the parent category, null if the category has no parent.

Since the category may not be known yet, we have to retroactively execute this action when a category (with id) is added.
