---
permalink: guides/couchbase-lite/java/index.html
---

<br />
⚠ Support in the current Developer Build is for Android only. The SDK cannot be used in Java applications.

## Getting Started

- In the top-level **build.gradle** file, add the following Maven repository in the `allprojects` section.

	```groovy
	allprojects {
		repositories {
			jcenter()
			maven {
				url "http://mobile.maven.couchbase.com/maven2/dev/"
			}
		}
	}
	```

- Next, add the following in the `dependencies` section of the application's **build.gradle** (the one in the **app** folder).

	```groovy
	dependencies {
		compile 'com.couchbase.lite:couchbase-lite-android:2.0.0-DB021'
	}
	```

### Resources

The API references for the Java SDK are available [here](http://docs.couchbase.com/mobile/2.0/couchbase-lite-java/db021).

## Database

### New Database

As the top-level entity in the API, new databases can be created using the `Database` class by passing in a name, configuration, or both. The following example creates a database using the `Database(name: String)` method.

```java
DatabaseConfiguration config = new DatabaseConfiguration(/* Android Context*/ context);
Database database = new Database("my-database", config);
```

Just as before, the database will be created in a default location. Alternatively, the `Database(string name, DatabaseConfiguration config)` initializer can be used to provide specific options in the [`DatabaseConfiguration`](http://docs.couchbase.com/mobile/2.0/couchbase-lite-java/db021/com/couchbase/lite/DatabaseConfiguration.html) object such as the database directory, encryption key through the  object.

###  Encryption

The following example demonstrates how to create a database with an encryption key (or open an existing one).

```java
DatabaseConfiguration config = new DatabaseConfiguration(/* Android Context*/ context);
config.setEncryptionKey(new EncryptionKey("secretpassword"));
Database database = new Database("my-database", config);
```

### Migrating from 1.x Databases

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

### Logging

The log messages are split into different domains (`LogDomains`) which can be tuned to different log levels. The following example enables `verbose` logging for the `replicator` and `query` domains.

```java
Database.setLogLevel(Database.LogDomain.REPLICATOR, Database.LogLevel.VERBOSE);
Database.setLogLevel(Database.LogDomain.QUERY, Database.LogLevel.VERBOSE);
```

### Singleton Pattern

The database instance must be used throughout the Couchbase Lite API to Create, Update, Delete and Query documents. Hence, the singleton pattern is useful to create a single instance of the `Database` object. The following example follows the Singleton Pattern in `Java`.

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

### Loading a pre-built database

If your app needs to sync a lot of data initially, but that data is fairly static and won't change much, it can be a lot more efficient to bundle a database in your application and install it on the first launch. Even if some of the content changes on the server after you create the app, the app's first pull replication will bring the database up to date.

To use a prebuilt database, you need to set up the database, build the database into your app bundle as a resource, and install the database during the initial launch. After your app launches, it needs to check whether the database exists. If the database does not exist, the app should copy it from the app bundle using the [`Database.copy(File path, String name, DatabaseConfiguration config)`](http://docs.couchbase.com/mobile/2.0/couchbase-lite-java/db021/com/couchbase/lite/Database.html#copy-java.io.File-java.lang.String-com.couchbase.lite.DatabaseConfiguration-) method as shown below.

```swift
let assetPath = Bundle.main.path(forResource: "travel-sample", ofType: "cblite2")!
if !Database.exists(withName: "travel-sample") {
	do {
		try Database.copy(fromPath: assetPath, toDatabase: "travel-sample", withConfig: nil)
	} catch {
		fatalError("Could not load pre-built database")
	}
}
```

## Document

In Couchbase Lite, a document's body takes the form of a JSON object — a collection of key/value pairs where the values can be different types of data such as numbers, strings, arrays or even nested objects. Every document is identified by a document ID, which can be automatically generated (as a UUID) or determined by the application; the only constraints are that it must be unique within the database, and it can't be changed.

### Initializers

The following methods/initializers can be used:

- The `MutableDocument()` initializer can be used to create a new document where the document ID is randomly generated by the database.
- The `MutableDocument(withID: String)` initializer can be used to create a new document with a specific ID.
- The `database.document(withID: String)` method can be used to  get a document. If it doesn't exist in the database, it will return `nil`. This method can be used to check if a document with a given ID already exists in the database.

The following code example creates a document and persists it to the database.

```java
Map<String, Object> dict = new HashMap<>();
dict.put("type", "task");
dict.put("owner", "todo");
dict.put("createdAt", new Date());
MutableDocument newTask = new MutableDocument(dict);
try {
    database.save(newTask);
} catch (CouchbaseLiteException e) {
    Log.e(TAG, "Failed to save the document", e);
}
```

### Mutability

The biggest change is that `MutableDocument` properties are now directly mutable. Instead of having to make a mutable copy of the properties dictionary, update it, and then save it back to the document, you can now modify individual properties in place and then save.

```java
// newTask is a MutableDocument
newTask.setString("name", "apples");
try {
    database.save(newTask);
} catch (CouchbaseLiteException e) {
    Log.e(TAG, "Failed to save the document", e);
}
```

This does create the possibility of confusion, since the document's in-memory state may not match what's in the database. Unsaved changes are not visible to other `Database` instances (i.e. other threads that may have other instances), or to queries.

### Typed Accessors

The `Document` class now offers a set of [`property accessors`](http://docs.couchbase.com/mobile/2.0/couchbase-lite-java/db021/com/couchbase/lite/Dictionary.html) for various scalar types, including boolean, integers, floating-point and strings. These accessors take care of converting to/from JSON encoding, and make sure you get the type you're expecting: for example, `document.string(forKey: String)` returns either a `String` or `nil`, so you can't get an unexpected object class and crash trying to use it as a string. (Even if the property in the document has an incompatible type, the accessor returns `nil`.)

In addition, as a convenience we offer `Date` accessors. Dates are a common data type, but JSON doesn't natively support them, so the convention is to store them as strings in ISO-8601 format. The following example sets the date on the `createdAt` property and reads it back using the `document.date(forKey: String)` accessor method.

```java
newTask.setValue("createdAt", new Date());
Date date = newTask.getDate("createdAt");
```

### Batch operations

If you're making multiple changes to a database at once, it's *much* faster to group them together, otherwise each individual change incurs overhead, from flushing writes to the filesystem to ensure durability. In 2.0 we've renamed the method from `inTransaction()` to `inBatch()` to emphasize that Couchbase Lite does not offer transactional guarantees, and that the purpose of the method is to optimize batch operations rather than to enable ACID transactions. The following example persists a few documents in batch.

```java
try {
    database.inBatch(new Runnable() {
        @Override
        public void run() {
            for (int i = 0; i < 10; i++) {
                MutableDocument doc = new MutableDocument();
                doc.setValue("type", "user");
                doc.setValue("name", String.format("user %d", i));
                doc.setBoolean("admin", false);
                try {
                    database.save(doc);
                } catch (CouchbaseLiteException e) {
                    Log.e(TAG, e.toString());
                }
                Log.i(TAG, String.format("saved user document %s", doc.getString("name")));
            }
        }
    });
} catch (CouchbaseLiteException e) {
    Log.e(TAG, e.toString());
}
```

At the **local** level this operation is still transactional: no other `Database` instances, including ones managed by the replicator can make changes during the execution of the block, and other instances will not see partial changes. But Couchbase Mobile is a distributed system, and due to the way replication works, there's no guarantee that Sync Gateway or other devices will receive your changes all at once.
