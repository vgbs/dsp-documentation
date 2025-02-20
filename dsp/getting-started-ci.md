---
title: Setting Up a CI
description: Setting up a Continuous Integration pipeline for your project using the Digital Solutions Platform (DSP)
---

[&larr; back to Overview](/dsp)

---

## 1. Introduction

This guide will help you set up a Continuous Integration (CI) pipeline for your project using the Digital Solutions Platform (DSP).

DSP Team is providing standard starter workflows which can be used in your project with minimal effort,
it enables developer to focus only on the code and not on the CI/CD configuration,
so they can develop only features that add value to the project.

---

## 2. Prerequisites

- GitHub Account: You need to have a GitHub account connected to the Vaillant Group organization.
- GitHub Repositories: You need to start with the GitOps and Application repository for your project. We recommend using this [template](https://github.com/Digital-Solution-Platform/gitops-digital-solution-platform), and follow the naming convention `gitops-<team-name>`.
- Service Principal (if using ACR): Please [contact](contact.md) the DSP team to get the service principal for your project, for pulling and pushing images to Docker registry.

---

## 3. Setting up CI

### 3.1 Contact DSP for Initial Setup

Please contact DSP Team via the [Help Desk](https://service.dsp.vaillant-group.com) to configure the initial integration for your project. The team will assist with:

- Configurating organization-level settings.
- Setting up the required GitHub Actions resources (secrets, deploy keys, GitHub Apps) for your repositories.

### 3.2 Using Starter Workflows

After the initial setup, explore the [Starter Workflows Repository](https://github.com/Digital-Solutions-Foundation/starter-workflows/tree/main).
These workflows are pre-configured and require minimal customization.

### 3.3 Configuring GitOps Automation

The GitOps automation workflow creates pull requests in your GitOps repository to update deployment manifests with new container image tags.

Add the GitOps update workflow to your main repository's `.github/workflows` directory.

An example file is available in the [Starter Workflows repository](https://github.com/Digital-Solutions-Foundation/starter-workflows/tree/main).

### 3.4 ArgoCD Deployment

Once changes are merged into the main branch of the GitOps repository:

- **ArgoCD** detects the updates automatically
- The changes are applied to the target environment using declarative manifests.

---

## 4. Using DSP Hosted GitHub Runner

The DSP team provides GitHub Runners for every organization, which is hosted on the Vaillant Group infrastructure.
This allows you to run your CI/CD workflows on internal network and dedicated hardware for cost-effective and secure execution.

Currently, we support the following runners:

- `dsp-linux`: x64 linux runner with Docker support.

### 4.1 Using DSP Hosted Runner

We are providing runners for each organization, which can be used for all repositories within the organization.
To use the DSP hosted runner, add the runner label to your workflow file into the `runs-on` section.

```yaml
jobs:
  build:
    runs-on: [dsp-linux]
```

## 5. Testing and Validation

After setting up the workflows you can trigger a workflows by committing and pushing changes to the main repository.
The CI should run build and test steps, and deploy the application to the target environment, with building and pushing the container image.

Check yours GitOps repository for the newly created Pull Request with the updated image tag.

If you want to deploy that changes,
just merge the PR in the GitOps repository and monitor as ArgoCD applies the changes to the environment.

---

## 6. Need Assistance?

For support or additional customization, contact the DSP team via the [Help Desk](https://service.dsp.vaillant-group.com).
The team will assist you in addressing any issues or implementing advanced features.

---

Happy deploying with DSP! 🚀
