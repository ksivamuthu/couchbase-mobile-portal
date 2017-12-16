---
id: manager
title: Manager
permalink: guides/couchbase-lite/native-api/manager/csharp.html
---

A `Manager` is the top-level object that manages a collection of Couchbase Lite `Database` instances. You need to create a `Manager` instance before you can work with Couchbase Lite objects in your Application.

## Creating a manager

You create a Manager object by calling a constructor or initializer on the Manager class.

```c
var manager = Manager.SharedInstance;
```

## Default database path

The Manager creates a directory in the filesystem and stores databases inside it. Normally, you don't need to care where that is -- your application shouldn't be directly accessing those files. But sometimes it does matter. Depending on the platform you are developing for, the default database path will be:


|Platform|Path|
|:-------|:---|
|Windows (WPF)|C:\Users\<username>\AppData\Local\|
|Unix|~/.local/share/|
|Xamarin Android|/data/data/<package-name>/files/.local/share/|
|Xamarin iOS|/Documents (inside sandbox)|

You can change the location of the databases by instantiating the `Manager` via a constructor/initializer that takes a path as a parameter. This directory will be created if it doesn't already exist.

```c
var options = new ManagerOptions();
options.ReadOnly = true;
Manager manager = new Manager(Directory.CreateDirectory(dbPath), options);
```

## Global logging settings

You can customize the global logging settings for Couchbase Lite via the `Manager` class. Log messages are tagged, allowing them to be logically grouped by activity. You can control whether individual tag groups are logged.

The available tags are:

```c
Log tags

Log.Domains.Database
Log.Domains.Query
Log.Domains.View
Log.Domains.Router
Log.Domains.Sync
Log.Domains.ChangeTracker
Log.Domains.Validation
Log.Domains.Upgrade
Log.Domains.Listener
Log.Domains.Discovery
Log.Domains.TaskScheduling
Log.Domains.All

Log levels

Log.LogLevel.Verbose
Log.LogLevel.Debug
Log.LogLevel.Error
Log.LogLevel.Warning
Log.LogLevel.Information
```

The following code snippet enables logging for the **Sync** tag.

```c
Log.Domains.Sync.Level = Log.LogLevel.Verbose
```

## Concurrency Support

In C#, a `Manager` instance should only be created once for any given directory and shared between threads.  Creating two managers that operate on the same directory will discard thread safety guarantees.
