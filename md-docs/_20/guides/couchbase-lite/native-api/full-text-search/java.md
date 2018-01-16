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
database.createIndex("nameFTSIndex", Index.fullTextIndex(FullTextIndexItem.property("name")).ignoreAccents(false));
```

Multiple properties to index can be specified in the `Index.fullTextIndex(withItems: [FullTextIndexItem])` method.

With the index created, an FTS query on the property that is being indexed can be constructed and ran. The full-text search criteria is defined as a `FullTextExpression`. The left-hand side is the full-text index to use and the right-hand side is the pattern to match: usually a word or a space-separated list of words, but it can be a more powerful [FTS4 search expression](https://www.sqlite.org/fts3.html#full_text_index_queries). The following code example matches all documents that contain the word 'buy' in the value of the `name` property.

```java
Expression whereClause = FullTextExpression.index("nameFTSIndex").match("buy");
Query ftsQuery = Query.select(SelectResult.expression(Meta.id))
        .from(DataSource.database(database))
        .where(whereClause);
ResultSet ftsQueryResult = ftsQuery.execute();
for(Result result : ftsQueryResult)
    Log.i(TAG, String.format("document properties %s", result.getString(0)));
```

It's very common to sort full-text results in descending order of relevance. This can be a very difficult heuristic to define, but Couchbase Lite comes with a fairly simple ranking function you can use. In the `OrderBy` array, use a string of the form `Rank(X)`, where `X` is the property or expression being searched, to represent the ranking of the result.