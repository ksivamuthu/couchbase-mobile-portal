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
database.CreateIndex("nameFTSIndex", Index.FullTextIndex(FullTextIndexItem.Property("name")).IgnoreAccents(false));
```

Multiple properties to index can be specified in the `Index.FullTextIndex(params FullTextIndexItem[] items)` method.

With the index created, an FTS query on the property that is being indexed can be constructed and ran. The full-text search criteria is defined as a `FullTextExpression`. The left-hand side is the full-text index to use and the right-hand side is the pattern to match: usually a word or a space-separated list of words, but it can be a more powerful [FTS4 search expression](https://www.sqlite.org/fts3.html#full_text_index_queries). The following code example matches all documents that contain the word 'buy' in the value of the `name` property.

```c#
var whereClause = FullTextExpression.Index("nameFTSIndex").Match("'buy'");
var ftsQuery = Query.Select(SelectResult.Expression(Meta.ID))
                  .From(DataSource.Database(database))
                  .Where(whereClause);

using(var ftsQueryResult = ftsQuery.Execute()) {
    foreach(var row in ftsQueryResult) {
	Console.WriteLine($"document properties ${row.GetString(0)}");
    }
}
```

It's very common to sort full-text results in descending order of relevance. This can be a very difficult heuristic to define, but Couchbase Lite comes with a fairly simple ranking function you can use. In the `OrderBy` array, use a string of the form `Rank(X)`, where `X` is the property or expression being searched, to represent the ranking of the result.
