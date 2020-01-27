# Secure your cloud data

A note of warning: This module is somewhat disorganized - microsoft seems to have made somewhat disconnected and overlapping modules and threw them together in one learning path. As a result, some stuff is doubled up, and title headers are not consistent.

In general, cloud doesn't solve your security problems, but azure will take part of the burden (like physical security, security updates in case of PaaS, etc). Thinks like exposing endpoints, rights management and account management is still always the user responsibility.

Layered security means that if one layer is breached, another layer is there. The Data is the final layer.

- Data
- Application - should be free of vulnerabilities, secrets stored in the right place
- Compute - should be kept patched
- Networking - Deny by default, limit communication
- Perimeter - Prevent DDOS, Firewall
- Identity and access - SSO, multifactor authentication, audit changes
- Physical security

List of top 10 web application security risks: <https://owasp.org/www-project-top-ten/>.

## Overview

### Azure security center

- Free version - assesment and recommendations only
- Standard tier - continuous monitoring, thread detection, just-in-time access control for ports and more. $15 per node per month

In general you detect suspicious behavior, assess what happened, and diagnose what can be improved. Security center of course helps with all three.

- Assessment, recommendations, security polity
- Network/VM threat detection, , adaptive application controls, JIT VM access

It also works for on-prem resources that are linked to Azure.

### Azure Active Directory

Has two things of interest: Single-Sign on and Multi-factor authentication.

Also, services have identities

- Service Principal
- Managed identities - similar to service principal but much easier, not available for all services. Automatically creates an account on your AD tenant

### Encryption

- Encryption at rest
- Encrpytion in transit

### Certificates

certificates are used for website data in transit

- Service certificates used for cloud services
- Management certificates used for deploying with the classic model, like Visual Studio orAzure SDK

### Network security

- Azure Firewall - managed service that provides protection for HTTP, RDP, SSH, FTP, and others
- Azure application gateway - load balancer that includes firewll
- Network virtual appliance - ideal for non-http, similar to hardware firewall
- Azure DDOs protection
- Virtual network security - limit resources within a VNET to only what is required
- Network Security Group - filter traffic in vnet to only what is required. and/or completely restrict public internet access for some resources
- VPN - security communication between channels, such as azure and on-prem resources

You can also protect documents with Microsoft Azure information Protection, by adding labels to documents and emails. You can then track access to documents...

### Azure Advanced Threat Protection

???

### Microsoft Security Development Lifecycle

- Provide training - security is everyone's job
- Define security requirements - legal, internal, based on previous incidents, known threads
- Define metrics and compliance reporting - KPI's such as all vulnerabilities with 'critical' must be fixed in x days
- Threat modeling - Discuss a model of a threat and establish risks and mitigations
- Establish design requirements - Set consistent requirements on design, don't give too much freedom in selecting security features
- Use crpytography standard - develop clear encryption standards, use only industry-vetted encryption libraries
- Manage risks from third-party components - Understand what happens if they contain a vulnerability
- Use approved tools, and security checks such as compilers and which warnings to enable
- Perform static analysis security testing - Automatic analysis after every commit for certain things
- Perform dynamic analysis security testing - pre-built attacks checking if they work or not
- Perform penetration testing
- Establish standard incident response process - who to contect, what to do, and test the protocol before it is needed

## Top 5 things to consider before deploying

- Azure Security Center - get recommendations etcetera (see section above)
- Validation of input/output. `Robert'); DROP TABLE Users;--`. Validate all data from third party or users. Validate/encode output data before sending to screen/users, to prevent XSS.
- Secrets in Azure Key Vault. You can rotate secrets, revoke leaked secrets, etcetera.
- Framework updates - chooise a framework that is industry-tested, well-supported (modern) and keep it updated.
- Use safe dependencies - and track CVE's on a site like <https://cve.mitre.org/>.

## Configure security policies to manage data

- Classify data at rest, in process, and in transit
- SQL Information Protection / Data Classification / Data Discovery is built into Azure SQL database and classifies/labels/protect sensitive data
- Advanced Data Security is build into Azure SQL database: combines: 15$/server/month
  - Data Classification (Data discovery)
  - Vulnerability Assessment
  - Advanced Thread protection
- Also monitor access to the sensitive data

Data recovery, retention and disposal policies:

- Blob storage has WORM (write once, read many) for critical data, e.g. for legal reasons

Data sovereignty:

- Often, we don't want data in a foreign countries to be subpoenaed by that foreign government. Instead, back up your data in a paired region.
- Now it talkes about the benefits of geo-redundancy... but not security related...? I guess it means that it's harder to attack both datacenters at once if they are hundreds of kilometers, although if your attacker can take down an entire datacenter, you probably have bigger problems.

## Securing storage account

In chapter 07_storing_data, header Security

## Securing SQL database

In chapter 06_relations_data, header Security considerations

## Azure Key Vault

There are two types of keys:

- hardware protected - uses HSM (hardware security module), decrypt/signing operations are always performed inside the HSM
- software protected - Just uses RSA or ECC (eliptic curve)

Generally, you would only use hardware protected if you have a high security compliance requirement.

Use the key vault for:

- Secret management - tokens, passwords, certificates, API keys, etc.
- Key management - create and control keys. For example App service can use Key Vault to decrypt data without ever knowing the key itself
- Certificate management - allows t to deploy public and provate SSL/TLS certificates, including lifecycle management

Be sure to manage access to keys/secrets, with RBAC, and restrict network access.

Retrieve the secret (in powershell):

```powershell
(Get-AzKeyVaultSecret -vaultName "VaultamortDiary" -name "HiddenLocation").SecretValueText
```

Generating certificates is more secure when the Key Vault acts as in-between between application and CA: ![example](https://docs.microsoft.com/en-us/learn/modules/configure-and-manage-azure-key-vault/media/5-certificate-authority-2.png)

Here, the Key vault can monitor the lifecycle and events such as the certificate being revoked. For your own IaaS apps, you will have to read the certificate from the key vault. For PaaS (such as App Service), you can easily join an App service to a Key vault.

## Role-based Access Control

In Azure, a role is a collection of permissions. Roles are inherited like this:

Management group -> Subscription -> Resource Group -> Resource.

The most common roles are Owner (Can change and give rights to others), Contributor (Can change), Reader (Can use but not change), User Access Administrator (Can give rights to others)

You can view activity logs to see changes in role assignments
