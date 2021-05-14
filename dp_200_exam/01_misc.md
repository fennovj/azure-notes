# DP-200 practice

There are various topics, here I am just listing the ones that are too short for a full page.

## CosmosDB partition design

Obviously, pick a partition key (x) that evenly distributes the data. If you often query on a second value (y), instead of fanning-out, make ADDITIONAL 'lookup collections': key-value from y to x. Then, when querying on y, you first use the lookup collection to get x, then do a query with x.

This lookup collection can be updated with the azure change feed, and an azure function that creates the new records.

The downside is a slightly longer query time (order of extra milliseconds). If that is not okay, you will need to completely duplicate the data, which is more complex and expensive (in dollar terms). And has more potential for mistakes since it is more complex.

## HDFS Nodes

Okay I don't think this will appear in the exam, but it appeared in a practice exam... And Data Lake is 'HDFS Compatible', meaning if you have a hadoop cluster, you can use Data Lake with the `hdfs://` protocol and it will work. Similarly, it will work with HDInsight which runs Apache Hive, Apache Spark, Hadoop, etcetera.

Hadoop has a driver/worker architecture just like Spark. There is a single 'NameNode', which is the driver server. There are other 'DataNodes', which manage storage attached to their nodes. The NameNode manages the metadata. Internally, files are split into blocks and split across the DataNodes. The NameNode gives out instructions like 'create block', 'delete block', or 'replicate block on node y'. Clients (users) talk to the NameNode first. If a user wants to download a block, the NameNode will then redirect them to the relevant DataNode. These node software runs in Java.

## Azure Blob Storage lifecycle

Lifecycle code can be configured in portal or with CLI/powershell

```json
{
  "rules": [
    {
      "enabled": true,
      "name": "myname",
      "type": "Lifecycle",
      "definition": { ... }
    }
  ]
}
```

A defintion looks like:

```json
"actions": {
    blobtype: {
        action: {run_condition: 30}, ...
    }, ...
},
"filters": {
    "blobTypes": ["blockBlob", "appendBlob"],
    "prefixMatch": [ "container1/foo" ]
}
```

Blobtype:

- baseBlob - normal blobs
- snapshot - snapshots - *only supports the delete action*

Actions:

- tierToCool - Move the tier (currently at hot tier) to cool tier
- enableAutoTierToHotFromCool - ?
- tierToArchive - Move the tier to archive tier
- delete - Delete the blob

Run conditions:

- daysAfterModificationGreaterThan
- daysAfterCreationGreaterThan
- daysAfterLastAccessTimeGreaterThan

## Caching

This means temporarily copying frequently accessed data to fast storage, for example close to the application. So in general, a different server than the database server, one that is closer to the application.

- Private cache: generally the computer that is also running the application. Most commonly an in-memory store.
- Shared cache: common source that can be accessed by multiple machines

Note that in general, the application chooses when it updates its cache. For example, you could choose to invalidate the cache 30 minutes after obtaining it from the server. This can mean that different applications have different versions of the data. If this cannot happen, use a shared cache instead.

When to use caching is kinda obvious: when you need performance, and you have a lot of users accessing a lot of data. It's especially useful when you have data that is read frequently but written infrequently.

Another idea if you have quickly changing data (for which caching is not ideal since the cache will quickly be outdated), is storing dynamically changing data *inside the application* and only occasionally syncing all the changes to the database. That way, you have the most recent version on the application, and minimize database interaction. The downside is you could lose data if the application experiences a failure.

Redis is a cache you can use for this purpose, although I haven't studied it extensively.