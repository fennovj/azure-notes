# Azure fundamentals Notes

[Big picture overview of azure](https://docs.microsoft.com/en-us/learn/modules/welcome-to-azure/3-tour-of-azure-services)
![azure](https://docs.microsoft.com/en-us/learn/modules/welcome-to-azure/media/3-azure-services.png)

## Big list - Do you know what all these are

- azure logic apps
- azure batch
- azure cosmos db
- azure machine learning studio
- azure advisor
- azure cli
- azure powershell
- azure resource manager
- azure policies
- management groups
- resource groups
- azure traffic manager
- azure government

## Practice tests

[Udemy](https://www.udemy.com/course/microsoft-azure-fundamentals-az-900-practice-tests/) - 161 questions, 20 euro

[Whizlabs (Yorick bought this, ask him if you need login!)](https://www.whizlabs.com/microsoft-azure-certification-az-900/practice-tests/)

## Introduction

## Architecture & SLA

## Core cloud services

## Azure Portal

### Management options

- Azure portal - GUI web option
  - you can also create dashboards, save them and share them, etc
- Azure powershell - a bunch of powershell commands- older
- Azure command line interface - newer, multiplatform (windows+linux) version of azure powershell
- Azure cloud shell - web based version of the two above (you can choose to use either one)
- Azure mobile app - a mobile app, similar to azure portal

### Azure advisor

- In azure portal, gives best practices recommendations, and proposed actions
- Improves performance, security, and availability, helps reduce costs
- The topics that it's about:
  - High availability
  - Security
  - Performance
  - Cost

### Public/private review

- new storage types, new azure services, or new apis
- private review: for specific azure customers, typically invite only by the team that created the feature
- public preview - for all customers. Just search for 'preview' when creating a new resource
- Portal preview: go to preview.portal.azure.com

## Compute

### VMS

- availability set - two VMS where one is always* available during planned(update domain)/unplanned(fault domain) maintenance -- no extra cost, so always do if you have/want multiple VMs
- scale set - duplicate the VM, no extra work in configuring routing to multiple vms
- azure batch - automatically starts a pool, runs jobs, requeues work, scales down at the end

### Containers

- azure container instance - Paas that just runs a docker (NACO I think)
- AKS - kubernetes - mcroservice (1 for frontend, 1 for backend, 1 for storage, etc) -- or you can use SLA-backed services for some parts (such as storage and AD)(for this use  Open Service Broker for Azure (OSBA) and AKS for the rest

### Azure app service

- Can be used for web apps, APIs, webjobs, mobile apps
- web apps - ASP.NET, ruby, jode, php, python, windows or linux
- apis - many languages/frameworks, with swagegr support
- webjob - .exe, java, php, python or node, or bash/.powershell, run by trigger
- mobile app backend - sql, social authentication, push notifications, etc

### Serverless

The platform manages server instances. Each function execution is on a different compute instance
Event driven, so more events is more triggers. Micro-billing (2 minutes of compute is 2 minutes of billing)

- Azure functions - Basically any modern programming language

## Storage

### Types of data

- structured data/relational data - adheres to schema: All of the data has the same fields/properties
- semi-structured data - no schema, but it does have tags/keys that provide hierarchy to data. e.g. NoSQL
- Unstructured data - no restrictions, e.g. pdf, jpg, json
  - these can be (semi-)structured (e.g. a structured hierarcy of jpg files), but unstructured means there is no restriction

### Options

- Azure SQL database - Runs on MSSQL. high performance, reliable, etc.
- Azure Cosmos DB - NoSQL, globally distributed. Good for constantly changing data and frequent reads/writes
- Azure Blob Storage - unstructured. similar to disk, but supports thousands of simultaneous users and massive file sizes.
  - Supports streaming to browser, and stores up to 8TB for VM's, 2PiB max total size
  - Blobs have a max size (can be gigabytes) but in general aren't used for super random data like Box
- Azure Data Lake - Large repository that stores structured and unstructured data
  - This is where you can dump everything including super random data like Box
  - In Data Lake, you can do analytics and for example move all images to Blob Storage or Data Warehouse
- Azure Files - File share that can be accessed in Windows/Linux/MacOS as network drives
- Azure Queue - Used to storage messages. Provides asynchronous message queueing. Senders/Receivers can subscribe to it.
  - Also can be used for load balancing/queueing, e.g. save up messages for later during high burst of traffic
- Disk Storage - Just a disk that you can use in an Azure VM. Mainly used for IaaS, and Lift-and-Shift of data between VMs.

### Misc

- Strorage tiers: the lower the cheaper, but most apps will just want hot
  - Hot storage tier - for frequent access
    - lower access costs, higher storage costs
  - Cool storage tier - for data thtat is stored for at least 30 days, infrequent access
    - lower storage costs, higher access costs
  - Archive storage tier - For rare access, stored at least 180 days, will have 'flexible latency'
        - You have to 'rehydrate' to hot or cool to access it, can take several hours
- Encryption/replication:
  - Storage Service Encrpytion (SSE) - encryption for data at rest - mainly for security/regulatory compliance
  - Client-side encryption - data is encrypted/decrypted by client application, azure never sees decrypted data
  - Replication - Belongs to storage account, you can have several tiers, like replicating across region or geography

### Comparison with on-premise

- More cost effective
- More reliable/fault tolerant
- More storage types
- easier/agile to create new storage types with changing technology

## Networking

### N-tier architecture

- Method of building a large enterprise systems with 'loosely coupled architecture'
  - loose coupling is better in general - each component have no knowledge of inner workings of other components
  - components can be added/changed/updated/replaced/scaled independently
- Uses multiple tiers - a higher tier can access lower tier, but not other way around
  - Web tier (higest tier), communicates with user through browser
  - Application tier with business logic
  - Data tier
- Azure provided pre-configured 3-tier vm environments, with virtual networks/security features

### What does that mean

- Virtual network - A logically isolated network on azure, allows resources to communicate with each other
  - can be segmented into subnets.
  - Only the web tier would have a public ip adress
  - You can also use on-premises services, and use VPN gateway to allow communication
  - You choose which IP adresses can reach the virtual network/other way around
- Network Security Group (NSG)
  - Allows or denies inbound traffic to azure resource. It's like a firewall
  - e.g. allow port 80 for everyone, but port 22 only from your office

### Azure Load balancer

- Used for high availability (e.g. 5 nines) and resiliency (natural disasters or traffic spikes or DDoS)
- Load balancer balances across multiple VMs, or even across availability zones/regions/geographies
- The load balancer does not need maintenance. It has 4 nines though...?

### Azure Application Gateway

- If you only have HTTP traffic, Azure Application Gateway may be better: designed for web apps
- It has URL rule-based routes, and some other stuff like headers/cookie rules

### Other

- CDN: A network of servers, especially in different geographies to deliver content faster. Can be hosted in azure like any other app
- DNS - 100% SLA !!!! (Thanks Yorick)

### Gateways

- [docs here](https://docs.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-about-vpngateways)

## Security

### security stuff

- IaaS means outsourcing physical security to microsoft. PaaS also means outsourcing OS security, etcetera
- You are always still responsible for the proper endpooints/accounts/access management
- Layers of security
  - Inner layer - data, stored somewhere - this is what attackers are often after
  - Application - make sure app is without vulnerability - you have to do this yourself unless you use SaaS
  - Compute - secure access to Compute - you have to do yourself only if you use IaaS or on-prem
  - networking - deny by default, etcetera
  - Perimeter - use firewall to detect/filter DDoS
  - Identity and Access - user management/SSO/ etc
  - Physical security - microsoft takes care of this

### Azure security center

- Free and Standard tier. Standard is $15 per node per month
- Provides reccomendations, continous monitoring, use ML to detect malware, JIT access control for ports
  - Incident reponse - detect/access/diagnose incidents
  - Enahcne security - reccomendations

### Identity/access

- Especially important with BYOD/work from home, and mobile apps
- Azure AD provides authentication, SSO, Application management, B2 identity
- Provide identity/authorization to servies
  - Service principal - A principal is an identity with certain roles. A service principal is an identity used by a service
  - Managed identities for azure services - manages the creation of service principals automatically.

### RBAC

- Roles are sets of permissions (e.g. you can read x, write y, etc. Call it 'Contributor' or some such)
  - Roles can be granted to a resource group, or an individual resource
- Privileged Identity Management (PIM) - additional service that provides oversight of roles

### Encryption

- You can encrypt all your data, at rest/in transit/etc
- Azure Key Vault (AKV) - Encrypt secrets/keys/certificates
  - Centralized application for secrets
  - Monitor access
  - Simplified administration
- HTTPS (TLS) Certificates can be issued per service, you can store them in AKV (AKV has features like auto-renewal)

### Firewall

- Azure Firewall - a firewall that supports non-HTTP protocols, such as FTP/SSH, also outbound
- Azure Application Gateway - a firewell designed for HTTP, also protects from common vulnerabilities
- Network virtual appliance (NVA) - similar to hardware firewall, for non-http or advanced
- Azure DDoS protection - does what it says. Basic (automatic, free) or Standard (even better)

- Virtual network Security
  - Network security group - group with inbound/outbound rules
  - Network integration - you can create a VPN and connect it to on-prem VPN
  - Azure Express Route - extend on-prem network into microsoft cloud, means no exposure to public internet

- Azure information protection - when using 365 enterprise, automatically detecs/labels credit card numbers, for example

### Azure threat protection (ATP)

- Azure ATP portal - monitor/respond to suspicious activity
- Azure ATP sensor - monitors domain controller traffic
- Azure ATP Cloud service - i have literally no idea

## Azure policy

You could allow nobody to create azure resources, only IT team. on-prem is nice, but is limiting.
In real life, you want to allow others to create stuff but it needs to meet checks

- create policies such as 'no more than 4 CPUs'
- This is like RBAC, but during deployment. Also, it is default-allow.
- policies such as:
  - allowed SKUs, resource types, locations
- You can also identify non-compliant resources and stop them

### Initiatives

- Initiative definition is a set of policies that help towards a larger goal
- Example initiative: 'Enable monitoring in Azure Security Center', has multiple policies

### Management groups

- Normally, access management occurs at the subscription level - each user has access to subscription(s)
- Management groups is a hierarchy above subscriptions.
- e.g.: HR group, has a DEV subscription and Apps subscription. They both inherit conditions for HR group
  - example conditions: VM locations US West only
  - Create RBAC that applies to all HR group subscriptions

### Azure Blueprints

- Declarative way to deploy roles/policies/Azure Resource Manager templates/resource groups
- First you create it, then you assign it. The assignment says what *was* deployed
- You can do this with Resource Manager Template as well, but this creates assignments as well which is nice for tracking
  - Blueprint can contain resource manager templates as well, so no need to choose
- Blueprint can also contain policies

### Compliance manager

- Four sources that Microsoft uses to provide full transparency:
  - Privacy statement - a statement that they promise to share your data with the US government
  - Trust center - basically a blog where they talk about how great and secure they are
  - Service trust portal - They publish audit reports here, and compliance guides for GDPR and such
  - Compliance manager - dashboard where you can track your regulatory compliance activitiees
    - Contains relevant microsoft audit results and 'risk scores' such as how non-gdpr compliant you are

### Monitoring

- Azure Monitor
  - Data about performance/functionality/operation of the code, and azure itself
  - some data by default, you can enable more yourself for $$$
  - there are tools for monitoring containers/VMs/web apps
  - It can raise alerts, take corrective actions, and autoscale
  - It can visualize in PowerBI or dashboard
- Azure service health - a suite of experiences
  - Azure status - contains global view of status of azure
  - service health - tracks state of your azure servies, such as upcoming planned maintenance
  - resource health - helps when azure service issue affects you, and understand if SLA was violated

## Azure Resource manager

- The layer that interpretes/executes your Azure CLI/Azure Portal commands. Also authenticates your requests
- Terminology:
  - Resource: an item such as VM/database
  - Resource group: a group of resource
  - Resource Provider: a service that supplies resources, such as Microsoft or yourself (hybrid cloud)
  - Resource Manager template: very similar to terraform

### Azure resource group

- can you access resources from a different group? - YES
- can a resource group have multiple regions? - YES
- can resources in a group have different tags? - YES
- if you set tag on the group, do all content also get that tag? - NO
- same as above, but for permission? - YES

### Tags

- you can apply a tag to every resource/resource group
- example: Department=Finance
- Simplest thing you can do is search/filter resources by tag
  - You can also use it to group billing - eg view billing per department
  - You can also monitor alerts - eg see which departments are affected by alert
  - Also, can create tags such as shutdown:6PM, that automation jobs can read those tags and act on them
- Tags are not inherited by resources in a resource group
- not all resources support tags

### Resource locks

- set resources to disallow deletion or editing
- Mainly just an extra step to prevent accidental deletion, but not really a security feature
- Can help as a reminder in case you forgot that a certain resource shouldn't be deleted or edited

## Pricing

- usage meters - a VM can have usage meters such as compute hours, data transfer, IO, different effet on pricing
  - It's always based on usage - e.g. if you disable the disk, you can never get charged for IO

### Factors affecting cost

- Resourrce type - e.g. storage is different than compute
- Services
- Location
- Billing Zones - Inbound data is free, Outpund data is different per zone which it goes to.

- Azure pricing calculator - drag and drop, you can make estimates
- Cost management - overview of where your money is going
- Azure Advisor - makes recommendations for how to save cosots
  - example, it can see that your VM has average CPU usage of <5% so you can make it smaller
- TCO calculator - sort of general calculator where you add your workloads and it will estimate costs Onprem vs azure (azure is always lower)

### Saving infrastructure cost tips

- visual studio subscribers get ~50-150$ per month for free
- You can enable spending limits
- You can use reserved instances if you have CONSTANT, STATIC, PREDICTABLE workloads
- choose low-cost locations/regions
- potentially use new services that are cheaper
- Right-size underutilized virtual machines
- Deallocate virtual machines in off hours
- Delete unused virtual machines
- Migrate to Paas, since it's cheaper than IaaS

### Saving Licencing costs

- Linux is cheaper than windows
- If you have subscriptions to SQL Server/Windows Server, you can reuse them in Hybrid cloud
- For a test/develiopment server, you don't have to pay Windows License/SQL Server costs (thanks microsoft!)

## Miscelaneous other stuff

### Support plans

- Basic (free):
  - Billing & Subscription support, online self help, support forums
  - Full access to all Azure Advisor recommendations
  - Service Health dashboard
- Developer (29/mo):
  - Get support engineers via email,
  - Software (PowerBI, Office 365, Azure Portal) configuration guidance and trouble shooting
  - <8 hours support time
- Standard (100/mo)
  - <4 hours or <1 hour response time depending on if the business impact is critical
- Professional Direct (1000/mo)
  - Architectural guidance and best practices, azure advisor consultations, web seminars
- Premier (contact us)
  - <15 minute response time if critical (does not come with premier by default, costs extra)
  - Customer specific architectural support delivered by Azure Technical specialists
  - On demand training/reporting

### Free account

- Free access for 12 months
  - Examples: 750 hours of B1S Linux VM, 5GB of CosmosDB, etc
- Additional 200$ credit for the first 30 days to use on ANYTHING (cant use on 3rd pary on azure marketplace)
- 25 products that are free for everyone regardless:
  - Examples: 10 web apps with App Service with max 1GB of storage, 5 users free with Azure Devops, etc
- After 30 or if $200 runs out, you have to convert to pay-as-you-go

## Questions I got wrong

### Test 1

I messed up  'Understand Azure Pricing and Support' - Outside of that I only got 1 wrong. Resources:

- [Overview of support plans](https://azure.microsoft.com/en-us/support/plans/)
- [Free account](https://azure.microsoft.com/en-us/free/free-account-faq/)
- [MEGA long page with service limits, not gonna read this](https://docs.microsoft.com/en-us/azure/azure-subscription-service-limits)
- Cost calculators overview:
  - Future costs:
    - Azure pricing calculator - Specific calculator if you know which resources you want
    - TCO Total Cost of Ownership - more general overview of what the costs savings of azure will be
    - Azure advisor - If you already use Azure! Recommends how to save in the future eg by downscaling underused VMs
  - Past costs:
    - Cost management - nice dashboard how much you spend and where
    - Cloudyn - basically worse version of cost management and advisor

- Does the basic support plan come along with a Free account - yes
- Does the free account allow you to host production-based resources - yes
- Do public preview features come with an SLA - no - PROVIDED AS IS WITH ALL FAULTS LIMITED WARRANTY
- Does azure provide a separate portal for public preview - yes - preview.portal.azure.com BUT public preview is also available through the normal portal
- You want regular architecture reviews, which plan do you need - Premier (Premier is highest, it goes Basic->Developer->Standard->Professional Direct->Premier)
- Should you use Azure Cost Management before migrating to cloud - No, cost management is to see costs you already made
- How do you increase the limit of 20 vCPUs - raise a customer support request

- Can you have resources in a resource group but different locations - YES - a resource group has a region, but it is normal to contain resources with multiple different regions

### Test 2

- How many free accounts can a customer have - maximum 1
- What is a feature of Azure SQL Data warehouse - Scalability
  It can have availability/disaster recovery, but you have to do that yourself e.g. with availability sets
- Does Azure Advisor give recommendations on Virtual Network Settings - No [See more here](https://docs.microsoft.com/en-us/azure/advisor/advisor-overview)
- How do you push events from multiple resources into a centralized repository for analysis - Azure Monitor
  - Azure Event Hubs is a Big Data Streaming/Ingestion service
- What is Azure Traffic Manager - [see here](https://docs.microsoft.com/en-us/azure/traffic-manager/traffic-manager-overview)
- Who can use Azure Government - US Government contractors/government entities
- How to connect On-Premise data Center to Azure Virtual Network - Azure Virtual Network Gateway

### Test 3

- A company has a VPN devise that will be used on a Site-to-Site connection from Azure to the on-premise location. Which would be used to represent the VPN device - Local Network Gateway
  - It's not a Virtual Network Gateway... I got a similar qestion wrong in test 2, maybe study up?
- Can you use Azure powershell on linux - yes
- Can a billing administrator or a co-administrator make support requests by default - co-administrator... A billing administrator is the person who has access to change/see/remove the billing info, but by default not a 'technical' person.

### Test 4

## Misc short notes of stuff I might forget

- Traffic manager - monitors endpoints availability/performance and directs to the one with the best performance
  - This is especially useful if you have deploymed in multiple regions
- azure advisor - is about security, performance, availability, cost
- Virtual Network, Virtual Network Gateway, Local network Gateway

- scale set - deploy identical vms (even across regions)
  - can increase availability
- azure information protection - for labeling sensitive emails etc
- azure storage accoutn - contains all your blobs/disks/tables/etc
- cosmodb can be accessed using mongodb api
- azure monitor can raise alerts if your stuff stops working
  for alerts about cost, use azure cost management
