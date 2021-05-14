# Synapse/Datawarehouse

## Loading data from Databricks into Synapse

This is mainly useful if you want/have to use databricks for transformations

- (optional): Mount datalake to databricks to load data from
- (optional) Load data (e.g. from datalake) and transform it
- (second optional): Any other way of creating dataframes in databricks

- Specify a temporary staging folder (tempdir) in blob storage (not neccecarily the same folder as in the previous steps)
- Write the dataframe to synapse. Scala code looks like:

```scala
sc.hadoopConfiguration.set("fs.azure.account.key.myacct.blob.core.windows.net", blobAccessKey)
spark.conf.set("spark.sql.parquet.writeLegacyFormat", "true")

renamedColumnsDF.write \
    .format("com.databricks.spark.sqldw") \
    .option("url", "jdbc:sqlserver://myserver.database.windows.net:1433;database=mydatabase;user=myuser;password=mypassword") \
    .option("dbtable", "SampleTable") \  // Target table name
    .option("forward_spark_azure_storage_credentials","True") \ // Forward the blob storage key to Synapse
    .option("tempdir", "wasbs://mycontainer@myacct.blob.core.windows.net") \  // Temporary staging directory
    .mode("overwrite").save()
```

## Loading data from Data Lake into Synapse with polybase

- Create master key (to encrypt the next step. only needs to be done once on entire database)
- Create database scoped credential containing the key for the data lake
- Create external source, consisting of the blob url and credential in previous step
- Create external file format (describing if it is parquet, or csv, etcetera)
- Create external (temporary) table, containing the external data logically (does not physically move data yet)
- Using a regular CREATE TABLE AS SELECT sql statement, create a table and physically move the data to sql
- Optionally, create statistics to improve query performance

## Loading data from Data Lake into Synapse with COPY INTO

- First, create the target endresult table, if it doesn't exist already. Something like:

```sql
CREATE TABLE [dbo].[dimTable]
(
    [Key] [int] NOT NULL,
    [Name] [nvarchar](500) NULL
)
WITH
(
    DISTRIBUTION = HASH([Key]),
    CLUSTERED COLUMNSTORE INDEX
);
```

- Then, create the copy statement. Remember, COPY INTO is usually better and more flexible than PolyBase. (but uses polybase under the hood)

```sql
COPY INTO [dbo].[dimTable]
(
    Key default -1 1,
    Name default 'myDefault' 2
)
FROM 'https://storageaccount.blob.core.windows.net/container/directory/'
WITH
(
    CREDENTIAL (IDENTITY = '', SECRET = '')
    FILE_TYPE = 'CSV'
    FIELDTERMINATOR = ','
    ROWTERMINATOR = '\n'
    ENCODING = 'UTF8'
    DATEFORMAT = 'ymd'
    MAXERRORS = 10
    ERRORFILE = '/errorsfolder'
) OPTION (LABEL = 'Just trying out ;)');
```

- Rebuild the columnstore index, because the loaded rows may not have been indexed yet.

```sql
ALTER INDEX ALL ON [dbo].[dimTable] REBUILD;
```

- Create statistics after a load, if you need them. It can look like below, but is honestly pretty complex because there are multiple cases. This is just the simplest possible case. Generally, you create statistics on the columns that feature in queries (joins, filters, aggregations, etc)

```sql
CREATE STATISTICS stat_dbo_dimTable_Key ON [dbo].[dimTable]([Key]) AS VARCHAR(8000)
CREATE STATISTICS stat_dbo_dimTable_Name ON [dbo].[dimTable]([Name]) AS VARCHAR(8000)
```

## Synapse monitoring

There are a couple of monitoring tables, called 'Dynamic Management Views' that appeared in a practice exam.

- `sys.dm_pdw_exec_sessions` - Shows the last 10.000 logins, including their status (open or closed) and session_id (a sequential number)
- `sys.dm_pdw_exec_requests` - The last 10.000 requests, including status (Completed, Failed, Cancelled, Running, Suspended), session_id, request_id, submit_time, total_elapsed_time, and the content of the query
- `sys.dm_pdw_request_steps` - If you have the request id from the above, you can investigate the query plan and see why it's so slow.
- `sys.dm_pdw_sql_requests` - While *exec request* is a query, *sql request* is a single sql excecution on a particular distribution of the data in a synapse 'cluster'.
- `sys.dm_pdw_waits` - Waiting queries (also contains request id)
- `sys.dm_pdw_dms_external_work` - External (polybase/COPY INTO) queries, also contains request_id

- `sys.dm_pdw_nodes_os_performance_counters` - Contains memory usage and transaction log size
- `sys.dm_pdw_nodes_tran_database_transactions` - Info on individual transactions (and if they are rolling back or not)
- `sys.dm_pdw_dms_workers` - Info on data movement on a distributed databse

Another obvious way to monitor is using Log Analytics. This allows you to log the following (that correspond to views above)

- ExecRequests - excecuting queries, one row per query
- RequestSteps - executing query steps, one row per step in the plan
- SqlRequests - executing SQL, one row per distribution or node, per step in the plan
- Waits - Waiting queries, one row per query
- DmsWorkers-  Workers doing data movement steps on distributed database. One row per worker/job. Worker(s) always work on a specific request step.