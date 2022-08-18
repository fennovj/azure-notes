# Dynamo


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