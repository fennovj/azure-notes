# Synapse documentation

This is mainly from  <https://docs.microsoft.com/en-us/azure/synapse-analytics/sql-data-warehouse/>

## Cheat sheet

- Migration - load your data
    - First, load your data in to Blob/Lake, then use PolyBase
    - Use Round Robin, Heap Indexing, No Partitioning
- Distribute/replicate tables
    - This is the strategy when dividing your data among multiple shards. Keep in mind that Synapse divides your data in 60 databases by default.
    - Replicated - copy data cross shards. Small dimension tables, less than ~10GB uncompressed.
    - Round Robin - slow performance, used for temporary tables or no candidate column
    - Hash - Fact tables, large dimension tables. Cannot update the distribution key
- Index table
    - Heap - no clustered index. You must use order by to guarantee a certain order. Temporary or small tables only.
    - Clustered index - Use on tables up to 100 million rows. 'Normal SQL'
    - CCI - clustered columnstore index - Data is stored/compressed in separate columns, with a delta store. Use for more than 100 million rows.
- Partitioning
    - Try partitioning with greater than 1 billion rows. Almost always partition based on date. Don't overpartition. Example: weekly or monthly partitiongs
- Resource class
    - You can allocate resources to certain queries.

## Security

### Connection security

Restrict ap access. Only allow port 1433. Synapse only supports server-level ip rules. You cannot disable connection encryption in Synapse.

### Authentication

You should not use the server admin role to authenticate users. The Admin can make an application user. Recommended is to create user in master database, this allows users to connect easier in SSMS.

Synapse also supports AD! Which is probably better because of the usual SSO reasons. In general, you still create database users, but just map them to AD identities (such as groups/users), then manage access like usual in SQL.

### Authorization

You can add roles, such as 'db_datareader', 'db_datawriter'. You can also do something like `GRANT SELECT ON SCHEMA::myschema to username`

### Transparent data encryption

Database is (should be) encrypted at rest. Also, backups and logs are encrypted the same way, with the database encryption key. This key is rotated automatically.

You can store the certificate in a key vault. I suspect that any users/readers will need access to this key vault to read the database?

### Column/row level security

You can restrict columns for certain users. Example: 

```sql
GRANT SELECT ON Users(FirstName, LastName, Email) TO TestUser;
SELECT * FROM Users;
-- The SELECT permission was denied on the column 'SSN' of the object 'Users', database 'My_TestDW', schema 'dbo'.
```

You can also restrict rows, for example: employees of a bank can only get data for their division. You do this by creating a predicate that returns 1 for allowed rows (with `USER_NAME()` to restrict users). After that, `SELECT` will only return rows where that predicate is 1. Example:

```sql
--- Here, namecol is the column with the name of the owner of the row.
CREATE FUNCTION Security.fn_securitypredicate(@NameCol AS sysname)  
    RETURNS TABLE  
WITH SCHEMABINDING  
AS  
    RETURN SELECT 1 AS fn_securitypredicate_result
WHERE @NameCol = USER_NAME() OR USER_NAME() = 'Manager';

CREATE SECURITY POLICY TableNameFilter  
ADD FILTER PREDICATE Security.fn_securitypredicate(NameCol)
ON dbo.TableName  
WITH (STATE = ON);  

GRANT SELECT ON security.fn_securitypredicate TO Username;

SELECT * FROM db.TableName;
-- Will only return the rows where NameCol = your username
```

### Dynamic data masking

Consists of three parts:

- A rule, of what field to mask, and what masking function is used
- A masking function to use
- SQL Users excluded from masking. Admins are always in this group, no matter what

Masking functions are stuff like:

- Credit card: XXXX-XXXX-XXXX-1234
- Email: aXX@XXXX.com
- Random number: Just a random number, in a range from n to m
- Custom text: Exposed prefix - Padding String - Exposed suffix
    - example: 3, 'XX', 2, will convert 'microsoft' to 'micXXft'
- Default:
    - varchar -> XXXX
    - numeric -> 0
    - date -> 01-01-1900
    - XML -> `<masked/>`
    - special (GUID, binary, etc) --> NULL

You can use the Azure REST API to set these (or az cli, etc)

## Data Loading

Normally, you would use ETL. However, Synapse uses ELT with MPP (massively parallel processing). That means you don't need resources to transform before loading. This also (must) use PolyBase.

- Extract the data into a supported format, such as CSV or Parquet. 
    - XML/JSON are not supported. SQL Dumps are not supported, you must dump to csv/parquet instead.
- Dump it into a Blob storage / Data Lake
    - Use AZCopy to move files less than 10 TB over the internet. Or create a pipeline in Azure Data Factory. Expressroute can also be used and is 'more reliable' since it doesn't use the public internet. (but a lil more expensive.)
- Load it with PolyBase or the COPY statement into Synapse.
    - Example of COPY statement:

```sql
COPY INTO dbo.myTable
FROM 'https://myaccount.blob.core.windows.net/myblobcontainer/folder0/*.txt'
WITH (
    FILE_TYPE = 'CSV',
    CREDENTIAL = (IDENTITY = 'Managed Identity'),
    FIELDQUOTE = '"',
    FIELDTERMINATOR=','
)
```

The COPY statement is recommended over Polybase, and has more capabilities. Polybase may still be used with SQL Server or Databricks.
The staging table should use heap index and round-robin distribution. You might consider using a hash index if the permanent table you are going to load into also has that same hash index.

- Once the data is loaded into a staging table, use MPP to transform it.
- use `INSERT INTO ... SELECT` to move it to a permanent table.

## Managebility

### Managing compute

In general, there is separate costs for compute and storage. You can scale compute with Data Warehouse Units. Example, DW100c has 1 compute nodes, meaning all 60 distributions are on that node. DW2000c has 4 compute nodes, meaning 15 distributions per node. DW30000c has 60 nodes, meaning 1 distribution per node.

In general, you should assume a linear scale (DWU vs performance), and test performance from there. You can also scale out automatically during busy hours. If data is skewed across partitions, or queries require a lot of data movement, it may not scale linearly with DWU.

### Monitoring resource utilization

You can use Azure Monitor to view CPU, IO percentage, Memory, active queries, failed connections, DWU percentage, Cache Miss percentage, Storage used, ectetera.

### Backup and restore

A *data warehouse snapshot* is a restore point to restore to a previous point. Snapshots are automatically created throughout the day, as incremental changes. Automatically, Azure aims to be able to recover to a point within 8 hours ago. (RPO)

A *data warehouse restore* is a new data warehouse created from a restore point. It can also be used for testing/development purposes.

User defined restore points are manual snapshots. They are available to restore from for 7 days. You can only have 42 max user defined AND automatic, even within 7 days. Oldest will be deleted if you go over. An example of UDF is during development. Or if you want, you can make a UDR every hour. That lowers your RPO considerably,but will also make it so you cannot recover to more than 42 hours ago.

Geo-backups are made once per day to a *paired data center*. The RPO is 24 hours.

#### Restoring

Creating a data warehouse restore goes a little like this:

- Create a data warehouse from a restore shapshot
- Update firewall rules on the new datawarehouse
- Update connection strings on existing applications

You can also replace the current datawarehouse with the restored datawarehouse, using some `ALTER DATABASE` or such command.

## Distributed tables

Distributed tables are split across 60 distributions. You should almost always use hash-distributed tables. Round robin is really only when there is no available hash column, or for temporary staging tables. In general, the data should be evenly distributed across distributions, so processing can run on all distributions in parallel. If this is not the case, it is called 'skewed'.

You should choose a hash as follows:

- Has many unique values. It is fine to use a unique column as hash.
- Does not have NULLs. It can have a few, but all NULL go to the same distribution.
- Is not used in WHERE clauses. This causes the query to only process on that one specific distribution
- Is used in JOIN/GROUP BY/DISTINCT/HAVING clauses. This improves performance. JOIN has a bigger performance increase than GROUP BY, so that has the priority.
- If there is no candidate, you can create a candidate as a composite of multiple columns. You should also aim to join on this composite column when possible.