---
---

## Getting Started

- Add `http://mobile.nuget.couchbase.com/nuget/Developer/` to your Nuget package sources and expect a new build approximately every 2 weeks!

Your app must call the relevant `Activate()` function inside of the class that is included in the support assembly (there is only one public class in each support assembly, and the support assembly itself is a nuget dependency).  For example, UWP looks like `Couchbase.Lite.Support.UWP.Activate()`.  Currently the support assemblies provide dependency injected mechanisms for default directory logic, and platform specific logging (i.e. Android will log to logcat with correct log levels and tags.  No more "mono-stdout" at always info level).

### Supported Versions

Couchbase Lite .NET will be a .NET Standard 2.0 library as of the Couchbase Lite .NET 2.0 GA release.

| .NET Runtime     | Minimum Runtime Version | Minimum OS version |
| ---------------- | ----------------------- | ------------------ |
| .NET Core Win    | 2.0                     | 10 (any supported) |
| .NET Core Mac    | 2.0                     | 10.12              |
| .NET Core Linux  | 2.0                     | n/a*               |
| .NET Framework   | 4.6.1                   | 10 (any supported) |
| UWP              | 6.0.1                   | 10.0.16299         |
| Xamarin iOS      | 10.14                   | 10.3.1             |
| Xamarin Android  | 8.0                     | 4.4 (API 19)       |

* There are many different variants of Linux, and we don't have the resources to test all of them.  They are tested on Ubuntu 16.04, but have been shown to work on CentOS, and in theory work on any distro supported by .NET Core.

Comparing this to 1.x you can see we've traded some lower obsolete versions for new platform support:

| .NET Runtime     | Minimum Runtime Version | Minimum OS version |
| ---------------- | ----------------------- | ------------------ |
| .NET Framework   | 3.5                     | XP                 |
| Mono Mac         | 5.2*                    | 10.9               |
| Mono Linux       | 5.2*                    | n/a**              |
| Xamarin iOS      | 8.0*                    | 8.0                |
| Xamarin Android  | 4.6*                    | 2.3 (API 9)        |

* These runtime versions are approximate.  Couchbase Lite 1.x is built using relatively new versions of all of these and an absolute minimum is unclear, and cannot actually be determined anymore due to lack of vendor support (i.e. Xamarin iOS 8 uses Xcode 6, etc).  So basically "any version" is an approximately good metric here.

** See above note about Linux

## API References

[`.NET SDK API References`](http://docs.couchbase.com/mobile/2.0/couchbase-lite-net/db022)

## Database

### New Database

As the top-level entity in the API, new databases can be created using the `Database` class by passing in a name, configuration, or both. The following example creates a database using the `Database(string name)` method.

```csharp
var database = new Database("my-database");
```

Just as before, the database will be created in a default location. Alternatively, the `Database(string name, DatabaseConfiguration config)` initializer can be used to provide specific options in the [`DatabaseConfiguration`](http://docs.couchbase.com/mobile/2.0/couchbase-lite-net/db022/html/T_Couchbase_Lite_DatabaseConfiguration.htm) object such as the database directory, encryption key through the  object.

###  Encryption

The following example demonstrates how to create a database with an encryption key (or open an existing one).

```c#
var dbConfig = new DatabaseConfiguration {
    EncryptionKey = new EncryptionKey("secretpassword")
};

Database = new Database("my-database", dbConfig);
```

**NOTE**: Encryption is an Enterprise Edition only feature

### Migrating from 1.x Databases

Databases that were created with Couchbase Mobile 1.2 or later can be read using the 2.0 API. Upon detecting it is a 1.x database file format, Couchbase Lite will automatically upgrade it to the new format used in 2.0. This feature is currently only available for the default storage type (i.e not for ForestDB databases).

### Finding a Database File

Where a database goes by default depends on the platform it is running on.  Here are the defaults for each platform:

- .NET Core: `Path.Combine(AppContext.BaseDirectory, "CouchbaseLite")` (unless the app context is altered \[e.g. by XUnit\], this will be the same directory as the output binary)
- UWP: `Windows.Storage.ApplicationData.Current.LocalFolder.Path` (Inside the installed app sandbox.  Note that this sandbox gets deleted sometimes when debugging from inside Visual Studio when the app is shutdown)
- Xamarin iOS: In a folder named CouchbaseLite inside of `ApplicationSupportDirectory` (this can be retrieved more easily from the simulator using the [SimPholders](https://simpholders.com/3/) utility)
- Xamarin Android: Using the `Context` passed in the `Activate()` method, `Context.FilesDir.AbsolutePath` (database can be retrieved using adb)

### Logging

The log messages are split into different domains (`LogDomains`) which can be tuned to different log levels. The following example enables `Verbose` logging for the `Replicator` and `Query` domains.

```c#
Database.SetLogLevel(LogDomain.Replicator, LogLevel.Verbose);
Database.SetLogLevel(LogDomain.Query, LogLevel.Verbose);
```

### Loading a pre-built database

If your app needs to sync a lot of data initially, but that data is fairly static and won't change much, it can be a lot more efficient to bundle a database in your application and install it on the first launch. Even if some of the content changes on the server after you create the app, the app's first pull replication will bring the database up to date.

To use a prebuilt database, you need to set up the database, build the database into your app bundle as a resource, and install the database during the initial launch. After your app launches, it needs to check whether the database exists. If the database does not exist, the app should copy it from the app bundle using the [`Database.Copy(string, DatabaseConfiguration)`](http://docs.couchbase.com/mobile/2.0/couchbase-lite-net/db022/html/M_Couchbase_Lite_Database_Copy.htm) method as shown below.

```c#
// assetPath is the path on the filesystem to the cblite2 folder
// On Android this means extracting from the APK into a temp directory first
if !(Database.Exists("travel-sample")) {
    Database.Copy(assetPath, "travel-sample", null);
}
```

## Document

In Couchbase Lite, a document's body takes the form of a JSON object — a collection of key/value pairs where the values can be different types of data such as numbers, strings, arrays or even nested objects. Every document is identified by a document ID, which can be automatically generated (as a UUID) or determined by the application; the only constraints are that it must be unique within the database, and it can't be changed.

### Initializers

The following methods/initializers can be used:

- The `MutableDocument()` constructor can be used to create a new document where the document ID is randomly generated by the database.
- The `MutableDocument(string documentID)` constructor can be used to create a new document with a specific ID.
- The `database.GetDocument(string documentID)` method can be used to  get a document. If it doesn't exist in the database, it will return `null`. This method can be used to check if a document with a given ID already exists in the database.

The following code example creates a document and persists it to the database.

```c#
var dict = new Dictionary<string, object> {
    ["type"] = "task",
    ["owner"] = "todo",
    ["createdAt"] = DateTimeOffset.UtcNow()
};

var newTask = new MutableDocument(dict);
database.Save(newTask);
```

### Mutability

By default, when a document is read from the database it is immutable. The `document.toMutable()` method should be used to create an instance of the document which can be updated.

```c#
// newTask is a MutableDocument
newTask.SetString("name", "apples");
database.Save(newTask);
```

Changes to the document are persisted to the database when the `saveDocument` method is called.

### Typed Accessors

The `Document` class now offers a set of [`property accessors`](http://docs.couchbase.com/mobile/2.0/couchbase-lite-net/db022/html/T_Couchbase_Lite_DictionaryObject.htm) for various scalar types, including boolean, integers, floating-point and strings. These accessors take care of converting to/from JSON encoding, and make sure you get the type you're expecting: for example, `document.GetString(string key)` returns either a `String` or `null`, so you can't get an unexpected object class and crash trying to use it as a string. (Even if the property in the document has an incompatible type, the accessor returns `null`.)

In addition, as a convenience we offer `DateTimeOffset` accessors. Dates are a common data type, but JSON doesn't natively support them, so the convention is to store them as strings in ISO-8601 format. The following example sets the date on the `createdAt` property and reads it back using the `document.GetDate(string key)` accessor method.

```c#
newTask.SetDate("createdAt", DateTimeOffset.UtcNow());
var date = newTask.GetDate("createdAt");
```

### Batch operations

If you're making multiple changes to a database at once, it's faster to group them together. The following example persists a few documents in batch.

```c#
database.InBatch(() => {
    foreach(int i in Enumerable.Range(1, 10)) {
        var doc = new MutableDocument();
        doc.SetString("type", "user");
        doc.SetString("name", $"user {i}");
        doc.SetBoolean("admin", false);
        database.Save(doc);
        Console.WriteLine($"saved user document {doc.GetString("name")}");
    }
});
```

At the **local** level this operation is still transactional: no other `Database` instances, including ones managed by the replicator can make changes during the execution of the block, and other instances will not see partial changes. But Couchbase Mobile is a distributed system, and due to the way replication works, there's no guarantee that Sync Gateway or other devices will receive your changes all at once.

### Blobs

We've renamed "attachments" to "blobs", for clarity. The new behavior should be clearer too: a `Blob` is now a normal object that can appear in a document as a property value. In other words, there's no special API for creating or accessing attachments; you just instantiate an `Blob` and set it as the value of a property, and then later you can get the property value, which will be a `Blob` object. The following code example adds a blob to the document under the `avatar` property.

```c#
// Also works with streams, i.e. File.OpenRead
var image = File.ReadAllBytes("avatar.jpg");

var blob = new Blob("image/jpg", image);

newTask.SetBlob("avatar", blob);
database.Save(newTask);
var taskBlob = newTask.GetBlob("avatar");
```

`Blob` itself has a simple API that lets you access the contents as in-memory data (a `Data` object) or as a `Stream`. It also supports an optional `type` property that by convention stores the MIME type of the contents. Unlike attachments, blobs don't have names; if you need to associate a name you can put it in another document property, or make the filename be the property name (e.g. `document.Set(string, Blob)`)

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

```csharp
database.CreateIndex("TypeNameIndex", IndexBuilder.ValueIndex(
	ValueIndexItem.Expression(Expression.Property("type")),
        ValueIndexItem.Expression(Expression.Property("name"))));
```

If there are multiple expressions, the first one will be the primary key, the second the secondary key, etc.

> **Note:** Every index has to be updated whenever a document is updated, so too many indexes can hurt performance. Thus, good performance depends on designing and creating the *right* indexes to go along with your queries.

### SELECT statement

With the SELECT statement, you can query and manipulate JSON data. With projections, you retrieve just the fields that you need and not the entire document. This is especially useful when querying for a large dataset as it results in shorter processing times and better performance.

A SelectResult represents a single return value of the query statement. Documents in Couchbase Lite comprise of the document properties specified as a Dictionary of Key-Value pairs and associated metadata. The metadata consists of document Id and sequence Id associated with the Document. When you query for a document, the document metadata is not returned by default. You will need to explicitly query for the metadata.

- [`SelectResult.All()`](http://docs.couchbase.com/mobile/2.0/couchbase-lite-net/db022/html/T_Couchbase_Lite_Query_SelectResult.htm): Returns all properties associated with a document.
- `SelectResult(`[`Expression`](http://docs.couchbase.com/mobile/2.0/couchbase-lite-net/db022/html/T_Couchbase_Lite_Query_IExpression.htm)`.property("name"))`: Returns the `name` property associated with a document.
- `SelectResult.expression(`[`Meta`](http://docs.couchbase.com/mobile/2.0/couchbase-lite-net/db022/html/T_Couchbase_Lite_Query_Meta.htm)`.id`)`: Returns the document ID.
- `SelectResult.expression(Expression.meta().sequence)`: Returns the sequence ID (used in replications).

You can specify a comma separated list of `SelectResult` expressions in the select statement of your query. For instance the following select statement queries for the document `_id` as well as the `type` and `name` properties of all documents in the database. In the query result, we print the `_id` and `name` properties of each row using the property name getter method.

```json
{
	"_id": "hotel123",
	"type": "hotel",
	"name": "Apple Droid"
}
```

```csharp
using(var query = QueryBuilder.Select(SelectResult.Property("name"))
	.From(DataSource.Database(database))
	.Where(
		Expression.Property("type").EqualTo(Expression.String("user"))
		.And(Expression.Property("admin").EqualTo(Expression.Boolean(false)))
	))
    var rows = query.Execute();
    foreach(var row in rows)
    {
	Console.WriteLine($"user name :: ${row.GetString(0)}");
    }
}
```

The `SelectResult.all()` method can be used to query all the properties of a document. In this case, the document in the result is embedded in a dictionary where the key is the database name. The following snippet shows the same query using `SelectResult.all()` and the result in JSON.

```csharp
var query = QueryBuilder
    .Select(SelectResult.All())
    .From(DataSource.Database(database));
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

The `Expression`'s [comparison operators](http://docs.couchbase.com/mobile/2.0/couchbase-lite-net/db022/Classes/Expression.html#/Comparison%20Operators) can be used in the WHERE statement to specify on which property to match documents. In the example below, we use the `equalTo` operator to query documents where the `type` property equals "hotel".

```json
{
	"_id": "hotel123",
	"type": "hotel",
	"name": "Apple Droid"
}
```

```csharp
using(var query = QueryBuilder.Select(SelectResult.Property("name"))
	.From(DataSource.Database(database))
	.Where(Expression.Property("type").EqualTo(Expression.String("hotel")))
	.Limit(10))
    var rows = query.Execute());
    foreach(var row in rows)
    {
        Console.WriteLine($"user name :: ${row.GetString("name")}");
    }
}
```

#### Collection Operators

[Collection operators](http://docs.couchbase.com/mobile/2.0/couchbase-lite-net/db022/Classes/Expression.html#/Collection%20operators:) are useful to check if a given value is present in an array. The following example uses the `Function.arrayContains` to find documents whose `public_likes` array property contain a value equal to "Armani Langworth".

```json
{
	"_id": "hotel123",
	"name": "Apple Droid",
	"public_likes": ["Armani Langworth", "Elfrieda Gutkowski", "Maureen Ruecker"]
}
```

```csharp
using(var query = QueryBuilder.Select(
        SelectResult.Expression(Meta.ID),
	SelectResult.Expression(Expression.Property("name")),
	SelectResult.Expression(Expression.Property("public_likes")))
	.From(DataSource.Database(database))
	.Where(Expression.Property("type").EqualTo(Expression.String("hotel"))
	    .And(ArrayFunction.Contains(Expression.Property("public_likes"), Expression.String("Armani Langworth")))))
    var rows = query.Execute();
    foreach(var row in rows)
    {
        // Serialize the array first to get meaningful string output
        Console.WriteLine($"public_likes :: ${row.GetArray("public_likes")}");
    }
}
```

#### Like Operator

The [`like`](http://docs.couchbase.com/mobile/2.0/couchbase-lite-net/db022/html/M_Couchbase_Lite_Query_IExpression_Like.htm) operator can be used for string matching. It is recommended to use the `like` operator for case insensitive matches and the `regex` operator (see below) for case sensitive matches.

In the example below, we are looking for documents of type `landmark` where the name property exactly matches the string "Royal engineers museum". Note that since `like` does a case insensitive match, the following query will return "landmark" type documents with name matching "Royal Engineers Museum", "royal engineers museum", "ROYAL ENGINEERS MUSEUM" and so on.

```csharp
using(var query = QueryBuilder.Select(
        SelectResult.Expression(Meta.ID),
	SelectResult.Expression(Expression.Property("country")),
	SelectResult.Expression(Expression.Property("name")))
	.From(DataSource.Database(database))
	.Where(Expression.Property("type").EqualTo(Expression.String("landmark"))
	    .And(Expression.Property("name").Like(Expression.String("Royal engineers museum"))))
	.Limit(10))
    var rows = query.Execute();
    foreach(var row in rows)
    {
        // Serialize the array first to get meaningful string output
        Console.WriteLine($"name property :: ${row.GetString("name")}");
    }
}
```

##### Wildcard Match

We can use `%` sign within a `like` expression to do a wildcard match against zero or more characters. Using wildcards allows you to have some fuzziness in your search string.

In the example below, we are looking for documents of `type` "landmark" where the name property matches any string that begins with "eng" followed by zero or more characters, the letter "e", followed by zero or more characters. The following query will return "landmark" type documents with name matching "Engineers", "engine", "english egg" , "England Eagle" and so on. Notice that the matches may span word boundaries.

```csharp
var query = QueryBuilder.Select(
    SelectResult.Expression(Meta.ID)
    SelectResult.Expression(Expression.Property("country")),
    SelectResult.Expression(Expression.Property("name")))
	.From(DataSource.Database(database))
	.Where(Expression.Property("type").EqualTo(Expression.String("landmark"))
	    .And(Expression.Property("name").Like(Expression.String("eng%e%"))))
	.Limit(limit)
```

##### Wildcard Character Match

We can use `_` sign within a like expression to do a wildcard match against a single character.

In the example below, we are looking for documents of type "landmark" where the `name` property matches any string that begins with "eng" followed by exactly 4 wildcard characters and ending in the letter "r".
The following query will return "landmark" `type` documents with the `name` matching "Engineer", "engineer" and so on.

```csharp
var query = QueryBuilder.Select(
    SelectResult.Expression(Meta.ID),
    SelectResult.Expression(Expression.Property("country")),
    SelectResult.Expression(Expression.Property("name")))
    .From(DataSource.Database(database))
    .Where(Expression.Property("type").EqualTo(Expression.String("landmark"))
        .And(Expression.Property("name").Like(Expression.String("eng____r"))))
    .Limit(limit))
```

#### Regex Operator

The [`regex`](http://docs.couchbase.com/mobile/2.0/couchbase-lite-net/db022/html/M_Couchbase_Lite_Query_IExpression_Regex.htm) operator can be used for case sensitive matches. Similar to wildcard `like` expressions, `regex` expressions based pattern matching allow you to have some fuzziness in your search string.

In the example below, we are looking for documents of `type` "landmark" where the name property matches any string (on word boundaries) that begins with "eng" followed by exactly 4 wildcard characters and ending in the letter "r".
The following query will return "landmark" type documents with name matching "Engine", "engine" and so on. Note that the `\b` specifies that the match must occur on word boundaries.

```csharp
var query = QueryBuilder.Select(
    SelectResult.Expression(Meta.ID),
    SelectResult.Expression(Expression.Property(Expression.String("name"))))
    .From(DataSource.Database(db))
    .Where(Expression.Property("type").EqualTo(Expression.String("landmark"))
        .And(Expression.Property("name").Regex(Expression.String("\\bEng.*e\\b"))))
    .Limit(limit))
```

### JOIN statement

The JOIN clause enables you to create new input objects by combining two or more source objects.

The following example uses a JOIN clause to find the airline details which have routes that start from RIX. This example JOINS the document of type "route" with documents of type "airline" using the document ID (`_id`) on the "airline" document and  `airlineid` on the "route" document.

```csharp
var query = QueryBuilder.Select(
        SelectResult.Expression(Expression.Property("name").From("airline")),
        SelectResult.Expression(Expression.Property("callsign").From("airline")),
        SelectResult.Expression(Expression.Property("destinationairport").From("route")),
        SelectResult.Expression(Expression.Property("stops").From("route")),
        SelectResult.Expression(Expression.Property("airline").From("route")))
    .From(DataSource.Database(database).As("airline"))
    .Joins(Join.DefaultJoin(DataSource.Database(database).As("route"))
        .On(Meta.ID.From("airline").EqualTo(Expression.Property("airlineid").From("route"))))
    .Where(
        Expression.Property("type").From("route").EqualTo(Expression.String("route"))
	    .And(Expression.Property("type").From("airline").EqualTo(Expression.String("airline")))
	    .And(Expression.Property("sourceairport").From("route").EqualTo(Expression.String("RIX"))))
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

```csharp
using(var query = QueryBuilder.Select(
        SelectResult.Expression(Function.Count(Expression.All())),
	SelectResult.Expression(Expression.Property("country")),
	SelectResult.Expression(Expression.Property("tz")))
    .From(DataSource.Database(database))
    .Where(
        Expression.Property("type").EqualTo(Expression.String("airport"))
	    .And(Expression.Property("geo.alt").GreaterThanOrEqualTo(300)))
    .GroupBy(
        Expression.Property("country"),
    var rows = query.Execute();
    foreach(var row in rows) {
        Console.WriteLine($"There are {row.GetInt("$1")} airports on the {row.GetString("tz")} timezone located in " +
	    $"{row.GetString("country")} and above 300 ft");
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

```csharp
var query = QueryBuilder.Select(
        SelectResult.Expression(Meta.ID),
	SelectResult.Expression(Expression.Property("title")))
    .From(DataSource.Database(database))
    .Where(Expression.Property("type").EqualTo(Expression.String("hotel")))
    .OrderBy(Ordering.Property("title").Ascending())
    .Limit(limit);
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

```c#
// Insert documents
var tasks = new[] { "buy groceries", "play chess", "book travels", "buy museum tickets" };
foreach(var task in tasks) {
    var doc = new MutableDocument();
    doc.SetString("type", "task");
    doc.SetString("name", task);
    database.Save(doc);
}

// Create index
database.CreateIndex("nameFTSIndex", IndexBuilder.FullTextIndex(FullTextIndexItem.Property("name")).IgnoreAccents(false));
```

Multiple properties to index can be specified in the `IndexBuilder.FullTextIndex(params FullTextIndexItem[] items)` method.

With the index created, an FTS query on the property that is being indexed can be constructed and ran. The full-text search criteria is defined as a `FullTextExpression`. The left-hand side is the full-text index to use and the right-hand side is the pattern to match.

```c#
var whereClause = FullTextExpression.Index("nameFTSIndex").Match("'buy'");
var ftsQuery = QueryBuilder.Select(SelectResult.Expression(Meta.ID))
                  .From(DataSource.Database(database))
                  .Where(whereClause);

    var ftsQueryResult = ftsQuery.Execute();
    foreach(var row in ftsQueryResult) {
	Console.WriteLine($"document properties ${row.GetString(0)}");
    }
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

Windows
```powershell
& 'C:\Program Files (x86)\Couchbase\sync_gateway.exe' sync-gateway-config.json
```

Unix
```bash
~/Downloads/couchbase-sync-gateway/bin/sync_gateway
```

For platform specific installation instructions, refer to the Sync Gateway [installation guide](https://developer.couchbase.com/documentation/mobile/1.5/installation/sync-gateway/index.html).

### Starting a Replication

Replication objects are now bidirectional, this means you can start a `push`/`pull` replication with a single instance. The replication's parameters can be specified through the [`ReplicatorConfiguration`](http://docs.couchbase.com/mobile/2.0/couchbase-lite-net/db022/html/T_Couchbase_Lite_Sync_ReplicatorConfiguration.htm) object; for example, if you wish to start a `push` only or `pull` only replication. The following example creates a `pull` only replication instance with Sync Gateway.

```c#
var url = new Uri("ws://localhost:4984/db");
var replConfig = new ReplicatorConfiguration(database, url);
replConfig.ReplicatorType = ReplicatorType.Pull;
var replication = new Replicator(replConfig);
replication.Start()
```

As shown in the code snippet above, the URL scheme for remote database URLs has changed in Couchbase Lite 2.0. You should now use `ws:`, or `wss:` for SSL/TLS connections. You can access the Sync Gateway `_all_docs` endpoint [http://localhost:4984/db/\_all\_docs?include_docs=true](http://localhost:4984/db/_all_docs?include_docs=true) to check that the documents are successfully replicated.

Couchbase Lite 2.0 uses WebSockets as the communication protocol to transmit data. Some load balancers are not configured for WebSocket connections by default (NGINX for example); so it might be necessary to explicitly enable them in the load balancer's configuration (see [Load Balancers](https://developer.couchbase.com/documentation/mobile/1.5/guides/sync-gateway/nginx/index.html)).

Starting in Couchbase Lite 2.0, replication between two local databases is now supported. This isn't often needed, but it can be very useful. For example, you can implement incremental backup by pushing your main database to a mirror on a backup disk.

### Troubleshooting

As always, when there is a problem with replication, logging is your friend. The following example increases the log output for activity related to replication with Sync Gateway.

```c#
Database.SetLogLevel(LogDomain.Replicator, LogLevel.Verbose);
```

### Replication Status

The `replication.Status.Activity` property can be used to check the status of a replication. For example, when the replication is actively transferring data and when it has stopped.

```c#
replication.AddChangeListener((sender, args) => 
{
    if (args.Status.Activity == ReplicatorActivityLevel.Stopped) {
        Console.WriteLine("Replication stopped")
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

```c#
replication.AddChangeListener((sender, args) =>
{
    if(args.Status.Error != null) {
        Console.WriteLine($"Error: {args.Status.Error.Message}");
    }
}
replication.start()
```

### Certificate Pinning

Couchbase Lite supports certificate pinning. Certificate pinning is a technique that can be used by applications to "pin" a host to it's certificate. The certificate is typically delivered to the client by an out-of-band channel and bundled with the client. In this case, Couchbase Lite uses this embedded certificate to verify the trustworthiness of the server and no longer needs to rely on a trusted third party for that (commonly referred to as the Certificate Authority).

The `openssl` command can be used to create a new self-signed certificate and convert the `.pem` file to a `.cert` file (see [creating your own self-signed certificate](https://developer.couchbase.com/documentation/mobile/1.5/guides/sync-gateway/configuring-ssl/index.html#creating-your-own-self-signed-certificate)). You should then have 3 files: `cert.pem`, `cert.cer` and `key.pem`.

The `cert.pem` and `key.pem` can be used in the Sync Gateway configuration (see [installing the certificate](https://developer.couchbase.com/documentation/mobile/1.5/guides/sync-gateway/configuring-ssl/index.html#installing-the-certificate)).

On the Couchbase Lite side, the replication must be configured with the `cert.cer` file.

```csharp
var config = new ReplicatorConfiguration();
// 1: Get the stream to the data (if using embedded resource, otherwise you can get it
// via your own logic, or just pass the file path if available)
using (var certStream = GetType().GetTypeInfo().Assembly.GetManifestResourcesStream("localhost-wrong.cert"))
using (var reader = new BinaryReader(certStream)) {
	// 2 : Load the certificate
	var bytes = reader.Read(certStream.Length);
	// 3: Create an X509Certificate2
	var cert = new X509Certificate2(bytes);
	// 4: Set the certificate on the replicator configuration
	config.PinnedServerCertificate = cert;
}
```

This example loads the certificate from the application sandbox, then converts it to the appropriate type to configure the replication object.
