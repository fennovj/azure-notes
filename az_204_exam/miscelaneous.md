# other

## Logic apps

The main strength of Logic Apps is the hundreds of pre-built components, such as Dropbox, Salesforce, SAP, Youtube, Azure Functions, Office 365, Slack, etcetera

Example: New tweet about your product: detect sentiment, if positive -> store link in database, if negative -> email customer service.

You can also write custom connectors, as long as the service has a REST api. You create an OPEN API file to describe the API, then upload and create a custom connector.

Use Logic apps when:

- You need to integrate multiple applications and systems
- You want scalability, but not neccecarily subsecond response time
- You don't need highly complex conditional logic, Azure Functions is better for that
- You have connectors, or are prepared to take a few hours to write a custom connector

Example: Social Media monitor is a perfect fit, integrates email/database/twitter.

Another example is checking daily if there are old videos to archive. Bit more complex, and not as much integration, but still logic apps will work nicely

Online sales process that alerts paymentprocessor/suppliers/shippers - will likely involve complex business logic based on which item/amount of item/location, and different shippers might all ave different methods of being alerted, meaning writing custom APIs for all of them. Logic apps could work, if we had the resources to write all the custom connectors. However, consider using code because it might be easier to write complex business rules in code instead of logic apps.

TODO question:
Which would you use to edit B2B workflows in logic apps:
Which would you use to edit definitions in JSON

## Logic apps tools to edit

- Logic Apps Designer - can visually add functionality
- Code View Editor - can edit definitions in JSON
- Enterprise Integration Pack - for B2B workflows
- API connections - can create custom APIS for logic apps

## Azure Web Jobs

Web Jobs can be used to run a program/script in the same context as a web app, as an App Service For now, it is not supported on Linux yet. It supports: `.cmd, .bat, .exe, .ps1, .sh, .php, .py, .js, .jar`

### Continuous

- Starts immediately when the job is created. Typically, these kinds of jobs contain some infinite loop. You can manually restart if it ends
- Runs on all instances that the Web App runs on, but you can optionally restrict to single instance
- Supports remote Debugging

### Triggered

- Starts only when manually triggered or scheduled
- Runs on a single instance (typically, the one that is currently not under load)
- Doesn't support remote debugging

SCHEDULED/INSTANCED do not exist! 'scheduled' is actually called 'triggered' as above

## Get/set roles

Here is info on creating a custom role: <https://docs.microsoft.com/en-us/azure/role-based-access-control/custom-roles-powershell>

First, you need to get an existing custom role:

```powershell
Get-AzRoleDefinition <role name> | ConvertTo-Json

{
    "Name": <role name>
    ...
    "Actions": [
        "Microsoft.Storage/*/read",
        "Microsoft.Network/*/read",
        "Microsoft.Compute/virtualMachines/start/action",
        "Microsoft.Support/*",
        etc.
    ]
    "NotActions": [], # The actions that are exceptions to the above list
    "AssignableScopes": [
      "/subscriptions/11111111-1111-1111-1111-111111111111"
    ]  # which scopes this role can be assigned to
}

# List all operations for virtualmachines:
Get-AzProviderOperation "Microsoft.Compute/virtualMachines/*"

# Create a new role from json file:
New-AzRoleDefinition -InputFile "C:\CustomRoles\customrole1.json"

# Updating a role:
# start from new or existing role. Here, new role
$role = [Microsoft.Azure.Commands.Resources.Models.Authorization.PSRoleDefinition]::new()
$role.Name = 'XXX'
...
$perms = 'Microsoft.Storage/*/read','Microsoft.Network/*/read','Microsoft.Compute/*/read'
$role.Actions = $perms
New-AzRoleDefinition -Role $role

# or if the role already existed:
Set-AzRoleDefinition -Role $role

# if you want to delete the role:
Remove-AzRoleDefinition -Role $role
```

## Azure Redis Cache

<https://docs.microsoft.com/en-us/azure/azure-cache-for-redis/cache-overview>

Temporarily copy frequently accessed data to fast storage close to the application. This will be loaded in-memory instead of on disk.

Common use cases:

- Cache-Aside: instead of entire database, load items in cache as needed.
- Content Caching: cache static content like headers/footers/toolbars/etc
- User Session Caching: instead of storing a lot of data in cookie, save key in cookie, and use key to query session cache.
- Job/message queuing: defer long running operations to queue
- Distributed transactions: store transactions in memory while commands are executing

Basic/Standard/Premium is mainly the number of nodes, SLA, and throughput/latency.

## OTHER

- Shared service tier vs Standard service tier App service, which can automatically scale?
  - answer: standard, because shared is basically free with custom domains
  - Note: premium also gives 250GB disk space, up to 30 instances, and 'Clone App'
  - Isolated makes it run on an isolated backbone network
- azure api management service
- Azure Application Insights - datamodels (metric/dependency/trace/event)