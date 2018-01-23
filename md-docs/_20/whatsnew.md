---
layout: whatsnew
cbl_features:
  - title: N1QL for Couchbase Lite
    description: |
      Couchbase Lite 2.0 introduces a new Query API with semantics based on Couchbase Server's N1QL query language.
    links:
      - name: swift
        value: couchbase-lite/swift.html#query
      - name: android
        value: couchbase-lite/java.html#query
      - name: c#
        value: couchbase-lite/csharp.html#query
  - title: WebSocket replication protocol
    description: |
      Couchbase Lite 2.0 uses a new replication protocol based on WebSockets. This protocol is designed to be faster and more efficient.
    links:
      - name: swift
        value: couchbase-lite/swift.html#replication
      - name: android
        value: couchbase-lite/java.html#replication
      - name: c#
        value: couchbase-lite/csharp.html#replication
  - title: Full-Text Search
    description: |
      This release includes the ability to run Full-Text Search queries on documents stored in the local database.
    links:
      - name: swift
        value: couchbase-lite/swift.html#full-text-search
      - name: android
        value: couchbase-lite/java.html#full-text-search
      - name: c#
        value: couchbase-lite/csharp.html#full-text-search
  - title: Database Replicas
    description: |
      Replication between two local databases is now supported. This feature can be used for local incremental backups for example.
    links:
      - name: swift
        value: couchbase-lite/swift.html#starting-a-replication
      - name: android
        value: couchbase-lite/java.html#starting-a-replication
      - name: c#
        value: couchbase-lite/csharp.html#starting-a-replication
sg_features:
  - title: No Conflicts Mode
    description: |
      Sync Gateway 2.0 introduces a 'no conflicts mode'. When enabled, Sync Gateway will reject any revision that would create a conflict. This mode is specifically designed for scenarios where you do not wish to use the multi version concurrency control aspect of Couchbase Mobile.
    links:
      - name: Sync Gateway Configuration
        value: 'guides/sync-gateway/config-properties/index.html#2.0/databases-foo_db-allow_conflicts'
other:
  - title: Travel Sample
    description: |
      This application synchronizes documents with Sync Gateway 1.5 and Couchbase Server 5.0. Shared bucket access is enabled to allow web and mobile clients to perform the same operations on the bucket.
    links:
      - name: Start building
        value: 'http://docs.couchbase.com/tutorials/travel-sample/'
---