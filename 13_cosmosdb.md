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
