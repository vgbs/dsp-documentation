# DSP - Digital Solution Platform

As DSP, we enable the Vaillant Group to use UI, CLI, IDE, to easily develop new applications in standarized way.

With the DSP, cluster and application deployment will be easier, as it come with a standarized way of doing that, with a set of tools and best practices prebuilt.

Key benefits of DSP:
1. Developer Enablment - Developers can focus on writing code, not on setting up infrastructure. Developers can spin up Kubernetes clusters confidentely, reducing risk of misconfiguration or errors.
2. Organizational efficiency - Standarized way of deploying applications, with a set of tools and best practices prebuilt.

The problem of incosistent environment is solved with standarization process, developers may apply the same process to all environments, and the process is repeatable.


---
# Development Environment
1. Application Config File (YAML)
2. Platform Config files
3. Developer Portal

---
1. Standarized CI/CD pipelines (GitHub Workflows)
2. Custom YAML Parsers
3. Automated Infrastructure Provisioning via Application Config Files, Platform Config Files, in future Developer Portal
4. GitOps Operator

## High level architecture of DSP

![DSP Architecture](./images/dsp-architecture.png)

Score application configuration file example:


# Infrastrcuture

On a very high level, infrastructure of one hub (master) and multiple spokes (tenants).

The hub is responsible for deploying workloads into the spokes, as well as provisioning new spokes.

# Technical Overview
Teams will have
- application code repo (for code of application)
- application gitops repo (for configuration of infrastructure)
