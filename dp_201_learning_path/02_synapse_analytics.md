# Synapse Analytics

Synapse provides a relational big data store that can scale to petabttes. It has a MPP (massively parallel processing) architecture. Synapse has a concept of compute nodes where data can be distributed over compute nodes (e.g. hash or round robin) to answer complex queries with high parallelism.

## Design a data warehouse

Azure SQL stores the data in columnar storage (jay) queries are generally much faster by default because of this. You can just use T-SQL for normal queries.

Synapse uses an *SQL Pool* as a unit of compute scale, which size is measured in DWU (data warehouse units). DWU can be used to scale up or down over time.\

### Clustered column store

In Synapse, because it's a column store, you will often create a *clustered columnstore index*. Remember, a clustered index is an index that reorders the way records are physically stored. Records are stored such that the index is ordered (which makes it faster to search by index value).

However, a clustered columnstore index is actually not a name for just an index on a column, it is a way of storing *the entire table*. The important thing is that it is the column segments  (max 1 million rows per column segments) plus a delta store. For small changes, the delta store is used. For big changes, you just commit directly to the column store. When querying, you query the column store, then check the delta store to see if anything changed. Over time, the delta store is committed to the column store.

You can have a nonclustered columnstore index - it's a way of doing the same thing, but you save this index separately, and update it when the data changes. It's slower because it does not physically affect the way data is stored.

### Workload management

You can prioritize query workloads with workload groups: reserve resources, or limit resources for a group of requests, and setting timeouts for 'runaway queries'. You can define these in your DB schema with T-SQL (`CREATE WORKLOAD GROUP xxx WITH (... )`)

You can map queries to a workload group with a workload classifier. You run a query with/as a classifier, and it is then mapped to a specific workload group that has the resources allocated. The queries go into a queue, and higher priority queries skip ahead of lower priority queries.

You can also do result_set_caching. The cache contains some results of recent queries.

Materialized views are basically views that are stored separately, and updated automatically. This improves query speed if you define them well, since results can be pre-computed.

### Massively Parallel processing

This is done by abstracting the CPU+memory+IO into an sql pool, that is allocated some DWU.
Applications connect and issue commands to a control node. This control node runes MPP engine and passes operations to the compute nodes. The data movement service moves data across nodes when needed (if one node needs data from another)
You can have 1 to 60 compute nodes (which I guess is enough for petabytes)

### Storage

Data is stored in Azure Blob Storage. (but that is sorta hidden away). The data is sharded to improve performance. It is important to choose a suitable sharding pattern when available. The default is Round-Robin.

- Replicated - Small-dimension tables in a star table with less than 2GB of compressed data (~10GB uncompressed) This means the data is completely copied if multiple nodes need it.
- Round-robin - rows are distributed one-by-one. This is good for a temporary table, but performance will be slower, because there will often be a lot of data movement between nodes, especially on joins.
- Hash - For large dimension tables or fact tables. Data is sharded according to some column. However, you cannot easily update that column.

Do not use a varchar as a hash column. You should try to avoid Round-Robin (except for temporary/staging tables), since hash is almost always faster.
For small tables, replicated is fastest, since there is basically no data movement needed at all after copying the data.

## Query data in Synapse

There is a demo in SQL Server Management Studio. It involves opening port 1433 (the default SQL port in microsoft sql server) and copying the connection string to SSMS. You can then just run a normal T-SQL query, so not the most interesting demo ever. You can also use the data in PowerBI, like you would use a normal SQL database.

## Polybase

Polybase aims to solve the problem of loading (csv or hadoop or zip) data into the compute node when you need to. Polybase allows the compute nodes to get data across the storage blob, even when the blob contains petabytes of data. You can then run T-SQL queries across it.

You can get better performance by having multiple csv files instead of 1 huge one, since you can split the data across compute nodes better. More than 60 gives no benefit since you can only have 60 nodes in parallel anyway.

Generally, the importing goes like:

- Create an import database, that contains credentials to the blob storage
- Create an external data source connection (with the connection string to the blob storage)
- Create an external file format (containing metadata such as the separator used for csv)
- Create a temporary SQL table for putting the data in
- Create a destination table (with a clustered columnstore index and a smart distribution such as hash)
- Optionally, for better performance in the future, create statistics on the columns that feature often in queries
- Press run!
