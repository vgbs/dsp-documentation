---
title: DSP Container Registry Provisioning
description: Team Container Registries in Azure
---

[&larr; back to Overview](/dsp)

# Container Registry Management

This page provides an overview of the Azure Container Registry provisioning process on the Digital Solutions Platform (DSP).

Centralised provisiooning of container registries and how they are created and managed within the DSP environment is crucial for organizing resources and ensuring proper access control.

Detailed user documentation is provided in Confluence and gives [step by step instructions on how to manage a team container registry](https://groupspace.vaillant-group.com/display/VIXP/2.6+Team+Container+Registriy).

---

## What is a Container Registry?

A container registry is a managed service that stores, manages, and distributes container images for your applications. It provides secure storage, versioning, and access control, enabling you to easily share and deploy containerized applications across different environments. 

**Key Features:**
- Centralized storage: Store and manage container images in a secure, central location.
- Versioning: Track changes and versions of container images.
- Authentication: Securely authenticate users and applications.
- Virtual network support: Connect to Azure Container Registry within your virtual network.

**Benefits:**
- Enhanced security: Securely manage and control access to container images.
- Simplified deployment: Streamline deployment processes across different environments.
- Reduced complexity: Reduce complexity associated with container management and creation.

---

## Container Registry Creation

During the registration of a team, a team container registry can be enabled and is then automatically created.

- Due to the fact that Azure container registry names need to be globally unigue the DSP will specify a CR name that closly matches the team name but allows for flexability.
- Container registry names can only consist of the following pattern: [a-zA-Z0-9]{1,20}
- Crossplane will prefix the uniqueName with dspacr

The general pattern is `dspacr<uniqueName>`.

Once the container registry has been provisioned it can be found in the teams global resource group. The pattern for the resource group name is: rg-<tenant-name>-<team-name>-global
Access is granted to the resource group by the Teams DevOps role assignment. The DevOps role can manage the container registry in full.

### Example

```
apiVersion: dsp.vaillant-group.com/v1alpha1
kind: XPlatformTeam
metadata:
  name: unused-example-team
spec:
  parameters:
    shortName: TEAM
    longName: Test Team
    tenantNameRef:
      name: unused-example-tenant
    acr:
      enabled: true
      uniqueName: TestTeam01
      sku: Standard  
```

---

# Key Takeaways

1. **Image Pull** access is granted to the Kubelet Identity of each cluster on which a team has an environment simplifying deployment of team applications.  
2. **Automated Provisioning**: Contrainer Registries are generated based on entries in the tenant and team claim files, reducing manual effort and potential errors.
3. **Clear Ownership**: Each Container Registry corresponds to a single team, streamlining usage and auditing.
4. **User Documentation**: Full user documentation is provided in confluence.
5. **Disabling** the container registry in the team claim file, will cause the container registry and its contents to be deleted.

If you have further questions about contrainer registries or need assistance with your scope / build authentication setup, please reach out to the DSP team.
