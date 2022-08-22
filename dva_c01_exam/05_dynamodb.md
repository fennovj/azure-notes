# Dynamo

Read request unit are the following: one strongly consistent request, or two eventually consistent request, for an item up to 4KB.

## Secondary Indexes

The fastest way to access DynamoDB is through a QUERY on a primary index. However, sometimes you want a secondary key to query on without having to do a full scan. E.g. you want to get the score for every `UserId`, but also want to filter on `DeviceType` 

A table can have multiple secondary indexes.
A secondary index contains a `partition key`, `sort key` and other `projected attributes` This index then essentially acts as a view/copy of the table. The Primary key is always projected no matter what.

- Global Secondary index - The partition key is different than the base table. The index is stored separately from the main index, and scales separately
- Local secondary index - Same partition key, but different sort key.  Every partition is scoped to the base table.

These work by essentially storing the data multiple times, so writing will be a little more expensive.

## Page size

Generally, it is better to have an even distribution of small requests, rather than a few random large requests, or many requests in bursts. <https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-query-scan.html#bp-query-scan-spikes>

This is because reads use `read operations`, and a BIG read will consume all the capacity for that interval, and then throttle (until the interval is over). Best practices:

- Reduce page size. By default, `Scan` reads 1MB of data at a time, which is quite expensive. Making it read less, means `Scan` will do more, smaller requests
- Isolate `Scans` - try not to scan on a mission-critical table. Instead, perhaps have a book-keeping or shadow table that you can do heavy scans on, and another primary for low-latency queries.

## DynamoDb Accelerator (DAX)

In-memory cache for DynamoDB. Can provide huge speedups and is basically neccecary to reach microsecond latency. However, this is expensive and should only done *after* you have done all neccecary optimizations in the way of setting indexes, page size, reducing scans, etcetera. A crappy app will not be helped by enabling DAX.

It uses Write through caching: writes data to dynamodb first, then replicates to DAX across all nodes. The data is kept in the cache for a while (I assume that when the cache is full, oldest records are purged first) This is opposed to 'lazy loading', which loads data into cache only when it's requested.

## DynamoDb Streams

## Atomic counters and conditional writes

`Atomic counter` is a way to use DynamoDB, not neccecarily a feature, just an example usecase. The goal is to have a counter that is incremented everytime you write to it, and allows for concurrent writes. This works by writing 'SET x = x + 1', as apposed to reading x, adding 1, and writing the new value, which would not be atomic. Importantly, this can lead to double counting when writes fail and are automatically retried. (since it's not idempotent), so only use if under/overcounting can be tolerated.

`Conditional writes` are writes where you add a condition, and only write if the item fulfills some condition. E.g. write an item with a certain primary key, but don't overwrite if it already exists.
This can be used to implement locking, e.g. read, do something, then do a conditional write that writes, but only if the item is still the same as when you read it.

## Optimistic/Pessimistic locking

Locking mechanisms ensure data is always updated to latest version. E.g. if you read, then update, then write, you kinda want to 'lock' the item during that process. There are three ways of doing this:

KEEP IN MIND THIS IS CLIENT_SIDE ONLY. The lock is not a 'real' lock, it is simply a client-side rule that it chooses to obey

- Optimistic locking: when you read an item, record the version number. If you write, check if the version number has changed, and if not, the update fails. You can simply retrieve item again and try again.
- Pessimistic Locking: When you retrieve an item, store a 'lock' for that item. Any other writes on that item will fail while the lock is active. This is much more intrusive and can lead to deadlocking etc.
- Overly Optimistic locking: Just do... no locking! Don't try to detect collisions, simply latest writer wins. Only really works if there is only one writer (for a given item) at a time.

## Misc

Stinker alert: you cannot create a local secondary index on an existing table. You must create a new table and migrate the data.