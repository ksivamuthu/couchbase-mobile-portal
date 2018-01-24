---
id: sg-release-notes
title: SG release notes
---

## New Features

- **Shared bucket access**
- **SSL and Multi-URL support**

## Notable Changes

- **Revs Limit lower limit:** The [databases.foo\_db.revs\_limit](../../../guides/sync-gateway/config-properties/index.html#1.5/databases-foo_db-revs_limit) property now has a minimal value. See the API reference for more detail.
- **Rev Tree endpoint:** The [/{db}/\_revtree/{doc}](../admin-rest-api/index.html?v=1.5#/document/get__db___revtree__doc_) endpoint returns the revision tree in dot syntax for the specified document. This endpoint is not officially supported and should only be used for troubleshooting and debugging purposes.

## 1.5.1

__Bugs__

- [__#3078__](https://github.com/couchbase/sync_gateway/issues/3078) Channel view results not being prepended to channel cache
- [__#3098__](https://github.com/couchbase/sync_gateway/issues/3098) Error migrating tombstoned documents: `Did not update document "DOC_ID" w/ xattr: invalid arguments`

## 1.5 release

__Performance Improvements__

- [__#2218__](https://github.com/couchbase/sync_gateway/issues/2218) SyncGateway/SyncGateway replicator memory keeps growing continuosly and resulting in OOM crash
- [__#2651__](https://github.com/couchbase/sync_gateway/issues/2651) SG gets killed due to excessive memory usage with continuous doc update

__Enhancements__

- [__#473__](https://github.com/couchbase/sync_gateway/issues/473) Implement Mobile Convergence to enable Sync Gateway to directly access Couchbase Server buckets
- [__#1462__](https://github.com/couchbase/sync_gateway/issues/1462) Sync Gateway Accelerator is reporting itself as Sync Gateway
- [__#2151__](https://github.com/couchbase/sync_gateway/issues/2151) The /{db}/\_changes?include_docs=true request shouldn't return _user/user docs
- [__#2259__](https://github.com/couchbase/sync_gateway/issues/2259) Add support for RHEL in service install scripts

__Bugs__

- [__#2218__](https://github.com/couchbase/sync_gateway/issues/2218) SyncGateway/SyncGateway replicator memory keeps growing continuosly and resulting in OOM crash
- [__#2367__](https://github.com/couchbase/sync_gateway/issues/2367) Omitting the logFilePath property in the logging configuration of Sync Gateway results in a null pointer exception crash
- [__#2381__](https://github.com/couchbase/sync_gateway/issues/2381) SG Replicate stops when an attachment with no content type is replicated
- [__#2400__](https://github.com/couchbase/sync_gateway/issues/2400) SG collect info is not case sensitive on the config's logFilePath property
- [__#2421__](https://github.com/couchbase/sync_gateway/issues/2421) Discard new rev sequence if it is less than a docs current rev sequence, to avoid issues with caching
- [__#2500__](https://github.com/couchbase/sync_gateway/issues/2500) Webhook event not fired when using bucket shadowing
- [__#2565__](https://github.com/couchbase/sync_gateway/issues/2565) Non-empty oldDoc being sent to sync function during import
- [__#2626__](https://github.com/couchbase/sync_gateway/issues/2626) Active changes feeds not notified when deferred sequences get cached
- [__#2651__](https://github.com/couchbase/sync_gateway/issues/2651) SG gets killed due to excessive memory usage with continuous doc update
- [__#2677__](https://github.com/couchbase/sync_gateway/issues/2677) Unable to install SG 1.5.0 on Win 10

__Known issues__

- [__#2744__](https://github.com/couchbase/sync_gateway/pull/2744) Failures due to multi operation deletes
- [__#149__](https://github.com/couchbaselabs/sync-gateway-accel/issues/149) Changes missing on Sync Gateway Accelerator re-shard
- [__#2068__](https://github.com/couchbase/sync_gateway/issues/2068) Update user doc sequence for access-based channel grants
- [__#160__](https://github.com/couchbaselabs/sync-gateway-accel/issues/160) The `_doc_ids` changes feed filter doesn't work with Accelerator