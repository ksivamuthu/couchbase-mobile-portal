---
layout: whatsnew
features:
  - title: Shared Bucket Access
    description: |
      With Sync Gateway 1.5, you can now read and write documents to a single bucket that is also being used with Couchbase Server client SDKs. This enables existing Couchbase Server deployments to connect with remote edge devices that are occasionally disconnected or connected.
    link: 'guides/sync-gateway/shared-bucket-access.html'
  - title: SSL and Multi-URL support
    description: |
      In Sync Gateway 1.5 you have the ability to define multiple server URLs in the Sync Gateway configuration, and full support for SSL between Sync Gateway and Couchbase Server.
    link: 'guides/sync-gateway/config-properties/index.html#1.5/databases-foo_db-server'
sample:
  title: Upgrading
  description: |
    To start using the shared bucket access feature on an existing cluster it should first be upgraded to use Sync Gateway 1.5 and Couchbase 5.0. Follow the upgrade documentation to learn about the different upgrade paths; including a step by step guide to perform a rolling upgrade.
  links:
    - name: Upgrade documentation
      value: 'guides/sync-gateway/upgrade/index.html'
---

> **Note:** The 1.5 release is a Sync Gateway only release. The latest release of Couchbase Lite remains 1.4.1.