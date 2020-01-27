# Manage resources in Azure

## Table of contents

- Cloud types (public/private) and service models (IaaS PaaS etc)
- Azure CLI
- Azure Powershell
- Predict costs and spending
- Azure Resource Manager

## Cloud types and service models

Public cloud consists of services offered over public internet available to everyone. Operated by third-party. Private cloud consists of resources exclusively used by one organization. Hybrid is both :).

Hopefully I won't struggle with any questions about these. Examples:

- An SQL server is needed for one week, after which it will be thrown away. Use Public because you don't need to buy an entire server to use for one week.
- You have VMs in the cloud on a VPN that connects to an on-prem datacenter with sensitive customer information. Hybrid obviously.
- You have two datacenters in your organization, one of the datacenters that because of regulatory reasons, has data that cannot be moved. TRICK QUESTION it's still private because they are both in your org.

I SKIPPED THE ENTIRE SECTION ABOUT IAAS/SAAS/PAAS because i was pretty confident I knew about it already.

## Azure CLI

You can install the Azure CLI with apt-get and such. Test if it's installed with `az --version`

To login, do `az login`, you are redirected to sign-in page. If you can't open a browser, it will instead show a code which you can enter at <https://aka.ms/devicelogin>.

You can find the most popular commands like this:

```bash
az find blob # popular commands related to blob
az find "az vm" # popular commands for the vm command group
az find "az vm create" # popular parameters/subcommands for az vm create
az vm create --help # more detailed information about parameters
```

The most common commands are `az XXX create --name YYY --location ZZZ` to craete something, and `az XXX list --output table` to see existing things.

You can also filter the `az X list` with queries: `az group list --query "[?name == 'XXX']"`. This filter is JMESPath. (<http://jmespath.org/>), a common language for filtering JSON.

### Example: Deploying a webapp in an app service plan

```bash
az group create --name $RESOURCE_GROUP --location $AZURE_REGION
az appservice plan create --name $AZURE_APP_PLAN --resource-group $RESOURCE_GROUP --location $AZURE_REGION --sku FREE
az webapp create --name $AZURE_WEB_APP --resource-group $RESOURCE_GROUP --plan $AZURE_APP_PLAN
az webapp deployment source config --name $AZURE_WEB_APP --resource-group $RESOURCE_GROUP --repo-url "https://github.com/Azure-Samples/php-docs-hello-world" --branch master --manual-integration
```

Then go to <webappname.azurewebsites.net> in the browser to see your hello world. (where webappname is `$AZURE_WEB_APP`)

## Azure Powershell

Azure powershell is mainly an alternative to the CLI. There is also the portal, but that is not really an alternative, it's just different. Portal is especially good for one-off things, like creating a storage account that will probably not be deleted any time soon.

The main reason to use Azure powershell (in my opinion) is if you already have powershell experience. Which I don't. However, note that Azure powershell is totally available in linux with apt-get and such.

Powershell uses 'cmdlet' as a command that does a single thing. They are formatted as 'Verb-Noun', e.g. 'Get-Process', 'Start-Service', etc. `Get-Help` shows the help for a cmdlet, e.g.: `Get-Help Get-ChildItem -detailed`.

Cmdlets are loaded in modules. You can show all modules and their commands with `Get-Module`. To add the azure module, do: `Install-Module -Name Az -AllowClobber`.

### Example: create resource group

```powershell
Import-Module Az  # Import the Az module
Connect-AzAccount  # Shows interactive prompt. If non-interactive, use parameters instead
Select-AzSubscription -Subscription "Visual Studio Enterprise" # Select your subscription
New-AzResourceGroup -Name <name> -Location <location>  # Create new resource group

Get-AzResource | Format-Table  # List existing resources
```

Creating and updating a VM:

```powershell
New-AzVm -ResourceGroupName X -Name Y -Credential (Get-Credential) -Location O -Image P
$vm = Get-AzVM -Name MyVM -ResourceGroup X
$vm.HardwareProfile.vmSize = "Standard_DS3_v2"
Update-AzVM -ResourceGroupName X -VM $vm
```

Powershell scripts:

```powershell
# Loops
$adminCredential = Get-Credential -Message "Enter a username and password for the VM administrator."
For ($i = 1; $i -lt 3; $i++)
{
    $vmName = "ConferenceDemo" + $i
    Write-Host "Creating VM: " $vmName
    New-AzVm -ResourceGroupName $resourceGroup -Name $vmName -Credential $adminCredential -Image UbuntuLTS
}
# Executing scripts from a file:
.\my_cool_script.ps1 -size 5 -location "East US"
# Capturing command-line parameters:
param([string]$location, [int]$size)
```

## Predict costs and optimize spending

First of all, a general note is:

- You can always check the billing page in the portal to get an overview of what is being charged
- Stuff is only charged on use. Compute resources are only charged when running or allocated, storage is charged as long as there is storage on it.

Pricing is affected by:

- Resource Type: duh
- Service: Enterprise, Web Direct, and Cloud Solution Provider (CSP) customers have different prices sometimes
- Location: North europe is cheaper because it's colder there lmao
- Billing zones: inbound traffic is usually free, outbound data depends on the billing zone. First 5GB per month is always free though. Generally, First world is cheapest, after that Japan/Australia/etc, then Brazil and South Africa are most expensive.

The Pricing calculator is a good place to check if a certain resource that is not location-sensitive can be cheaper somewhere else.

### Azure Pricing calculator

See it here: <https://azure.microsoft.com/pricing/calculator/>.

There are two tabs: Products for adding new products to your current estimate, and Estimate for viewing past saved estimates.

You can share an estimate through excel spreadsheet, or URL.

### Azure Advisor

Azure Advisor makes these kinds of recommendations in the cost section. (there is also a High Availability, Security, Performance section not covered here)

- Eliminate unprovisioned circuits (Azure ExpressRoute) that still cost money
- Buy reserved instead of pay-as-you-go
- Downsize or shutdown underused VMs

You go to the portal, and it will show you a list of recommendations.

### Azure Cost management

Historical overview of costs, pie charts etc. You can also set budgets and schedule reports.

### TCO calculator

General calculator where you input labor/security/cooling costs etcetera, and it will compare public to private cloud.

## Saving on costs

- Use Azure credits for development/testing (150 per month baby), but is shut down after 120 hours and cannot be used for production
- Set spending limits
- Use reserved instances, can save up to 80%
- Choose low cost regions, north europe is colder so less cooling costs baby
- Sometimes there are [updates here](https://azure.microsoft.com/en-us/updates/) that have a significant effect on pricing.
- Right-size/Delete underutilized VMs
- Deallocate VMs in off hours, Azure has auto-shutdown.
- Migrate to PaaS/SaaS, it's cheaper

### Licensing costs

- Use linux, it's cheaper than windows (and better :eyes:)
- You can use existing Windows server licences. (maximum 16 cores or some stupid shit)
- You can use existing SQL server licenses. Excerpt: `If you have Standard Edition per core licenses with active Software Assurance, you can get one vCore in the General Purpose service tier for every one license core you own on-premises.` I don't really understand it nor do I care.
- Dev/test environments don't have licence charges, and bill you at the linux rate.

## Azure Resource Manager

This section is about stuff like policies. We don't want critical resources to be deleted, we want a standard for resource names/sizes etcetera.

- Resource groups
- Tags
- Policies
- Resource locks

### Resource groups

Resource groups are logical groups. It's also nice that you can put all your stuff in one group, then delete the group when you are done, everything in it is also deleted. They are also a scope for RBAC, and the resources inherit the permissions from the group.

Guidelines for resource groups:

- Consistent naming. Example: `learning-infrastructure-rg` is better than `my-resource-group-1`.
- Organize with a principle. For example, group by department, resource type, prod/dev/test, or multiple (dev-finance-rg, prod-marketing-rg)
- Put resources that are created/deleted at the same time in the same group, so you can manage them together easier.
- Put resources together that incur costs together, so your billing is easier to understand.

### Tags

Every resource can have some tags, that are just key=value. They can be stuff like: `Department:Finance` or `Environment:Training`, or even: `shutdown:6PM`.
The main uses are:

- Grouping billing data however you want.
- Making filtering in the portal/CLI easier
- Automation, automation jobs can read/use tags

Note that not all resources support tags

### Policies

Policies apply and enforce rules that resources need to follow. Examples

- All resources must have the `Department` tag.
- All resources must be in a certain location

Example rule in JSON: (generally you will go to the portal and export a template)

```json
{
  "mode": "Indexed",
  "policyRule": {
    "if": {
      "field": "[concat('tags[', parameters('tagName'), ']')]",
      "exists": "false"
    },
    "then": {
      "effect": "deny"
    }
  },
  "parameters": {
    "tagName": {
      "type": "String",
      "metadata": {
        "displayName": "Tag Name",
        "description": "Name of the tag, such as 'environment'"
      }
    }
  }
}
```

You can apply a policy to a scope (for example a resource group or a subscription.)

### RBAC on resources

RBAC uses a whitelist (allow model) to permissions. Best practices:

- Only allow specific access instead of unrestricted for everyone
- Grant users the absolute lowest that they need
- Use Resource locks (next section)

### Resource locks

You can block modification or deletion. Generally nice to have for critical infrastructure. Mainly used to prevent accidents, since everyone who has the right to modify/delete should already be trusted anyway.

Resource locks apply to everyone, even the owner. However, the owner can remove the lock. This extra step helps to prevent accidents, it does not really help against the admin getting phished, since anyone with the rights to delete also has the rights to remove the lock.
