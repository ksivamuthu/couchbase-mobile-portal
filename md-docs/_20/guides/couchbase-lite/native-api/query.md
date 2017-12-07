---
permalink: guides/couchbase-lite/native-api/query/index.html
---

<block class="java" />

⚠ Support in the current Developer Build is for Android only. The SDK cannot be used in Java applications.

<block class="all" />

Database queries have changed significantly. Instead of the map/reduce algorithm used in 1.x, they're now based on expressions, of the form "return ____ from documents where \_\_\_\_, ordered by \_\_\_\_", with semantics based on Couchbase Server's N1QL query language. If you've used {% tx Core Data|Core Data|LINQ|LINQ %}, or other query APIs based on SQL, you'll find this familiar.

There are several parts to specifying a query:

- SELECT: specifies the projection, which is the part of the document that is to be returned.
- FROM: specifies the database to query the documents from.
- JOIN: specifies the matching criteria in which to join multiple documents.
- WHERE: specifies the query criteria that the result must satisfy.
- GROUP BY: specifies the query criteria to group rows by.
- ORDER BY: specifies the query criteria to sort the rows in the result.

## Indexing

Before we begin querying documents, let's briefly mention the importance of having a query index. A query can only be fast if there's a pre-existing database index it can search to narrow down the set of documents to examine.

The following example create a new index for the `type` and `name` properties.

```json
{
    "_id": "hotel123",
    "type": "hotel",
    "name": "Apple Droid"
}
```

<block class="swift" />

```swift
database.createIndex(Index.valueIndex().on(
                ValueIndexItem.expression(Expression.property("type")),
                ValueIndexItem.expression(Expression.property("name"))),
         withName: "TypeNameIndex")
```

<block class="net" />

```csharp
database.CreateIndex("TypeNameIndex", Index.ValueIndex().On(
	ValueIndexItem.Expression(Expression.Property("type")),
        ValueIndexItem.Expression(Expression.Property("name"))));
```

<block class="java" />

```java
database.createIndex("TypeNameIndex",
        Index.valueIndex(ValueIndexItem.property("type"),
                ValueIndexItem.property("name")));
```

<block class="all" />

If there are multiple expressions, the first one will be the primary key, the second the secondary key, etc.

> **Note:** Every index has to be updated whenever a document is updated, so too many indexes can hurt performance. Thus, good performance depends on designing and creating the *right* indexes to go along with your queries.

## SELECT statement

With the SELECT statement, you can query and manipulate JSON data. With projections, you retrieve just the fields that you need and not the entire document. This is especially useful when querying for a large dataset as it results in shorter processing times and better performance.

A SelectResult represents a single return value of the query statement. Documents in Couchbase Lite comprise of the document properties specified as a Dictionary of Key-Value pairs and associated metadata. The metadata consists of document Id and sequence Id associated with the Document. When you query for a document, the document metadata is not returned by default. You will need to explicitly query for the metadata.

- `SelectResult.all()`: Returns all properties associated with a Document.
- `SelectResult(Expression)`: Returns properties of a Document based on the Expression.
- `SelectResult.expression(Expression.meta().id)`: Return Document Id.
- `SelectResult.expression(Expression.meta().sequence)`: Returns sequence Id (used in replications).

You can specify a comma separated list of `SelectResult` expressions in the select statement of your Query. For instance the following select statement queries for the document `_id` as well as the `type` and `name` properties of all documents in the database. In the query result, we print the `_id` and `name` properties of each row using the property name getter method.

```json
{
	"_id": "hotel123",
	"type": "hotel",
	"name": "Apple Droid"
}
```

<block class="swift" />

```swift
let query = Query
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

<block class="objc" />

```objectivec
CBLQuery* query = [CBLQuery select:@[[CBLQueryExpression property:@"name"]]
                              from:[CBLQueryDataSource database:database]
                             where:[
                                    [[CBLQueryExpression property:@"type"] equalTo:@"user"]
                                    and: [[CBLQueryExpression property:@"admin"] equalTo:@FALSE]]];

NSEnumerator* rows = [query run:&error];
for (CBLQueryRow *row in rows) {
    NSLog(@"user name :: %@", [row stringAtIndex:0]);
}
```

<block class="csharp" />

```csharp
using(var query = Query.Select(SelectResult.Expression(Expression.Property("name")))
	.From(DataSource.Database(database))
	.Where(
		Expression.Property("type").EqualTo("user")
		.And(Expression.Property("admin").EqualTo(false))
	))
using(var rows = query.Run()) {
    foreach(var row in rows)
    {
	Console.WriteLine($"user name :: ${row.GetString(0)}");
    }
}
```

<block class="java" />

```java
Query query = Query
        .select(SelectResult.expression(Meta.id),
                SelectResult.property("name"),
                SelectResult.property("type"))
        .from(DataSource.database(database))
        .where(Expression.property("type").equalTo("hotel"))
        .orderBy(Ordering.expression(Meta.id));;
try {
    ResultSet rs = query.execute();
    for (Result result : rs) {
        Log.i("Sample", String.format("hotel id :: %s", result.getString("_id")));
        Log.i("Sample", String.format("hotel name :: %s", result.getString("name")));
    }
} catch (CouchbaseLiteException e) {
    Log.e("Sample", e.getLocalizedMessage());
}
```

<block class="all" />

The `SelectResult.all()` method can be used to query all the properties of a document. In this case, the document in the result is embedded in a dictionary where the key is the database name. The following snippet shows the same query using `SelectResult.all()` and the result in JSON.

<block class="swift" />

```swift
let query = Query
	.select(SelectResult.all())
	.from(DataSource.database(database))
```

<block class="objc" />



<block class="csharp" />

```csharp
var query = Query
    .Select(SelectResult.All())
    .From(DataSource.Database(database));
```

<block class="java" />
```java
Query query = Query
        .select(SelectResult.all())
        .from(DataSource.database(database))
        .where(Expression.property("type").equalTo("hotel"));
    ResultSet rs = query.execute();
    for (Result result : rs)
        Log.i("Sample", String.format("hotel -> %s", result.getDictionary(DATABASE_NAME).toMap()));
```


<block class="all" />

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

<block class="all" />

## WHERE statement

Similar to SQL, you can use the where clause to filter the documents to be returned as part of the query. The select statement takes in an `Expression`. You can chain any number of Expressions in order to implement sophisticated filtering capabilities.

### Comparison

In the example below, we use `Expression.property` type in conjunction with `equalTo` comparison expression to filter documents based on a specific document property.

```json
{
	"_id": "hotel123",
	"type": "hotel",
	"name": "Apple Droid"
}
```

<block class="swift" />

```swift
let query = Query
	.select(SelectResult.all())
	.from(DataSource.database(database))
	.where(Expression.property("type").equalTo("hotel"))
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

<block class="objc" />



<block class="csharp" />

```csharp
using(var query = Query.Select(SelectResult.Expression(Expression.Property("name")))
	.From(DataSource.Database(database))
	.Where(Expression.Property("type").EqualTo("hotel"))
	.Limit(10))
using(var rows = query.Run()) {
    foreach(var row in rows)
    {
        Console.WriteLine($"user name :: ${row.GetString("name")}");
    }
}
```

<block class="java" />
```java
Query query = Query
        .select(SelectResult.all())
        .from(DataSource.database(database))
        .where(Expression.property("type").equalTo("hotel"))
        .limit(10);
ResultSet rs = query.execute();
for (Result result : rs) {
    Dictionary all = result.getDictionary(DATABASE_NAME);
    Log.i("Sample", String.format("name -> %s", all.getString("name")));
    Log.i("Sample", String.format("type -> %s", all.getString("type")));
}
```


<block class="all" />

The list of supported comparison operators include, among others, `lessThan`, `lessThanOrEqualTo`, `greaterThan`, `greaterThanOrEqualTo`, `equalTo`, `notEqualTo`.

### Collection Operators

Collection operators are useful to check if a given value is present in an array. The following example uses the `Function.arrayContains` to find documents whose `public_likes` array property contain a value equal to "Armani Langworth".

```json
{
	"_id": "hotel123",
	"name": "Apple Droid",
	"public_likes": ["Armani Langworth", "Elfrieda Gutkowski", "Maureen Ruecker"]
}
```

<block class="swift" />

```swift
let query = Query
	.select(
		SelectResult.expression(Expression.meta().id),
		SelectResult.expression(Expression.property("name")),
		SelectResult.expression(Expression.property("public_likes"))
	)
	.from(DataSource.database(database))
	.where(Expression.property("type").equalTo("hotel")
		.and(Function.arrayContains(Expression.property("public_likes"), value: "Armani Langworth"))
	)

do {
	for row in try query.run() {
		print("public_likes :: \(row.array(forKey: "public_likes")?.toArray())")
	}
}
```

<block class="objc" />



<block class="csharp" />

```csharp
using(var query = Query.Select(
        SelectResult.Expression(Expression.Meta().ID),
	SelectResult.Expression(Expression.Property("name")),
	SelectResult.Expression(Expression.Property("public_likes")))
	.From(DataSource.Database(database))
	.Where(Expression.Property("type").EqualTo("hotel")
	    .And(Function.ArrayContains(Expression.Property("public_likes"), "Armani Langworth"))))
using(var rows = query.Run()) {
    foreach(var row in rows)
    {
        // Serialize the array first to get meaningful string output
        Console.WriteLine($"public_likes :: ${row.GetArray("public_likes")}");
    }
}
```

<block class="java" />
```java
Query query = Query
        .select(SelectResult.expression(Meta.id),
                SelectResult.property("name"),
                SelectResult.property("public_likes"))
        .from(DataSource.database(database))
        .where(Expression.property("type").equalTo("hotel")
                .and(ArrayFunction.contains(Expression.property("public_likes"), "Armani Langworth")));
ResultSet rs = query.execute();
for (Result result : rs)
    Log.i("Sample", String.format("public_likes -> %s", result.getArray("public_likes").toList()));
```


<block class="all" />

### Pattern Matching

The `like` and `regex` expressions can be used for string matching. The former is typically used for case insensitive matches while the `regex` expressions is used for case sensitive matches.

In the example below, we are looking for documents of type `landmark` where the name property exactly matches the string "Royal engineers museum". Note that since `like` does a case insensitive match, the following query will return "landmark" type documents with name matching "Royal Engineers Museum", "royal engineers museum", "ROYAL ENGINEERS MUSEUM" and so on.

<block class="swift" />

```swift
let query = Query
	.select(
		SelectResult.expression(Expression.meta().id),
		SelectResult.expression(Expression.property("country")),
		SelectResult.expression(Expression.property("name"))
	)
	.from(DataSource.database(db))
	.where(Expression.property("type").equalTo("landmark")
		.and( Expression.property("name").like("Royal engineers museum"))
	)
	.limit(10)

do {
	for row in try query.run() {
		print("name property :: \(row.string(forKey: "name")!)")
	}
}
```

<block class="objc" />



<block class="csharp" />


```csharp
using(var query = Query.Select(
        SelectResult.Expression(Expression.Meta().ID),
	SelectResult.Expression(Expression.Property("country")),
	SelectResult.Expression(Expression.Property("name")))
	.From(DataSource.Database(database))
	.Where(Expression.Property("type").EqualTo("landmark")
	    .And(Expression.Property("name").Like("Royal engineers museum")))
	.Limit(10))
using(var rows = query.Run()) {
    foreach(var row in rows)
    {
        // Serialize the array first to get meaningful string output
        Console.WriteLine($"public_likes :: ${row.GetArray("public_likes")}");
    }
}
```

<block class="java" />
```java
Query query = Query
        .select(SelectResult.expression(Meta.id),
                SelectResult.property("country"),
                SelectResult.property("name"))
        .from(DataSource.database(database))
        .where(Expression.property("type").equalTo("landmark")
                .and(Expression.property("name").like("Royal engineers museum")));
ResultSet rs = query.execute();
for (Result result : rs)
    Log.i("Sample", String.format("name -> %s", result.getString("name")));
```


<block class="all" />

### Wildcard Match

We can use `%` sign within a `like` expression to do a wildcard match against zero or more characters. Using wildcards allows you to have some fuzziness in your search string.

In the example below, we are looking for documents of `type` "landmark" where the name property matches any string that begins with "eng" followed by zero or more characters, the letter "e", followed by zero or more characters. The following query will return "landmark" type documents with name matching "Engineers", "engine", "english egg" , "England Eagle" and so on. Notice that the matches may span word boundaries.

<block class="swift" />

```swift
let query = Query
	.select(
		SelectResult.expression(Expression.meta().id),
		SelectResult.expression(Expression.property("country")),
		SelectResult.expression(Expression.property("name"))
	)
	.from(DataSource.database(db))
	.where(Expression.property("type").equalTo("landmark")
		.and( Expression.property("name").like("eng%e%")))
	.limit(limit)
```

<block class="objc" />



<block class="csharp" />

```csharp
var query = Query.Select(
    SelectResult.Expression(Expression.Meta().ID)
    SelectResult.Expression(Expression.Property("country")),
    SelectResult.Expression(Expression.Property("name")))
	.From(DataSource.Database(database))
	.Where(Expression.Property("type").EqualTo("landmark")
	    .And(Expression.Property("name").Like("eng%e%")))
	.Limit(limit)
```

<block class="java" />
```java
Query query = Query
        .select(SelectResult.expression(Meta.id),
                SelectResult.property("country"),
                SelectResult.property("name"))
        .from(DataSource.database(database))
        .where(Expression.property("type").equalTo("landmark")
                .and(Expression.property("name").like("%eng%e%")));
ResultSet rs = query.execute();
for (Result result : rs)
    Log.i("Sample", String.format("name -> %s", result.getString("name")));
```


<block class="all" />

### Wildcard Character Match

We can use `_` sign within a like expression to do a wildcard match against a single character.

In the example below, we are looking for documents of type "landmark" where the `name` property matches any string that begins with "eng" followed by exactly 4 wildcard characters and ending in the letter "r".
The following query will return "landmark" `type` documents with the `name` matching "Engineer", "engineer" and so on.

<block class="swift" />

```swift
let query = Query
	.select(SelectResult.expression(Expression.meta().id),
		SelectResult.expression(Expression.property("country")),
		SelectResult.expression(Expression.property("name")))
	.from(DataSource.database(db))
	.where(Expression.property("type").equalTo("landmark")
		.and(Expression.property("name").like("eng____r")))
	.limit(limit)
```

<block class="objc" />



<block class="csharp" />

```csharp
var query = Query.Select(
    SelectResult.Expression(Expression.Meta().ID),
    SelectResult.Expression(Expression.Property("country")),
    SelectResult.Expression(Expression.Property("name")))
    .From(DataSource.Database(database))
    .Where(Expression.Property("type").EqualTo("landmark")
        .And(Expression.Property("name").Like("eng____r")))
    .Limit(limit))
```

<block class="java" />
```java
Query query = Query
        .select(SelectResult.expression(Meta.id),
                SelectResult.property("country"),
                SelectResult.property("name"))
        .from(DataSource.database(database))
        .where(Expression.property("type").equalTo("landmark")
                .and(Expression.property("name").like("eng____r")));
ResultSet rs = query.execute();
for (Result result : rs)
    Log.i("Sample", String.format("name -> %s", result.getString("name")));
```


<block class="all" />

### Regex Match

The `regex` expression can be used for case sensitive matches. Similar to wildcard `like` expressions, `regex` expressions based pattern matching allow you to have some fuzziness in your search string.

In the example below, we are looking for documents of `type` "landmark" where the name property matches any string (on word boundaries) that begins with "eng" followed by exactly 4 wildcard characters and ending in the letter "r".
The following query will return "landmark" type documents with name matching "Engine", "engine" and so on. Note that the `\b` specifies that the match must occur on word boundaries.

<block class="swift" />

```swift
let query = Query
	.select(
		SelectResult.expression(Expression.meta().id),
		SelectResult.expression(Expression.property("name"))
	)
	.from(DataSource.database(db))
	.where(Expression.property("type").equalTo("landmark")
		.and(Expression.property("name").regex("\\bEng.*e\\b")))
	.limit(limit)
```

<block class="objc" />



<block class="csharp" />

```csharp
var query = Query.Select(
    SelectResult.Expression(Expression.Meta().ID),
    SelectResult.Expression(Expression.Property("name")))
    .From(DataSource.Database(db))
    .Where(Expression.Property("type").EqualTo("landmark")
        .And(Expression.Property("name").Regex("\\bEng.*e\\b")))
    .Limit(limit))
```

<block class="java" />
```java
Query query = Query
        .select(SelectResult.expression(Meta.id),
                SelectResult.property("country"),
                SelectResult.property("name"))
        .from(DataSource.database(database))
        .where(Expression.property("type").equalTo("landmark")
                .and(Expression.property("name").regex("\\bEng.*r\\b")));
ResultSet rs = query.execute();
for (Result result : rs)
    Log.i("Sample", String.format("name -> %s", result.getString("name")));
```


<block class="all" />

## JOIN statement

The JOIN clause enables you to create new input objects by combining two or more source objects.

The following example uses a JOIN clause to find the airline details which have routes that start from RIX. This example JOINS the document of type "route" with documents of type "airline" using the document ID (`_id`) on the "airline" document and  `airlineid` on the "route" document.

<block class="swift" />

```swift
let query = Query.select(
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
	Expression.property("type").from("route").equalTo("route")
	.and(Expression.property("type").from("airline").equalTo("airline"))
	.and(Expression.property("sourceairport").from("route").equalTo("RIX"))
)
```

<block class="objc" />



<block class="csharp" />

```csharp
var query = Query.Select(
        SelectResult.Expression(Expression.Property("name").From("airline")),
        SelectResult.Expression(Expression.Property("callsign").From("airline")),
        SelectResult.Expression(Expression.Property("destinationairport").From("route")),
        SelectResult.Expression(Expression.Property("stops").From("route")),
        SelectResult.Expression(Expression.Property("airline").From("route")))
    .From(DataSource.Database(database).As("airline"))
    .Joins(Join.DefaultJoin(DataSource.Database(database).As("route"))
        .On(Expression.Meta().ID.From("airline").EqualTo(Expression.Property("airlineid").From("route"))))
    .Where(
        Expression.Property("type").From("route").EqualTo("route")
	    .And(Expression.Property("type").From("airline").EqualTo("airline"))
	    .And(Expression.Property("sourceairport").From("route").EqualTo("RIX")))
```

<block class="java" />

```java
            Query query = Query.select(
                    SelectResult.expression(Expression.property("name").from("airline")),
                    SelectResult.expression(Expression.property("callsign").from("airline")),
                    SelectResult.expression(Expression.property("destinationairport").from("route")),
                    SelectResult.expression(Expression.property("stops").from("route")),
                    SelectResult.expression(Expression.property("airline").from("route")))
                    .from(DataSource.database(database).as("airline"))
                    .join(Join.join(DataSource.database(database).as("route"))
                            .on(Meta.id.from("airline").equalTo(Expression.property("airlineid").from("route"))))
                    .where(Expression.property("type").from("route").equalTo("route")
                            .and(Expression.property("type").from("airline").equalTo("airline"))
                            .and(Expression.property("sourceairport").from("route").equalTo("RIX")));
            ResultSet rs = query.execute();
            for (Result result : rs)
                Log.w("Sample", String.format("%s", result.toMap().toString()));
```


<block class="all" />

## GROUP BY statement

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

<block class="swift" />

```swift
let query = Query.select(
	SelectResult.expression(Function.count("*")),
	SelectResult.expression(Expression.property("country")),
	SelectResult.expression(Expression.property("tz"))
	)
	.from(DataSource.database(database))
	.where(
		Expression.property("type").equalTo("airport")
				.and(Expression.property("geo.alt").greaterThanOrEqualTo(300))
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

<block class="objc" />



<block class="csharp" />

```csharp
using(var query = Query.Select(
        SelectResult.Expression(Function.Count("*")),
	SelectResult.Expression(Expression.Property("country")),
	SelectResult.Expression(Expression.Property("tz")))
    .From(DataSource.Database(database))
    .Where(
        Expression.Property("type").EqualTo("airport")
	    .And(Expression.Property("geo.alt").GreaterThanOrEqualTo(300)))
    .GroupBy(
        Expression.Property("country"),
	Expression.Property("tz")))
using(var rows = query.Run()) {
    foreach(var row in rows) {
        Console.WriteLine($"There are {row.GetInt("$1")} airports on the {row.GetString("tz")} timezone located in " +
	    $"{row.GetString("country")} and above 300 ft");
    }
}
```

<block class="java" />
```java
Query query = Query.select(
        SelectResult.expression(Function.count("*")),
        SelectResult.property("country"),
        SelectResult.property("tz"))
        .from(DataSource.database(database))
        .where(Expression.property("type").equalTo("airport")
                .and(Expression.property("geo.alt").greaterThanOrEqualTo(300)))
        .groupBy(Expression.property("country"),
                Expression.property("tz"))
        .orderBy(Ordering.expression(Function.count("*")).descending());
ResultSet rs = query.execute();
for (Result result : rs)
    Log.i("Sample",
            String.format("There are %d airports on the %s timezone located in %s and above 300ft",
                    result.getInt("$1"),
                    result.getString("tz"),
                    result.getString("country")));
```


<block class="all" />

```text
There are 138 airports on the Europe/Paris timezone located in France and above 300 ft
There are 29 airports on the Europe/London timezone located in United Kingdom and above 300 ft
There are 50 airports on the America/Anchorage timezone located in United States and above 300 ft
There are 279 airports on the America/Chicago timezone located in United States and above 300 ft
There are 123 airports on the America/Denver timezone located in United States and above 300 ft
```

<block class="all" />

## ORDER BY statement

It is possible to sort the results of a query based on a given expression result. The example below returns documents of type equal to "hotel" sorted in ascending order by the value of the title property.

<block class="swift" />

```swift
let query = Query
	.select(
		SelectResult.expression(Expression.meta().id),
		SelectResult.expression(Expression.property("title")))
	.from(DataSource.database(database))
	.where(Expression.property("type").equalTo("hotel"))
	.orderBy(Ordering.property("title").ascending())
	.limit(limit)
```

<block class="objc" />



<block class="csharp" />

```csharp
var query = Query.Select(
        SelectResult.Expression(Expression.Meta().ID),
	SelectResult.Expression(Expression.Property("title")))
    .From(DataSource.Database(database))
    .Where(Expression.Property("type").EqualTo("hotel"))
    .OrderBy(Ordering.Property("title").Ascending())
    .Limit(limit);
```

<block class="java" />
```java
Query query = Query
        .select(SelectResult.expression(Meta.id),
                SelectResult.property("name"))
        .from(DataSource.database(database))
        .where(Expression.property("type").equalTo("hotel"))
        .orderBy(Ordering.property("name").ascending())
        .limit(10);
ResultSet rs = query.execute();
for (Result result : rs)
    Log.i("Sample", String.format("%s", result.toMap()));
```

<block class="all" />

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

<block class="all" />

## Live Query

A live query stays active and monitors the database and query index for changes. When there's a change it re-runs itself automatically, and if the query results changed it notifies any observers.
