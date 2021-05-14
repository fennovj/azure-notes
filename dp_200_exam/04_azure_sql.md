# Azure SQL

## Disaster recovery for Azure SQL

More on the concept of this in the dp 201 exam chapter.

There are several ways to restore:

Automated backups: The main way of restoring from some random error or accident (such as accidentally deleting a table or something). You can use these to:

- PITR (point in time restore): Use the Azure Portal or Powershell/CLI. It creates a new database with a different name. After the restore is done, you can delete the original database.
- PITR Deleted database: Same as above, also done in Portal/Powershell/CLI. Note that Azure always makes a final transaction log before deleting, to make sure there is no data loss at all if you restore.
- Geo-restore: same as above, but to a different region. Note that this is for disaster recovery, not for business continuity. For business continuity, use auto-failover groups.

Example of restoration:

```powershell
$Database = Get-AzSqlDatabase -ResourceGroupName "ResourceGroup01" -ServerName "Server01" -DatabaseName "Database01"

Restore-AzSqlDatabase -FromPointInTimeBackup -PointInTime (Get-Date -Date "2019/06/25 12:30:22") -ResourceGroupName $Database.ResourceGroupName -ServerName $Database.ServerName -TargetDatabaseName "RestoredDatabase" -ResourceId $Database.ResourceID -Edition "Standard" -ServiceObjectiveName "S2"
```

After running that, you have to go in SQL and rename `RestoredDatabase` to the old database name.

To do a geo-restore, you have to create a new SQL database. When creating a SQL databse, you can choose to use 'existing data'. You can then choose a backup. Programatically, you can select a backup, for example in powershell with `Get-AzSqlDeletedDatabaseBackup` or `Get-AzSqlDatabaseGeoBackup`

You can change the PITR retention period in the portal, from 1-35 days, or just turn it off. Or in powershell with `Set-AzSqlDatabaseBackupShortTermRetentionPolicy`

## Partitioning

Partitioning is the idea of using multiple database servers, and dividing the data among them. This is useful for increasing scalability/performance. You also have more flexibility, for example the physical location of partitions can be different, or some partitions can use different properties like more/less resources.

- Horizontal partitioning: Each partition has the same schema but holds different rows. This is known as *sharding*. For example, store A-M in shard 1, N-Z in shard 2. Or store European customers in shard 1, American customers in shard 2.
- Vertical partioning: Store different columns in different shard. Usually there will be overlap. For example, store columns commonly used together in the same partition.
- Functional partitioning: Seperate data by function. For example one partition for login data, and one partition for reccomender system data. Or one read-only partition.

For horizontal partitioning, it is important to choose the right sharding key. For example: A-M, N-Z is a fast sharding key that always works, but may cause unbalanced partitions. Other strategies are:

- Lookup strategy: store a separate lookup table from id to partition. Offers full control but may cause overhead
- Hash strategy: partition based on hash range. Even distribution but overhead, and rebalancing shards may be difficult.
- Range strategy: easy but may cause unbalanced shards.

Soem considerations:

- minimize cross-partition queries
- Consider replicating static reference data across partitions
- Regularly rebalance partitions

## Auditing Azure SQL/Synapse

Auditing tracks events and writes to a log in Storage (Data Lake not supported), Log Analytics or Event Hubs.

It can be use for all kinds of things like regulatory compliance, or reporting/analyzing database activity.

There is server level or database level auditing. For geo-replicated databases, it is recommended to server-level audit both databases independently.

Auditing is different from advanced threat protection.

The audit logs contain the following:

- Name of the action
- Number of affected rows
- Source IP of the client
- Query duration
- ID of the service principal that performed the action
- A bunch of other info

## Automatic Tuning for Azure SQL

Azure SQL has automatic tuning where it will automatically do the following, if it decides it would improve performance.

- Force last good query plan - default ON
- Create indexes - default OFF
- Drop indexes - default OFF

Again you can enable on server or on database level

## Database jobs

Automate management tasks to run them every weekday/after hours/etc.

For example:

- Deploy schema changes, rebuild statistics or indexes
- Collect/aggregate data for reporting
- Data movements across databases

You can use SQL Agent Jobs, or Elastic jobs (to excecute job on multiple databases)

A job consists of a Step (basically just a query), a Schedule (when to run it), and a Notification (what to do when it completes)

A notification is usually an email, which is supported in Azure by creating an email profile. (lot of code, not gonna copy it all here)