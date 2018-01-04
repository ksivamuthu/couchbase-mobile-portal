---
---

## New Database

As the top-level entity in the API, new databases can be created using the `Database` class by passing in a name, configuration, or both. The following example creates a database using the `Database(name: String)` method.

```objectivec
NSError *error;
CBLDatabase* database = [[CBLDatabase alloc] initWithName:@"my-database" error:&error];
if (!database) {
    NSLog(@"Cannot open the database: %@", error);
}
```

Just as before, the database will be created in a default location. Alternatively, the `Database(name: Strings, config: DatabaseConfiguration?)` method can be used to provide specific options (the directory to create the database in, encryption key, etc.)

## Migrating from 1.x Databases

Databases that were created with Couchbase Mobile 1.2 or later can be read using the 2.0 API. Upon detecting it is a 1.x database file format, Couchbase Lite will automatically upgrade it to the new format used in 2.0. This feature is currently only available for the default storage type, SQLite (i.e not for ForestDB databases).

## Finding a Database File

[//]: # (TODO: replace content with best practice for Android)

When the application is running on the iOS simulator, you can easily locate the application's sandbox directory using the [SimPholders](https://simpholders.com/3/) utility.

## CLI tool

The Couchbase Lite `.zip` file available from the [downloads page](https://www.couchbase.com/downloads) contains a CLI which can be used on **.cblite2** files.

## Logging

The log messages are split into different domains (`LogDomains`) which can be tuned to different log levels. The following example enables `verbose` logging for the `replicator` and `query` domains.

[//]: # (TODO: replace below with ObjC/C#/Java)

```swift
Database.setLogLevel(.verbose, domain: .replicator)
Database.setLogLevel(.verbose, domain: .query)
```

## Singleton Pattern

The database instance must be used throughout the Couchbase Lite API to Create, Update, Delete and Query documents. Hence, the singleton pattern is useful to create a single instance of the `Database` object. The following example follows the Singleton Pattern in `Swift`.

[//]: # (TODO: replace below with ObjC/C#/Java)

```swift
class DataManager {
  static let sharedInstance: DataManager = DataManager()
	
  private init() {
    do {
  	  self.database = try Database(name: "dbname")
    } catch {
  	  fatalError("Could not initialize database")
    }
  }
}
```

The database instance can then be access throughout the codebase using the class property: `DataManager.sharedInstance.database`.

##  Encryption

The following example demonstrates how to create a database with an encryption key (or open an existing one).

[//]: # (TODO: replace below with ObjC/C#/Java)

```swift
var dbConfig = DatabaseConfiguration()
dbConfig.encryptionKey = EncryptionKey.password("secretpassword")
self.database = try Database(name: "my-database", config: dbConfig)
```