# CosmosDB

This document contains some more specifics and code examples for cosmosdb

## Table API in .NET SDK

Example of insterting an entity:

```csharp
string storageConnectionString = '...';
string tableName = 'customers';
CloudStorageAccount storageAccount = CreateStorageAccountFromConnectionString(storageConnectionString);
CloudTableClient tableClient = storageAccount.CreateCloudTableClient(new TableClientConfiguration());
CloudTable table = tableClient.GetTableReference(tableName);
bool created = await table.CreateIfNotExistsAsync();

// CustomerEntity is a subclass of TableEntity with attribute Email and PhoneNumber (among others)
public CustomerEntity(string lastName, string firstName) {
    PartitionKey = lastName;
    RowKey = firstName;
}
CustomerEntity entity = CustomerEntity('Smith', 'John');
TableOperation insertOrMergeOperation = TableOperation.InsertOrMerge(entity);
TableResult result = await table.ExecuteAsync(insertOrMergeOperation);
CustomerEntity insertedCustomer = result.Result as CustomerEntity;
Console.WriteLine(result.RequestCharge) // RUs used
```
