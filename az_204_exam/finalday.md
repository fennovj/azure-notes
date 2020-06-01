# Miscellaneous topics

Miscellaneous topics I struggled with

- optional claims
    - Basically this entire How-to guide: <https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-enterprise-app-role-management>

- Azure funciton
    - If your function times out after 5 minutes, can you increase the timeout in host.json?
        - For a job/background task, yes that's fine. For a premium app service backend you can even do 60 minutes
        - In this host.json you can also define logging, healthmonitor, extensions
        - However, for a http request, you shouldn't do that. Instead start a background task with webhook (built in, in durable functions!)
    - If you want to reduce latency on azure function, should you use serverless or app service plan?
        - The big difference is cold start. serverless can have cold start if not ran in a while.
        - This latency can be up to 10 minutes (wow that seems long...)
        - However, if you have a high surge in requests, serverless could potentially have less latency under heavy load.
    - If you want an azure function that interacts with blob storage (as input/output), what do you need to do?
        - You can use an Azure storage Trigger, where a create/update is used to trigger. content of blob is given as input to function
        - You can use an input binding, where you get some blobdata as input. You can use a queue trigger with a string to use a different blob each time
        - You can use an output binding, where you get blobs with write access. On blob trigger, you can use the name to configure the name of the output blob

- funnel/impact/user flow stuff
    - Azure application Insights:
        - Implement: there is a default package in ASP.NET, package for django, etc. So no coding effort after setting it up.
            also it's of course supported for web apps/azure functions/etc (you need to have always on on your web app though right?)
        - Usage analytics
            - Funnel: The progression through a series of steps, and step-by-step conversion rates
            - Cohorts: complex filters, such as 'engaged users', you can then split by country/view number of sessions/etc
            - Impact: analyse effect of load time (or other properties like country) on conversion rate
            - Retention: How many users return to app
            - User Flow: View how users navigate the site, find repetitive actions, etc

- Key vault operations:
    - All:
        - Backup: returns an encrypted version of the secret (all versions)
        - Restore: Upload a backed up version TO THE SAME SUBSCRIPTION, but can be a different vault
        - Get, Get deleted (if recoverable), Get versions, Purge (perma-delete), Recover, Set, Update - speak for themselves
    - Keys:
        - Encrypt/Decrypt: send some data, get the encrypted/decrypted data
        - Sign: Creates signature with the key
        - Import: Creates a key (with a BUNCH of RSA parameters), Set is not allowed for keys
        - Verify: Verify a signature with the key
        - Unwrap/Wrap - wrap/unwrap a symmetric key
    - Certificate:
        - Get issuer, get operation, get policy, get contacts, etc.
        - Merge Certificate
        - Udpdate issuer, update operation, update policy

- Logic apps
    - Enterprise Integration Pack (sharing business data, for example flight ticket availability/order) (formats: EDI/EAI)
        - It's kinda just an extension of Logic Apps, but it needs an 'integration account' to work and store artifacts
        - This integration account is the same as the javascript one, it just happens to have a bunch of B2B features added to it
    - Code view editor: Literally you just write json files. Like datafactory. Not really userfriendly or anything, but it is more powerful than the UI
    - Visual editor: the classic 'drag-and-drop' interface for making logic apps

- AzCopy
    - Can be used of course for copying to/from azure storage accounts
        - `azcopy make 'https://mystorageaccount.blob.core.windows.net/mycontainer'`
        - `azcopy copy 'C:\myDirectory\myTextFile.txt' 'https://mystorageaccount.blob.core.windows.net/mycontainer/myTextFile.txt'`
        - `azcopy copy 'https://mystorageaccount.blob.core.windows.net/mycontainer/myTextFile.txt' 'C:\myDirectory\myTextFile.txt'`
        - you can add '--recursive' for directories
    - Copy between blobs:
        - `azcopy copy 'https://<source-storage-account-name>.blob.core.windows.net/<container-name>/<blob-path><SAS-token>' 'https://<destination-storage-account-name>.blob.core.windows.net/<container-name>/<blob-path>'`
        - You do need a SAS token

- Azure Service Bus
    - gebruik je azure service bus of queue storage als je FIFO, transactional support wil? - SERVICE BUS, BOTH NOT IN QUEUE STORAGE
    - Filters - each subscription has a filter
        - No filter means TRUE filter which means all messages
        - Boolean filter (all or none)
        - SQL Filter (SQL-like property (>, <, !=, arithmetic, LIKE, IN, NOT IN, EXISTS, IS NULL, etc)
        - Correlation Filter - Select property(s) and a value for each to match. When selecting multiple, ALL conditions must match
            - Usually you want to use CorrelationId, since that is the property that you should use to define context for the purpose of correlation

- Azure Front door
    - Service for managing/monitoring global routing
        - Accelerate performance: uses microsoft global network to immediately route client to nearest POP, ensures least latency
        - Smart health probes, automatic failover when one of the backends goes down
        - URL based routing, e.g. <http://website/users/*> goes to a different backend than <http://website/products/*>
        - Multisite hosting, you can host multiple sites on the same front door, and use the same/different backend for them
        - Session affinity, make sure a session stays routed to the same backend
        - TLS termination, setup up TLS with front door instead of (long haul) with backend
        - manage certificates/domain names (not unique, but hey you get it for free)
        - Firewall (it comes with DDoS protection as a free extra)
        - Auto redirect HTTP to HTTPS
        - URL rewrite, and even configure host header
        - Supports IPv6 and HTTP/2
        - Pricing: 0.2 euro per GB outbound, 0.01 euro per GB inbound, 0.026 euro per routing rule per hour (19 eur per month)
