# SQL Database

The skills measured mostly mention Synapse. However, there are some things that are a little different, such as backups, sharding, and elastic pools. So they are written here.

https://docs.microsoft.com/en-us/azure/azure-sql/database/elastic-scale-introduction#sharding
https://docs.microsoft.com/en-us/azure/architecture/patterns/sharding

## Sharding

Sharding is not really only SQL Database, but still interesting to know. The main problem is a limit of computing resources, network bandwidth, or geography of a single server. You can scale vertically (add more resources), but that may be temporary. For supporting large amounts of users/data, you need to horizontally scale eventually.

### Horizontal sharding / partitioning

Each shard has the same schema but different subset of data. You can scale and use off-the-shelf, cheaper hardware. Balance workloads across shards. Or locate shards physically close to users that will need that part of the data.

The shard key should be static, and (almost) never change. You generally want to shard on the thing that is most often queries on (WHERE). You can even use a composite shard if there really are multiple values often queries on.

- The lookup Strategy
    - The shard key is a unique value. The sharding logic is essentially a lookup table. The advantage over hash is that you have more control over the physical locations of certian records.
- The range strategy
    - The shard key is ordered, and shards are defined based on a range. Useful when there are a lot of range lookups (wow), and easy to implement.
-  The hash strategy
    - reduces hotspots (shards with way more data). Sharding strategy is just computing the hash(es) on every query. Rebalancing shards may be difficult if it's still unbalanced.

### Multi-tenant vs Single-Tenant

A tenant here is basically what tenant means in azure, except imagine we are Azure, and we have Tenants (customers). You might want to really isolate the databases of your different tenants, and even have different pricing tiers, offering better performance, backups, etcetera.

Single-tenant means each tenant has their own database. This is simple and you have isolation, backups, etcera. However, sometimes a Multi-tenant solution may be better, where multiple tenants are grouped on the same database. Especially when you have a large number of small tenants.

## Elastic queries

Elastic queries are those that span multiple databases. Maybe don't use in the above customer example, but it can be used for building reports from multiple databases. There are two main ways of doing this in Azure SQL:

- OLTP (online transaction processing) Cloud App:
    - Use the Elastic Database Tools Library (.net and other languages)
- PowerBI/ODBC/etc:
    - Elastic Database Query

There are two secnarios:

- Vertical partitioning: There are two different databases with different schemas we want to cross-query
- Horizontal partitioning: There are horizontal partitions of the same schema.

Basically, an elastic query makes external data available in another sql table. There are some possibilities:

### Vertical patterns

- Reference data: Create a new database with 'views' to external tables. That database combines the tables and returns results.
- Cross-database querying: Each database has access to all others. Each database will ad-hoc query other databases as needed. This should mainly be used for small queries.

### Horizontal patterns

You need an elastic database shard map. Then, there is essentially a 'gateway' SQL database that acts as the normal sql database and hosts the shard map. In this way, it's very similar to the 'Reference data' pattern above.

## Elastic pools

This mainly follows up on the whole 'single-tenant vs multi-tenant' story, and is mainly to address the problem as follows:

- You have several databases for multiple tenants. Each tenant has different usage patterns. You now have to either other-provision each tenant, or under-provision meaning everyone suffers during their peak time.

The obvious solution: Buy resources for a pool, and share them around!

You should consider elastic pools when you have different databases with different utilization patterns. For example, if christmas is coming up, and everyone is going to hammer your website, elastic pool is probably not the best solution. (in that case, just start scaling vertically if possible, or maybe even horizontally)

In general, eDTU (elastic DTU) are 1.5x more expensive than DTU, so do the math and see if it saves money. If you have 2 databases with mutually exclusive usage patterns (not really realistic but whatever), it would already be worth it.

Elastic pools have the usual features: PITR, Geo-restore, Active-geo-replication...!

## Business continuity

### Active geo-replication

Keeps a secondary database in a same or different data center. Generally, you want a traffic manager that will automatically redirect traffic to the secondary data center if the primary fails.

(useless factoid!) A lot of incidents can be self-mitigated by a single database. The most common case where geo-replication is needed is the tenant ring going down due to OS kernel memory leaks on the compute nodes, or someone cutting a wrong network cable during a routine operation. Of course Azure will fix these eventually, but they are main causes of extended downtime.

### Failover group

This is something you do ON TOP OF active geo-replication. It simplifies/abstracts the deployment of (many) replicated databases. Basically, you enable 'Auto-failover' and that takes care of everything for you. It can also contains apps that are also failed over.

A Failover group is a group of databases and apps managed on a single server, that can fail over as a unit

A server hosts some databases. You can have multiple failover groups on a single server.

A Planned failover is a failover that happens right after a full synchronization. This ensures there is no data loss whatsoever. This is mainly done for a recovery drill, or to relocate the primary to a different region (such as in case of a fallback - returning to the primary region after outage is over)

After a planned or unplanned failover, the old region automatically becomes a new secondary.

You can configure how long to wait for initiating failover. For example, you can say 'Wait 23 hours before initiating failover', meaning you might get 24 hours of downtime, but are much less likely to lose data, because the primary might come back online. By default, it waits 1 hour.

Applications connect to some `fog-name.database.windows.net`. In case of failover, the url will keep working.

## Backup and recovery

SQL makes:
    - Full backups every week
    - Differential backups every 12-24 hours (diffs to the full backup, not to each other)
    - Transaction log backups every 5-10 minutes
The backups are stored in RA-GRS storage blobs and replicated to a paired region.

Restoring to a point in time (PITR) requires an uninterrupted backup chain to a full backup, optionally a differential backup. You can define a retention period across which you want PITR. Azure will automatically delete backups that aren't needed for PITR in that period. With a retention of 7 days, this generally means that every week, all backups of 7-14 days ago are purged. Retention period can be 1-35 days.

You can enable long-term retention for retention up to 10 years. This means the weekly backups are copied and stored long-term somewhere. You can choose something like only store monthly/yearly full backups to lower the costs a bit.

The costs are included for DTU model, and you pay basically for double the storage costs if you use vCore model.

Note that in general, you can automatically restore in case of some network outage or such. But I think usually azure should do this by itself? Also you can manually restore, you do this by creating a new database and selecting 'backup' as the source data.

### Database recovery

ADR (accelerated database recovery) is a feature that accelerates recovery, and therefore improves database availability. Features are:

- Fast database recovery irrespective of the number of active transactions
- Instantaneous transaction rollback, irrespective of the time the transaction has been active
- Aggresive log truncation, preventing the log from growing out of control.

#### Normal (ARIES) recovery

Algorithm for Recovery and Isolation Exploiting Semantics

(note: this will probably not be on the exam, but I still think it's interesting)

- Analysis phase, analyse the transaction log and the last known state of the database to determine the state of each transaction (committed to database, or still active)
- Redo phase: redo all committed operations to the database
- Undo phase: For each active transaction, undo the operations that the transaction performed.

Recovery time is proportional to the size of the longest active transaction. With large bulk insert operations, recovery can take a LONG time.

#### Accelerated (ADR) recovery

Make it constant time. This is done by versioning all physical modifications. Physical modifications are copied to an additional log, called sLog.

Note that physical modifications are things like lock acquisition, cache invalidation, etcetera. Logical modifications are stuff like selecting and updating values. Logical modifications are VERY easy to undo.

- Analysis phase: Same as above. Also reconstruct 
- Redo Phase:
    - First, redo only the few operations from sLog
    - Then, redo from the last checkpoint of the database (since all other operations were already committed anyway)
- Undo Phase:
    - Use the sLog to undo the non-versioned stuff (this is normally slow, but is now fast)
    - Use the regular log to undo the versioned stuff (These are now only logical modifications which are also fast to revert)

So in conclusion: a lot of text, and I still don't fully understand the 'versioning' stuff and other details. But the main conclusion is: Use this when you have long-running transactions, or cases where the transaction log grows significantly large. Or in general, when recovery takes too long and affects availability too much.

## Managed instance

I don't really understand the name, but by far the main feature of this is it is near 100% compatible with SQL Server (enterprise edition). The whole thing is hosted in a VNet for security reasons (existing on-prem SQL servers are most likely also isolated in a private network, so this is a good replacement), and is basically intended to be an easy lift-and-shift for existing SQL Server customers.

It's still a PaaS, so you don't really 'Manage' the instance directly or anything, which is why I don't really understand the name. I guess it's to differentiate from SQL Server, not from Azure SQL Database. Like, "it's just like SQL Server, but the instance is completely managed by Microsoft!". Of note: there is an alternative if you really want to manage the instance yourself... Hosting your own SQL Database on a VM!

## Pricing / Service Tiers

You generally pay for DTU model or vCore model. For DTU model, there is also a service tier. For vCore, you generally sorta choose these things yourself, instead of an entire package. All of them can be used for production workloads. The main differences are:

- Basic: max 7 day backup retention. Intended for low CPU workloads. Max 2GB per database
- Standard: max 35 day backup retention. max 1TB per database
- Premium: same as above, but supports OLTP in-memory. max 4TB per database

Also, premium has a higher response time requirement. This following numbers are ran on a 'representative' benchmark of real-world data.

- Basic: 80th percentile at 2 seconds.
- Standard: 90th percentile at 1 seconds.
- Premium: 95th percentile at 0.5 seconds.
