---
id: sg-release-notes
title: SG release notes
---

## 2.0.0

__Upgrading__

Starting in Sync Gateway 2.0, Sync Gateway’s design documents include the version number in the design document name. In this release for example, the design documents are named `_design/sync_gateway_2.0` and `_design/sync_housekeeping_2.0`.

On startup, Sync Gateway will check for the existence of these design documents, and only attempt to create them if they do not already exist. Then, Sync Gateway will wait until views are available and indexed before starting to serve requests. To evaluate this, Sync Gateway will issue a `stale=false&limit=1` query against the Sync Gateway views (channels, access and role_access).

If the view request exceeds the default timeout of 75s (which would be expected when indexing large buckets), Sync Gateway will log additional messages and retry. The logging output will look like this:

```bash
14:26:41.039-08:00 Design docs for current SG view version (2.0) found.
14:26:41.039-08:00 Verifying view availability for bucket default...
14:26:42.045-08:00 Timeout waiting for view "access" to be ready for bucket "default" - retrying...
14:26:42.045-08:00 Timeout waiting for view "channels" to be ready for bucket "default" - retrying...
14:26:42.045-08:00 Timeout waiting for view "role_access" to be ready for bucket "default" - retrying...
14:26:44.065-08:00 Timeout waiting for view "access" to be ready for bucket "default" - retrying...
14:26:44.065-08:00 Timeout waiting for view "role_access" to be ready for bucket "default" - retrying...
14:26:44.065-08:00 Timeout waiting for view "channels" to be ready for bucket "default" - retrying...
14:26:44.072-08:00 Views ready for bucket default.
```

Sync Gateway 2.0 will **not** automatically remove the previous design documents. Removal of the obsolete design documents is done via a call to the new  [`/{db}/_post_upgrade`](../admin-rest-api/index.html#/server/post__post_upgrade) endpoint in Sync Gateway’s Admin REST API. This endpoint can be run in preview mode (`?preview=true`) to see which design documents would be removed. To summarize, the steps to perform an upgrade to Sync Gateway 2.0 are:

1. Upgrade one node in the cluster to 2.0, and wait for it to be reachable via the REST API (for example at [http://localhost:4985/](http://localhost:4985/)).
2. Upgrade the rest of the nodes in the cluster.
3. Clean up obsolete views:
	- **Optional** Issue a call to `/_post_upgrade?preview=true` on any node to preview which design documents will be removed. To upgrade to 2.0, expect to see "sync_gateway" and "sync_housekeeping" listed.
	- Issue a call to `/post_upgrade` to remove the obsolete design documents. The response should indicate that "sync_gateway" and "sync_housekeeping" were removed.

## New Features

- No conflicts mode ([`databases.$db.allow\_conflicts`](../../../guides/sync-gateway/config-properties/index.html#2.0/databases-foo_db-allow_conflicts))
- Expiry value for local documents managed by Sync Gateway ([`databases.$db.local\_doc\_expiry\_secs`](../../../guides/sync-gateway/config-properties/index.html#2.0/databases-foo_db-local_doc_expiry_secs))

__Performance Improvements__

- [__#1383__](https://github.com/couchbase/sync_gateway/issues/1383) Nginx load balancer needs plugins to detect db offline state.
- [__#1850__](https://github.com/couchbase/sync_gateway/issues/1850) Avoid duplicate parsing of HTTP query string
- [__#2410__](https://github.com/couchbase/sync_gateway/issues/2410) SG gets into a 100% cpu state, where restart is the only recovery
- [__#2871__](https://github.com/couchbase/sync_gateway/issues/2871) Review bucket_gocb concurrent op limits
- [__#2928__](https://github.com/couchbase/sync_gateway/issues/2928) Optimize document unmarshalling for GetDoc

__Enhancements__

- [__#744__](https://github.com/couchbase/sync_gateway/issues/744) When try to put nonexistent document with rev: Document revision conflict
- [__#955__](https://github.com/couchbase/sync_gateway/issues/955) Support group reduce queries
- [__#1280__](https://github.com/couchbase/sync_gateway/issues/1280) Auto-expire unused _local checkpoint documents 
- [__#1580__](https://github.com/couchbase/sync_gateway/issues/1580) Use latest version of Otto
- [__#1720__](https://github.com/couchbase/sync_gateway/issues/1720) Replication inaccurate 403 forbidden
- [__#1850__](https://github.com/couchbase/sync_gateway/issues/1850) Avoid duplicate parsing of HTTP query string
- [__#2354__](https://github.com/couchbase/sync_gateway/issues/2354) Windows service wrapper should write to a configurable stderr log file
- [__#2709__](https://github.com/couchbase/sync_gateway/issues/2709) Conflict-free mode (allow_conflicts:false)
- [__#3123__](https://github.com/couchbase/sync_gateway/issues/3123) Log _sync:seq on startup
- [__#3136__](https://github.com/couchbase/sync_gateway/issues/3136) Inter-document compression in BLIP replicator
- [__#3168__](https://github.com/couchbase/sync_gateway/issues/3168) Move allow_conflicts out of unsupported config 

__Bugs__

- [__#1003__](https://github.com/couchbase/sync_gateway/issues/1003) SG with bucket shadowing doesn't work properly after CB restarted
- [__#1047__](https://github.com/couchbase/sync_gateway/issues/1047) Runtime panic while running tests on Jenkins
- [__#1406__](https://github.com/couchbase/sync_gateway/issues/1406) Feedtype=DCP not working sometimes
- [__#1488__](https://github.com/couchbase/sync_gateway/issues/1488) Rebalance -in a CBS node results in timeouts which manifests as empty changes feed
- [__#1720__](https://github.com/couchbase/sync_gateway/issues/1720) Replication inaccurate 403 forbidden
- [__#2108__](https://github.com/couchbase/sync_gateway/issues/2108) Missing sequences in _changes feed causing sg-replicate missing documents replication
- [__#2223__](https://github.com/couchbase/sync_gateway/issues/2223) Wrong name being set for users
- [__#2371__](https://github.com/couchbase/sync_gateway/issues/2371) Logging - "Replicate" works when in *.json but not when you PUT _logging
- [__#2383__](https://github.com/couchbase/sync_gateway/issues/2383) Channel cache missing data when request instantiating cache times out
- [__#2410__](https://github.com/couchbase/sync_gateway/issues/2410) SG gets into a 100% cpu state, where restart is the only recovery
- [__#2441__](https://github.com/couchbase/sync_gateway/issues/2441) RedHat/Centos 6 'initctl restart sync_gateway' does not work as expected
- [__#2458__](https://github.com/couchbase/sync_gateway/issues/2458) Log rotation skipping BLIP keys
- [__#2489__](https://github.com/couchbase/sync_gateway/issues/2489) XATTRs - Error unmarshalling doc
- [__#2497__](https://github.com/couchbase/sync_gateway/issues/2497) XATTRS: Getting doc after restart fails
- [__#2505__](https://github.com/couchbase/sync_gateway/issues/2505) XATTRs: View not returning results after restart
- [__#2521__](https://github.com/couchbase/sync_gateway/issues/2521) XATTRs: GET on doc after deletion returns inconsistent responses
- [__#2717__](https://github.com/couchbase/sync_gateway/issues/2717) SG Blip handler not reloading user channels
- [__#3048__](https://github.com/couchbase/sync_gateway/issues/3048) Panic when attempting to make invalid update to a conflicting document
- [__#3049__](https://github.com/couchbase/sync_gateway/issues/3049) Allow non-winning tombstone revisions when running with allow_conflicts=false