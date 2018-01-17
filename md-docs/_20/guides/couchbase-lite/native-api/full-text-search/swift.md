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
	try database.createIndex(Index.fullTextIndex(withItems: FullTextIndexItem.property("name")).ignoreAccents(false), withName: "nameFTSIndex")
} catch let error {
	print(error.localizedDescription)
}
```

Multiple properties to index can be specified in the `Index.fullTextIndex(withItems: [FullTextIndexItem])` method.

With the index created, an FTS query on the property that is being indexed can be constructed and ran. The full-text search criteria is defined as a `FullTextExpression`. The left-hand side is the full-text index to use and the right-hand side is the pattern to match: usually a word or a space-separated list of words, but it can be a more powerful [FTS4 search expression](https://www.sqlite.org/fts3.html#full_text_index_queries). The following code example matches all documents that contain the word 'buy' in the value of the `name` property.

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

It's very common to sort full-text results in descending order of relevance. This can be a very difficult heuristic to define, but Couchbase Lite comes with a fairly simple ranking function you can use. In the `OrderBy` array, use a string of the form `Rank(X)`, where `X` is the property or expression being searched, to represent the ranking of the result.