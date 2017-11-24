---
layout: whatsnew
features:
  - title: Shared Bucket Access
    description: |
      With Sync Gateway 1.5, you can now read and write documents to a single bucket that is also being used with Couchbase Server client SDKs. This enables existing Couchbase Server deployments to connect with remote edge devices that are occasionally disconnected or connected.
    link: 'guides/sync-gateway/shared-bucket-access.html'
sample:
  title: Travel Sample
  description: |
    This sample app allows users to search for flight and hotels, and subsequently to create a booking. The application then synchronizes documents with Sync Gateway 1.5 and Couchbase Server 5.0. Shared bucket access is enabled to allow web and mobile clients to perform the same operations on the bucket. It uses the Couchbase Lite 2.0 API that includes support for a N1QL like query interface and Full Text Search.
  link: 'http://docs.couchbase.com/tutorials/travel-sample/'
---

> **Note:** The 1.5 release is a Sync Gateway only release. The latest release of Couchbase Lite remains 1.4.1.