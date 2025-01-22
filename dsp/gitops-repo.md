---
title: Setting Up a Fresh GitOps Repository
description: This guide explains the process of creating and organizing a new GitOps repository, leveraging Kustomize for both single and composite applications, and provisioning infrastructure resources (e.g., databases).
---

[&larr; back to Overview](/dsp)

## 1. Introduction

Modern cloud-native applications require robust,
automated workflows that rapidly deliver new features while minimizing operational overhead.
The DSP addresses these needs through GitOps-driven principles,
where configuration and deployment logic are stored in Git repositories.
GitOps ensures traceability, versioning, and consistency across environments.

This guide sets forth DSP best practices for establishing a fresh GitOps repository for:
1. **Single (monolithic) applications**
2. **Composite (microservice) applications**
3. **Infrastructure resources** (e.g., databases, caches)

With DSP,
developers can focus on creating value-added features while relying on automated pipelines for consistent,
secure, and scalable deployments.

---

## 2. Repository Structure Overview
A typical GitOps repository under this model comprises distinct directories for applications (`apps`)
and infrastructure (`infrastructure`),
plus the optional `templates`.
```bash
├── README.md
├── apps
│   ├── service-tracker
│   └── spring-petclinic
├── infrastructure
│   └── postgres-db
└── templates
    └── deployment.yaml
```

### 2.1 `apps` Directory
- **Purpose**: Hosts Kubernetes manifests for each application.
- **Recommended Structure**:
  - A folder per application, with a `base` subdirectory for raw manifests and multiple `overlays` subdirectories to handle environment-specific differences.

### 2.2 `infrastructure` Directory
- **Purpose**: Stores definitions for persistent or shared resources (e.g., databases, message queues) that your application depends upon.
- **Recommended Structure**:
  - A similar base/overlays organization, so environment-level overrides remain consistent.

### 2.3 `templates` Directory
- **Purpose**: Contains generic YAML templates (e.g., a standard `deployment.yaml`), that can be copied or referenced for new services.

---

## 3. Application Patterns and Kustomize Examples

### 3.1 Single Application (Monolithic)
**Context**: A single, self-contained workload such as Spring Petclinic.
**Key Idea**: All manifests reside in a single `base` folder, with environment-specific overlays in environments.

```
├── apps
│   └── spring-petclinic
│       ├── base
│       │   ├── deployment.yaml
│       │   ├── ingressroute.yaml
│       │   ├── kustomization.yaml
│       │   ├── postgres-claim.yaml
│       │   └── service.yaml
│       └── overlays
│           ├── dev
│           │   ├── kustomization.yaml
│           ├── prod
│           │   ├── kustomization.yaml
│           └── qa
│               ├── kustomization.yaml
```

### 3.2 Composite Application (Microservice)
**Context**: The application spans multiple microservices, each potentially with its own deployment or stateful resource.
**Key Idea**: Each microservice has its own subdirectory in `base`; a top-level `kustomization.yaml` aggregates them. Overlays tailor environment-specific details.

```
├── apps
│   ├── service-tracker
│   │   ├── base
│   │   │   ├── data-api/
│   │   │   ├── flights-api/
│   │   │   ├── kustomization.yaml (includes all sub-bases)
│   │   │   ├── quakes-api/
│   │   │   ├── service-tracker-ui/
│   │   │   ├── tracker-postgres-db/
│   │   │   └── weather-api/
│   │   └── overlays
│   │       ├── dev/
│   │       ├── prod/
│   │       └── qa/
```

For monorepos containing multiple microservices,
a composite pattern streamlines joint deployments while preserving clear boundaries between workloads.

---

## 4. Infrastructure

### 4.1 Postgres Database & Purpose and Directory Structure

In addition to application manifests,
your solution may require external resources such as a database.
The DSP supports defining infrastructure resources in the `infrastructure` directory,
following the same base/overlays structure:

```bash
├── infrastructure
│   └── postgres-db
│       ├── base
│       │   ├── kustomization.yaml
│       │   └── psql.yaml
│       └── overlays
│           ├── dev
│           ├── prod
│           └── qa
```

- **base**: Holds default resource definitions (CPU/memory requests, storage classes).
- **overlays**: Environments can override base definitions to suit environmental constraints (e.g., namespace, credentials).

#### Provisioning Flow

1. **Declare Base Resources**
In the `base` directory, reference the resource files via a `kustomization.yaml`.
2. **Environment Overlays**
For each environment, create a dedicated folder referencing `../../base`, adding environment-specific patches as needed.
3. **Pipeline Integration**
The DSP handles the underlying provisioning or "claiming" of these resources as part of the deployment process.

#### Best Practices
- Use **Kustomize patches** for environment differences.
- Keep **resources small and focused** to avoid accidental side effects across services.
- Store **sensitive data** in secure stores (Secrets, Vault).
- Maintian consistent patterns in environments for easy comparisons and maintenance.

---

## 5. Working with Overlays

### 5.1 Kustomization File Essentials

Each overlay folder includes a `kustomization.yaml` pointing to `../../base` and optionally:
- **namespace**: The target namespace.
- **images**: Updated container references or digests.
- **patches**: Additional environment-specific YAML that modifies base resource definitions.

Sample kustomization.yaml for `dev` environment:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: dsp-namespace-dev

resources:
- ../../base

images:
- digest: sha256:f6a17e3053d76da90734a38304ba0d442cc3f95f59c0d3ed827f71fad3d79c00
  name: spring-petclinic
  newName: dspcontainerregistry.azurecr.io/spring-petclinic

patches:
- path: deployment.yaml
- path: service.yaml
```

### 5.2 Example Deployment Patch

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spring-petclinic

spec:
  replicas: 1
  template:
    spec:
      containers:
      - name: spring-petclinic
        env:
        - name: SPRING_PROFILES_ACTIVE
          value: dev
```

---

## 6. End-to-End Setup Steps

Before initiating these steps, please ensure you have been **onboarded to the DSP** and **invited to the Vaillant Group Organization** on GitHub.
These prerequisites guarantee that you possess the necessary privileges to create, manage, and integrate repositories within the DSP's deployment environment.
For further details on the onboarding and invitation process, consult the [Getting Started](getting-started.md) guide.

1. **Obtain the DSP GitOps Template**
  - **Objective**: Jumpstart your project with a standarized, pre-structured [GitOps repository template](https://github.com/Digital-Solution-Platform/gitops-digital-solution-platform).
  - **Significance**: This template aligns with industry best practices, ensuring a consistent folder layour, pre-configured Kustomize files, and example overlays for multiple environments.
  - **Details**: If the target organization is not onboarded onth the DSP yet, contact the DSP Team regarding the [onboarding steps](https://groupspace.vaillant-group.com/x/iUdVGw).
2. **Create a Dedicated Application Directory**
  - **Action**: Within the `apps/` directory, create a subfolder matching your applicaion's name (e.g., `my-cool-app`).
  - **Details**: Add a `base` subdirectory containing the foundational YAML files (e.g., Deployment, Service). These form the baseline configuration that overlays (environment) will reference.
  - **Outcome**: Enforces a clear separation of concerns between core manifests and environment-specific modifications.
3. **Provision Infrastructure Resources**
  - **Action**: If your application needs external resources (e.g., Postgres), define them within the `infrastructure/` directory. Organize them similarly into `base` and `overlays`.
  - **Rationale**: Centralizing resource definitions simplifies audits, version control, and environment parity. It also ensures that underlying resources (databases, message queues, etc.) adhere to a consistent, template-driven standard.
4. **Configure Environment Overlays**
  - **Action**: For each environment (e.g., `dev`, `qa`, `prod`), add a subfolder under `apps/[my-cool-app]/overlays/` (and similarly under `infrastructure/[resource]/overlays/` if applicable).
  - **Details**:
    - Reference `../../base` in `kustomization.yaml`.
    - Customize parameters such as replicas, environment variables, or container images via Kustomize patches.
  - **Benefit**: Promotes maintainability by clearly isolating environment-level changes (like resource scaling, domain names, or credentials) in dedicated folders.
5. **Contact the DSP Team for Pipeline Configuration**
  - **Action**: Once your GitOps repository is structured and the overlays are in place, reach out to the DSP Team. Provide them with the repository URL and relevant environment details.
  - **Result**: The DSP Team will set up and configure the pipeline for you, enabling automatic synchronization of your Git-defined manifests to the appropriate Kubernetes clusters.
6. **Validate Deployment**
  - **Action**: Make s mall test commit - such as increasing replicas from 1 to 2 - within one of your overlay folders (e.g., `apps/[my-cool-app]/overlays/dev`).
  - **Verification**: After the pipeline is configured by the DSP Team, confirm that your environment reflects the updated configuration.
  - **Conclusion**: A successful deployment indicates that your GitOps workflow is operational, ensuring reliable and traceable updates for each environment.

By following these six steps, teams establish a dependable, version-controlled delivery pipeline for cloud-native workloads.
Standarizing on Kustomize-based overlays for both applications and infrastructure maintains a coherent and scalable approach that simplifies future modifications,
fosters collaboration, and supports ongoing evolution of your services within the DSP ecosystem.

---

## 7. Example Applications

### 7.1 Spring Petclinic (Single)
A monolithic Java application used as a canonical demonstration.
Once properly configured,
you can deploy it to different environments by pushing changes to your Git repository.

[View Spring Petclinic Files](https://github.com/Digital-Solution-Platform/gitops-digital-solution-platform/tree/main/apps/spring-petclinic)

### 7.2 Service Tracker (Composite)
A multi-service application featureing microservices like data-api, flights-api, and quakes-api.
Each service resides in its own subdirectory, enabling fine-grained control and easy environment.

[View Service Tracker Files](https://github.com/Digital-Solution-Platform/gitops-digital-solution-platform/tree/main/apps/service-tracker)

---

## 8. Best Practices & Tips
1. **Consistent Naming**
  - Use clear, lowercase directories that reflect each application or service.
2. **Parametrization**
  - Use overlays or external secrets for environment-specific adjustments (replicas, environment variables)
3. **Security & Permissions**
  -Restrict GitOps repo write access to authorized DSP service accounts or maintainers.
4. **Regular Cleanup**
  - Remove or archive overlays, images, or resources that are no longer in use.

---

## 9. Summary & Conclusion

A fresh GitOps repository, organized around `apps`, `infrastructure`, and optionally `templates`,
forms the backbone of an efficient, scalable DevOps workflow.
Teams can quickly iterate on both single (monolithic) and composite (microservice) applications,
while maintaining environment-specific concerns in a robust, version-controlled manner.

Integrating infrastructure definitions into the same Git repository further ensures consistency,
auditability, and traceability, making it easier for teams to reason about changes at all layers - from code modifications to database provisioning.
This unified approach simplifies governance, fosters better collaboration, and paves the way for advanced features such as automated security scans,
policy enforcement, and continuous delivery at scale.

> **Next Steps**:
> 1. **Refine Overlays**: Adjust environment patches for your environment-specfic requirements.
> 2. **CI Integration**: Look into the [CI setup](getting-started-ci.md) to automate deployments.

[&larr; back to Overview](/dsp)
