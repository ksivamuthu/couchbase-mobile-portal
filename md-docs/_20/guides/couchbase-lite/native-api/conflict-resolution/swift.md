

### Conflict Handling

We're approaching conflict handling differently, and more directly. Instead of requiring application code to go out of its way to find conflicts and look up the revisions involved, Couchbase Lite will detect the conflict (while saving a document, or during replication) and invoke an app-defined conflict-resolver handler. The conflict resolver is given "source" document properties, "target" document properties, and (if available) the properties of the common ancestor revision.

* When saving a `Document`, "target" properties will be the in-memory properties of the object, and "source" properties will be one ones already saved in the database (by some other application thread, or by the replicator.)
* During replication, "target" properties will be the ones in the local database, and "source" properties will be the ones coming from the server.

The resolver is responsible for returning the resulting properties that should be saved. There are of course a lot of ways to do this. By the time 2.0 is released we want to include some resolver implementations for common algorithms (like the popular "last writer wins" that just returns "my" properties.) The resolver can also give up by returning {% st nil|nil|null|null %}, in which case the save fails with a "conflict" error. This can be appropriate if the merge needs to be done interactively or by user intervention.

If the database doesn't have a conflict resolver (the default situation), a default algorithm is used that picks the revision with the larger number of changes in its history.