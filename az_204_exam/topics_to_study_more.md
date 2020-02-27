# other topics and links

- Shared access signature
  - <https://docs.microsoft.com/en-us/azure/storage/common/storage-sas-overview>
  - Gives access during a specified time, mainly useful for external parties. For internal services, In azure use Gen2 storage which has RBAC, to give to Managed Identity.

- Authentication
  - Managed Service Identity - Basically an automatic identity a service gets while running in azure
  - Role Binding, Cluster Role Binding are AKS specific
  - Service Principal <https://docs.microsoft.com/en-us/azure/active-directory/develop/app-objects-and-service-principals> - a user identity for an app

QUESTION: what is the difference between MSI and Service Principal? Which is each better/suitable for?
Managed service identity is a service principal managed by azure, so you can't access the secrets or anything.

- Events/messages stuff, including code
  - <https://docs.microsoft.com/en-us/azure/service-bus-messaging/topic-filters>
  - Service bus queue

- Active directory integration
  - <https://docs.microsoft.com/en-us/azure/aks/azure-ad-integration> (this is AKS though which wont appear)
  - AD service principal
  - Conditional access policy

- Key Vault
  - Destination (HSM or software) MUST BE SPECIFIED (along with key name) when creating key
  - Disk encrpytion <https://docs.microsoft.com/en-us/azure/virtual-machines/windows/encrypt-disks>
    - If you want key vault for use with disk encryption, you need to pass '-EnabledForDiskEncrpytion' when creating the key vault. Then, use `Set-AzVMDiskEncryptionExtension` where you pass the KeyVault URL/ID, and Key URL/ID.
