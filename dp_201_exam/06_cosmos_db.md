# Cosmosdb

Mainly some miscellaneous stuff.

## Change feed

The change feed in Azure Cosmos DB listens to a container, then outputs a sorted list of documents in order in which they were modified. This output will then be passed to a processor. You can choose multiple processors to handle the change feed in parallel (for example, have three separate feeds for A-G, H-K, L-N, each with their own processor. Change feed is enabled by default. Reading from the change feed costs DBU.

## Security

 Some layers of security are:

- Network security: CosmosDB has IP-level controls. Also there are virtual network service tags: a certain group of IP addresses of a certain network security group.
- Authorization: CosmosDB uses HMAC (hash message authentication code) - each request is sent with a signed hash, CosmosDB hashes the request and sees if it's correct, and if the key is allowed to make that request. You can use resource tokens to let users sign requests, but only to certain containers/partition keys/UFs/etcetera. Or just give out the master key :D
- RBAC - You can use RBAC to give resource tokens, to make sure users only make requests they are allowed to.
    - Of note: when not using the master key, you have to use CosmosDB users, not AD users. Access rules are handled within CosmosDB. AD users can create CosmosDB users though, with the right role. Then, these CosmosDB users can request resource tokens.
- Global replication - Famous CosmosDB feature. not really a security thing, but I guess it helps against regional failures. Cosmos automatically failsover if you replicated to multiple datacenters.
- Various other security features, such as automatied backups, audit logging, HTTPS encryption, Encryption at rest, and it's impossible to have an admin account with no password!

## SLA

99.99% SLA for single region, 99.999% SLA for multi region.

