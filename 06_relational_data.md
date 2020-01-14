# Work with relational data in Azure

- Azure SQL Database
- Azure Database for PostgreSQL
- SQL Elastic pools
- Security
- Exercise: Develop an ASP.NET application with Azure SQL Database

## Azure SQL Database

It's very nice compared to on-premise. Basically the usual cloud advantages: scale, convenience, cost, security.

Things to consider when creating an SQL database:

- logical server
  - You create a *logical server*. This is a server with firewall rules/security/logins, that can contain many databases.
- Purchasing models
  - DTUs - units of compute/storage/IO combination. Cheaper to get started, provides a good balance.
  - vCores - virtual cores, gives greater control over which of the three you want more/less of.
- Elastic pools
  - eDTUs are DTUs that are shared with multiple databases in a pool.
- Collation
  - Rules that govern the data, such as encoding, case/accent sensitivity in SQL. Usually, default is fine.

### Example code

See current database info: `az sql db show --group RESOURCE_GROUP --sql-server SERVER_NAME --name DB_NAME | jq '{name: .name, maxSizeBytes: .maxSizeBytes, status: .status}'`

```json
{
  "name": "Logistics",
  "maxSizeBytes": 2147483648,
  "status": "Online"
}
```

`az sql db show-connection-string --client sqlcmd --name Logistics`

Now you can write T-SQL in the command line if you connect to that url.

## Security considerations

You have to restrict access in some way, unless it's open data of course. There are three general ways:

- User accounts
- Virtual networks
  - Logically isolated network. Only some resources can connect to the database. The server/database are on different subnets, and have their rules for connecting from/to other networks
- Firewall rules
  - Basically, you allow input based on ranges of IP adresses

## Azure database for PostgreSQL

Again, the usual benefits of cloud. They mostly compare to on-prem PostgreSQL, not to Azure SQL database. You choose PostgreSQL if you want/need to use PostgreSQL.

I just paste the entire command here, just to get an idea, and it seems useful to have an insight in what kind of parameters there are

```bash
az postgres server create \
   --name $serverName \
   --resource-group xxxxxxx \
   --location westeurope \
   --sku-name B_Gen5_1 \
   --storage-size 20480 \
   --backup-retention 15 \
   --version 10 \
   --admin-user $userName \
   --admin-password "P@ssw0rd"
```

### Security

You can use native PostgreSQL user authentication. You can also make a virtual network or a firewall. Exampe: `az postgres server firewall-rule create`.

## Elastic pools

Elastic pools are mainly used to scale multiple databases with unpredictable resource requirements, in a cost-effective way. For example: you have a chain of stores with promotions at different times of day and year, rates of growth vary, etcetera. And you want all databases to stay performant, in a cost-effective way (e.g. don't just make each individual database capable of handling high load all at the same time, because that will never happen anyway)

They are ideal if you have multiple databases with low average, but high peak utilization. In general, you have two options:

1. Make all individual databases capable of meeting spikes (the 'simple option')
2. Put all databases in an elastic pool capable of meeting spikes

If option 1 requires 150% or more of the resources of option 2, then option 2 is worth it. (e.g. if option 1 means only 110% of the resources of option 2, then option 1 is probably simpler and cheaper)

You should add at least two S3 databases or 15 S0 databases, and you can up to 100 or 500 databases depending on performance tier.
