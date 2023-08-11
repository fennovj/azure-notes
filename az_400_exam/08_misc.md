# Misc

This is just miscelaneous stuff I didn't know yet but picked up during studying.

## Privileged Identity Management

<https://learn.microsoft.com/en-us/azure/active-directory/privileged-identity-management/pim-configure>

This allows you to give limited-time role to a user. For example:

- Give Emily `Blob Data Reader` on `storage-01`, from 7:30 to 8:30 UTC.
- Give support engineer in other company temporary access to resources

Also it allows 'just in time' access - you make Emily `eligible` for blob data reader on storage-01. This means she will get assigned the role as soon as she uses it for the first time. I honestly am not sure what difference that makes from a security perspective, but oh well.

## Conditional Access

Conditional access is a different Active Directory feature, but it does not affect the role assignment directly. Rather, you can set conditions on access:

- Requiring MFA
- Blocking risky sign-ins
- Requiring organization-managed devices
- Blocking certain ip ranges (e.g. only allowing Netherlands)

## Key vault access control

<https://learn.microsoft.com/en-us/azure/key-vault/general/rbac-access-policy>
<https://learn.microsoft.com/en-us/answers/questions/816270/provide-access-to-key-vault-keys-certificates-and>

I'm not sure if this has changed or if I just forgot since doing dp_200/dp_201, but for key vault access, Microsoft now recommends using RBAC, not access policy. The same goes for if you want to give access to even a single (or subset) of secrets. The trick is that every secret is an individual (child) resource that can have its own RBAC. So e.g., you can give `Secret Reader` permission to Emily on the `mySecret` secret, but not any others.

Access policies still exist in key vaults but probably should not be used where RBAC is an option.

## Third party software

- SonarQube - Static Code Analysis, both during development and during CI. I was kinda biased since we use SonarQube only on master branch (for now). But you absolutely can use it for development. e.g. run a sonarQube analysis on your branch to see what you did wrong, before even requesting a code review. Or alternatively, a reviewer checking the SonarQube results to see if there are any weak points.
- PMD - another static code analyzer
  - Both SonarQube and PMD are supported out of the box for Maven projects.
- CheckStyle - Static code analyzier, *only for maven*. Is also supported out of the box in DevOps.
- BlackDuck by Synopsys - Identifies open source packages for vulnerabilities.
- Whitesource Bolt -  Identifies open source packages for vulnerabilities.
- Selenium - Automated UI testing
- Cobertura/Jacoco - Code Coverage. local packages, but there are DevOps pipeline tasks that calculate/publish code coverage results.
