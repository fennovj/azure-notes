# Standard vs premium

This document contains all info about standard vs premium (and also shared/isolated)

## App service

- Free/shared: Almost no features. 1 hour compute time per day. Max 10 apps. 1 GB disk space
- Shared: Like Free, but not free. Custom domains. 1 hour per day. Max 100 apps.
- Basic: Like shared, but max 3 instances and 10GB disk space. Unlimited apps
- Standard: For production workloads. Up to 10 instances, autoscaling, VPN hybrid connectivity.
- Premium also gives 250GB disk space, up to 30 instances, and 'Clone App'
- Isolated makes it run on an isolated backbone network within Azure

Summary: Free/Shared/Basic is for dev/test, Standard/Premium is for production. The second group has way more features, but is more expensive. Within the groups, the main difference is the amount of diskspace/instances.

## Azure AD

Free - up to 10 apps for SSO, bunch of features like MFA, User management, etc.
Office 365 - like free, but with company branding, and SLA
Premium - unlimited apps for SSO, and:
    - group access management
    - Hybrid identities
    - Advanced group access management
    - Conditional acces (based on group/location/device status)
Premium (P2)
    - Identity production (Risky account detection)
    - Identity governance (PIM, monitoring/protection of superusers)

Example: requiring 2FA for a certain app is contitional access, therefore you need Premium.
