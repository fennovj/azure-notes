# Storing data in azure

There are three types of data:

- Structured, or relational data
- Semistructured data, like XML, JSON, or YAML
- Unstructured data, like photos, videos, pdf, word, text, log files

Example: product catalog data can be structured, but if you add many specialty fields like 'bluetooth-enabled', etcetera, it is too cumbersome to add new columns for all of them, so semi-structured is easier.

## Operational need

Think about the following

- Are you just doing simple lookups on id? (complex queries are easier with SQL)
- How many operations per second do you expect? Data science not much, hotly visited website: much more
- How quickly do they need to complete? Few seconds okay for analysis or cron job, user interaction need much lower latency.

### Transactions

Transactions are database operations that operate together. They are **ACID**

- Atomic - exactly once, either all or none
- Consistent - Data is consistent before/after operation
- Isolated - Transaction is not affected by other transaction
- Durable - Changes are saved permanently, even if the system fails after the transaction

### OLTP vs OLAP

Online Transaction Process vs Online Analytical Processing. Mainly, OLAP is for fewer uses and longer response times, but large and complex transactions. OLTP is for many concurrent users, a lot of data being sent/received, quick response times, but simple queries and transactions.

### Conclusions

For OLTP semi-structed customer data, Azure reccommends Cosmos DB. It's ACID-compliant and can handle many requests concurrently. You could also use SQL if you can make a good relational schema.

For unstructured data with low latency and high throughput, but no transactional support, Blob storage will work fine. For scaling up, use a CDN.

For structured business data, of course SQL storage makes the most sense. Data warehouse will also work, but does not support cross-database queries (oh well?)

## Storage accounts

A storage account is a container that groups the following together:

- Azure Blobs
- Azure Files
- Azure Queues
- Azure Tables

It does not include CosmosDB, Azure SQL, etcetera - those have their own storage servers. The main advantage is that it makes it easier to use those 4 together in an application, and they can share settings. If you delete the storage account, it deletes all the data.

There are settings such as location, performance, replication, security, virtual networks. How many you need is defined by

- Data diversity - is data specific to a country/region, or some public/private data? If so, they may need separate storage accounts
- Cost sensitivity - is some data fine with lower performance? If so, you can save money with more storage accounts with different settings
- Management overhead - different settings can be nice for more control, but it also means more management, also with deciding with storage account to use for each part of the application

There are three account options

- GP v2 (general purpose) lowest per-gigabyte costs
- GP v1 - does not have latest features, but lower prices
- Blob storage account: legacy, only supports block/append blobs. Similar pricing to GP v2

### Creating a storage account

You can use the Portal, CLI, Powershell, or other stuff like Blueprints/terraform/etc. Keep in mind that storage accounts are relatively stable and aren't created all the time. (like functions)

A storage account name must be globally unique.

## Security

There are several security features, like Encryption at rest, encryption in transit, RBAC, CORS, auditing.

### Storage account keys

each storage account has 2 shared keys that give access to EVERYTHANG. Only use for in-house applications ;) It might be a good idea to regenerate them periodically anyway. To do that, switch everyone to the second key, and then change the first key. Later, switch everyone to the first key, and then change the second key, etcetera.

### Shared access signature

This is for external third-party applications. You can add constraints and time range of access. This is for a situation where clients can read/write data to the storage account. It's easiest to make the front-end proxy do the authentication and only forward good requests, but it is more scalable to have a separate authentication service, that communicates with both server and client to get/authenticate a key.

### Network access

By default, storage accounts are public. You can additionally restrict them to certain IP ranges or virtual networks.

### Advanced thread protection

Basically, check in real time if something weird is happening. It's an additional service offered by microsoft, that sends an email to ad admin if there is some alert.

### ACL (Access Control List)

Basically kind of a RBAC.

## Blob

![Blob storage](https://fp2w.org/assets/ext/blob.jpg)

Blob storage is for unstructued data. They have higher latency than Memory/local disk, and don't have indexing like SQL, but they have very high throughput. E.g.: send the url in an API, then download from that URL with high throughput thanks to blob storage. It can also be used like a fily system or a static webserver.

Every blob lives inside a blob container. Containers can only store blobs, not other containers.

Best practice is making an app check if container exists, and if not, create it

- You can either make blob storage very structured, with tags and file names, or just dump everything and automate it with GUID etcetera
- You can make a blob public-access, which is very nice for scalability
- You can give a blob a hierarchical name like 'finance/budgets/2017.xlsx', and then using the blob API, you can filter on prefix. In this way, it resembles a normal file system.

### Blob types

- Block blobs - composed of blobs that can be uploaded independently. Generally good and fast for most types of data
- Append blobs - you can only append new data, but very efficient (for example for streaming data)
- Page blobs - great for random access read/writes

### Code examples

As mentioned, it is best practice to check for the container on startup, and create it if it doesn't exist.

It turned out to be a lot of code, [Here it is though.](https://github.com/MicrosoftDocs/mslearn-store-data-in-azure/blob/master/store-app-data-with-azure-blob-storage/src/final/Models/BlobStorage.cs)

Then, they deploy the app like this: (maybe nice to get some inspiration for deployment to an app service)

```bash
app_service_name=blob-exercise
app_name=my_app_name
storage_account_name=my_storage_account
resource_group=XXXXXX

az appservice plan create --name $app_service_name --resource-group $resource_group --sku FREE --location westeurope

az webapp create --name $app_name --plan $app_service_name --resource-group $resource_group

CONNECTIONSTRING=$(az storage account show-connection-string --name $storage_account_name --output tsv)

az webapp config appsettings set --name $app_name --resource-group $resource_group --settings AzureStorageConfig:ConnectionString=$CONNECTIONSTRING AzureStorageConfig:FileContainerName=files

dotnet publish -o pub
cd pub
zip -r ../site.zip *
az webapp deployment source config-zip --src ../site.zip --name $app_name --resource-group $resource_group
```

## Javascript app

There is a javascript app as well, which is easier for me than C#! Jay!

- Create node app with index.js
- Create storage account
- Try the REST API: `https://[url-for-service-account]/?comp=list&include=metadata`
  - Or a python/javascript library, which is just a thin wrapper over the REST API

Initialization:

```javascript
const util = require('util');
const storage = require('azure-storage');
const blobService = storage.createBlobService();
const createContainerAsync = util.promisify(blobService.createContainerIfNotExists).bind(blobService);
await createContainerAsync('testblob');
```
