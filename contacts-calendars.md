Contacts/Calendars
==================

For contacts/calendars, a list of resources (contacts/events) is stored in the vCard/iCalendar format. Furthermore, some general information about the collection, like its name, is stored.

Storing the vCard/iCalendar resource as a single value is not ideal. For example, if the name and location of an event are changed independently, one of those changes is ignored. It would be better to store each property with a different key. However, the vCard and iCalendar formats are supported very well in programs, which makes the implementation way easier.

Mappings
--------

The following DecSync paths are used.

#### `["info"]`

* key: "name"; value: String, name of the collection.
* key: "deleted"; value: Boolean, indicating the deleted status of the collection.
* (calendars only) key: "color"; value: String with format "#rrggbb", indicating the color of the calendar.

#### `["resources", uid]`

* key: null.
* value: String or null, the resource (contact/event) as vCard/iCalendar, null if the resource is removed.
* uid: the uid of the resource.
