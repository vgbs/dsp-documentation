---
title: DSP User Management and RBAC
description: User and RBAC Management
---

[&larr; back to Overview](/dsp)

# User and RBAC Management

This page provides an overview of the User and RBAC management process on the Digital Solutions Platform (DSP).

How Users are managed and role based access is provided to tenants and teams to the DSP provisioned resources is crucial for organizing resources and ensuring proper access control.

By following a standarized prcess , teams can efficiently organize their users and access their resources in a multi-tenant environment.

---

## User Management in DSP?

During on-boarding process of tenants/teams, DSP provisions Entra ID (previously Azure AD) groups and assign owners for every usecase to effectively manage users. These groups further can be managed by individual tenant/team owners. DSP uses these groups for RBAC management.

- **Tenant level grouos**: To manage the users at each tenant level.
- **Team specific groups**: To manage users of a team.


## RBAC in Multi Tenant Kubernetes Environment?

DSP provided Kubernetes cluster is a multi tenant shared resource which has a logical subdivision that groups and isolates resources within a cluster using Namespaces. In order to provide a clear isolation to teams when accessing the shared resource, DSP came up with roles and assigning these roles to dedicated groups at specific scope.

- **Kubernetes Cluster Users**: Set of users who have access to kubernetes cluster to call API Server using kubectl. This role is assigned to all the tenant users.
- **Kubernetes Cluster Namespace Admins**: Set of users who want to perform kubernetes oprtaions at their team specific namespace.

---

## How DSP Provides access to kubernetes cluster and access to the team namespace?

During the registration of a tenant & team,  Entra ID group is created for each tenant and team.
These groups are assigned with owners specified in the tenant and teams configuration files. Later these groups are assigned with azure roles. 

Group names follow a **standarized naming convention** to ensure cosistency across the platform.
The general pattern is `EID-DSP-<tenant-name>-users` and `EID-DSP-<tenant-name>-<team-name>-users`.


### Example

For a customer-solutions tenant a group with name `EID-DSP-customer-solutions-users` is created to manage the users and this group will be assigned a `Azure Kubernetes Service Cluster User Role` role in azure to give access to kubernetes cluster.

---

# Key Takeaways

1. **Consistent Naming**: The `EID-DSP-<tenant-name>-<team-name>-users` format unifies team grouos and simplifies user management.
2. **Automated Provisioning**: Groups and role assignments are provisioned during on-boarding based on entries in the tenant and team claim files, reducing manual effort and potential errors.
3. **Clear Ownership**: Each team specific groups corresponds to a single team where members of these groups are able to access their own namespace to provide a clear isolation for access control.

If you have further questions about groups and assignments or need assistance with your environment setup, please reach out to the DSP team.
