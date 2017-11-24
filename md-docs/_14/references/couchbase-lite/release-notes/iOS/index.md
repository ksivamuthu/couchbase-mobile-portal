---
id: ios-release-notes
title: iOS release notes
---

__Deprecation notices__

The following features are being deprecated in Couchbase Mobile 1.4 and will be unsupported in 2.0.

- ForestDB
- Geo queries

The following platforms are being deprecated in Couchbase Mobile 1.4 and will be unsupported in 2.0.

- iOS 8

## 1.4.1 release

__Performance Improvements__

- [__#1648__](https://github.com/couchbase/couchbase-lite-ios/issues/1648) Purge does not remove row from 'docs' table
- [__#1737__](https://github.com/couchbase/couchbase-lite-ios/issues/1737) View indexing is very slow, due to mis-optimized SQLite query
- [__#1751__](https://github.com/couchbase/couchbase-lite-ios/issues/1751) Push replication becomes slower as local DB grows

__Bugs__

- [__#1354__](https://github.com/couchbase/couchbase-lite-ios/issues/1354) Deferred pulling of attachments problem
- [__#1572__](https://github.com/couchbase/couchbase-lite-ios/issues/1572) Idle crash [CBLRemoteSession close]
- [__#1613__](https://github.com/couchbase/couchbase-lite-ios/issues/1613) Crash in askDelegateToValidateServerTrust
- [__#1622__](https://github.com/couchbase/couchbase-lite-ios/issues/1622) Error opening!: 23 - when trying to load data from iOS to Apple Watch
- [__#1649__](https://github.com/couchbase/couchbase-lite-ios/issues/1649) Missing role appears to cause continuous push replication to stop [v1.4]
- [__#1655__](https://github.com/couchbase/couchbase-lite-ios/issues/1655) Fatal Exception: NSInternalInconsistencyException [1.4]
- [__#1707__](https://github.com/couchbase/couchbase-lite-ios/issues/1707) Replication Stops working after lots of syncing / lots of time - Too much memory used during replication
- [__#1758__](https://github.com/couchbase/couchbase-lite-ios/issues/1758) Updating a design doc view map function does not take effect
- [__#1770__](https://github.com/couchbase/couchbase-lite-ios/issues/1770) Crash observed while saving documents to DB
- [__#1779__](https://github.com/couchbase/couchbase-lite-ios/issues/1779) IOS App Extension replication gets suspended "prematurely"
- [__#1853__](https://github.com/couchbase/couchbase-lite-ios/issues/1853) CBL_BlobStoreWriter appendData assertion failure
- [__#1887__](https://github.com/couchbase/couchbase-lite-ios/issues/1887) 1.4 Encrypting a new database fails. Existing database encrypt fine.
- [__#1898__](https://github.com/couchbase/couchbase-lite-ios/issues/1898) One shot sync not pulling data properly in background
- [__#1912__](https://github.com/couchbase/couchbase-lite-ios/issues/1912) Peer sync between High Sierra and iOS 11 CBLWarnUntrustedCert error
- [__#1918__](https://github.com/couchbase/couchbase-lite-ios/issues/1918) Channel removal potentially disrupts attachments
- [__#1921__](https://github.com/couchbase/couchbase-lite-ios/issues/1921) CBLListener retaining CBL_RunLoopServer so both never get freed.
- [__#1923__](https://github.com/couchbase/couchbase-lite-ios/issues/1923) CBLRemoteSession finishTasksAndInvalidate crash

## 1.4 release

__API change__

- An `isDeletion` property is now available on the `DatabaseChange` object to identify if a change is tombstone document.

__Performance Improvements__

- [__#1389__](https://github.com/couchbase/couchbase-lite-ios/issues/1389) Allow SQLite reads concurrently with writer
- [__#1497__](https://github.com/couchbase/couchbase-lite-ios/issues/1497) Update ForestDB to get improved compaction / space reuse

__Enhancements__

- [__#1538__](https://github.com/couchbase/couchbase-lite-ios/issues/1538) Make CBLDatabaseChange.isDeletion public?

__Bugs__

- [__#794__](https://github.com/couchbase/couchbase-lite-ios/issues/794) "!_changeTracker" assertion failure in -[CBLRestPuller startChangeTracker]
- [__#1207__](https://github.com/couchbase/couchbase-lite-ios/issues/1207) Push replication stops with error `400 bad_request`
- [__#1243__](https://github.com/couchbase/couchbase-lite-ios/issues/1243) Replication task missing source or target values
- [__#1285__](https://github.com/couchbase/couchbase-lite-ios/issues/1285) REST API does not allow GET on a db with "/" in its name
- [__#1343__](https://github.com/couchbase/couchbase-lite-ios/issues/1343) Need to close all other db handles before rekeying
- [__#1365__](https://github.com/couchbase/couchbase-lite-ios/issues/1365) Non-continuous push replication gets stuck in kCBLReplicationIdle status
- [__#1422__](https://github.com/couchbase/couchbase-lite-ios/issues/1422) ForestDB crashes frequently in the background because of protection data level
- [__#1429__](https://github.com/couchbase/couchbase-lite-ios/issues/1429) 1.3.1-6: crashed EXC_BAD_ACCESS KERN_INVALID_ADDRESS 0x0000000000000000
- [__#1442__](https://github.com/couchbase/couchbase-lite-ios/issues/1442) Collection class properties not propagating from CBLDocument to CBLModel after sync
- [__#1443__](https://github.com/couchbase/couchbase-lite-ios/issues/1443) Handle iOS file protection gracefully when in background
- [__#1461__](https://github.com/couchbase/couchbase-lite-ios/issues/1461) ForestDB: Assertion failure in -[MYBackgroundMonitor beginBackgroundTaskNamed:]() MYBackgroundMonitor.m:66
- [__#1467__](https://github.com/couchbase/couchbase-lite-ios/issues/1467) Memory leak caused by ref cycle between replicator and CBLRemoteSession
- [__#1556__](https://github.com/couchbase/couchbase-lite-ios/issues/1556) Assertion failed: (_changesTimeout > 0)
- [__#1557__](https://github.com/couchbase/couchbase-lite-ios/issues/1557) Exception caught in CBLDatabase transaction
- [__#1558__](https://github.com/couchbase/couchbase-lite-ios/issues/1558) Not all updates propagating while under load
- [__#1568__](https://github.com/couchbase/couchbase-lite-ios/issues/1568) Attachments failing to push after delete / create (ForestDB)
- [__#1579__](https://github.com/couchbase/couchbase-lite-ios/issues/1579) [NSLock lock]: deadlock when deleting database (ForestDB)
- [__#1580__](https://github.com/couchbase/couchbase-lite-ios/issues/1580) Wrong "reason" response for GET on purged documents
- [__#1621__](https://github.com/couchbase/couchbase-lite-ios/issues/1621) OIDC failing with Grocery Sync when creating new google project

## Where to get it

You can download this release from the [downloads page](http://www.couchbase.com/nosql-databases/downloads#couchbase-mobile).