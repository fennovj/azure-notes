# Resources

## Moving On-Premise from/to AWS

- Direct Connect - Essentically a VPN between you and AWS. However, it goes further: You *physically* take your hardware to a Partner Location (e.g. Equinix in Amsterdam), or use specialized hardware to connect to them. Then, the traffic doesn't even travel across the internet. You rent (or buy?) special hardware, or meet with the partner.
- DataSync - After you set up an AWS Direct Connect, you can use this to copy data to and from AWS. You can copy from/to S3, EFS, FsX for Windows, OpenZFS, Lustre, etcetera. If you want to transfer databases, you can only do that by transferring database snapshots e.g. to S3.
- Storage Gateway - 
  - Gateway Cached Volume - 
  - Gateway Stored Volme - 
- Server Migration Service
- Database Migration Service
- Migration Hub - 

## Networking

- Elastic Fabric Adapter (EFA) - Network interface for inter-node communcations. Works well for Message-Passing-Interface or NVIDIA-Collective-Communications-Library to scale to thousands of CPU's, but in general for high-performance communications. Works by bypassing host OS for inter-node communications. No additional cost.
- Elastic Network Interface (ENI) - The 'normal' network interface that represents a virtual network card. It usually has an (internal) IP adress.
- CloudFront - Content Delivery Network, mostly for Edge Caching. e.g. caching of images/html/javascript in edge servers. 
- S3 Transfer Acceleration - Speeds up the S3 requests over long distances by using Amazon's special networking. For applications where s3 transfer speed is critical. 
- Global Accelerator - This is like provisioning edge locations. Users are router through the edge locations to your resources. You can check the edge nodes here: <https://aws.amazon.com/global-accelerator/features/>. There are 200 nodes across the world, and you pay per edge location ($0.02/hr), and also a few cents per GB. Again, more expensive than CloudFront for high-throughput applications.

## Storage

- Aurora - Amazon SQL database. Basically the most 'managed' SQL database you can have in AWS. There is also a serverless version that scales automatically.
- Redshift - The Synapse/Snowflake alternative. OLAP, columnar based, that can do Massively-Parallel-Processing.

## Streaming

- Kinesis Data Streams - Serverless streaming service, where you can put in data (from e.g. microservices, logs, or mobile apps), and consume them using the below 2, or lambda.
- Kinesis Data Analytics - Stream Analytics kinda thing, allows you to run serverless SQL or Java on a Kinesis Data Stream
- Kinesis Data Firehose - ETL service, gets data from AWS, *optionally* transforms it (using built-in-transformations), and stores it in S3 or Redshift (or a few others).
- SQS - Simple queue service, for when Data Streams would be overkill.

## Other

- Resource Access Manager - 
- EMR (Elastic MapReduce) - Big data platform, can be used to run Spark/Hadoop/Hive/Presto workloads.