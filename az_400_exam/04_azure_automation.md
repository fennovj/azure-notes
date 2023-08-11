# Azure Automation

<https://learn.microsoft.com/en-us/azure/automation/automation-dsc-getting-started>

In a nutshell, Azure Automation is a combination of terraform and ansible. But with a lot of Azure sauce over it of course. There are also some other functionalities, such as tracking changes/inventory, and integrating with power automate/azure monitor/etcetera.

Also, it works with Windows and Linux servers, both in Azure and on-premise.

Essentially, you make 'runbooks jobs' and 'watchers'. runbooks deploy something and can also do post-deployment stuff, like e.g. running install scripts or triggering a power automate flow. Watchers track changes and configuration etc.

## Desired State configuration (DSC)

This is kind of similar to terraform/arm/ansible, in that instead of defining the steps to undertake, you describe the desired state.

Example DSC with a single Node that has IIS installed:

```powershell
configuration TestConfig
{
    Node IsWebServer
    {
        WindowsFeature IIS
        {
            Ensure               = 'Present'
            Name                 = 'Web-Server'
            IncludeAllSubFeature = $true
        }
    }
}
```

The way you use DSC is:

- Onboard a VM to Azure Automation, making it a managed node
- Upload a configuration such as the above to Azure Automation
- Compile the configuration into a node configuration (also called a MOF document)
- Assign a node configuration to one or more managed nodes (uses a runbook)
- Using a watcher, check the compliance status of your managed nodes
