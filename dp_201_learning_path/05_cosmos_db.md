# Working with CosmosDB

This basic intro is basically the same as az_204, so I only repeated some basics here. Just check under az 204 for more.

## Basic concepts

### Request unit

Cost of performing a GET request on a 1-KB document using document ID. RU cost is deterministic and can be calculated in the portal. You provision RUs per second, and can change them any time.
    - Item size
    - Item indexing (you can chooise not to index items)
    - Item property count (writing more indexes costs more)
    - Data consistency (Strong consistency is twice as expensive)
    - Query pattern (such as number of predicates, result size, etc)

### Partition key

The key used to define the partition the data is logically stored in. You want to optimize queries (by filtering on partition key), as well as avoid hot partition. A good partition key is userID or productID, since those are often filtered/queried on, and not hot.

Generally we prefer small partitions, using a unique key is totally fine. Just beware that cross-partition queries are more expensive, so for example, if you often query the products in a shopping basket, it would be better to have CustomerID as a partition key rather than ProductID or `concat(customerID, productID)`, because customerID means you only need to query a single partition.

Also, you cannot change the partition key of a container after creation.

### Creating a CosmosDB

- Step 1: Make a database
- Step 2: Make a container
    - define RUs, and partition key, and optionally some unique keys

## APIs

SQL, Gremlin, MongoDB, Cassandra, Table

## Consistency models

Strong, Session, Bounded Staleness, Prefix, Eventual

## Multi-master model

Instead of asking one region for every write, multi master allows to write to each region as a master, the masters communicate with each other to figure out who wins, this is either with LWW (last writer wins) or a custom UDF.
