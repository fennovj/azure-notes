# Azure Data Lake Storage

Azure Data Lake builds on Blob Storage to optimize it specifically for analytics workloads. It can deal with data at the exabyte scale and hundreds of gigabytes of throughput. It also has hdfs support, posix permissions, organizes data hierarchically (better performance), and has redundancy just like blob storage.

The main difference between a data lake and blob storage is that blob storage stores all data in a single hierarchy (flat namespace). The folders are 'fake', and simulated by putting slashes in the filename. For example, renaming a folder in a blob storage means renaming every file 'in' that folder.

This has an effect on Hive, Spark, etc, which often write to temporary directories and then rename the directory at the conclusion of the job. Without hierarchical namespaces, this rename can take longer than the job itself.

Data Lake can use both Data Lake API's as well as Blob APIs, since it's built on top of blob storage.

## Some example usecases

In general, the big data prorcess consists of four steps: ingest, store, prep&train, model&serve.

- A modern data warehouse
    - Ingest in Data Factory, store in Data Lake. Use PolyBase and/or databricks to put data in Data Warehouse (synapse?).
    - Use PowerBI to model&serve. (Azure Analysis Services creates a semantic data model and caches data for PowerBI)

- Advanced analytics against big data
    - Very similar to above. Data Factory -> Data Lake -> Data Bricks -> (Polybase) -> Synapse -> PowerBI.
    - Also, Databricks can write to CosmosDB for real-time apps. The real-time app is for example a reccommender system that is updated (by databricks) hourly.

![Infra picture](https://docs.microsoft.com/en-us/learn/data-ai-cert/introduction-to-azure-data-lake-storage/media/6-advanced-analytics.jpg)

- A real-time analytical solution
    - Again, data goes from Data Factory -> Data Lake -> Synapse (with polybase). With a Databricks processing engine that writes to Synapse/Cosmos.
    - Also, there is some sensors/IoT that writes to Data lake and/or databricks. For example with Kafka, but you could probably use an IoT Hub or Event Hub as well.

In general, Data Lake is used to store the (raw) data, not yet to model it in some relational way.

## Uploading data

For a single file, Portal is the best. Storage Explorer is more user friendly. When you get to hundreds of files, or are interfacing with non-Azure storage (or stuff like sql tables that are difficult to do manually), use Data Factory.

## Security

Some features include:

- Encryption at rest: Blob storage automatically uses AES, or encrypt VHD (with key in key vault)
- Encryption in transit: You can use HTTPS (even enforce it)
- CORS: you can lock down GET requests for images/videos to specific domains.
- RBAC
- Auditing access: built-in, Storage Analytics

### Storage account keys

#### Shared account key

these give access to everything. You should only use them for in-house apps, and probably regenerate them. (change app to secondary value, refresh the primary value, switch them)

#### Shared access signature

Basically, you give read/write/etc accesss to a part of the storage account, to control access better

### Data Lake

Data lake also has ACL (access control list) that is posix-compliant. It authenticats through OAuth2. You can use this for services (databricks, synapse, etc) as well.
