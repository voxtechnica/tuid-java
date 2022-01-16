# Time-based Unique Identifiers (TUID)

A `TUID` is an immutable, chronologically sortable, unique identifier with an embedded nanosecond-resolution createdAt
timestamp. Internally, it is a `BigInteger` value, expressed as a case-sensitive base-62 string (digits 0-9, characters
A-Z, and characters a-z). The left bits are the embedded nanoseconds since epoch, and the right 32 bits are entropy (a
`SecureRandom` integer). Valid IDs in the 21st century are 16 characters long, and the bit length ranges from 92-94
bits. Example: `91Mq07yx9IxHCi5Y` is a `TUID` with a UTC timestamp of `2021-03-08T05:54:09.208207Z`. Because the
left-most bits are the nanoseconds since epoch, the IDs sort in chronological order.

A `TUID` compares nicely with a
128-bit `UUID` ([Universally Unique Identifier](https://en.wikipedia.org/wiki/Universally_unique_identifier)), but it's
not a [UUID](https://www.ietf.org/rfc/rfc4122.txt). While a Version 1 `UUID` has an embedded timestamp, it's not
chronologically sortable. A random Version 4 `UUID` is more common. Example: `05cd1093-c52c-48e5-a343-9e0017454067`.
And, while a `UUID` is readily available in the Java standard library, a `TUID` has significant advantages:

* A `TUID` is sortable chronologically (alphabetically), making it easy to order a collection of things with IDs.
* A `TUID` has an embedded nanosecond resolution timestamp, making it easy to track when things were created.
* A `TUID` is shorter than a `UUID` (16 characters vs. 36 characters), requiring less space in a database or URL.
* A `TUID` has no hyphens, making it easy to copy and paste into other systems.

A `TUID` also compares nicely with a `FriendlyID` ([Friendly Identifier](https://github.com/Devskiller/friendly-id)).
Example: `5wbwf6yUxVBcr48AMbz9cb`. A `FriendlyID` is a base-62 encoded 128-bit `UUID` without the hyphens, making it
shorter and easier to work with than a `UUID`. A `FriendlyID` has the great advantage of being interoperable with any
existing `UUID`, but it's missing the embedded timestamp and is not chronologically sortable. Also, a `FriendlyID`
version of a full 128-bit `UUID` is 22 characters, whereas a `TUID` is only 16 characters.

So, why use a `TUID`? Because you like the embedded timestamp, and you like being able to sort things chronologically.

## Using TUIDs

Creating a new `TUID` is simple:

* `TUID tuid = new TUID();` :: Create a `TUID` with the current system time and a random entropy value.
* `TUID tuid = new TUID(Instant.now());` :: Create a `TUID` with a specified timestamp and random entropy value.
* `TUID tuid = new TUID("91Mq07yx9IxHCi5Y");` :: Instantiate a `TUID` from an ID string.
* `TUID tuid = TUID.fromString("91Mq07yx9IxHCi5Y");` :: Instantiate a `TUID` from an ID string.
* `TUID tuid = TUID.withTimestamp(Instant.now());` :: Create a `TUID` with a specified timestamp and random entropy
  value.

Often, you'll just be working with a `TUID` as an ID string, which is a compact representation of the `TUID` with all
the relevant information embedded:

* `String id = tuid.getId();` :: Get the ID string.
* `String id = tuid.toString();` :: Get the ID string.
* `String id = TUID.createId();` :: Get an ID string with the current system time and a random entropy value.
* `String firstId = TUID.firstId(instant);` :: Get the "first" ID at the specified timestamp, with zero entropy. These
  are useful for paging through a collection of things with IDs.
* `Boolean valid = TUID.isValidId(id);` :: Check if the ID string is a valid `TUID` in the 21st century.

To get access to the embedded timestamps, you can use the following methods:

* `Instant createdAt = tuid.getCreatedAt();` :: Get the timestamp as a UTC Instant with nanosecond resolution.
* `Instant createdAt = TUID.createdAt("91Mq07yx9IxHCi5Y");` :: Get the timestamp from an ID string as a UTC Instant.
* `ZonedDateTime timestamp = tuid.timestamp();` :: Get the timestamp as a UTC ZonedDateTime with nanosecond resolution.
* `ZonedDateTime timestamp = tuid.timestamp(ZoneId.of("US/Pacific"));` :: Get the timestamp as a ZonedDateTime in the
  specified time zone.
* `String timestamp = tuid.timestamp(ZoneId.of("US/Pacific"), DateTimeFormatter.ofPattern("E MM-dd-yyyy hh:mm:ss v"));` ::
  Get the timestamp as a formatted string in the specified time zone.
* `LocalDate createdOn = tuid.localDate();` :: Get the timestamp as a UTC LocalDate with no resolution.
* `LocalDate createdOn = tuid.localDate(ZoneId.of("US/Pacific"));` :: Get the timestamp as a LocalDate with no
  resolution in the specified time zone. LocalDates are useful for grouping things that happen on the same day.
* `Duration duration = startTuid.duration(stopTuid);` :: Get the time duration between two `TUID`s.
* `Duration duration = stopTuid.durationSince(startTuid);` :: Get the time duration between two `TUID`s.
* `Duration duration = TUID.duration(startId, stopId);` :: Get the time duration between two ID strings.

Usually, when working with things that have IDs, you'll want to use just the ID string and the createdAt timestamp.
Example:

```
TUID tuid = new TUID();
Event event = new Event();
event.setId(tuid.getId());
event.setCreatedAt(tuid.getCreatedAt());
```

Or, since the `createdAt` timestamp is embedded in the ID string, you can use the static `TUID.createdAt` method in a
getter for the `createdAt` timestamp for your entity:

```
String getId() {
  return id;
}

Instant getCreatedAt() {
  return TUID.createdAt(id);
}
```
