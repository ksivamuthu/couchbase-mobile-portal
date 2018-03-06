---
---

## Getting Started

### Xcode project

Create or open an existing Xcode project and install Couchbase Lite using one of the following methods.

{% include install-tabs.html %}

{% include_relative installation/swift.md %}

### Starter code

Open **ViewController.swift** in Xcode and copy [the following](https://github.com/couchbaselabs/couchbase-mobile-portal/blob/master/md-docs/_20/example.swift) code in the `viewDidLoad` method. This snippet demonstrates how to run basic CRUD operations, a simple Query and running bi-directional replications with Sync Gateway.

Build and run. You should see the document ID and property printed to the console. The document was successfully persisted to the database.

![](img/getting-started-ios.png)

Before synchronizing documents to Sync Gateway you will need to disable App Transport Security. In the Xcode navigator, right-click on **Info.plist** and open it as a source file.

<img src="img/info-plist.png" class=center-image />

Append the following inside of the `<dict>` XML tags to disable ATS.

```xml
<key>NSAppTransportSecurity</key>
<dict>
<key>NSAllowsArbitraryLoads</key><true/>
</dict>
```

## Supported Versions

| Platform         | Minimum OS version |
| ---------------- | ------------------ |
| iOS              | 10.3.1             |
| tvOS             | 10.0.1             |
| macOS            | 10.9               |

## API References

[Swift SDK API References](http://docs.couchbase.com/mobile/2.0/couchbase-lite-swift/db022)

## Database

### New Database

As the top-level entity in the API, new databases can be created using the `Database` class by passing in a name, configuration, or both. The following example creates a database using the `Database(name: String)` method.

```swift
do {
  let database = try Database(name: "my-database")
} catch {
  print(error)
}
```

Just as before, the database will be created in a default location. Alternatively, the `Database(name: Strings, config: DatabaseConfiguration?)` initializer can be used to provide specific options in the [`DatabaseConfiguration`](http://docs.couchbase.com/mobile/2.0/couchbase-lite-swift/db022/Classes/DatabaseConfiguration.html) object such as the database directory, encryption key through the  object.

###  Encryption

Encryption is available in the **Enterprise Edition** only. The following example demonstrates how to create a database with an encryption key (or open an existing one). Note that this code won't compile if you're running the **Community Edition** of Couchbase Lite.

```swift
let config = DatabaseConfiguration()
config.encryptionKey = EncryptionKey.password("secretpassword")
self.database = try Database(name: "my-database", config: config)
```

### Migrating from 1.x Databases

Databases that were created with Couchbase Mobile 1.2 or later can be read using the 2.0 API. Upon detecting it is a 1.x database file format, Couchbase Lite will automatically upgrade it to the new format used in 2.0. This feature is only available for the default storage type (i.e not for ForestDB databases). Additionally, the automatic migration feature does not support encrypted database. Database encryption is an **Enterprise Edition** feature. If the 1.x database is encrypted, you will first need to disable encryption using the Couchbase Lite 1.x SDK (see the [1.x Database Guide](https://developer.couchbase.com/documentation/mobile/1.5/guides/couchbase-lite/native-api/database/index.html#step-2-enabling-encryption)).

### Finding a Database File

When the application is running on the iOS simulator, you can easily locate the application's sandbox directory using the [SimPholders](https://simpholders.com/3/) utility.

### CLI tool

The Couchbase Lite `.zip` file available from the [downloads page](https://www.couchbase.com/downloads) contains a CLI which can be used on **.cblite2** files.

### Logging

The log messages are split into different domains (`LogDomains`) which can be tuned to different log levels. The following example enables `verbose` logging for the `replicator` and `query` domains.

```swift
Database.setLogLevel(.verbose, domain: .replicator)
Database.setLogLevel(.verbose, domain: .query)
```

### Loading a pre-built database

If your app needs to sync a lot of data initially, but that data is fairly static and won't change much, it can be a lot more efficient to bundle a database in your application and install it on the first launch. Even if some of the content changes on the server after you create the app, the app's first pull replication will bring the database up to date.

To use a prebuilt database, you need to set up the database, build the database into your app bundle as a resource, and install the database during the initial launch. After your app launches, it needs to check whether the database exists. If the database does not exist, the app should copy it from the app bundle using the [`Database.copy(fromPath:toDatabase:withConfig:)`](http://docs.couchbase.com/mobile/2.0/couchbase-lite-swift/db022/Classes/Database.html#/s:18CouchbaseLiteSwift8DatabaseC4copyySS8fromPath_SS02toD0AA0D13ConfigurationVSg10withConfigtKFZ) method as shown below.

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

```swift
let newTask = MutableDocument()
		.setString("task", forKey: "type")
		.setString("todo", forKey: "owner")
		.setDate(Date(), forKey: "createdAt")
try database.saveDocument(newTask)
```

### Mutability

By default, when a document is read from the database it is immutable. The `document.toMutable()` method should be used to create an instance of the document which can be updated.

```swift
guard let document = database.document(withID: "xyz") else { return }
let mutableDocument = document.toMutable()
mutableDocument.setString("apples", forKey: "name")
try database.saveDocument(mutableDocument)
```

Changes to the document are persisted to the database when the `saveDocument` method is called.

### Typed Accessors

The `Document` class now offers a set of [`property accessors`](http://docs.couchbase.com/mobile/2.0/couchbase-lite-swift/db022/Classes/Document.html#/DictionaryProtocol) for various scalar types, including boolean, integers, floating-point and strings. These accessors take care of converting to/from JSON encoding, and make sure you get the type you're expecting: for example, `document.string(forKey: String)` returns either a `String` or `nil`, so you can't get an unexpected object class and crash trying to use it as a string. (Even if the property in the document has an incompatible type, the accessor returns `nil`.)

In addition, as a convenience we offer `Date` accessors. Dates are a common data type, but JSON doesn't natively support them, so the convention is to store them as strings in ISO-8601 format. The following example sets the date on the `createdAt` property and reads it back using the `document.date(forKey: String)` accessor method.

```swift
newTask.setDate(Date(), forKey: "createdAt")
let date = newTask.date(forKey: "createdAt")
```

### Batch operations

If you're making multiple changes to a database at once, it's faster to group them together. The following example persists a few documents in batch.

```swift
do {
    try database.inBatch {
        for i in 0...10 {
            let doc = MutableDocument()
            doc.setValue("user", forKey: "type")
            doc.setValue("user \(i)", forKey: "name")
            doc.setBoolean(false, forKey: "admin")
            try database.saveDocument(doc)
            print("saved user document \(doc.string(forKey: "name"))")
        }
    }
} catch let error {
    print(error.localizedDescription)
}
```

At the **local** level this operation is still transactional: no other `Database` instances, including ones managed by the replicator can make changes during the execution of the block, and other instances will not see partial changes. But Couchbase Mobile is a distributed system, and due to the way replication works, there's no guarantee that Sync Gateway or other devices will receive your changes all at once.

### Blobs

We've renamed "attachments" to "blobs", for clarity. The new behavior should be clearer too: a `Blob` is now a normal object that can appear in a document as a property value. In other words, you just instantiate a `Blob` and set it as the value of a property, and then later you can get the property value, which will be a `Blob` object. The following code example adds a blob to the document under the `avatar` property.

```swift
let appleImage = UIImage(named: "avatar.jpg")!
let imageData = UIImageJPEGRepresentation(appleImage, 1)!

let blob = Blob(contentType: "image/jpeg", data: imageData)
newTask.setBlob(blob, forKey: "avatar")
try? database.save(newTask)

if let taskBlob = newTask.blob(forKey: "image") {
	UIImage(data: taskBlob.content!)
}
```

The `Blob` API lets you access the contents as in-memory data (a `Data` object) or as a `InputStream`. It also supports an optional `type` property that by convention stores the MIME type of the contents.

In the example above, "image/jpeg" is the MIME type and "avatar" is the key which references that `Blob`. That key can be used to retrieve the `Blob` object at a later time.

When a document is synchronized, the Couchbase Lite replicator will add an `_attachments` dictionary to the document's properties if it contains a blob. A random access name will be generated for each `Blob` which is different to the "avatar" key that was used in the example above. On the image below, the document now contains the `_attachments` dictionary when viewed in the Couchbase Server Admin Console.

<img class="portrait" style="width: 450px" src="img/attach_replicated.png" />

A blob also has properties such as `"digest"` (a SHA-1 digest of the data), `"length"` (the length in bytes), and optionally `"content_type"` (the MIME type.) The data is not stored in the document, but in a separate content-addressable store, indexed by the digest.

This `Blob` can be retrieved on the Sync Gateway REST API at [localhost:4984/justdoit/user.david/blob_1](http://localhost:4984/justdoit/user.david/blob_1). Notice that the blob identifier in the URL path is "blob_1" (not "avatar").

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

```swift
let index = IndexBuilder.valueIndex(items:
            ValueIndexItem.expression(Expression.property("type")),
            ValueIndexItem.expression(Expression.property("name")))
try database.createIndex(index, withName: "TypeNameIndex")
```

If there are multiple expressions, the first one will be the primary key, the second the secondary key, etc.

> **Note:** Every index has to be updated whenever a document is updated, so too many indexes can hurt performance. Thus, good performance depends on designing and creating the *right* indexes to go along with your queries.

### SELECT statement

With the SELECT statement, you can query and manipulate JSON data. With projections, you retrieve just the fields that you need and not the entire document. This is especially useful when querying for a large dataset as it results in shorter processing times and better performance.

A SelectResult represents a single return value of the query statement. Documents in Couchbase Lite comprise of the document properties specified as a Dictionary of Key-Value pairs and associated metadata. The metadata consists of document Id and sequence Id associated with the Document. When you query for a document, the document metadata is not returned by default. You will need to explicitly query for the metadata.

- [`SelectResult.all()`](http://docs.couchbase.com/mobile/2.0/couchbase-lite-swift/db022/Classes/SelectResult.html#/s:18CouchbaseLiteSwift12SelectResultC3allAC4FromCyFZ): Returns all properties associated with a document.
- `SelectResult(`[`Expression`](http://docs.couchbase.com/mobile/2.0/couchbase-lite-swift/db022/Classes/Expression.html)`.property("name"))`: Returns the `name` property associated with a document.
- `SelectResult.expression(`[`Meta`](http://docs.couchbase.com/mobile/2.0/couchbase-lite-swift/db022/Classes/Meta.html)`.id`)`: Returns the document ID.
- `SelectResult.expression(Expression.meta().sequence)`: Returns the sequence ID (used in replications).

You can specify a comma separated list of `SelectResult` expressions in the select statement of your query. For instance the following select statement queries for the document `_id` as well as the `type` and `name` properties of all documents in the database. In the query result, we print the `_id` and `name` properties of each row using the property name getter method.

```json
{
	"_id": "hotel123",
	"type": "hotel",
	"name": "Apple Droid"
}
```

```swift
let query = QueryBuilder
	.select(
		SelectResult.expression(Expression.meta().id),
		SelectResult.expression(Expression.property("type")),
		SelectResult.expression(Expression.property("name"))
	)
	.from(DataSource.database(database))

do {
	for row in try query.run() {
		print("document id :: \(row.string(forKey: "id"))")
		print("document name :: \(row.string(forKey: "name"))")
	}
} catch {
	print(error)
}
```

The `SelectResult.all()` method can be used to query all the properties of a document. In this case, the document in the result is embedded in a dictionary where the key is the database name. The following snippet shows the same query using `SelectResult.all()` and the result in JSON.

```swift
let query = QueryBuilder
	.select(SelectResult.all())
	.from(DataSource.database(database))
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

The `Expression`'s [comparison operators](http://docs.couchbase.com/mobile/2.0/couchbase-lite-swift/db022/Classes/Expression.html#/Comparison%20Operators) can be used in the WHERE statement to specify on which property to match documents. In the example below, we use the `equalTo` operator to query documents where the `type` property equals "hotel".

```json
{
	"_id": "hotel123",
	"type": "hotel",
	"name": "Apple Droid"
}
```

```swift
let query = QueryBuilder
	.select(SelectResult.all())
	.from(DataSource.database(database))
	.where(Expression.property("type").equalTo(Expression.string("hotel")))
	.limit(10)

do {
	for row in try query.run() {
		if let dict = row.dictionary(forKey: "travel-sample") {
			print("document name :: \(dict.string(forKey: "name"))")
		}
	}
} catch {
	print(error)
}
```

#### Collection Operators

[Collection operators](http://docs.couchbase.com/mobile/2.0/couchbase-lite-swift/db022/Classes/Expression.html#/Collection%20operators:) are useful to check if a given value is present in an array. The following example uses the `Function.arrayContains` to find documents whose `public_likes` array property contain a value equal to "Armani Langworth".

```json
{
	"_id": "hotel123",
	"name": "Apple Droid",
	"public_likes": ["Armani Langworth", "Elfrieda Gutkowski", "Maureen Ruecker"]
}
```

```swift
let query = QueryBuilder
	.select(
		SelectResult.expression(Expression.meta().id),
		SelectResult.expression(Expression.property("name")),
		SelectResult.expression(Expression.property("public_likes"))
	)
	.from(DataSource.database(database))
	.where(Expression.property("type").equalTo(Expression.string("hotel"))
                .and(ArrayFunction.contains(Expression.property("public_likes"), value: Expression.string("Armani Langworth")))
	)

do {
	for row in try query.run() {
		print("public_likes :: \(row.array(forKey: "public_likes")?.toArray())")
	}
}
```

#### Like Operator

The [`like`](http://docs.couchbase.com/mobile/2.0/couchbase-lite-swift/db022/Classes/Expression.html#/s:18CouchbaseLiteSwift10ExpressionC4likeACypF) operator can be used for string matching. It is recommended to use the `like` operator for case insensitive matches and the `regex` operator (see below) for case sensitive matches.

In the example below, we are looking for documents of type `landmark` where the name property exactly matches the string "Royal engineers museum". Note that since `like` does a case insensitive match, the following query will return "landmark" type documents with name matching "Royal Engineers Museum", "royal engineers museum", "ROYAL ENGINEERS MUSEUM" and so on.

```swift
let query = QueryBuilder
	.select(
		SelectResult.expression(Expression.meta().id),
		SelectResult.expression(Expression.property("country")),
		SelectResult.expression(Expression.property("name"))
	)
	.from(DataSource.database(db))
	.where(Expression.property("type").equalTo(Expression.string("landmark"))
                .and(Expression.property("name").like(Expression.string("eng%e%")))
	)
	.limit(10)

do {
	for row in try query.run() {
		print("name property :: \(row.string(forKey: "name")!)")
	}
}
```

##### Wildcard Match

We can use `%` sign within a `like` expression to do a wildcard match against zero or more characters. Using wildcards allows you to have some fuzziness in your search string.

In the example below, we are looking for documents of `type` "landmark" where the name property matches any string that begins with "eng" followed by zero or more characters, the letter "e", followed by zero or more characters. The following query will return "landmark" type documents with name matching "Engineers", "engine", "english egg" , "England Eagle" and so on. Notice that the matches may span word boundaries.

```swift
let query = QueryBuilder
	.select(
		SelectResult.expression(Expression.meta().id),
		SelectResult.expression(Expression.property("country")),
		SelectResult.expression(Expression.property("name"))
	)
	.from(DataSource.database(db))
	.where(Expression.property("type").equalTo(Expression.string("landmark"))
                .and(Expression.property("name").like(Expression.string("eng%e%")))
	.limit(limit)
```

##### Wildcard Character Match

We can use `_` sign within a like expression to do a wildcard match against a single character.

In the example below, we are looking for documents of type "landmark" where the `name` property matches any string that begins with "eng" followed by exactly 4 wildcard characters and ending in the letter "r".
The following query will return "landmark" `type` documents with the `name` matching "Engineer", "engineer" and so on.

```swift
let query = QueryBuilder
	.select(SelectResult.expression(Expression.meta().id),
		SelectResult.expression(Expression.property("country")),
		SelectResult.expression(Expression.property("name")))
	.from(DataSource.database(db))
	.where(Expression.property("type").equalTo(Expression.string("landmark"))
                .and(Expression.property("name").like(Expression.string("eng____r")))
	.limit(limit)
```

#### Regex Operator

The [`regex`](http://docs.couchbase.com/mobile/2.0/couchbase-lite-swift/db022/Classes/Expression.html#/s:18CouchbaseLiteSwift10ExpressionC5regexACypF) operator can be used for case sensitive matches. Similar to wildcard `like` expressions, `regex` expressions based pattern matching allow you to have some fuzziness in your search string.

In the example below, we are looking for documents of `type` "landmark" where the name property matches any string (on word boundaries) that begins with "eng" followed by exactly 4 wildcard characters and ending in the letter "r".
The following query will return "landmark" type documents with name matching "Engine", "engine" and so on. Note that the `\b` specifies that the match must occur on word boundaries.

```swift
let query = QueryBuilder
	.select(
		SelectResult.expression(Expression.meta().id),
		SelectResult.expression(Expression.property("name"))
	)
	.from(DataSource.database(db))
	.where(Expression.property("type").equalTo(Expression.string("landmark"))
                .and(Expression.property("name").like(Expression.string("\\bEng.*e\\b")))
	.limit(limit)
```

### JOIN statement

The JOIN clause enables you to create new input objects by combining two or more source objects.

The following example uses a JOIN clause to find the airline details which have routes that start from RIX. This example JOINS the document of type "route" with documents of type "airline" using the document ID (`_id`) on the "airline" document and  `airlineid` on the "route" document.

```swift
let query = QueryBuilder.select(
	SelectResult.expression(Expression.property("name").from("airline")),
	SelectResult.expression(Expression.property("callsign").from("airline")),
	SelectResult.expression(Expression.property("destinationairport").from("route")),
	SelectResult.expression(Expression.property("stops").from("route")),
	SelectResult.expression(Expression.property("airline").from("route"))
)
.from(
	DataSource.database(database!).as("airline")
)
.join(
	Join.join(DataSource.database(database!).as("route"))
	.on(
		Expression.meta().id.from("airline")
		.equalTo(Expression.property("airlineid").from("route"))
	)
)
.where(
	Expression.property("type").from("route").equalTo(Expression.string("route"))
                    .and(Expression.property("type").from("airline").equalTo(Expression.string("airline")))
                    .and(Expression.property("sourceairport").from("route").equalTo(Expression.string("RIX")))
)
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

```swift
let query = QueryBuilder.select(
	SelectResult.expression(Function.count("*")),
	SelectResult.expression(Expression.property("country")),
	SelectResult.expression(Expression.property("tz"))
	)
	.from(DataSource.database(database))
	.where(
		Expression.property("type").equalTo(Expression.string("airport"))
                    .and(Expression.property("geo.alt").greaterThanOrEqualTo(Expression.int(300)))
	)
	.groupBy(
		Expression.property("country"),
		Expression.property("tz")
	)

do {
	for row in try query.run() {
		print("There are \(row.int(forKey: "$1")) airports on the \(row.string(forKey: "tz")!) timezone located in \(row.string(forKey: "country")!) and above 300 ft")
	}
}
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

```swift
let query = QueryBuilder
	.select(
		SelectResult.expression(Expression.meta().id),
		SelectResult.expression(Expression.property("title")))
	.from(DataSource.database(database))
	.where(Expression.property("type").equalTo(Expression.string("hotel")))
	.orderBy(Ordering.property("title").ascending())
	.limit(limit)
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

```swift
// Insert documents
let tasks = ["buy groceries", "play chess", "book travels", "buy museum tickets"]
for task in tasks {
	let doc = MutableDocument()
	doc.setString("task", forKey: "type")
	doc.setString(task, forKey: "name")
	try? database.saveDocument(doc)
}

// Create index
do {
	let index = IndexBuilder.fullTextIndex(items: FullTextIndexItem.property("name")).ignoreAccents(false)
        try database.createIndex(index, withName: "nameFTSIndex")
} catch let error {
	print(error.localizedDescription)
}
```

Multiple properties to index can be specified in the `Index.fullTextIndex(withItems: [FullTextIndexItem])` method.

With the index created, an FTS query on the property that is being indexed can be constructed and ran. The full-text search criteria is defined as a `FullTextExpression`. The left-hand side is the full-text index to use and the right-hand side is the pattern to match.

```swift
let whereClause = FullTextExpression.index("nameFTSIndex").match("'buy'")
let ftsQuery = Query.select(SelectResult.expression(Meta.id))
                  .from(DataSource.database(database))
                  .where(whereClause)

do {
	let ftsQueryResult = try ftsQuery.execute()
	for row in ftsQueryResult {
		print("document properties \(row.string(at: 0))")
	}
} catch let error {
	print(error.localizedDescription)
}
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

⚠️ The new protocol is **incompatible** with CouchDB-based databases. And since Couchbase Lite 2 only supports the new protocol, you will need to run a version of Sync Gateway that [supports it](../references/couchbase-lite/release-notes/index.html#compatibility-matrix).

To use this protocol with Couchbase Lite 2.0, the replication URL should specify WebSockets as the URL scheme (see the "Starting a Replication" section below). Mobile clients using Couchbase Lite 1.x can continue to use **http** as the URL scheme. Sync Gateway 2.0 will automatically use the 1.x replication protocol when a Couchbase Lite 1.x client connects through "http://localhost:4984/db" and the 2.0 replication protocol when a Couchbase Lite 2.0 client connects through "ws://localhost:4984/db".

### Starting Sync Gateway

[Download Sync Gateway](https://www.couchbase.com/downloads) and start it from the command line with the configuration file created above.

```bash
~/Downloads/couchbase-sync-gateway/bin/sync_gateway
```

For platform specific installation instructions, refer to the Sync Gateway [installation guide](https://developer.couchbase.com/documentation/mobile/1.5/installation/sync-gateway/index.html).

### Starting a Replication

Replication objects are now bidirectional, this means you can start a `push`/`pull` replication with a single instance. The replication's parameters can be specified through the [`ReplicatorConfiguration`](http://docs.couchbase.com/mobile/2.0/couchbase-lite-swift/db022/Classes/ReplicatorConfiguration.html) object; for example, if you wish to start a `push` only or `pull` only replication. The following example creates a `pull` only replication instance with Sync Gateway.

```swift
let url = URL(string: "ws://localhost:4984/db")!
var replConfig = ReplicatorConfiguration(withDatabase: database, targetURL: url)
replConfig.replicatorType = .pull
let replication = Replicator(withConfig: replConfig)
replication.start()
```

As shown in the code snippet above, the URL scheme for remote database URLs has changed in Couchbase Lite 2.0. You should now use `ws:`, or `wss:` for SSL/TLS connections. You can access the Sync Gateway `_all_docs` endpoint [http://localhost:4984/db/\_all\_docs?include_docs=true](http://localhost:4984/db/_all_docs?include_docs=true) to check that the documents are successfully replicated.

Couchbase Lite 2.0 uses WebSockets as the communication protocol to transmit data. Some load balancers are not configured for WebSocket connections by default (NGINX for example); so it might be necessary to explicitly enable them in the load balancer's configuration (see [Load Balancers](https://developer.couchbase.com/documentation/mobile/1.5/guides/sync-gateway/nginx/index.html)).

Starting in Couchbase Lite 2.0, replication between two local databases is now supported. This isn't often needed, but it can be very useful. For example, you can implement incremental backup by pushing your main database to a mirror on a backup disk.

### Troubleshooting

As always, when there is a problem with replication, logging is your friend. The following example increases the log output for activity related to replication with Sync Gateway.

```swift
Database.setLogLevel(.verbose, domain: .replicator)
```

### Replication Status

The `replication.status.activity` property can be used to check the status of a replication. For example, when the replication is actively transferring data and when it has stopped.

```swift
self.replication.addChangeListener { (change) in
    if change.status.activity == .stopped {
        print("Replication stopped")
    }
}
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

```swift
self.replication.addChangeListener { (change) in
    if let error = change.status.error as NSError? {
        print("Error code :: \(error.code)")
    }
}
self.replication.start()
```

## Handling Conflicts

Starting in Couchbase Lite 2.0, it is possible to delegate the resolution of conflicts to the database (also known as automatic conflict resolution). There are 2 different `save` method signatures to specify how to resolve a conflict:

- `save(document: MutableDocument)`: when concurrent writes to an individual record occur, the conflict is automatically resolved and only one non-conflicting document update is stored in the database. The Last-Write-Win (LWW) algorithm is used to pick the winning revision.
- `save(document: MutableDocument, concurrencyControl: ConcurrencyControl)`: attempts to save the document with a concurrency control. The concurrency control parameter has two possible values:
	- `none`: The last operation wins if there is a conflict.
	- `optimistic`: The operation will fail if there is a conflict.

Similarly to the save operation, the delete operation also has two method signatures to specify how to resolve a possible conflict:

- `delete(document: Document)`: The last write will win if there is a conflict.
- `delete(document: Document, concurrencyControl: ConcurrencyControl)`: attemps to delete the document with a concurrency control. The concurrency control parameter has two possible values:
	- `none`: The last operation wins if there is a conflict.
	- `optimistic`: The operation will fail if there is a conflict.

## Certificate Pinning

Couchbase Lite supports certificate pinning. Certificate pinning is a technique that can be used by applications to "pin" a host to it's certificate. The certificate is typically delivered to the client by an out-of-band channel and bundled with the client. In this case, Couchbase Lite uses this embedded certificate to verify the trustworthiness of the server and no longer needs to rely on a trusted third party for that (commonly referred to as the Certificate Authority).

The `openssl` command can be used to create a new self-signed certificate and convert the `.pem` file to a `.cert` file (see [creating your own self-signed certificate](https://developer.couchbase.com/documentation/mobile/1.5/guides/sync-gateway/configuring-ssl/index.html#creating-your-own-self-signed-certificate)). You should then have 3 files: `cert.pem`, `cert.cer` and `key.pem`.

The `cert.pem` and `key.pem` can be used in the Sync Gateway configuration (see [installing the certificate](https://developer.couchbase.com/documentation/mobile/1.5/guides/sync-gateway/configuring-ssl/index.html#installing-the-certificate)).

On the Couchbase Lite side, the replication must be configured with the `cert.cer` file.

```swift
let sslCert = Bundle.main.path(forResource: "cert", ofType: "cer")
if let path = sslCert {
	if let data = NSData(contentsOfFile: path) {
		let certificate = SecCertificateCreateWithData(nil, data)
		replConfig.pinnedServerCertificate = certificate
	}
}
```

This example loads the certificate from the application sandbox, then converts it to the appropriate type to configure the replication object.
