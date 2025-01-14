---
title: DSP Namespace Management
description: Namespace Management
---

[&larr; back to Overview](/dsp)

# Namespace Management

This page provides an overview of the namespace management process on the Digital Solutions Platform (DSP).

How namespaces are created and managed within the DSP environment is crucial for organizing resources and ensuring proper access control.

By following a standarized naming convention, teams can efficiently organize and deploy their applications in a multi-tenant Kubernetes environment.

---

## What is a Namespace?

A **namespace** in Kubernetes is a logical subdivision that groups and isolates resources within a cluster. It helps in:
- **Logical Separation**: Preventing resource name collissions across different teams or projects.
- **Access Control**: Restricting user permissions to specified namespaces.
- **Resource Quotas**: Limiting the amount of resources that can be consumed by a namespace. Defining CPU, memory, and other usage limits for specific namespaces.

---

## Kubernetes Namespace Creation

During the registration of a team, a Kubernetes namespace is automatically created.
This namespace follow a **standarized naming convention** to ensure cosistency across the platform.
The general pattern is `<tenant-name>-<team-name>-<env>`.

### Example

`customer-solutions-dsp-dev`

1. **Tenant Name** (see example [Tenant Claim File](!localhost)).
2. **Team Name** (see example [Team Claim File](!localhost)).
3. **Environment**
4. [Public GitOps DSP Template](https://github.com/Digital-Solution-Platform/gitops-dsp-demo)

During the namespace creation step of team on-boarding along with the tenant cluster a namespace will be created for each team in the shared tenant (spoke) cluster.

The same namespace is also created on the **hub cluster**, mirroring the tenant cluster setup.
This ensures a seamless experience for teams operating in multiple clusters.

---

# Key Takeaways

1. **Consistent Naming**: The `<tenant-name>-<team-name>-<env>` format unifies team environments and simplifies resource identification.
2. **Automated Provisioning**: Namespaces are generated based on entries in the tenant and team claim files, reducing manual effort and potential errors.
3. **Clear Ownership**: Each namespace clearly correspons to a signle team and environment, streamlining both usage and auditing.

If you have further questions about namespaces or need assistance with your environment setup, please reach out to the DSP team.
