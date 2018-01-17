Couchbase Mobile 2.0 uses a new replication protocol based on WebSockets. This protocol has been designed to be fast, efficient, easier to implement, and symmetrical between the client and server.

## Compatibility

⚠️ The new protocol is **incompatible** with Couchbase Lite 1.x, and with CouchDB-based databases including PouchDB and Cloudant. Since Couchbase Lite 2 developer builds support only the new protocol, to test replication you will need to run a version of Sync Gateway that supports it.

To use this protocol with Couchbase Lite 2.0, the replication URL should specify **blip** as the URL scheme (see the [Replication API](index.html#replication-api) section below). Mobile clients using Couchbase Lite 1.x can continue to use **http** as the URL scheme. Sync Gateway 1.5 will automatically use the 1.x replication protocol when a Couchbase Lite 1.x client connects through "http://localhost:4984/db" and the 2.0 replication protocol when a Couchbase Lite 2.0 client connects through "blip://localhost:4984/db".

## Starting Sync Gateway

To run an example, create a new file named **sync-gateway-config.json** with the following.

```javascript
{
  "databases": {
    "db": {
      "server":"walrus:",
      "users": {
        "GUEST": {"disabled": false, "admin_channels": ["*"]}
      },
      "unsupported": {
        "replicator_2":true
      }
    }
  }
}
```

In the configuration file above, the **replicator_2** property enables the new replication protocol.

[Download Sync Gateway](https://www.couchbase.com/downloads) and start it from the command line with the configuration file created above.

[//]: # (TODO: for csharp.md only, update command below for Windows dev)

```bash
~/Downloads/couchbase-sync-gateway/bin/sync_gateway sync-gateway-config.json
```

For platform specific installation instructions, refer to the Sync Gateway [installation guide](../../../../../current/installation/sync-gateway/index.html).

## Starting a Replication

Replication objects are now bidirectional, this means you can start a `push`/`pull` replication with a single instance. The replication's parameters can be specified through the [`ReplicatorConfiguration`](http://docs.couchbase.com/mobile/2.0/couchbase-lite-objc/db021/Classes/CBLReplicatorConfiguration.html) object; for example, if you wish to start a `push` only or `pull` only replication. The following example creates a `pull` only replication instance with Sync Gateway.

```objectivec
let url = URL(string: "blip://localhost:4984/db")!
var replConfig = ReplicatorConfiguration(withDatabase: database, targetURL: url)
replConfig.replicatorType = .pull
let replication = Replicator(withConfig: replConfig)
replication.start()
```

As shown in the code snippet above, the URL scheme for remote database URLs has changed in Couchbase Lite 2.0. You should now use `blip:`, or `blips:` for SSL/TLS connections (or the more-standard `ws:` / `wss:` notation). You can access the Sync Gateway `_all_docs` endpoint [http://localhost:4984/db/\_all\_docs?include_docs=true](http://localhost:4984/db/_all_docs?include_docs=true) to check that the documents are successfully replicated.

Starting in Couchbase Lite 2.0, replication between two local databases is now supported. This isn't often needed, but it can be very useful. For example, you can implement incremental backup by pushing your main database to a mirror on a backup disk.

## Troubleshooting

As always, when there is a problem with replication, logging is your friend. The following example increases the log output for activity related to replication with Sync Gateway.

[//]: # (TODO: replace code snippet for Java/C#/Objc)

```swift
Database.setLogLevel(.verbose, domain: .replicator)
```

## Replication Status

The `replication.status.activity` property can be used to check the status of a replication. For example, when the replication is actively transferring data and when it has stopped.

[//]: # (TODO: replace code snippet for Java/C#/Objc)

```swift
self.replication.addChangeListener { (change) in
    if change.status.activity == .stopped {
        print("Replication stopped")
    }
}
```

The following table lists the different activity levels in the API and the meaning of each one.

|State|Meaning|
|:----|:------|
|`STOPPED`|The replication is finished or hit a fatal error.|
|`OFFLINE`|The replicator is offline as the remote host is unreachable.|
|`CONNECTING`|The replicator is connecting to the remote host.|
|`IDLE`|The replication caught up with all the changes available from the server. The `IDLE` state is only used in continuous replications.|
|`BUSY`|The replication is actively transferring data.|

## Handling Network Errors

A running replication can be interrupted for a variety of reasons such as network errors or unauthorized access. In this case, the replication status will be updated with an `Error` which follows the standard HTTP error codes. The replication change event can be used to monitor the status of the replication. The following example monitors the replication for errors and logs the error code to the console.

[//]: # (TODO: replace code snippet for Java/C#/Objc)

```swift
self.replication.addChangeListener { (change) in
    if let error = change.status.error as NSError? {
        print("Error code :: \(error.code)")
    }
}
self.replication.start()
```
