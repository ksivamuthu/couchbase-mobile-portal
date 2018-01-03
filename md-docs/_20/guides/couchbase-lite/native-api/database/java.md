---
---

⚠ Support in the current Developer Build is for Android only. The SDK cannot be used in Java applications.

## New Database

As the top-level entity in the API, new databases can be created using the `Database` class by passing in a name, configuration, or both. The following example creates a database using the `Database(name: String)` method.

```java
DatabaseConfiguration config = new DatabaseConfiguration(/* Android Context*/ context);
Database database = new Database("my-database", config);
```

Just as before, the database will be created in a default location. Alternatively, the `Database(name: Strings, config: DatabaseConfiguration?)` method can be used to provide specific options (the directory to create the database in, whether it is read-only etc.)

## Migrating from 1.x Databases

Databases that were created with Couchbase Mobile 1.2 or later can be read using the 2.0 API. Upon detecting it is a 1.x database file format, Couchbase Lite will automatically upgrade it to the new format used in 2.0. This feature is currently only available for the default storage type, SQLite (i.e not for ForestDB databases).

## Finding a Database File

When the application is running on the Android emulator, you can locate the application's data folder and access the database file by using the **adb** CLI tools. For example, to list the different databases on the emulator, you can run the following commands.

```bash
$ adb shell
$ su
$ cd /data/data/{APPLICATION_ID}/files
$ ls
```

The **adb pull** command can be used to pull a specific database to your host machine.

```bash
$ adb root
$ adb pull /data/data/{APPLICATION_ID}/files/{DATABASE_NAME}.cblite2 .
```

## Logging

The log messages are split into different domains (`LogDomains`) which can be tuned to different log levels. The following example enables `verbose` logging for the `replicator` and `query` domains.

```java
Database.setLogLevel(Database.LogDomain.REPLICATOR, Database.LogLevel.VERBOSE);
Database.setLogLevel(Database.LogDomain.QUERY, Database.LogLevel.VERBOSE);
```

## Singleton Pattern

The database instance must be used throughout the Couchbase Lite API to Create, Update, Delete and Query documents. Hence, the singleton pattern is useful to create a single instance of the `Database` object. The following example follows the Singleton Pattern in `Swift`.

```java
public class DataManager {
    private static DataManager sharedInstance;
    private Database database;

    public static DataManager instance(DatabaseConfiguration config) {
        if (sharedInstance == null && config != null)
            sharedInstance = new DataManager(config);
        return sharedInstance;
    }

    private DataManager(DatabaseConfiguration config) {
        try {
            database = new Database("dbname", config);
        } catch (CouchbaseLiteException e) {
            e.printStackTrace();
        }
    }
}
```

The database instance can then be access throughout the codebase using the class property: `DataManager.sharedInstance.database`.

##  Encryption

The following example demonstrates how to create a database with an encryption key (or open an existing one).

```java
DatabaseConfiguration config = new DatabaseConfiguration(/* Android Context*/ context);
config.setEncryptionKey(new EncryptionKey("secretpassword"));
Database database = new Database("my-database", config);
```