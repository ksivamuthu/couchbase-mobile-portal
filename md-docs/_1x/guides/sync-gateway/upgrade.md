---
permalink: guides/sync-gateway/upgrade/index.html
---

This section is an overview of the different options to upgrade a running cluster to the latest version of Sync Gateway and Couchbase Server. For a complete list of instructions, we recommend to follow the [upgrade section](http://docs.couchbase.com/tutorials/travel-sample/deploy/centos#/0/4/0) in the travel sample tutorial. You will learn how to perform a rolling upgrade and enable the shared bucket access introduced in Sync Gateway 1.5 in order to use N1QL, Mobile and Server SDKs on the same bucket.

## Sync Gateway

In each of the scenarios described below, the upgrade process will trigger views in Couchbase Server to be re-indexed. During the re-indexing, operations that are dependent on those views will not be available. The main operations relying on views to be indexed are:

- A user requests data that doesn't reside in the [channel cache](../config-properties/index.html#1.5/databases-foo_db-cache-channel_cache_max_length).
- A new channel or role is granted to a user in the [Sync Function](../sync-function-api-guide/index.html).

The unavailability of those operations may result in some requests not being process. The duration of the downtime will depend on the data set and frequency of replications with mobile clients.

| From       | To | Steps 
| ------------- | -- | ------
| 1.3 | 1.4 | <ul><li>A rolling upgrade is supported: modify your load balancer's config to stop any HTTP traffic going to the node that will be upgraded, perform the upgrade on the given node and re-balance the traffic across all nodes. Repeat this operation for each node that needs to be upgraded.</li></ul>
| 1.4 | 1.5 xattrs disabled | <ul><li>A rolling upgrade is supported: modify your load balancer's config to stop any HTTP traffic going to the node that will be upgraded, perform the upgrade on the given node and re-balance the traffic across all nodes. Repeat this operation for each node that needs to be upgraded.</li></ul>
| 1.5 xattrs disabled | 1.5 xattrs enabled | <ul><li>A rolling upgrade is supported: modify your load balancer's config to stop any HTTP traffic going to the node that will be upgraded, perform the upgrade on the given node and re-balance the traffic across all nodes. Repeat this operation for each node that needs to be upgraded.</li><li>The mobile metadata for existing documents is automatically migrated.</li><li>The first node to be upgraded should have the `import_docs=continuous` property enabled.</li></ul>
| 1.4 | 1.5 xattrs enabled | <ul><li>This upgrade, if done directly, will result in application downtime because all the nodes must be taken offline during the upgrade.</li><li>The first node to be restarted should have the `import_docs=continuous` property enabled.</li></ul><br /> That being said, it is possible to avoid this downtime by running the 2 upgrade paths mentioned above (first, an upgrade from 1.4 to 1.5, and second, an upgrade from 1.5 to 1.5 with xattrs enabled).
| 1.4 | 2.0 xattrs disabled | TODO
| 1.5 xattrs disabled | 2.0 xattrs disabled | TODO
| 1.5 xattrs enabled | 2.0 xattrs enabled | TODO


> **Note:** Enabling convergence on your existing deployment (i.e XATTRs) is **not** reversible. It is recommended to test the upgrade on a staging environment before upgrading the production environment.

## Couchbase Server

All of the different upgrade paths mentioned above assume that Couchbase Server is running a [compatible version](../../../installation/index.html) for Sync Gateway. There are 3 commonly used upgrade paths for Couchbase Server. Depending on the one you choose, there may be additional consideration to keep in mind when using Sync Gateway:

| Upgrade Strategy  | Downtime | Additional Machine Requirements | Impact when using Sync Gateway
| ------------- | ------------- | ------------- | --------------- |
| Rolling Online Upgrade  | None  | Low | <ul><li>**Potential transient connection errors:** The Couchbase Server re-balance operations can result in transient connection errors between Couchbase Server and Sync Gateway, which could result in Sync Gateway performance degradation.</li><li>**Potential for unexpected server errors during re-balance:** There is an increased potential to lose in-flight ops during a fail-over.</li></ul>
| Upgrade Using Inter-cluster Replication  | Small amount during switchover  | High - duplicate entire cluster | Using an XDCR (Cross Data Center Replication) approach will have incur some Sync Gateway downtime, but less downtime than other approaches where Sync Gateway is shutdown during the entire Couchbase Server upgrade. <br/><br/> It's important to note that the XDCR replication must be a **one way** replication from the existing (source) Couchbase Server cluster to the new (target) Couchbase Server cluster, and that no other writes can happen on the new (target) Couchbase Server cluster other than the writes from the XDCR replication, and no Sync Gateway instances should be configured to use the new (target) Couchbase Server cluster until the last step in the process. <ol><li>Start XDCR to do a one way replication from the existing (source) Couchbase Server cluster to the new (target) Couchbase Server cluster running the newer version.</li><li>Wait until the target Couchbase Server has caught up to all the writes in the source Couchbase Server cluster.</li><li>Shutdown Sync Gateway to prevent any new writes from coming in.</li><li>Wait until the target Couchbase Server has caught up to all the writes in the source Couchbase Server cluster -- this should happen very quickly, since it will only be the residual writes in transit before the Sync Gateway shutdown.</li><li>Reconfigure Sync Gateway to point to the target cluster.</li><li>Restart Sync Gateway.</li></ol>Caveats:<br/><ul><li>**Small amount of downtime during switchover:** Since there may be writes still in transit after Sync Gateway has been shutdown, there will need to be some downtime until the target Couchbase Server cluster is completely caught up.</li><li>**XDCR should be monitored:** </li>Make sure to monitor the XDCR relationship as per [XDCR docs](https://developer.couchbase.com/documentation/server/current/xdcr/xdcr-intro.html)</ul>
| Offline Upgrade  | During entire upgrade  | None  | <ul><li>Take Sync Gateway offline</li><li>Upgrade Couchbase Server using any of the options mentioned in the [Upgrading Couchbase Server](https://developer.couchbase.com/documentation/server/current/install/upgrading.html) documentation.</li><li>Bring Sync Gateway online</li></ul> |
