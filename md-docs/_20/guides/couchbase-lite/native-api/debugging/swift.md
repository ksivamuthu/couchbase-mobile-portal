---
permalink: guides/couchbase-lite/native-api/swift/debugging/index.html
---

## Finding a Database File

When the application is running on the iOS simulator, you can easily locate the application's sandbox directory using the [SimPholders](https://simpholders.com/3/) utility.

## CLI tool

The Couchbase Lite `.zip` file available from the [downloads page](https://www.couchbase.com/downloads) contains CLI which can be used on **.cblite2** files.

<img src="../../img/tools-folder.png" />

## Logging

The log messages are split into different domains (`LogDomains`) which can be tuned to different log levels. The following example enables `verbose` logging for the `replicator` and `query` domains.

```swift
Database.setLogLevel(.verbose, domain: .replicator)
Database.setLogLevel(.verbose, domain: .query)
```