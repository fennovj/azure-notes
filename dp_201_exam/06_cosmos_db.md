# Cosmosdb

Mainly some miscellaneous stuff.

## Change feed

The change feed in Azure Cosmos DB listens to a container, then outputs a sorted list of documents in order in which they were modified. This output will then be passed to a processor. You can choose multiple processors to handle the change feed in parallel (for example, have three separate feeds for A-G, H-K, L-N, each with their own processor. Change feed is enabled by default. Reading from the change feed costs DBU.

## Security

 Some layers of security are:

- Network security: CosmosDB has IP-level controls. Also there are virtual network service tags: a certain group of IP addresses of a certain network security group.
- Authorization: CosmosDB uses HMAC (hash message authentication code) - but that's kind of a technical detail that doesn't concern developers. More importantly, you can use resource tokens to let users sign requests, but only to certain containers/partition keys/UFs/etcetera. Or just give out the master key :D
A resource token is created when a CosmosDB user is given a permission, for a limited time. A resource key is kind of similar to a SAS token, and is also used for better permission management than just giving the master key.
- RBAC - You can use RBAC to give resource tokens, to make sure users only make requests they are allowed to.
    - Of note: when not using the master key, you have to use CosmosDB users, not AD users. Access rules are handled within CosmosDB. AD users can create CosmosDB users though, with the right role. Then, these CosmosDB users can request resource tokens, or someone else can create resource tokens for them.
- Global replication - Famous CosmosDB feature. not really a security thing, but I guess it helps against regional failures. Cosmos automatically failsover if you replicated to multiple datacenters.
- Various other security features, such as automated backups, audit logging, HTTPS encryption, Encryption at rest, and it's impossible to have an admin account with no password!

## SLA

99.99% SLA for single region, 99.999% SLA for multi region.

## Misc

There are several ways of connecting with Cosmos:

- Regular API or .NET SDK, Python SDK, etcetera: especially useful for just firing sql-like queries
- BulkExecutor Library - like the above, but especially for bulk queries. Also used by DataFactory
- Change Feed - a sorted list of documents that were changed. You can read it as follows:
    - Push: Use an Azure Function to process each item
    - Push: Use the SDK (see above) to build your own processor.
    - Pull: Just read directly with the SDK. You don't get at-least-once delivery guarantee. .NET SDK only for now.
