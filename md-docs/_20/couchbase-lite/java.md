---
---

<br />
⚠ Currently, Couchbase Lite 2.0 is available for Android only. The SDK cannot be used in Java applications.

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
		compile 'com.couchbase.lite:couchbase-lite-android:2.0.0-DB022'
	}
	```

## API References

[Java SDK API References](http://docs.couchbase.com/mobile/2.0/couchbase-lite-java/db021)

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
DatabaseConfiguration config = new DatabaseConfiguration(/* Android Context*/ context)
    .setEncryptionKey(new EncryptionKey("secretpassword"));
Database database = new Database("my-database", config);
```

### Migrating from 1.x Databases

Databases that were created with Couchbase Mobile 1.2 or later can be read using the 2.0 API. Upon detecting it is a 1.x database file format, Couchbase Lite will automatically upgrade it to the new format used in 2.0. This feature is currently only available for the default storage type (i.e not for ForestDB databases).

### Finding a Database File

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

### Loading a pre-built database

If your app needs to sync a lot of data initially, but that data is fairly static and won't change much, it can be a lot more efficient to bundle a database in your application and install it on the first launch. Even if some of the content changes on the server after you create the app, the app's first pull replication will bring the database up to date.

To use a prebuilt database, you need to set up the database, build the database into your app bundle as a resource, and install the database during the initial launch. After your app launches, it needs to check whether the database exists. If the database does not exist, the app should copy it from the app bundle using the [`Database.copy(File path, String name, DatabaseConfiguration config)`](http://docs.couchbase.com/mobile/2.0/couchbase-lite-java/db021/com/couchbase/lite/Database.html#copy-java.io.File-java.lang.String-com.couchbase.lite.DatabaseConfiguration-) method as shown below.

```swift
DatabaseConfiguration config = new DatabaseConfiguration(/* Android Context*/ context);
ZipUtils.unzip(getAsset("replacedb/android200-sqlite.cblite2.zip"), context.getFilesDir());
File path = new File(context.getFilesDir(), "android-sqlite");
try {
    Database.copy(path, "travel-sample", config);
} catch (CouchbaseLiteException e) {
    Log.e(TAG, "Could not load pre-built database");
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
    Log.e(TAG, e.toString());
}
```

### Mutability

By default, when a document is read from the database it is immutable. The `document.toMutable()` method should be used to create an instance of the document which can be updated.

```java
// newTask is a MutableDocument
newTask.setString("name", "apples");
try {
    database.save(newTask);
} catch (CouchbaseLiteException e) {
    Log.e(TAG, e.toString());
}
```

Changes to the document are persisted to the database when the `saveDocument` method is called.

### Typed Accessors

The `Document` class now offers a set of [`property accessors`](http://docs.couchbase.com/mobile/2.0/couchbase-lite-java/db021/com/couchbase/lite/Dictionary.html) for various scalar types, including boolean, integers, floating-point and strings. These accessors take care of converting to/from JSON encoding, and make sure you get the type you're expecting: for example, `document.string(forKey: String)` returns either a `String` or `nil`, so you can't get an unexpected object class and crash trying to use it as a string. (Even if the property in the document has an incompatible type, the accessor returns `nil`.)

In addition, as a convenience we offer `Date` accessors. Dates are a common data type, but JSON doesn't natively support them, so the convention is to store them as strings in ISO-8601 format. The following example sets the date on the `createdAt` property and reads it back using the `document.date(forKey: String)` accessor method.

```java
newTask.setValue("createdAt", new Date());
Date date = newTask.getDate("createdAt");
```

### Batch operations

If you're making multiple changes to a database at once, it's faster to group them together. The following example persists a few documents in batch.

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

### Blobs

We've renamed "attachments" to "blobs", for clarity. The new behavior should be clearer too: a `Blob` is now a normal object that can appear in a document as a property value. In other words, there's no special API for creating or accessing attachments; you just instantiate an `Blob` and set it as the value of a property, and then later you can get the property value, which will be a `Blob` object. The following code example adds a blob to the document under the `avatar` property.

```java
InputStream is = getAsset("attachment.png");
try {
    MutableDocument newTask = new MutableDocument();
    Blob blob = new Blob("image/png", is);
    newTask.setBlob("attachment", blob);
    Document doc = database.save(newTask);

    Blob taskBlob = doc.getBlob("attachment");
    byte[] bytes = taskBlob.getContent();
} catch (CouchbaseLiteException e) {
    Log.e(TAG, e.toString());
} finally {
    try {
        is.close();
    } catch (IOException e) {
    }
}
```

`Blob` itself has a simple API that lets you access the contents as in-memory data (a `Data` object) or as a `InputStream`. It also supports an optional `type` property that by convention stores the MIME type of the contents. Unlike attachments, blobs don't have names; if you need to associate a name you can put it in another document property, or make the filename be the property name (e.g. `document.set(imageBlob, forKey: "thumbnail.jpg")`)

> **Note:** A blob is stored in the document's raw JSON as an object with a property `"@type":"blob"`. It also has properties such as `"digest"` (a SHA-1 digest of the data), `"length"` (the length in bytes), and optionally `"content_type"` (the MIME type.) As always, the data is not stored in the document, but in a separate content-addressable store, indexed by the digest.

## Query

Database queries have changed significantly. Instead of the map/reduce algorithm used in 1.x, they're now based on expressions, of the form "return ____ from documents where \_\_\_\_, ordered by \_\_\_\_", with semantics based on Couchbase Server's N1QL query language.

There are several parts to specifying a query:

- SELECT: specifies the projection, which is the part of the document that is to be returned.
- FROM: specifies the database to query the documents from.
- JOIN: specifies the matching criteria in which to join multiple documents.
- WHERE: specifies the query criteria that the result must satisfy.
- GROUP BY: specifies the query criteria to group rows by.
- ORDER BY: specifies the query criteria to sort the rows in the result.

### Indexing

Before we begin querying documents, let's briefly mention the importance of having a query index. A query can only be fast if there's a pre-existing database index it can search to narrow down the set of documents to examine.

The following example creates a new index for the `type` and `name` properties.

```json
{
    "_id": "hotel123",
    "type": "hotel",
    "name": "Apple Droid"
}
```

```java
database.createIndex("TypeNameIndex",
        IndexBuilder.valueIndex(ValueIndexItem.property("type"),
                ValueIndexItem.property("name")));
```

If there are multiple expressions, the first one will be the primary key, the second the secondary key, etc.

> **Note:** Every index has to be updated whenever a document is updated, so too many indexes can hurt performance. Thus, good performance depends on designing and creating the *right* indexes to go along with your queries.

### SELECT statement

With the SELECT statement, you can query and manipulate JSON data. With projections, you retrieve just the fields that you need and not the entire document. This is especially useful when querying for a large dataset as it results in shorter processing times and better performance.

A SelectResult represents a single return value of the query statement. Documents in Couchbase Lite comprise of the document properties specified as a Dictionary of Key-Value pairs and associated metadata. The metadata consists of document Id and sequence Id associated with the Document. When you query for a document, the document metadata is not returned by default. You will need to explicitly query for the metadata.

- [`SelectResult.all()`](http://docs.couchbase.com/mobile/2.0/couchbase-lite-java/db021/com/couchbase/lite/SelectResult.html#all--): Returns all properties associated with a document.
- `SelectResult(`[`Expression`](http://docs.couchbase.com/mobile/2.0/couchbase-lite-java/db021/index.html?com/couchbase/lite/Dictionary.html)`.property("name"))`: Returns the `name` property associated with a document.
- `SelectResult.expression(`[`Meta`](http://docs.couchbase.com/mobile/2.0/couchbase-lite-java/db021/index.html?com/couchbase/lite/Dictionary.html)`.id`)`: Returns the document ID.
- `SelectResult.expression(Expression.meta().sequence)`: Returns the sequence ID (used in replications).

You can specify a comma separated list of `SelectResult` expressions in the select statement of your query. For instance the following select statement queries for the document `_id` as well as the `type` and `name` properties of all documents in the database. In the query result, we print the `_id` and `name` properties of each row using the property name getter method.

```json
{
	"_id": "hotel123",
	"type": "hotel",
	"name": "Apple Droid"
}
```

```java
Query query = QueryBuilder
        .select(SelectResult.expression(Meta.id),
                SelectResult.property("name"),
                SelectResult.property("type"))
        .from(DataSource.database(database))
        .where(Expression.property("type").equalTo(Expression.string("hotel")))
        .orderBy(Ordering.expression(Meta.id));

try {
    ResultSet rs = query.execute();
    for (Result result : rs) {
        Log.i("Sample", String.format("hotel id -> %s", result.getString("id")));
        Log.i("Sample", String.format("hotel name -> %s", result.getString("name")));
    }
} catch (CouchbaseLiteException e) {
    Log.e("Sample", e.getLocalizedMessage());
}
```

The `SelectResult.all()` method can be used to query all the properties of a document. In this case, the document in the result is embedded in a dictionary where the key is the database name. The following snippet shows the same query using `SelectResult.all()` and the result in JSON.

```java
Query query = QueryBuilder
        .select(SelectResult.all())
        .from(DataSource.database(database))
        .where(Expression.property("type").equalTo(Expression.string("hotel")));
ResultSet rs = query.execute();
for (Result result : rs)
    Log.i("Sample", String.format("hotel -> %s", result.getDictionary(DATABASE_NAME).toMap()));
```

```json
[
	{
		"travel-sample": {
			"callsign": "MILE-AIR",
			"country": "United States",
			"iata": "Q5",
			"icao": "MLA",
			"id": 10,
			"name": "40-Mile Air",
			"type": "airline"
		}
	},
	{
		"travel-sample": {
			"callsign": "TXW",
			"country": "United States",
			"iata": "TQ",
			"icao": "TXW",
			"id": 10123,
			"name": "Texas Wings",
			"type": "airline"
		}
	}
]
```

### WHERE statement

Similar to SQL, you can use the where clause to filter the documents to be returned as part of the query. The select statement takes in an `Expression`. You can chain any number of Expressions in order to implement sophisticated filtering capabilities.

#### Comparison

The `Expression`'s [comparison operators](http://docs.couchbase.com/mobile/2.0/couchbase-lite-java/db021/Classes/Expression.html#/Comparison%20Operators) can be used in the WHERE statement to specify on which property to match documents. In the example below, we use the `equalTo` operator to query documents where the `type` property equals "hotel".

```json
{
	"_id": "hotel123",
	"type": "hotel",
	"name": "Apple Droid"
}
```

```java
Query query = QueryBuilder
        .select(SelectResult.all())
        .from(DataSource.database(database))
        .where(Expression.property("type").equalTo(Expression.string("hotel")))
        .limit(Expression.intValue(10));
ResultSet rs = query.execute();
for (Result result : rs) {
    Dictionary all = result.getDictionary(DATABASE_NAME);
    Log.i("Sample", String.format("name -> %s", all.getString("name")));
    Log.i("Sample", String.format("type -> %s", all.getString("type")));
}
```

#### Collection Operators

[Collection operators](http://docs.couchbase.com/mobile/2.0/couchbase-lite-java/db021/Classes/Expression.html#/Collection%20operators:) are useful to check if a given value is present in an array. The following example uses the `Function.arrayContains` to find documents whose `public_likes` array property contain a value equal to "Armani Langworth".

```json
{
	"_id": "hotel123",
	"name": "Apple Droid",
	"public_likes": ["Armani Langworth", "Elfrieda Gutkowski", "Maureen Ruecker"]
}
```

```java
Query query = QueryBuilder
        .select(SelectResult.expression(Meta.id),
                SelectResult.property("name"),
                SelectResult.property("public_likes"))
        .from(DataSource.database(database))
        .where(Expression.property("type").equalTo(Expression.string("hotel"))
                .and(ArrayFunction.contains(Expression.property("public_likes"), Expression.string("Armani Langworth"))));
ResultSet rs = query.execute();
for (Result result : rs)
    Log.i("Sample", String.format("public_likes -> %s", result.getArray("public_likes").toList()));
```

#### Like Operator

The [`like`](http://docs.couchbase.com/mobile/2.0/couchbase-lite-java/db021/com/couchbase/lite/Expression.html#like-java.lang.Object-) operator can be used for string matching. It is recommended to use the `like` operator for case insensitive matches and the `regex` operator (see below) for case sensitive matches.

In the example below, we are looking for documents of type `landmark` where the name property exactly matches the string "Royal engineers museum". Note that since `like` does a case insensitive match, the following query will return "landmark" type documents with name matching "Royal Engineers Museum", "royal engineers museum", "ROYAL ENGINEERS MUSEUM" and so on.

```java
Query query = QueryBuilder
        .select(SelectResult.expression(Meta.id),
                SelectResult.property("country"),
                SelectResult.property("name"))
        .from(DataSource.database(database))
        .where(Expression.property("type").equalTo(Expression.string("landmark"))
                .and(Expression.property("name").like(Expression.string("Royal Engineers Museum"))));
ResultSet rs = query.execute();
for (Result result : rs)
    Log.i("Sample", String.format("name -> %s", result.getString("name")));
```

#### Wildcard Match

We can use `%` sign within a `like` expression to do a wildcard match against zero or more characters. Using wildcards allows you to have some fuzziness in your search string.

In the example below, we are looking for documents of `type` "landmark" where the name property matches any string that begins with "eng" followed by zero or more characters, the letter "e", followed by zero or more characters. The following query will return "landmark" type documents with name matching "Engineers", "engine", "english egg" , "England Eagle" and so on. Notice that the matches may span word boundaries.

```java
Query query = QueryBuilder
        .select(SelectResult.expression(Meta.id),
                SelectResult.property("country"),
                SelectResult.property("name"))
        .from(DataSource.database(database))
        .where(Expression.property("type").equalTo(Expression.string("landmark"))
                .and(Expression.property("name").like(Expression.string("Eng%e%"))));
ResultSet rs = query.execute();
for (Result result : rs)
    Log.i("Sample", String.format("name -> %s", result.getString("name")));
```

##### Wildcard Character Match

We can use `_` sign within a like expression to do a wildcard match against a single character.

In the example below, we are looking for documents of type "landmark" where the `name` property matches any string that begins with "eng" followed by exactly 4 wildcard characters and ending in the letter "r".
The following query will return "landmark" `type` documents with the `name` matching "Engineer", "engineer" and so on.

```java
Query query = QueryBuilder
        .select(SelectResult.expression(Meta.id),
                SelectResult.property("country"),
                SelectResult.property("name"))
        .from(DataSource.database(database))
        .where(Expression.property("type").equalTo(Expression.string("landmark"))
                .and(Expression.property("name").like(Expression.string("Eng____r"))));
ResultSet rs = query.execute();
for (Result result : rs)
    Log.i("Sample", String.format("name -> %s", result.getString("name")));
```

#### Regex Match

The `regex` expression can be used for case sensitive matches. Similar to wildcard `like` expressions, `regex` expressions based pattern matching allow you to have some fuzziness in your search string.

In the example below, we are looking for documents of `type` "landmark" where the name property matches any string (on word boundaries) that begins with "eng" followed by exactly 4 wildcard characters and ending in the letter "r".
The following query will return "landmark" type documents with name matching "Engine", "engine" and so on. Note that the `\b` specifies that the match must occur on word boundaries.

```java
Query query = QueryBuilder
        .select(SelectResult.expression(Meta.id),
                SelectResult.property("country"),
                SelectResult.property("name"))
        .from(DataSource.database(database))
        .where(Expression.property("type").equalTo(Expression.string("landmark"))
                .and(Expression.property("name").regex(Expression.string("\\bEng.*r\\b"))));
ResultSet rs = query.execute();
for (Result result : rs)
    Log.i("Sample", String.format("name -> %s", result.getString("name")));
```

### JOIN statement

The JOIN clause enables you to create new input objects by combining two or more source objects.

The following example uses a JOIN clause to find the airline details which have routes that start from RIX. This example JOINS the document of type "route" with documents of type "airline" using the document ID (`_id`) on the "airline" document and  `airlineid` on the "route" document.

```java
Query query = QueryBuilder.select(
        SelectResult.expression(Expression.property("name").from("airline")),
        SelectResult.expression(Expression.property("callsign").from("airline")),
        SelectResult.expression(Expression.property("destinationairport").from("route")),
        SelectResult.expression(Expression.property("stops").from("route")),
        SelectResult.expression(Expression.property("airline").from("route")))
        .from(DataSource.database(database).as("airline"))
        .join(Join.join(DataSource.database(database).as("route"))
                .on(Meta.id.from("airline").equalTo(Expression.property("airlineid").from("route"))))
        .where(Expression.property("type").from("route").equalTo(Expression.string("route"))
                .and(Expression.property("type").from("airline").equalTo(Expression.string("airline")))
                .and(Expression.property("sourceairport").from("route").equalTo(Expression.string("RIX"))));
ResultSet rs = query.execute();
for (Result result : rs)
    Log.w("Sample", String.format("%s", result.toMap().toString()));
```

### GROUP BY statement

You can perform further processing on the data in your result set before the final projection is generated. The following example looks for the number of airports at an altitude of 300 ft or higher and groups the results by country and timezone.

```json
{
	"_id": "airport123",
	"type": "airport",
	"country": "United States",
	"geo": { "alt": 456 },
	"tz": "America/Anchorage"
}
```

```java
Query query = QueryBuilder.select(
        SelectResult.expression(Function.count(Expression.string("*"))),
        SelectResult.property("country"),
        SelectResult.property("tz"))
        .from(DataSource.database(database))
        .where(Expression.property("type").equalTo(Expression.string("airport"))
                .and(Expression.property("geo.alt").greaterThanOrEqualTo(Expression.intValue(300))))
        .groupBy(Expression.property("country"),
                Expression.property("tz"))
        .orderBy(Ordering.expression(Function.count(Expression.string("*"))).descending());
ResultSet rs = query.execute();
for (Result result : rs)
    Log.i("Sample",
            String.format("There are %d airports on the %s timezone located in %s and above 300ft",
                    result.getInt("$1"),
                    result.getString("tz"),
                    result.getString("country")));
```

```text
There are 138 airports on the Europe/Paris timezone located in France and above 300 ft
There are 29 airports on the Europe/London timezone located in United Kingdom and above 300 ft
There are 50 airports on the America/Anchorage timezone located in United States and above 300 ft
There are 279 airports on the America/Chicago timezone located in United States and above 300 ft
There are 123 airports on the America/Denver timezone located in United States and above 300 ft
```

### ORDER BY statement

It is possible to sort the results of a query based on a given expression result. The example below returns documents of type equal to "hotel" sorted in ascending order by the value of the title property.

```java
Query query = QueryBuilder
        .select(SelectResult.expression(Meta.id),
                SelectResult.property("name"))
        .from(DataSource.database(database))
        .where(Expression.property("type").equalTo(Expression.string("hotel")))
        .orderBy(Ordering.property("name").ascending())
        .limit(Expression.intValue(10));
ResultSet rs = query.execute();
for (Result result : rs)
    Log.i("Sample", String.format("%s", result.toMap()));
```

```text
Aberdyfi
Achiltibuie
Altrincham
Ambleside
Annan
Ardèche
Armagh
Avignon
```

## Full-Text Search

To run a full-text search (FTS) query, you must have created a full-text index on the expression being matched. Unlike regular queries, the index is not optional. The following example inserts documents and creates an FTS index on the `name` property.

```java
// Insert documents
String[] tasks = {"buy groceries", "play chess", "book travels", "buy museum tickets"};
for (String task : tasks) {
    MutableDocument doc = new MutableDocument();
    doc.setString("name", task);
    doc.setString("type", "task");
    database.save(doc);
}

// Create index
database.createIndex("nameFTSIndex", 
	IndexBuilder.fullTextIndex(FullTextIndexItem.property("name")).ignoreAccents(false));
```

Multiple properties to index can be specified in the `Index.fullTextIndex(withItems: [FullTextIndexItem])` method.

With the index created, an FTS query on the property that is being indexed can be constructed and ran. The full-text search criteria is defined as a `FullTextExpression`. The left-hand side is the full-text index to use and the right-hand side is the pattern to match.

```java
Expression whereClause = FullTextExpression.index("nameFTSIndex").match("buy");
Query ftsQuery = QueryBuilder.select(SelectResult.expression(Meta.id))
        .from(DataSource.database(database))
        .where(whereClause);
ResultSet ftsQueryResult = ftsQuery.execute();
for (Result result : ftsQueryResult)
    Log.i(TAG, String.format("document properties %s", result.getString(0)));
```

In the example above, the pattern to  match is a word, the full-text search query matches all documents that contain the word 'buy' in the value of the `doc.name` property.

Full-text search is supported in the following languages: danish, dutch, english, finnish, french, german, hungarian, italian, norwegian, portuguese, romanian, russian, spanish, swedish and turkish.

The pattern to match can also be in the following forms:

- **prefix queries:** the query expression used to search for a term prefix is the prefix itself with a '*' character appended to it. For example:

	```bash
	"'lin*'"
	-- Query for all documents containing a term with the prefix "lin". This will match
	-- all documents that contain "linux", but also those that contain terms "linear",
	--"linker", "linguistic" and so on.
	```

- **overriding the property name that is being indexed:** Normally, a token or token prefix query is matched against the document property specified as the left-hand side of the `match` operator. This may be overridden by specifying a property name followed by a ":" character before a basic term query. There may be space between the ":" and the term to query for, but not between the property name and the ":" character. For example:

	```bash
	'title:linux problems'
	-- Query the database for documents for which the term "linux" appears in
	-- the document title, and the term "problems" appears in either the title
	-- or body of the document.
	```

- **phrase queries:** a phrase query is a query that retrieves all documents that contain a nominated set of terms or term prefixes in a specified order with no intervening tokens. Phrase queries are specified by enclosing a space separated sequence of terms or term prefixes in double quotes ("). For example:

	```bash
	"'"linux applications"'"
	-- Query for all documents that contain the phrase "linux applications".
	```

- **NEAR queries:** A NEAR query is a query that returns documents that contain a two or more nominated terms or phrases within a specified proximity of each other (by default with 10 or less intervening terms). A NEAR query is specified by putting the keyword "NEAR" between two phrase, token or token prefix queries. To specify a proximity other than the default, an operator of the form "NEAR/<N>" may be used, where <N> is the maximum number of intervening terms allowed. For example:

	```bash
	"'database NEAR/2 "replication"'"
	-- Search for a document that contains the phrase "replication" and the term
  -- "database" with not more than 2 terms separating the two.
	```

- **AND, OR & NOT query operators:** The enhanced query syntax supports the AND, OR and NOT binary set operators. Each of the two operands to an operator may be a basic FTS query, or the result of another AND, OR or NOT set operation. Operators must be entered using capital letters. Otherwise, they are interpreted as basic term queries instead of set operators. For example:

	```bash
	'couchbase AND database'
	-- Return the set of documents that contain the term "couchbase", and the
	-- term "database". This query will return the document with docid 3 only.
	```

	When using the enhanced query syntax, parenthesis may be used to specify the precedence of the various operators. For example:

	```bash
	'("couchbase database" OR "sqlite library") AND linux'
	-- Query for the set of documents that contains the term "linux", and at least
	-- one of the phrases "couchbase database" and "sqlite library".
	```

### Ordering results

It's very common to sort full-text results in descending order of relevance. This can be a very difficult heuristic to define, but Couchbase Lite comes with a fairly simple ranking function you can use. In the `OrderBy` array, use a string of the form `Rank(X)`, where `X` is the property or expression being searched, to represent the ranking of the result.

## Replication

Couchbase Mobile 2.0 uses a new replication protocol based on WebSockets. This protocol has been designed to be fast, efficient, easier to implement, and symmetrical between the client and server.

### Compatibility

⚠️ The new protocol is **incompatible** with CouchDB-based databases. And since Couchbase Lite 2 only supports the new protocol, you will need to run a version of Sync Gateway that supports it.

To use this protocol with Couchbase Lite 2.0, the replication URL should specify WebSockets as the URL scheme (see the "Starting a Replication" section below). Mobile clients using Couchbase Lite 1.x can continue to use **http** as the URL scheme. Sync Gateway 2.0 will automatically use the 1.x replication protocol when a Couchbase Lite 1.x client connects through "http://localhost:4984/db" and the 2.0 replication protocol when a Couchbase Lite 2.0 client connects through "ws://localhost:4984/db".

### Starting Sync Gateway

To run an example, create a new file named **sync-gateway-config.json** with the following.

```javascript
{
  "databases": {
    "db": {
      "server":"walrus:",
      "users": {
        "GUEST": {"disabled": false, "admin_channels": ["*"]}
      },
      "unsupported": {
        "replicator_2":true
      }
    }
  }
}
```

In the configuration file above, the **replicator_2** property enables the new replication protocol.

[Download Sync Gateway](https://www.couchbase.com/downloads) and start it from the command line with the configuration file created above.

```bash
~/Downloads/couchbase-sync-gateway/bin/sync_gateway sync-gateway-config.json
```

For platform specific installation instructions, refer to the Sync Gateway [installation guide](https://developer.couchbase.com/documentation/mobile/1.5/installation/sync-gateway/index.html).

### Starting a Replication

Replication objects are now bidirectional, this means you can start a `push`/`pull` replication with a single instance. The replication's parameters can be specified through the [`ReplicatorConfiguration`](http://docs.couchbase.com/mobile/2.0/couchbase-lite-java/db021/index.html?com/couchbase/lite/ReplicatorConfiguration.html) object; for example, if you wish to start a `push` only or `pull` only replication. The following example creates a `pull` only replication instance with Sync Gateway.

```java
URI uri = new URI("ws://localhost:4984/db");
Endpoint endpoint = new URLEndpoint(uri);
ReplicatorConfiguration config = new ReplicatorConfiguration(database, endpoint);
config.setReplicatorType(ReplicatorConfiguration.ReplicatorType.PULL);
Replicator replication = new Replicator(config);
replication.start();
```

As shown in the code snippet above, the URL scheme for remote database URLs has changed in Couchbase Lite 2.0. You should now use `ws:`, or `wss:` for SSL/TLS connections. You can access the Sync Gateway `_all_docs` endpoint [http://localhost:4984/db/\_all\_docs?include_docs=true](http://localhost:4984/db/_all_docs?include_docs=true) to check that the documents are successfully replicated.

Couchbase Lite 2.0 uses WebSockets as the communication protocol to transmit data. Some load balancers are not configured for WebSocket connections by default (NGINX for example); so it might be necessary to explicitly enable them in the load balancer's configuration (see [Load Balancers](https://developer.couchbase.com/documentation/mobile/1.5/guides/sync-gateway/nginx/index.html)).

Starting in Couchbase Lite 2.0, replication between two local databases is now supported. This isn't often needed, but it can be very useful. For example, you can implement incremental backup by pushing your main database to a mirror on a backup disk.

### Troubleshooting

As always, when there is a problem with replication, logging is your friend. The following example increases the log output for activity related to replication with Sync Gateway.

```java
Database.setLogLevel(Database.LogDomain.REPLICATOR, Database.LogLevel.VERBOSE);
```

### Replication Status

The `replication.Status.Activity` property can be used to check the status of a replication. For example, when the replication is actively transferring data and when it has stopped.

```java
replication.addChangeListener(new ReplicatorChangeListener() {
    @Override
    public void changed(ReplicatorChange change) {
        if (change.getStatus().getActivityLevel() == Replicator.ActivityLevel.STOPPED)
            Log.i(TAG, "Replication stopped");
    }
});
```

The following table lists the different activity levels in the API and the meaning of each one.

|State|Meaning|
|:----|:------|
|`STOPPED`|The replication is finished or hit a fatal error.|
|`OFFLINE`|The replicator is offline as the remote host is unreachable.|
|`CONNECTING`|The replicator is connecting to the remote host.|
|`IDLE`|The replication caught up with all the changes available from the server. The `IDLE` state is only used in continuous replications.|
|`BUSY`|The replication is actively transferring data.|

### Handling Network Errors

A running replication can be interrupted for a variety of reasons such as network errors or unauthorized access. In this case, the replication status will be updated with an `Error` which follows the standard HTTP error codes. The replication change event can be used to monitor the status of the replication. The following example monitors the replication for errors and logs the error code to the console.

```java
replication.addChangeListener(new ReplicatorChangeListener() {
    @Override
    public void changed(ReplicatorChange change) {
        CouchbaseLiteException error = change.getStatus().getError();
        if (error != null)
            Log.w(TAG, "Error code:: %d", error.getCode());
    }
});
replication.start();
```

### Certificate Pinning

Couchbase Lite supports certificate pinning. Certificate pinning is a technique that can be used by applications to "pin" a host to it's certificate. The certificate is typically delivered to the client by an out-of-band channel and bundled with the client. In this case, Couchbase Lite uses this embedded certificate to verify the trustworthiness of the server and no longer needs to rely on a trusted third party for that (commonly referred to as the Certificate Authority).

The `openssl` command can be used to create a new self-signed certificate and convert the `.pem` file to a `.cert` file (see [creating your own self-signed certificate](https://developer.couchbase.com/documentation/mobile/1.5/guides/sync-gateway/configuring-ssl/index.html#creating-your-own-self-signed-certificate)). You should then have 3 files: `cert.pem`, `cert.cer` and `key.pem`.

The `cert.pem` and `key.pem` can be used in the Sync Gateway configuration (see [installing the certificate](https://developer.couchbase.com/documentation/mobile/1.5/guides/sync-gateway/configuring-ssl/index.html#installing-the-certificate)).

On the Couchbase Lite side, the replication must be configured with the `cert.cer` file.

```java
InputStream is = getAsset("cert.cer");
byte[] cert = IOUtils.toByteArray(is);
is.close();
config.setPinnedServerCertificate(cert);
```

This example loads the certificate from the application sandbox, then converts it to the appropriate type to configure the replication object.
