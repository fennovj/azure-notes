# Azure Storage (Lake and Blob)

<https://docs.microsoft.com/en-us/azure/storage/blobs/>

( The 'Blob Storage' chapter is the same for blob and lake, it also contains a 'Data Lake Gen2' chapter)

## Security

- Data Protection
    - Enable Advanced thread protection (detects potentially harmful stuff).
    - Turn on soft delete to recover deleted stuff.
    - Use immutable blobs (WORM, write once, read many) for business-critical data
    - Limit SAS Tokens to HTTPS only.
- Identity
    - Use Azure AD (see below)
    - Grant SAS tokens instead of permant access when that suffices. User delegated SAS tokens are even better - they are secured with the user credentials instead of the storage account key.
    - Use principle of leasy privilege when granting SASs
    - Share account key in Key Vault instead of plaintext somewhere
    - Regenerate account keys periodically.
    - Disable anonymous access
- Networking
    - Enable secure transfer required
    - Enable firewall rules to limit IP addresses
    - Allow trusted microsoft services.
    - Use private entpoints (such as a VNet)
- Logging/monitoring
    - Enable Azure Storage logging to monitor requests authorization.

### Access control (with AD?)

- RBAC - RBAC appies sets of permissions to service principals. In Blob storage you can only apply permissions to the entire account. In Data Lake Gen2, you can apply permissions to an individual container. (but not to individual folders!)

## Access control lists (ACL)

Note that Data Lake have this, but Blob storage don't. They require hierarchical namespace!

- Access control lists - Azure Data Lake Only!!! In such a list, you can associate Read/Write/Execute privileges (POSIX baby) with service principals. Note that this is on top of the permissions granted other ways. For example, you cannot use ACL to deny access to someone that already has the account key, or deny someone that is already Blob Data Contributor, or deny someone with a relevant SAS token.
- Default ACL - Default ACL's are associated with a directory, and will be applied to any newly created child items (such as files or new subdirectories)

Example of how ACL works:

- You need Execute privileges on a directory to access any child items of the directory. This also applies to all points below.
- You need Read privileges to list the contents of the directory.
- You need write privileges to update a file, or to create a new file
- You do not need write privileges to delete a file or empty directory (but you do need write privileges on the parent)
- To delete a non-empty directory, you need write privileges on the parent (as listed in previous point), and also need read+write privileges on the non-empty directory to delete its content.

There is also a *super user group*, who can basically do anything, delete anything, change owning groups, change ACL's, etcetera. By default this group is empty.

Each file/directory also has an *Owner group*, a service principal who is the owner. The owning group is by default the primary group from the user who created the file/directory. The owner of the root is the user who created the container.

It is reccommended to never assign users directly, but always assign groups instead. This makes it much easier to manage permissions and the reasons for them. Azure helps this by assigning the *primary group* from the user as the owner, instead of the user identity itself.

- example: Alice has access to folder foo, bar and baz because she is a sysadmin, and also to foo because she is a developer. she stops being a sysadmin. When she is assigned directly, her permission needs to be revoked from bar and baz, but not from foo. It is better to assign only groups, in which case Alice can just be removed from the sysadmins group, and will still have access to the foo folder.

