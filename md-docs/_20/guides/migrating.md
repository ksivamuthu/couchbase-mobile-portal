---
permalink: guides/migrating/index.html
---

## Opening 1.x Databases

Databases that were created with Couchbase Mobile 1.2 or later can be read using the 2.0 API. Upon detecting it is a 1.x database file format, Couchbase Lite will automatically upgrade it to the new format used in 2.0. This feature is currently only available for the default storage type, SQLite (i.e not for ForestDB databases).

## Sync Gateway compatibility

⚠️ The new protocol is **incompatible** with Couchbase Lite 1.x, and with CouchDB-based databases including PouchDB and Cloudant. Since Couchbase Lite 2 developer builds support only the new protocol, to test replication you will need to run a version of Sync Gateway that supports it.

To use this protocol with Couchbase Lite 2.0, the replication URL should specify **blip** as the URL scheme (see the [Replication API](index.html#replication-api) section below). Mobile clients using Couchbase Lite 1.x can continue to use **http** as the URL scheme. Sync Gateway 1.5 will automatically use the 1.x replication protocol when a Couchbase Lite 1.x client connects through "http://localhost:4984/db" and the 2.0 replication protocol when a Couchbase Lite 2.0 client connects through "blip://localhost:4984/db".