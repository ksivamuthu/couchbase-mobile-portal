---
id: sg-release-notes
title: SG release notes
---

## 2.0.0

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