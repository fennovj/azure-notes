# Work with NoSQL data in Azure CosmosDB

General terms

- NoSQL
- Request Unit
- Partition Key

## Creating CosmosDB

You need the following

- Resource group, region - like usual
- Account Name - unique name. The URI will be something like `<name>.documents.azure.com`
- API - There are 5: SQL, Gremlin, MongoDB, Azure Table, Cassandra.
- Spark - disable unless you need
- Geo-redundancy - can be done later
- Multi-region writes - write to multuple regions at the same time

### Request unit

Request unit is approximately the cost of a GET on a 1-KB document using a document's ID. Creating/replacing/deleting or not getting by the ID costs more. You provision RU in increments of 100 RUs per second. Factors that affect it:

- Number of query results
- Number/nature of predicates
- Number of user-defined functions
- Size of source data/result set

Same query on same data always costs the same on repeated executions. If you exceed the RUs, you are rate-limited. Ideally, it will be retried after a certain time limit, depends on which SDK you use. 400RU/s is the minimum, 250.000 RU/s is the maximum

### Partition key

A single server will run out of space, the smartest way to address this is to scale out, with smart partition keys.

Logical divisions like userID make sense since you will often filter on userID. Avoid hot partitions such as day, since the current day will be requested way more. Don't be afraid of choosing a partition key with a lot of possible values. (like userid), and choose a partition key you filter on often.
You cannot change the partition key after creating a cosmosdb database.

## How to choose the API

There are 5: SQL, MongoDB, Cassandra, Azure Table and Gremlin (graph).

NOTE THAT SQL does not mean the data is stored in a column-wise, relational way. It is still NoSQL at its core, you just query it with SQL.

Example GET queries (the gremlin one also filter on an outward edge):

- SQL: `SELECT c.productname FROM Items c`
- MongoDB: `db.Items.find({}, {productname:1,_id:0})`
- Cassandra: `SELECT productname FROM catalog.items`
- Table API: `SELECT c.productname FROM Items c`
- Gremlin: `g.V().hasLabel('item').has('productName', 'Refridgerator').outE('BoughtWith')`

The GENERAL RULE is to use SQL for everything, except:

- Existing MongoDB/Cassandra/Azure Table/Gremlin data -> use that
- Need graph data or metadata about relations between data -> use Gremlin

SQL is also a good fit for simple key-value pairs. Previously, Redis or Table API were recommended, but these days, just use CosmosDB with SQL, since you get a richer query experience than Redis, and improved indexing over Table API. In general, it may still be a good idea to migrate from Cassandra or MongoDB to SQL (according to Microsoft), if it doesn't cause too much disruption to the existing business.

Note that it is of course possible to model Graph data in NoSQL. For example, edges are just tuples of source/destination. However, the lack of native queries means that performance will suffer. Example: you want to store products, and the amount of times that a customer bought multiple products together, or maybe model relations between properties of customers, and flag certain relations as suspicious.

## Actually using CosmosDB

Below you have to add resource group parameter to everything as well

```bash
az cosmosdb create --name $NAME --kind GlobalDocumentDB
az cosmosdb sql database create --account-name $NAME --name "Products"
az cosmosdb sql container create --account-name $NAME --database-name "Products" --name "Clothing" --partition-key-path "/productID" throughput 1000
```

You can use the data explorer to navigate/view/query/modify data, and run stored procedures. It is a good way to get acquainted, e.g. by adding a single item manually without messing around with commands.
An item just needs an 'id' and ideally your partition key as well, as well as other variables.

Note that azure adds some metadata like '_rid', '_self', etcetera, no need to worry about it.

Queries look like 'SELECT X FROM Y WHERE Z', you can also order by, join etcetera. A JSON in list-of-dicts-format is returned.

### Stored procedures/UDFs

You can also define stored-procedures and user-defined functions, that will run complex procedures on the data.

- Stored procedures are ACID because they run on the server.
- UDF are complex and are used for computational logic on values/documents.

They are written in JAVASCRIPT :O

Example:

```javascript
function sample(prefix) {
    var collection = getContext().getCollection();
    var isAccepted = collection.queryDocuments(
        collection.getSelfLink(),
        'SELECT * FROM root r',
        callback_function_that_returns_json // func(err, feed, options) {}
    );
}
// or
function create() {
    var collection = getContext().getCollection();
    var accepted = collection.createDocument(
        collection.getSelfLink(),
        {'id': '3', 'data': 'xxx'},
        callback_function_that_returns_json // func(err, documentCreated) {}
    );

}
```

Example user defined function:

```javascript
function producttax(price) {
    if (price == undefined)
        throw 'no input';

    var amount = parseFloat(price);

    if (amount < 1000)
        return 10;
    else
        return 20;
}
```

Then, you can do: `SELECT c.productId, udf.producttax(c.price) AS producttax FROM c` to call the function.

## Using the Graph API

Recall that a Graph database consists of Vertices (objects), Edges (relationships) and Properties - metadata about vertices/edges.

Advantages are that you can add more types of relationships as edges, which in NoSQL, would add a lot more complexity. The weakness is that it is worse at high volumes/aggregations of data, and not effective at queries that traverse the entire database.

I did javascript, it was a lot of boilerplate, here is some pseudocode:

```javascript
const Gremlin = require("gremlin");
const config = require("./config"); //contains endpoint for gremlin, authkey, and database/collection
const authenticator = new Gremlin.driver.auth.PlainTextSaslAuthenticator(
    `/dbs/${config.database}/colls/${config.collection}`,
    config.authKey
    );
const client = new Gremlin.driver.Client(config.endpoint, {authenticator, ...})
const result = await client.submit(query);
```

Then you can execute queries like:

```javascript
g.addV('Product').property('id', 'p1').property('name', 'Phone Charger').property('price', 12.99)
g.addV('Category').property('id', 'c1').property('name', 'Mobile Phones')
g.V('p1').addE('belongsto').to(g.V('c1'))
```

You can also view the graph in the data explorer.

More queries:

```javascript
g.V().hasLabel('Category').has('id','p1') // filter
g.V().hasLabel('Category').values('name', 'price') // only select certain properties
g.V().hasLabel('Product').order().by('name', incr).values('name','price') // order by
g.V('p1').property('price', 15.99) // update properties
```

## Move app with Table API to Cosmos

The main reasons to move Azure Tables to Cosmos is better performance, especially globally. Lower latency, more throughput. Also, better global data consistency. There are some differences compared to Tables:

- Max 255 bytes in row keys, max 2MB batch operation
- no CORS, tablenames are case-sensitive

There are two migration tools:

- Azure Cosmos DB Data Migration Tool: open source, from many sources into CosmosDB.
- AzCopy: command line, copy from/to Azure Storage accounts

The exercise didn't do any of this, they just took an existing app, changed the connection string from Tables to Cosmos, and repopulated the storage.

## Create a .NET app with VSCode

You can create a cosmosDB in vscode: images here:

![creating a cosmosdb account](https://i.imgur.com/uCDYesX.png)

![creating a database](https://i.imgur.com/npq2HLv.png)

Then, initialize a dotnet app:

```powershell
dotnet new console
dotnet add package System.Net.Http
dotnet add package System.Configuration
dotnet add package System.Configuration.ConfigurationManager
dotnet add package Microsoft.Azure.DocumentDB.Core
dotnet add package Newtonsoft.Json
dotnet add package System.Threading.Tasks
dotnet add package System.Linq
dotnet restore
```

Also, in `App.config`, add the connectionstring and key. [See code here](https://docs.microsoft.com/en-us/learn/modules/build-cosmos-db-app-with-vscode/3-setup)

Code looks like:

```csharp
this.client = new DocumentClient(endpoint_url, key)
await this.client.CreateDatabaseIfNotExistsAsync(new Database { Id = "Users" });
await this.client.CreateDocumentCollectionIfNotExistsAsync(UriFactory.CreateDatabaseUri("Users"), new DocumentCollection { Id = "WebCustomers" });
Console.WriteLine("Database and collection validation complete");
```

The `DocumentClient` contains the most important CosmosDB methods, like `CreateDatabaseIfNotExistsAsync`, `CreateDocumentCollectionIfNotExistsAsync`, but also:

- CreateDocumentAsync
- ReadDocumentAsync
- ReplaceDocumentAsync
- UpsertDocumentAsync  :eyes:
- DeleteDocumentAsync

Note there is no update!!!! You can only create/replace/upsert. Replace will error if the document doesn't exist yet, Create will error if the document already exists. Upsert will not error, but instead create a new one/update existing one.

Example code:

```csharp
    try
    {
        await this.client.ReadDocumentAsync(UriFactory.CreateDocumentUri(databaseName, collectionName, user.Id), new RequestOptions { PartitionKey = new PartitionKey(user.UserId) });
    }
    catch (DocumentClientException de)
    {
        if (de.StatusCode == HttpStatusCode.NotFound)
        {
            await this.client.CreateDocumentAsync(UriFactory.CreateDocumentCollectionUri(databaseName, collectionName), user);
        }
    }
```

### Quering using Linq

Linq is a .NET programming model that expresses computations as streams of objects. You can create an object that queries CosmosDB directly in Linq.

Example: `input.Select(family => family.parents[0].familyName)` is like selecting the familyName from the first parent from every family.

```csharp
IQueryable<User> userQuery = this.client.CreateDocumentQuery<User>(
        UriFactory.CreateDocumentCollectionUri(databaseName, collectionName), queryOptions)
        .Where(u => u.LastName == "Pindakova");
// Or just plain SQL
IQueryable<User> userQuery = this.client.CreateDocumentQuery<User>(
        UriFactory.CreateDocumentCollectionUri(databaseName, collectionName),
        "SELECT * FROM User WHERE User.lastName = 'Pindakova'", queryOptions );
```

### Running stored procedures

```csharp
await client.ExecuteStoredProcedureAsync<string>(UriFactory.CreateStoredProcedureUri(databaseName, collectionName, "UpdateOrderTotal"), new RequestOptions { PartitionKey = new PartitionKey(user.UserId) });
```

## Partitioning and indexing

There are three metrics to consider:

- Total request per second
- Total throughput in RU/s
- Total storage in KB

Note that RUs consider the volume of data that's read/written, but also the complexity, number of fields, indexing, and data consistency model. If you exceed the maximum RU/s, you get throttled. For that reasoning, it is smart to think about partitioning and indexing, because it makes you use less RU/s.

- Partitioning is the distribution and grouping of data across underlying resources.
- Indexing is a catalog of document properties and values. It makes reading faster, but writing slower.

### Examples of writing to collections

- Allocate a collection with 400 throughput - it is throttled at 400/s. In case of the example, that is 26 operations per second.
- Create a collection with a Hot Partition. Essentially a badly chosen partition key. It starts at 60 operations, 900 RU every second. But it keeps growing to 60 operations, 1000 RU per second as it gets harder to insert.
- Use the id as a partition key. This seems to be the smartest option? However, it seems almost the same as the previous.

### Measure throughput of individual queries

Go to the portal, Data Explorer, new SQL query.

![screenshot here](https://docs.microsoft.com/en-us/learn/modules/monitor-and-scale-cosmos-db/media/4-querystats-tab.png)

What we see is that when quering on the partition ID, the RU/s is about half of the RU when quering on a different field.

### Partition strategy

- Estimate scale of data needs: what is the size of documents, and volume of documents being queried?
- Understand workload: Is it read or write heavy? If read, what are the top queries? For write, do you need transactions?
- Select partition key:
  - Does the partition key have many possible values?
  - Is there a consistent spread
  - Are some values accessed more than others?
  - For read-heavy, do common queries go across partitions?
  - For write-heavy, do tranasctions go across partitions?

Example: don't choose ordertime - too large of a resolution if you choose seconds, hot partition if you choose hour/day. Item category is not a good choice if the cardinality is too low. Item Id is a good choice.

You can see the impact in portal -> metrics -> throughput.

### Indexing strategy

There are three modules:

- Consistent - index is updated every time new document is written, writing is more expensive but ensures consistency
- Lazy - Index is updated eventually, reads/writes have higher priority. Writes are cheaper. Reads are only consistent when index is updated.
- None - No index is created, reading is more expensive

You can change the index strategy at any time. Note that the CosmosDB default is indexing all properties. This is the best for read-heavy workloads. For write-heavy workloads where you only read/query on a certain field, consider changing the default.

Here are some estimates:

- Read document directly - 1 RU
- Query document with indexes - 3 RU
- Query indexed document with partial index - 3+ RU
- Query non-indexed document, partial index - 10+ RU
- Query document with no index at all - 20+ RU
- Insert document with full index - 13 RU
- Insert document with lazy index - 7 RU
- Insert document with some index - 6 RU
- Insert document with no index - 5 RU

## Distribute globally

Advantages are low-latency around the globe, and resiliancy and disaster recovery.

Here is an example of creating global replication:

![nice gif](https://docs.microsoft.com/en-us/learn/modules/distribute-data-globally-with-cosmos-db/media/2-global-distribution/2-global-replication.gif)

### Multi-master support

Multi-master support means that each region is equal, in a write-anywhere model. Once you write anywhere, it is propagated too the other regions.

Advantages are far lower write-latency, and higher scalability for writing: you don't have to all write to the same region that happens to be the master. However, we do have to deal with conflict resolution, when 2 regions write to the same record. There are three methos:

- Last-Writer-Wins (default) - a custom integer property, whoevers record has the highest value wins. By default its `_ts` which means last writer wins
- Custom - UDF - Write a function to decide what happens, you can 'merge' two records in a custom way
- Custom - Async - conflicts are not committed but instead registered to a read-only feed. The application can asynchronously read the queue and resolve it in a custom way.

### Automated failover

If a failure happens in a master region, the SDK redirects to the next closest region (that also is a master). So manual failover only really applies to single-region-write apps.

- If a read region has an outage, calls are redirected to the next availably region. No settings are required in the app.
- If a write region has an outage, and automatic failover is enabled, an alternative region is automatically promoted as the write region.

You can enable automatic failover in the portal, and set the priority list.

### Consistency level

Considering someone can write to one region, and someone else can read that same record from another region, there needs to be an agreement about a consistency level. There are 5 consistency level:

(Note: If I say letters here, that is the same document, but different values. Assume the values are written to the database in the order A, B, C but in different regions)

- Strong - linear operations, reads are guaranteed to return the most recent version of an item.
- Bounded Staleness - Reads lag behind writes by at most t interval
- Session - Strong consistency within a single client session. It guarantees that reads/writes within a session are processed in order, and you can read your own writes in the correct order (RYW). Outside of a session, there is eventual consistency.
- Consistent Prefix - Guarantees that you don't see out-of-order writes, so you might see A, B, (and not C yet) but never A, C.
- Eventual consistency - Out of order reads. You might see out-of-order values, like B, C, A, but eventually you will see C.

Keep in mind that the first two are far easier with a single-write region. This has effects on latency and cost. If you want low latency globally with multiple write regions, the first two are probably not suitable.

Session can be used globally, because we can assume that a single client session only writes to a single region. Example: someone fills a shopping basket in US West, if they are also logged in in EU West, their shopping basket might not update immediately, but if they are in the same session in US West, their shopping basket will update correctly.
