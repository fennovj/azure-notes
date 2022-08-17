# Resources

## Moving On-Premise from/to AWS

- Direct Connect - At first looks like a VPN between you and AWS. However, it goes further: You *physically* take your hardware to a Partner Location (e.g. Equinix in Amsterdam), or use specialized hardware to connect to them. Then, the traffic doesn't even travel across the internet. You can also rent (or buy?) special hardware to connect with the partner, so it still doesn't travel across the internet? But I'm not sure if this always works, or if you have to be close to the partner or something like that.
- DataSync - After you set up an AWS Direct Connect, you can use this to copy data to and from AWS. You can copy from/to S3, EFS, FsX for Windows, OpenZFS, Lustre, etcetera. If you want to transfer databases, you can only do that by transferring database snapshots e.g. to S3.
- Storage Gateway - A gateway that gives on-premise access to cloud storage. This is for hybrid architectures, e.g. you have on-premise video editing stations, but you use AWS as a network share. You *can* access Storage Gateway via Direct Connect, but also just via a regular VPN. In practice, this will mean having an on-premise VM act as a gateway, similar to PowerBI Gateway.
- Application Migration Service - Lift-and-shift migration. You install the 'Replication Agents' on all your servers, the server will be entirely replicated to EC2, and when it's all done and tested, you do a 'cutover' and shift to AWS. The cutover shouldn't take more than a few minutes.
- Database Migration Service - Another lift-and-shift. The goal is to migrate something like Oracle-to-Oracle, or even Oracle-to-Aurora if you want. For heterogeneous migrations, there is the 'Schema Conversion tools' that converts schemas from your existing database. Again, the goal is to minimize downtime when migrating.
- Snowball - This is the one where AWS literally ships you a hard drive, up to 80TB. After you transfer your data on it, it's shipped back and dumped in S3. Snowball is the service, 'Snowball Edge' is the name of the device itself. You can also order a Compute-Optimized Snowball Edge, which is like a data-center in a box, that you can run EC2 instances on if you want. Mainly useful if you are in a remote location but still want to use like Terraform or something.
  - There is also Snowcone which is much smaller but portable (meant for remote locations like power stations), and Snowmobile, which is a 45-foot truck that can store up to 100PB. Tanenbaum was right again.
- Migration Hub - This is a place to track your on-premise/aws resources, and the progress of migrations. It doesn't cost/do anything by itself, I guess it just exists to organize all the myriad of stuff above.
- Transit Gateway - A gateway to multiple AWS services to connect to your on-prem data centre. The main goal is if you are using VPC Peering on multiple VPC's and/or DirectConnect, AWS Transit Gateway means you just VPN to one server (on-prem?) and from there go to all the other AWS stuff. Hub-and-Spoke model.

## Networking

- Elastic Fabric Adapter (EFA) - Network interface for inter-node communcations. Works well for Message-Passing-Interface or NVIDIA-Collective-Communications-Library to scale to thousands of CPU's, but in general for high-performance communications. Works by bypassing host OS for inter-node communications. No additional cost.
- Elastic Network Interface (ENI) - The 'normal' network interface that represents a virtual network card. It usually has an (internal) IP adress.
- CloudFront - Content Delivery Network, mostly for Edge Caching. e.g. caching of images/html/javascript in edge servers. 
- S3 Transfer Acceleration - Speeds up the S3 requests over long distances by using Amazon's special networking. For applications where s3 transfer speed is critical. 
- Global Accelerator - This is like provisioning edge locations. Users are router through the edge locations to your resources. You can check the edge nodes here: <https://aws.amazon.com/global-accelerator/features/>. There are 200 nodes across the world, and you pay per edge location ($0.02/hr), and also a few cents per GB. Again, more expensive than CloudFront for high-throughput applications.
- PrivateLink - May also be called 'Private VPC Endpoint', as these VPCE's are powered by PrivateLink. The idea is you have a VPC with some service, e.g. a load-balancer, and you want others to access that service without going over the public internet. So you make them connect to some private vpce, and it appears as if your resource is in their VPC. The traffic will be routed over the AWS Backbone instead of over public internet. Note that the source requests must also come from within an AWS VPC.
- WAF - web application firewall - protects against 'common web exploits' and bots/ddosing. E.g. you can configure it to block requests that look like they are exploiting vulnerabilities specific to PHP/Javascript/whatever you use.
- NAT Gateway - The *main* usecase is to make instances connect outside of the VPC, but not the other way around. By itself, it does not allow access to the public internet. You generally just need one, in the public subnet. Then, from the private network, you redirect 0.0.0.0/0 to the NAT gateway. The NAT gateway can be in a different route table, that redirects 0.0.0.0/0 to the IGW. Basically, just see it as a weird router that sits in your public subnet. You can have multiple NAT gateways - the only reason is higher resiliency.

## Storage

- Aurora - Amazon SQL database. Basically the most 'managed' SQL database you can have in AWS. There is also a serverless version that scales automatically.
- Redshift - The Synapse/Snowflake alternative. OLAP, columnar based, that can do Massively-Parallel-Processing.
- Glue - Data catalog, containing metadata and such for data. It's not just for 'organization', you can also use it to e.g. store configuration and connection details for tables so that other AWS resources can read those tables easier without having to all separately having to store configuration.
- Athena - Serverless query service for S3 data. Point it at S3, define the schema, and start using SQL. Most queries take seconds at most. This works on csv, json, parquet, etcetera. The main use cases are often business analytics and ad-hoc queries.
- S3 Select - kind of the same thing as Athena. However, it only works on a single file, and is more meant for programmatic use. It' can also be used for push down filtering.'
- S3 Glacier Select - Similar to S3 select, but it works in glacier! Pricing differs based on your requested speed can be either a few minutes (expensive) or a few hours (cheaper). An advantage is you don't need to move the entire object out of glacier, you pay for the subset.


## Streaming

- Kinesis Data Streams - Serverless streaming service, where you can put in data (from e.g. microservices, logs, or mobile apps), and consume them using the below 2, or lambda.
- Kinesis Data Analytics - Stream Analytics kinda thing, allows you to run serverless SQL or Java on a Kinesis Data Stream. You can input/output directly from S3 or Kinesis Data Streams.
- Kinesis Data Firehose - ETL service, gets data from AWS, *optionally* transforms it (using built-in-transformations), and stores it in S3 or Redshift (or a few others).
- SQS - Simple queue service, for when Data Streams would be overkill. The main difference is Kinesis is more like Kafka, and Kinesis supports multiple consumers per topic/stream. Kinesis is better for real-time analytics and IoT, SQS is better for microservice decoupling.
- Data Pipeline - Managed ETL service. Analog to Data Factory. Main usecase is moving data from A to B, and do some minor transformations. Example usecase: move data from S3 to Aurora. Or move data from DynamoDB to S3.

## Other

- Resource Access Manager - Used to share resources across accounts. For a resource in account A, the goal is that any policies/roles/etcetera that apply in account B, also apply to the resource. You can also share across organizations.
- Organizations - Management tool for your AWS environment. Used to programatically create accounts, group accounts and apply policies to accounts, such as SSO or central logging and auditing.
- Trusted Advisor - Provides recommendations on 'best practices'. Account-level.
- EMR (Elastic MapReduce) - Big data platform, can be used to run Spark/Hadoop/Hive/Presto workloads. Can also be used for ETL.
- OpenSearch - An open source analytics suite that has a data store, search engine, and dashboard. Probably this won't come up as a solution, but it might come up as some sort of data sink. Kinesis Firehose and Cloudwatch have OpenSearch as a built in sink, For S3 or DynamoDb, you will need to use lambda to load data into it.
- CloudHSM - Hardware Security Module. Can generate and use encryption keys. You can also provision hardware that will do things like TLS/SSL processing for you, without the private key ever leaving the HSM(s)

## Last minute additions

- CodeBuild - Continuous Integration pipeline runner. mostly CI, can test/build your code, and dump artifacts somewhere.
- CodePipeline - Continuous Integration pipeline runner. Can run tests, builds, and run Cloudformation or AWS CLI commands. There are also Github/Jenkins plugins. It is a bit of both CI and CD
- CodeDeploy - Continuous Deployment. Can deploy to EC2, Fargate, Lambda, and on-prem.
- X-ray - A tool for debugging microservice applications. You can 'trace' data through your microservices, and create a view service map to analyze issues.
- Batch - A tool to schedule/run some (big) workload. Can run on Fargate or EC2. You submit a job to a 'job queue', configure how many resources you need, AWS does everything else. No additional costs beyond the costs of Fargate/EC2.
- Polly - SaaS for Text-to-speech. Can mostly be used to create a robot voice for your applications. Mostly used in lambda by calling the Polly API in lambda. (And storing the Audio in S3 if neccecary)
- Step Functions - Low-code application builder, with built-in connectors. You can do stuff like invoke lambda, run ecs task, etcetera. Not really meant for non-programmers to process their Outlook message or whatever, it's mostly about connecting microservices.
- Simple Workflow Service - A tool to build microservice application in Java. Mainly, you build asynchronous code using the 'flow framework'. You can use this to e.g. distribute work across multiple machines, wait for output, recover from lost nodes, etcetera. I don't see myself using it since Spark exists, but I'm sure there are good usecases for it.
- Fargate - Version of Elastic Container Service. Basically, it manages your EC2 instances for you, you just have to define the compute/memory you need, so you don't need to make your own autoscaling groups. Basically more managed and the 'real' version of ECS in my opinion. Can also be used with EKS to automatically provision instances based on your compute needs.
- Lambda@Edge - Function to run codes in cloudfront edge nodes. Probably the main usecase is to improve cache-hit ratio by doing some preprocessing on the request:
  - Modify headers, 'normalize' the user agent or query string.
  - Modify an image (that is in cache) e.g. by resizing, without having to get a different version from the source.
  - You can also do more complicated stuff like A/B testing