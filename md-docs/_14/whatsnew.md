---
layout: whatsnew
features:
  - title: Sync Gateway Accelerator
    description: |
     Sync Gateway Accelerator is a new component in the Couchbase ecosystem to increase <b>write</b> performance of a Sync Gateway cluster. The goal of Sync Gateway Accelerator is to move the mutation feed processing off Sync Gateway nodes, and instead distribute this work across a cluster of Sync Gateway Accelerator nodes.
    links: 
      - name: Learn more
        value: 'guides/sync-gateway/accelerator.html'
  - title: Log Rotation
    description: |
      Introduced in 1.4, Sync Gateway can now be configured to perform. Previously this add to be done through external scripts. This feature can be enabled in the Sync Gateway configuration file. Log rotation is a recommended method to minimize disk space usage.
    links: 
      - name: Learn more
        value: 'guides/sync-gateway/deployment/index.html#built-in-log-rotation'
  - title: ToDo Sample
    description: |
      This step by steps tutorial walks you through some of the more advanced features of Couchbase Lite and Sync Gateway. Such as running complex map-reduce queries, custom conflict resolution rules, user login and managing all aspects of security in the Sync Function.
    links:
      - name: Start building
        value: 'training/index.html'
---